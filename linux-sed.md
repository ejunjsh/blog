---
title: linux sed命令详解
date: 2014-01-05 23:53:22
tags: [linux,sed]
categories: linux命令
---
功能说明：
利用script来处理文本文件。 
语　　法：
````bash
sed [-hnV][-e script][-f script文件][文本文件] 
````
<!-- more -->

补充说明：
sed可依照script的指令，来处理、编辑文本文件。 

参数：  
````bash
-e script或—expression=script   以选项中指定的script来处理输入的文本文件。  
-f script文件或—file=script文件   以选项中指定的script文件来处理输入的文本文件。  
-h或—help 显示帮助。  
-n或—quiet或--silent 仅显示script处理后的结果。  
-V或—version 显示版本信息。
````

例子： 
````bash
$ sed -e 's/123/1234/' a.txt
````
将a.txt文件中所有行中的123用1234替换（-e表示命令以命令行的方式执行；参数s，表示执行替换操作） 

````bash
$ sed -e '3,5 a4' a.txt
````
将a.txt文件中的3行到5行之间所有行的后面添加一行内容为4的行（参数a，表示添加行，参数a后面指定添加的内容） 

````bash
$ sed -e '1 s/12/45/' a.txt
````
把第一行的12替换成45 

````bash
$ sed -i "s/oldstring/newstring/g" `grep oldstring -rl yourdir`
````
批量处理通过grep搜索出来的所有文档，将这些文档中所有的oldstring用newstring替换（-i参数表示直接对目标文件操作） 

````bash
$ sed -n 's/^test/mytest/p' example.file
````
(-n)选项和p标志一起使用表示只打印那些发生替换的行。也就是说，如果某一行开头的test被替换成mytest，就打印它。(^这是正则表达式中表示开头，该符号后面跟的就是开头的字符串)（参数p表示打印行） 

````bash
$ sed 's/^wangpan/&19850715/' example.file
````
表示被替换换字符串被找到后，被替换的字符串通过＆符号连接给出的字符串组成新字符传替换被替换的字符串,所有以wangpan开头的行都会被替换成它自已加19850715，变成wangpan19850715 

````bash
$ sed -n 's/\(love\)able/\1rs/p' example.file
````
love被标记为1，所有loveable会被替换成lovers，而且替换的行会被打印出来。需要将这条命令分解，s/是表示替换操作，\(love\)表示选中love字符串，\(love\)able/表示包含loveable的行，\(love\)able/\l表示love字符串标记为1，表示在替换过程中不变。rs/表示替换的目标字符串。这条命令的操作含义：只打印替换了的行 

````bash
$ sed 's#10#100#g' example.file
````
不论什么字符，紧跟着s命令的都被认为是新的分隔符，所以，“#”在这里是分隔符，代替了默认的“/”分隔符。表示把所有10替换成100。 

````bash
$ sed -n '/love/,/unlove/p' example.file
````
只打印包含love字符串行到包含unlove字符串行之间的所有行（确定行的范围就是通过逗号实现的） 

````bash
$ sed -n '5,/^wang/p' example
````
只打印从第五行开始到第一个包含以wang开始的行之间的所有行 

````bash
$ sed '/love/,/unlove/s/$/wangpan/' example.file
````
对于包含love字符串的行到包含unlove字符串之间的行，每行的末尾用字符串wangpan替换。  
字符串$/表示以字符串结尾的行，$/表示每一行的结尾，s/$/wangpan/表示每一行的结尾添加wangpan字符串 

````bash
$ sed -e '11,53d' -e 's/wang/pan/' example.file
````
(-e)选项允许在同一行里执行多条命令。如例子所示，第一条命令删除11至53行，第二条命令用pan替换wang。命令的执行顺序对结果有影响。如果两个命令都是替换命令，那么第一个替换命令将影响第二个替换命令的结果。(参数d，表示删除指定的行) 

````bash
$ sed --expression='s/wang/pan/' --expression='/love/d' example.file
````
一个比-e更好的命令是--expression。它能给sed表达式赋值。 

````bash
$ sed '/wangpan/r file' example.file
````
file里的内容被读进来，显示在与wangpan匹配的行后面，如果匹配多行，则file的内容将显示在所有匹配行的下面。参数r，表示读出文件，后面空格紧跟文件名称 

````bash
$ sed -n '/test/w file' example.file
````
在example.file中所有包含test的行都被写入file里。参数w，表示将匹配的行写入到指定的文件file中 

````bash
$ sed '/^test/a\oh! My god!' example.file
````
'oh! My god!'被追加到以test开头的行的后面，sed要求参数a后面有一个反斜杠。 

````bash
$ sed '/test/i\oh! My god!' example.file
````
'oh! My god!'被追加到包含test字符串行的前面，参数i表示添加指定内容到匹配行的前面，sed要求参数i后面有一个反斜杠 

````bash
$ sed '/test/{ n; s/aa/bb/; }' example.file
````
如果test被匹配，则移动到匹配行的下一行，替换这一行的aa，变为bb。参数n，表示读取匹配行的下一个输入行，用下一个命令处理新的行而不是匹配行。Sed要求参数n后跟分号 

````bash
$ sed '1,10y/abcde/ABCDE/' example.file
````
把1—10行内所有abcde转变为大写，注意，正则表达式元字符不能使用这个命令。参数y，表示把一个字符翻译为另外的字符（但是不用于正则表达式） 

````bash
$ sed -i 's/now/right now/g' test_sed_command.txt
````
表示直接操作文件test_sed_command.txt，将文件test_sed_command.txt中所有的now用right now替换。参数-i，表示直接操作修改文件，不输出。 

````bash
$ sed '2q' test_sed_command.txt
````
在打印完第2行后，就直接退出sed。参数q，表示退出 

````bash
$ sed -e '/old/h' -e '/girl-friend/G' test_sed_command.txt
````
首先了解参数h，拷贝匹配成功行的内容到内存中的缓冲区。在了解参数G，获得内存缓冲区的内容，并追加到当前模板块文本的后面。上面命令行的含义：将包含old字符串的行的内容保存在缓冲区中，然后将缓冲区的内容拿出来添加到包含girl-friend字符串行的后面。隐含要求搜集到缓冲区的匹配行在需要添加行的前面。 

````bash
$ sed -e '/test/h' -e '/wangpan/x' example.file
````
将包含test字符串的行的内容保存在缓冲区中，然后再将缓冲区的内容替换包含wangpan字符串的行。参数x，表示行替换操作。隐含要求搜集到缓冲区的匹配行在需要被替换行的前面。 
