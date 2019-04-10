# 0x00 简介

本章简单的介绍一下sleep2.1的语法主要是写给跟我一样不会英语的朋友，用在编写脚本上，大家可以查询文档http://sleep.dashnine.org/download/sleep21manual.pdf 
用pdf方便搜索函数,本文只是简单的介绍语法，想深入学习的还请看官方文档，AggressorScripts主要是对CS提供的函数的一些调用所以用不到太多的sleep相关的。
大家也可以下载sleep.jar 来测试语法。


# 0x01 sleep 2.1 基本语法


### 打印

都先来学习 打印hello word吧 

```
println("Hello World")

```

`load` 加载 `x` 用于调试函数

```
 $  java -jar sleep.jar
>> Welcome to the Sleep scripting language
> help
debug [script] <level>
env [functions/other] [regex filter]
help [command]
interact
list
load <file>
unload <file>
tree [key]
quit
version
x <expression>
? <predicate expression>
> load 1.txt
G:\教程\Cobalt Strike2\img\1.txt loaded successfully.
Hello World
```


### 变量

定义变量用`$`  

```
> interact
>> Welcome to interactive mode.
Type your code and then '.' on a line by itself to execute the code.
Type Ctrl+D or 'done' on a line by itself to leave interactive mode.
$name = "404";
println($name);
.
404
```


### 运算

```
$x = 5 + "1";
$x = 5 - $y;
$x = $x * $2;
$x = $z / 9.9;
$x = $1 % 3; # modulus
$x = $1 ** 4; # exponentation
```

字符串

```
$x = "Ice" . "cream"; 组合 "Icecream"

$x = "abc" x 3;  abcabcabc

```


### 数组

定义一个数组用 `@`

```
@foo = @("a", "b", "c");                                          
$x = @foo[1];                                                     
println($x);                                                      
.                                                                 
b                                                                 

```

### Hash Scalars

类似字典

```
%bar = %(x => "x-ray", y => "yabboes");

```


### 函数

定义函数 `sub function` 参数为`$1,$2,$3`

```
sub add
{
    $a = $1;
    println($a);
}

```

### if-else 条件判断

```
$nub = 1;
$nub2 = 2;
if ($nub>$nub2)
{
    println("1>2");
}
else
{
    println("1<2");
}
```

### 逻辑运算符

`and(&&)` `or(||)`

```
sub check
{
if (($1 > 0) && ($2 <= 10))
{
println("$1 is within 0 .. 10");
}
else
{
println("$1 $2 is out of range!!");
}
}

```

### 循环语句

For 循环

```
for ($total = 99; $total > 0; $total--)
{
    println($total);
}

```

while 循环

```
$total = 99;
while ($total > 0)
{
    println($total);
}

```

Foreach 循环

```
%data = %(foo => "a", bar => "b", baz => "c", jaz => "d");
foreach $key => $value (%data)
{
    println($key);

}

```


# 0x02 sleep 2.1 函数

这里我就不讲了 看官网文档把自己想用什么函数搜索下,不会英语就机翻下,大概就知道了。


# 0x02 文末


### 本文如有错误，请及时提醒，以免误导他人