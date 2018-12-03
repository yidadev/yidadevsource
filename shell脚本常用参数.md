shell的常用关键字

``` xml
echo "shell 输出脚本名称及参数";
echo "执行的脚本名: $0 ";
echo "第一个参数： $1";
echo "第二个参数： $2";
echo "传递到脚本的参数 $#"
echo "脚本运行的当前进程ID号 $$"

以一个单字符串显示所有向脚本传递的参数。
如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done
```

``` xml
//拼接字符串
your_name="qinjx"
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"

//获取字符串的长度
string="abcd" echo ${#string} #输出 4

//提取字符串
string="runoob is a great site"
echo ${string:1:4} # 输出 unoo
```

数组的定义：
``` xml
array_name=(value0 value1 value2 value3)

array_name[0]=value0
array_name[1]=value1
array_name[n]=valuen

读取数组：
${数组名[下标] valuen=${array_name[n]}
echo ${array_name[@]}  使用@符号可以获取数组中的所有元素

# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}

```
expr
(1)."string : REGEX"字符串匹配示例。要输出匹配到的字符串结果，需要使用"\("和"\)"，否则返回的将是匹配到的字符串数量。
(2)."index string chars"用法示例。
(3)."substr string pos len"用法示例。


数字

-eq	检测两个数是否相等，相等返回 true。
ne	检测两个数是否相等，不相等返回 true。	[ $a -ne $b ]
-gt	检测左边的数是否大于右边的，如果是，则返回 true。	[ $a -gt $b ]
-lt	检测左边的数是否小于右边的，如果是，则返回 true。	[ $a -lt $b ]
-ge	检测左边的数是否大于等于右边的，如果是，则返回 true。 [ $a -ge $b ]
-le	检测左边的数是否小于等于右边的，如果是，则返回 true。 [ $a -le $b ]

if [ $a -eq $b ]
then
   echo "$a -eq $b : a 等于 b"
else
   echo "$a -eq $b: a 不等于 b"
fi

!	非运算，表达式为 true 则返回 false，否则返回 true。	[ ! false ]
-o	或运算，有一个表达式为 true 则返回 true。	[ $a -lt 20 -o $b -gt 100 ]
-a	与运算，两个表达式都为 true 才返回 true。	[ $a -lt 20 -a $b -gt 100 ]

&&	逻辑的 AND	[[ $a -lt 100 && $b -gt 100 ]] 返回 false
||	逻辑的 OR	[[ $a -lt 100 || $b -gt 100 ]] 返回 true

字符串：

=	检测两个字符串是否相等，相等返回 true。	[ $a = $b ] 返回 false。
!=	检测两个字符串是否相等，不相等返回 true。	[ $a != $b ] 返回 true。
-z	检测字符串长度是否为0，为0返回 true。	[ -z $a ] 返回 false。
-n	检测字符串长度是否为0，不为0返回 true。	[ -n $a ] 返回 true。
str	检测字符串是否为空，不为空返回 true。	[ $a ] 返回 true。


-b file	检测文件是否是块设备文件，如果是，则返回 true。	[ -b $file ]
-c file	检测文件是否是字符设备文件，如果是，则返回 true。	[ -c $file ]
-d file	检测文件是否是目录，如果是，则返回 true。	[ -d $file ]
-f file	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。	[ -f $file ] 返回 true。
-g file	检测文件是否设置了 SGID 位，如果是，则返回 true。	[ -g $file ] ]
-k file	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。	[ -k $file ] 返回 false。
-p file	检测文件是否是有名管道，如果是，则返回 true。	[ -p $file ] 返回 false。
-u file	检测文件是否设置了 SUID 位，如果是，则返回 true。	[ -u $file ] 返回 false。
-r file	检测文件是否可读，如果是，则返回 true。	[ -r $file ] 返回 true。
-w file	检测文件是否可写，如果是，则返回 true。	[ -w $file ] 返回 true。
-x file	检测文件是否可执行，如果是，则返回 true。	[ -x $file ] 返回 true。
-s file	检测文件是否为空（文件大小是否大于0），不为空返回 true。	[ -s $file ] 返回 true。
-e file	检测文件（包括目录）是否存在，如果是，则返回 true。	[ -e $file ] 返回 true。

//输出到文件
echo "It is a test" > myfile
echo `date`输出时间
