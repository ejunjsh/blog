# 什么是尾递归

尾递归是指所有递归形式的调用，一定是发生在函数的末尾。 形式上只要最后一个return语句是单纯函数就可以。如：

````java
return tailrec(x+1);
````

而

````java
return tailrec(x+1) + x;
````

或者

````java
return x*tailrec(x+1);
````

都不可以。因为无法更新tailrec()函数内的实际变量，只是新建一个栈。

# 尾递归的好处

尾递归和一般的递归不同在对内存的占用，普通递归创建stack累积而后计算收缩，尾递归只会占用恒量的内存（和迭代一样）。
我们知道递归调用是通过栈来实现的，每调用一次函数，系统都将函数当前的变量、返回地址等信息保存为一个栈帧压入到栈中，那么一旦要处理的运算很大或者数据很多，有可能会导致很多函数调用或者很大的栈帧，这样不断的压栈，很容易导致栈的溢出。 

我们回过头看一下尾递归的特性，函数在递归调用之前已经把所有的计算任务已经完毕了，他只要把得到的结果全交给子函数就可以了，无需保存什么，子函数其实可以不需要再去创建一个栈帧，直接把就着当前栈帧，把原先的数据覆盖即可。相对的，如果是普通的递归，函数在递归调用之前并没有完成全部计算，还需要调用递归函数完成后才能完成运算任务。
Scala，Erlang，C等语言支持尾递归优化，即编译期间把递归优化成了迭代。

但是Python，Java等语言不支持尾递归优化。JVM本身是不支持尾递归优化的，需要编译器支持，而Java编译器不支持，但是Scala支持。

所以即便在Java中用尾递归的形式写递归，仍然会出现stackoverflow的异常。

所以在不支持尾递归优化的语言中，就要尽量避免深递归的调用，尽量使用循环代替递归。

但是在支持尾递归优化的语言中，可以使用尾递归，毕竟递归的代码更好理解些。

# 尾递归的一些实例应用
1. fibonacci数列
````java
public static int fibonacci_tail(int n, int ret1, int ret2) {
		if (n == 0)
			return ret1;
		else
			return fibonacci_tail(n - 1, ret2, ret1 + ret2);
	}
````
2. 求最大公约数
````java
public static int gcd(int big,int small){  
        if(big%small==0) return small;  
        return gcd(small, big%small);  
    }
````
3. 阶乘
````java
public static int fn2Helper(int ret, int n){  
        if(n<2) return ret;  
        return fn2Helper(ret*n,n-1);  
    }
````
4. 翻转字符串
````java
public static String reverse2Helper(String s,String init,int len){  
           if(len==0) return init;  
           return reverse2Helper(s, init+s.charAt(len-1), len-1);  
       }
````
5. 验证字符串是否是回文
````java
public static boolean testHuiwenHelper(String s,int begin, int len){  
         if(begin< len>>1 && s.charAt(begin)==s.charAt(len-begin-1))   
             return testHuiwenHelper(s, begin+1, len);  
         else if(begin==len>>1) return true;  
         else return false;  
     }
````
6. 翻转整数
````java
public static int reverseIntHelper(int i, int init){  
         if(i==0) return init;  
         return reverseIntHelper(i/10, init*10+i%10);  
     }
````

ref https://my.oschina.net/hosee/blog/616968
