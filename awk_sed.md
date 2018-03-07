# 1 介绍
&emsp;&emsp;使用shell处理文档是很常见的需求，相比于Java、Python，shell是一种非常轻量级的选择。Awk和Sed是两种常用的shell文本处理工具，本文将结合实际经验介绍这两个工具。
## 1-1 sed功能
&emsp;&emsp;Sed(Stream EDitor)--流式编辑器。乍一看编辑器，自然会想到notepad、vim这些，但是sed是非交互式的，在command line中像pipe那样一级一级传递。
``` shell
	sed OPTIONS... [SCRIPT] [INPUTFILE...]
```
其中__[SCRIPT]__部分，支持用表达式，来表达操作行内文本的逻辑。__[OPTIONS]__是参数设置，来配置sed的输入输出方式、是否备份等功能。下面我们通过操作text.txt中的内容，来学习详细配置。
```
	$ cat text.txt
    abc abc abc
    xyz xzy xyz
      abc abc abc
    mq mq mz
    
    $ cat text2.txt
    name: xiaoming subject: Math score: 59
    name: xiaoming subject: English score: 89
    name: xiaoming subject: ComputerScience score: 98
    name: xiaohong subject: Math score: 96
    name: xiaohong subject: English score: 98
    name: xiaohong subject: ComputerScience score: 72
```
### 使用s参数，替换目标文本

默认替换每行行首第一次匹配的内容，注意表达式的要__包含三个'/'__。
``` shell
	$ sed 's/abc/opq/' text.txt
    opq abc abc
    xyz xzy xyz
      opq abc abc
    mq mq mz
```

### 使用g参数，进行全文替换

添加g参数，替换全文匹配内容。“n”匹配每行第n个。
``` shell
	$ sed 's/abc/opq/g' text.txt
    opq opq opq
    xyz xzy xyz
      opq opq opq
    mq mq mz
    
    $ sed 's/abc/opq/2' text.txt
    abc opq abc
    xyz xzy xyz
      abc opq abc
    mq mq mz
```
与正则语法相同。匹配行首使用“^”，“/^abc/” 以abc开头的行。匹配行尾使用“$”，“/}$/”匹配以“}”结尾的行。

``` shell
	$ sed 's/^/# /g' text.txt
    # abc abc abc
    # xyz xzy xyz
    #   abc abc abc
    # mq mq mz
    
    $ sed 's/$/ end /g' text.txt
    abc abc abc end 
    xyz xzy xyz end 
      abc abc abc end 
    mq mq mz end 
```

### 替换指定区间内行的内容

__“n,ms/a/b/g”__匹配从第n行到第m行的内容。

### 匹配与参数传递

__“&”__代表被匹配的变量，可以编辑左右内容。
```
	$ sed 's/abc/{ & }/g' text.txt
    { abc } { abc } { abc }
    xyz xzy xyz
      { abc } { abc } { abc }
    mq mq mz
```
__"()"__使用圆括号匹配的内容，可以作为参数（形式为\1,\2..）向下游传递。下面的例子中，通过匹配姓名、科目、得分三项，将匹配内容使用方括号（[]）括上，其中正则表达式[^,]*表达匹配任意开头的n个字符。这个功能，与awk所做的很相似。
```
	$ sed 's/name: \([^,]*\).* subject: \([^,]*\).* score: \([^,]*\)/[\1]:[\2]:[\3]/g' test2.txt
    [xiaoming]:[Math]:[59]
    [xiaoming]:[English]:[89]
    [xiaoming]:[ComputerScience]:[98]
    [xiaohong]:[Math]:[96]
    [xiaohong]:[English]:[98]
    [xiaohong]:[ComputerScience]:[72]
```
## 1-2 awk功能

与Sed相比，被称作编程语言的AWK就显得功能复杂的多。它的名字来源于其三位创始人的姓氏首字母（[阿尔佛雷德·艾侯](https://zh.wikipedia.org/wiki/%E9%98%BF%E5%B0%94%E4%BD%9B%E9%9B%B7%E5%BE%B7%C2%B7%E8%89%BE%E4%BE%AF)、[彼得·温伯格](https://zh.wikipedia.org/wiki/%E5%BD%BC%E5%BE%97%C2%B7%E6%BA%AB%E4%BC%AF%E6%A0%BC)和[布莱恩·柯林](https://zh.wikipedia.org/wiki/%E5%B8%83%E8%90%8A%E6%81%A9%C2%B7%E6%9F%AF%E6%9E%97%E6%BC%A2)）

### 执行周期中三个阶段

Awk在执行逻辑时，包含了三个阶段：Begin、Exec和End。其中Begin和End都是可略过的，甚至无需声明。可以由前至后传递参数，如下例中的words数组。

```shell

    awk \
    # begin 阶段
    'BEGIN { \
        FS="[^a-zA-Z]+" \
    } \
    # exec 阶段
    { \
         for(i=1; i<=NF; ++i) \
              words[tolower($i)]++ \
    } \
    # end 阶段
    END { \
        for(i in words) \
             print i, words[i] \
    }' \

```

### 自定义列分割符

awk比较擅长分割文本，默认以空格作为分割符。我们还是以上面的test.txt和test2.txt为例，来展示一下awk的功能

```shell
	$ cat text.txt
    abc abc abc
    xyz xzy xyz
      abc abc abc
    mq mq mz
    
    $ cat text2.txt
    name: xiaoming subject: Math score: 59
    name: xiaoming subject: English score: 89
    name: xiaoming subject: ComputerScience score: 98
    name: xiaohong subject: Math score: 96
    name: xiaohong subject: English score: 98
    name: xiaohong subject: ComputerScience score: 72
```

```shell
	>$ awk '{print $0}' test.txt
    >abc abc abc abc
    >xyz xzy xyz
    >  abc abc
    >mq mq mz
    
	>$ awk '{print $1}' text.txt
    >abc
    >xzy
    >abc
    >mq
    
    >$ awk '{print $2}' text.txt
    >abc
    >xzy
    >abc
    >mq
    
    >$ awk '{print $3}' test.txt
    >abc
    >xyz
	>
    >mz
```
像例子中展示的那样，awk默认将文本以空格为分隔符做了列切分，这里要注意text.txt的第三行是以空格起始的，但是awk对此做了trim，并没有出现第一列为空的情况。而因为第三行只有两列字符，所以第三列是空的。

那如何自定义换行符呢？-F参数可以解决这个问题。效果与空格是一样的。

```shell
	$cat test3.txt
    abc:abc:abc:abc
    xyz:xzy:xyz
    abc:abc
    mq:mq:mz
    
    $awk -F ':' '{print $1}' test3.txt
    >abc
	>xyz
	>abc
	>mq
    
    $awk -F ':' '{print $3}' test3.txt
	>abc
	>xyz
	>
	>mz
```
让我们再升级一下，如果想混合多种分隔符（如空格+冒号），该怎么做？

这时候就得使用BEGIN了。在下面的test4.txt中，混合了冒号和空格作为分隔符，让我们来使用awk切分它。

```
	$cat test4.txt
    abc:abc:abc abc
    xyz xzy:xyz
    abc:abc
    mq mq mz
    
    $awk 'BEGIN { FS=":| " } {print $1}' test4.txt
    >abc
    >xyz
    >abc
    >mq
    
    $awk 'BEGIN { FS=":| " } {print $2}' test4.txt
    >abc
	>xzy
	>abc
	>mq
    
    $awk 'BEGIN { FS=":| " } {print $3}' test4.txt
	>abc
	>xyz
	>
	>mz
```

这里刚才的-F函数已经化名成FS了，而且还可以用“|”来分隔多个。从效果来看，确实与单个分隔符的效果一样。

### 内置变量

在上段文字中我们已经见过了$0~$n这样的变量，和shell中的函数参数很类似，$n代表了第n列元素。而$0会输出整段文章。除了这些参数外，awk还提供了如下表中的参数。

<table>
	<thead>
		<tr>
        	<th>变量名</th>
        	<th>解释</th>
        </tr>
    </thead>
    <tbody>
		<tr>
        	<td>FS</td>
        	<td>输入列分标识，默认是空格</td>
        </tr>
        <tr>
        	<td>RS</td>
        	<td>输入换行标识，默认是换行符</td>
        </tr>
        <tr>
        	<td>OFS</td>
        	<td>输出列分标识，默认是空格</td>
        </tr>
        <tr>
        	<td>ORS</td>
        	<td>输出换行标识，默认是换行符</td>
        </tr>
        <tr>
        	<td>NF</td>
        	<td>当前记录中的字段个数，切分后的列数，可用在循环中</td>
        </tr>
        <tr>
        	<td>NR</td>
        	<td>已经读出的记录数，就是行号，从1开始</td>
        </tr>
        <tr>
        	<td>FILENAME</td>
        	<td>当前输入文件的名字</td>
        </tr>
        <tr>
        	<td>FNR</td>
        	<td>当前记录数</td>
        </tr>
    </tbody>
</table>

### 循环、判断与打印

awk支持循环和判断来表达分支逻辑。awk循环支持while和for常见的循环形式。如下面的例子，我们把"{{ }}"之间的内容打印出来。

```shell

    >$cat test5.txt
    >This is {{ namea }}, who is {{ nameb }}?
    >{{ namec }} is speaking.

    >$awk 'BEGIN{FS="{{ | }}"} {for (i=2; i<NF; i=i+2) print $i}' test5.txt
    >namea
    >nameb
    >namec
	
```

使用while，将行内内容转置

```shell

    >$cat test2.txt
    >name: xiaoming subject: Math score: 59
    >name: xiaoming subject: English score: 89
    >name: xiaoming subject: ComputerScience score: 98
    >name: xiaohong subject: Math score: 96
    >name: xiaohong subject: English score: 98
    >name: xiaohong subject: ComputerScience score: 72

    >$awk '{i=1; while(i<NF) {print $i$(i+1); i=i+2; if(i%7 == 0) print "*******"}}' test2.txt
    >name:xiaoming
    >subject:Math
    >score:59
    >*******
    >name:xiaoming
    >subject:English
    >score:89
    >*******
    >name:xiaoming
    >subject:ComputerScience
    >score:98
    >*******
    >name:xiaohong
    >subject:Math
    >score:96
    >*******
    >name:xiaohong
    >subject:English
    >score:98
    >*******
    >name:xiaohong
    >subject:ComputerScience
    >score:72
    >*******
    
```

# 2 应用

## 2-1 使用awk获取pid

在工作中，使用脚本重启应用是非常常见的场景。所以使用awk获取pid，并传给kill命令，可以满足自动杀死应用的场景。

```

	>$ps aux| grep tomcat | grep -v 'grep tomcat' | awk '{print $2}' | xargs kill -9

```

这里使用awk截取了ps aux结果中的第二列（对，第二列就是进程号）。其中因为这句grep tomcat会把grep进程的进程号也读出来，所以有个第二个grep -v，来去掉第一个grep的进程号。然后使用xargs将进程号传给kill，完成杀死进程的操作。

## 2-3 使用awk、sed替换模板内容

来吧，前面铺垫那么多，就是为了下面这个例子。下面我们要用awk和sed来解析配置文件。test_final.txt中包含了一个模板，"{{  }}"代表将要被替换的值。

```
	>$cat test_final.txt
	>db.url={{ tpl_url }}
	>db.name={{ tpl_name }}
	>db.password={{ tpl_password }}
```
```
	#!/bin/bash

	# 转义反斜杠
	tpl_url=jdbc:mysql:\/\/8.8.8.8:3306\/awk_sed
	tpl_name=username
	tpl_password=password

	# 读取占位符，到数组placeHolderList中
	placeHolderList=($(awk 'BEGIN { FS="{{ | }}"} {for (i=2; i<=NF;i=i+2) print $i}' test11.txt))
	# 遍历占位符
    for placeHolder in ${placeHolderList[@]}
	do
    	# ${!placeHolder}可以取到$placeHolder作为引用所指向的值（就是右边表达式：${placeHolder}=${!placeHolder}）
    	sed -i.bak 's/{{ '${placeHolder}' }}/'${!placeHolder}'/g' test11.txt
	done
```

就写这些吧，希望能帮助到您。有任何问题，请给我留言。谢谢！

欢迎转发此文，请注明出处https://www.gkwen.com/blog/awk_sed

