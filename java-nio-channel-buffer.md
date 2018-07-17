---
title: NIO之Channel、Buffer
date: 2016-08-22 21:16:51
tags: [nio,channel,buffer,java]
categories: java
---
# Buffer
一块缓存区，内部使用字节数组存储数据，并维护几个特殊变量，实现数据的反复利用。
1、mark：初始值为-1，用于备份当前的position;
2、position：初始值为0，position表示当前可以写入或读取数据的位置，当写入或读取一个数据后，position向前移动到下一个位置；
3、limit：写模式下，limit表示最多能往Buffer里写多少数据，等于capacity值；读模式下，limit表示最多可以读取多少数据。
4、capacity：缓存数组大小

[![](http://idiotsky.top/images3/java-nio-channel-buffer.png)](http://idiotsky.top/images3/java-nio-channel-buffer.png)
<!-- more -->

mark()：把当前的position赋值给mark
````java
public final Buffer mark() {
    mark = position;
    return this;
}
````

reset()：把mark值还原给position
````java
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
````

clear()：一旦读完Buffer中的数据，需要让Buffer准备好再次被写入，clear会恢复状态值，但不会擦除数据。
````java
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
````

flip()：Buffer有两种模式，写模式和读模式，flip后Buffer从写模式变成读模式。
````java
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
````

rewind()：重置position为0，从头读写数据。
````java
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
````

目前Buffer的实现类有以下几种：
* ByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer
* MappedByteBuffer

[![](http://idiotsky.top/images3/java-nio-channel-buffer-1.png)](http://idiotsky.top/images3/java-nio-channel-buffer-1.png)

# ByteBuffer
ByteBuffer的实现类包括"HeapByteBuffer"和"DirectByteBuffer"两种。

## HeapByteBuffer
````java
public static ByteBuffer allocate(int capacity) {
    if (capacity < 0)
        throw new IllegalArgumentException();
    return new HeapByteBuffer(capacity, capacity);
}
HeapByteBuffer(int cap, int lim) {  
    super(-1, 0, lim, cap, new byte[cap], 0);
}
````
HeapByteBuffer通过初始化字节数组hd，在虚拟机堆上申请内存空间。

## DirectByteBuffer
````java
public static ByteBuffer allocateDirect(int capacity) {
    return new DirectByteBuffer(capacity);
}
DirectByteBuffer(int cap) {
    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);

    long base = 0;
    try {
        base = unsafe.allocateMemory(size);
    } catch (OutOfMemoryError x) {
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
        // Round up to page boundary
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}
````
DirectByteBuffer通过unsafe.allocateMemory申请堆外内存，并在ByteBuffer的address变量中维护指向该内存的地址。
unsafe.setMemory(base, size, (byte) 0)方法把新申请的内存数据清零。

# Channel
channel是一个带缓冲的可读可写的I/O对象，所谓带缓冲就是数据要经过一道缓冲的buffer之后，才会到用户层（也就是java代码直接访问的那一层）

目前已知Channel的实现类有：
* FileChannel
* DatagramChannel
* SocketChannel
* ServerSocketChannel

这里用FileChannel做代表吧

# FileChannel
FileChannel的read、write和map通过其实现类FileChannelImpl实现。

## read实现
````java
public int read(ByteBuffer dst) throws IOException {
    ensureOpen();
    if (!readable)
        throw new NonReadableChannelException();
    synchronized (positionLock) {
        int n = 0;
        int ti = -1;
        try {
            begin();
            ti = threads.add();
            if (!isOpen())
                return 0;
            do {
                n = IOUtil.read(fd, dst, -1, nd);
            } while ((n == IOStatus.INTERRUPTED) && isOpen());
            return IOStatus.normalize(n);
        } finally {
            threads.remove(ti);
            end(n > 0);
            assert IOStatus.check(n);
        }
    }
}
````
FileChannelImpl的read方法通过IOUtil的read实现：
````java
static int read(FileDescriptor fd, ByteBuffer dst, long position,
                NativeDispatcher nd) IOException {
    if (dst.isReadOnly())
        throw new IllegalArgumentException("Read-only buffer");
    if (dst instanceof DirectBuffer)
        return readIntoNativeBuffer(fd, dst, position, nd);

    // Substitute a native buffer
    ByteBuffer bb = Util.getTemporaryDirectBuffer(dst.remaining());
    try {
        int n = readIntoNativeBuffer(fd, bb, position, nd);
        bb.flip();
        if (n > 0)
            dst.put(bb);
        return n;
    } finally {
        Util.offerFirstTemporaryDirectBuffer(bb);
    }
}

````
通过上述实现可以看出，基于channel的文件数据读取步骤如下：
1. 申请一块和缓存同大小的DirectByteBuffer bb。
2. 读取数据到缓存bb，底层由NativeDispatcher的read实现。
3. 把bb的数据读取到dst（用户定义的缓存，在jvm中分配内存）。

read方法导致数据复制了两次。内核->bb,bb->dst

## write实现
````java
public int write(ByteBuffer src) throws IOException {
    ensureOpen();
    if (!writable)
        throw new NonWritableChannelException();
    synchronized (positionLock) {
        int n = 0;
        int ti = -1;
        try {
            begin();
            ti = threads.add();
            if (!isOpen())
                return 0;
            do {
                n = IOUtil.write(fd, src, -1, nd);
            } while ((n == IOStatus.INTERRUPTED) && isOpen());
            return IOStatus.normalize(n);
        } finally {
            threads.remove(ti);
            end(n > 0);
            assert IOStatus.check(n);
        }
    }
}
````
和read实现一样，FileChannelImpl的write方法通过IOUtil的write实现：
````java
static int write(FileDescriptor fd, ByteBuffer src, long position,
                 NativeDispatcher nd) throws IOException {
    if (src instanceof DirectBuffer)
        return writeFromNativeBuffer(fd, src, position, nd);
    // Substitute a native buffer
    int pos = src.position();
    int lim = src.limit();
    assert (pos <= lim);
    int rem = (pos <= lim ? lim - pos : 0);
    ByteBuffer bb = Util.getTemporaryDirectBuffer(rem);
    try {
        bb.put(src);
        bb.flip();
        // Do not update src until we see how many bytes were written
        src.position(pos);
        int n = writeFromNativeBuffer(fd, bb, position, nd);
        if (n > 0) {
            // now update src
            src.position(pos + n);
        }
        return n;
    } finally {
        Util.offerFirstTemporaryDirectBuffer(bb);
    }
}
````
通过上述实现可以看出，基于channel的文件数据写入步骤如下：
1. 申请一块DirectByteBuffer，bb大小为byteBuffer中的limit - position。
2. 复制byteBuffer src中的数据到bb中。
3. 把数据从bb中写入到文件，底层由NativeDispatcher的write实现.

write方法也导致了数据复制了两次,src -> bb, bb-> 内核

# Channel和Buffer示例
````java
File file = new RandomAccessFile("data.txt", "rw");
FileChannel channel = file.getChannel();
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = channel.read(buffer);
while (bytesRead != -1) {
    System.out.println("Read " + bytesRead);
    buffer.flip();
    while(buffer.hasRemaining()){
        System.out.print((char) buffer.get());
    }
    buffer.clear();
    bytesRead = channel.read(buffer);
}
file.close();
````
注意buffer.flip() 的调用，首先将数据写入到buffer，然后变成读模式，再从buffer中读取数据。

# 总结
channel里面的那一层缓冲跟传统意义的带缓冲io不同，传统的缓冲io是为了减少io操作，但是channel的缓冲更像是纯粹的把从内核读出来的数据（这数据不在java堆里面）拷贝到java堆中。所以性能不是很好，才会有`MappedByteBuffer`,下一篇会谈谈这个。

参考 https://www.jianshu.com/p/052035037297