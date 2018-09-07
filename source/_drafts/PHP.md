---
title: PHP备忘录(1)
date: 2018-07-08 13:00:28
categories: Notes
tags:
description:
---

记录一下学习PHP基础中的要点
<!-- more -->

# PHP基本语法 

## PHP标记语法
PHP标记语法总共有4种
```php
// 标准格式(XML格式)
<?php echo 1; ?>

// script格式
<script language="php">
    echo 1;
</script>

// 短格式
<? echo 1; ?> // 默认不开启，需要配置PHP配置文件 short_open_tag = On

// asp格式
<% echo 1; %> // 默认不开启，需要配置PHP配置文件 asp_tags=On
```
注意：
 - 如果没有结束符`?>`，那么在开始标签之后都会被当做是PHP去执行
 - 最后一行PHP代码可以没有`;`，因为`?>`可以看成`;?>`，它默认带上了`;`符号

## 注释
```php
// 单行注释

/* 多行注释 */
```

<br/><br/>
# 变量
变量是内存中用于临时存储数据的一个空间，这个空间有一个名字，这个名字就是变量名。变量名是用于对这个内存中的数据进行引用。
[很好的理解](https://www.zhihu.com/question/34266997)

## 变量基本操作
```php
// 语法
$变量名 = 值;

// 变量的声明
$v1 = 1;

// 变量修改
$v1 = 2;

// 变量删除
unset($v1); // unset()就是将变量从内存中销毁
```
**定义变量注意点：**
 - PHP中的变量以$开头
 - 变量名只能包含字母、数字、下划线，只能以字母或者下划线开头



## 可变变量
```php
// 1. 通过一个变量访问到另一个变量
$v1 = 'age';
$age = '20';
echo $v1; // age
echo $$v1; // 20 $$v1 > $age > 20

// 2. 通过一个变量创建一个变量
$v1 = 'age';
$$v1 = 20; // $$v1 > $age > 20
echo $v1; // 20
echo $age; // age
```

## 预定义变量
PHP为了我们预先定义了一组变量，这些变量会在不同的需求中使用
```
$_GET         // 接收前台表单使用GET方式提交的数据
$_POST        // 接收前台表单使用POST方式提交的数据
$_REQUESTS    // 接收前台表单使用GET或POST方式提交的数据
$_SERVER      // 头信息(header)、路径(path)、以及脚本位置(script locations)等等信息的数组
$_COOKIE
$_SESSION
$_FILE        // 用户上传的文件信息
$GLOBAL       // 记录全局变量
```

<br/><br/>
# 内存原理
程序语言就是对内存进行操作(对内存进行读和写)

## 内存结构
![内存结构](2018-07-08-14-37-11.png)
![内存结构](2018-07-08-14-36-51.png)
![内存结构](2018-07-08-14-37-28.png)



## PHP的执行过程

### 编译阶段
会先对程序进行
 - 语法检查(例如for循环结构)
 - 词法检查(对代码关键字function,while等进行检查)
 - 代码优化(将某些冗余代码优化)

如果编译通过之后会将源代码转换为机器指令。

### 执行阶段
如果编译通过后，会将源代码对应的机器指令存储在代码段中，在开始执行代码段中的机器指令。


![PHP执行过程实例](2018-07-08-15-07-12.png)


## PHP代码嵌入到HTML中
PHP功能模块在处理一个PHP文件时，仅会处理PHP代码(`<?php ... ?>`)，会把其他内容(`HTML、CSS、JS`)都当作字符串


它会将代码从上到下执行，将不是PHP代码原样存入到 **内存中的输出缓存区**
将是PHP代码执行，如果有输出命令，则将输出的内存也存储到 **内存的输出缓存区**
最后将 **内存的输出缓存区的所有内容**返回给Apache

```php
// PHP程序源码
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
<?php echo 'hello world' ?>
</body>
</html>

// 内存中输出缓存区结果
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
hello world
</body>
</html>
```

## PHP中的变量赋值方式
有两种传值方式
1. 赋值传值
`$a=$b`传递的是`$a`的值
2. 引用传值
`$a=&$b`传递的是`$a`的地址

<br/><br/>
# 常量
常量是一种特殊的变量，也是用于存储数据。

说明：
 - 常量一旦定义就不能修改和删除
 - 常量的值只能是标量(整数、字符串、布尔...)
 - 常量是默认区分大小写的，一般常量名在命名时，我们会使用全大写的方式

```php
// 常量的定义
// 1
define('常量名', 值, 是否不区分大小写); // 默认区分大小写
// 2
const 常量名 = 值;

// 判断常量是否存在
defined(const name);

// 获取所有常量
get_defined_constants();
```

## define VS const
define可以在分支结构(for循环中)中定义常量，const不允许。
define可以自定义是否区分大小写。
const必须PHP>=5.3


## 魔术常量
```php
__FILE__         // 当前文件名完整路径
__DIR__          // 当前文件的路径 PHP>=5.3
__LINE__         // 获取当前行数
__FUNCTION__     // 获取当前函数名
__METHOD__       
__CLASS__        // 获取当前类名
__NAMESPACE__    // 获取当前命名空间
```


<br/><br/>
# 数据类型
PHP的数据类型分为三大类，八小类
注意：以下描述与手册相比较不全

 - 标量(scalar)数据类型
   - int
   - float(也称double)
   - boolean
   - string
 - 复合数据类型
   - array 
   - object   对象
 - 特殊数据类型
   - null     
   - resource 资源类型

## 整数(Int)
 - 十进制
   - `echo 255`
 - 八进制
   - 以0开头，并且不能有超过7的数字
   - `echo 076`
 - 十六进制
   - 以0x开头，包含0-9,a-f
   - `echo 0xFF`


## 浮点(Float)
小数有两种表示方式：
1. 普通方式
`3.1415926`
2. 科学技术法
`12e9` 或者 `12E+9`

在PHP中需要注意浮点数中的坑
```php
if(0.7-0.3==0.4){
	echo 'yes';
}else{
	echo 'no';
}
// result:no
```
![浮点数的警告](2018-07-08-16-10-54.png)


## 字符串(String)
字符串中需要注意：
 - 单引号和双引号的区别和使用
 - 转义符的理解

```php
$age = 20;
// 单引号
$str = 'This age is\t$age'; // 单引号中不识别变量名和空白字符

// 双引号
$str2 = "This age is\t{$age}"; // 双引号中可以识别变量名和空白字符

// heredoc   定义长字符串
// 语法要求：开始标记和结束标记必须相同名称，并且结束标记必须定格
$hereDoc = <<<here
123
123
123
here;

// 转义符 \
// 用于将程序语言所赋予的某些字符的特殊功能转义掉
\'
\"
\$
\\  
\t  TAB
\r  回车
\n  换行
```

## 布尔类型(Boolean)
布尔类型只有两个值：
 - `true`
 - `false`

## 数组(Array)
数组中分为：
1. 索引数组
数组元素的下标是数值
2. 关联数组
数组元素的下标是字符串

```php
// 变量(数组)与字符串进行拼接
echo "This is a $_SERVER[REMOTE_ADDR]"; 
// 注意：不需要给数组加单引号，因为REMOTE_ADDR已经是一个字符串了，它在双引号内!!!不会被当做常量来解析
// 如果使用花括号，那么必须加单引号，因为花括号相当拼接字符串，会被报出没找到常量的错误
```


## NULL
NULL类型只有一个值就是`NULL`

## 资源类型(Resource)
资源数据类型是一个特殊的变量，程序员是无法直接定义一个资源，必须使用PHP提供的获取资源的函数
不属于PHP所管辖范围的，都是资源类型，例如打开文件的连接，就是一个资源
```php
$file = fopen('data.txt','r');
var_dump($file); // resource(3) of type (stream) 
```



## 数据类型转换
数据就是用于运算，当参与运算的两个数据，类型不同时，PHP会自动进行转换。
有时，我们也会进行强制转换

### 自动转换


### 强制转换
```php
$v1 = 10;
(integer)$v1;
(float)$v1;
(array)$v1;
(object)$v1;
(string)$v1;
(boolean)$v1;
```

强制转换中的一个说明：
```php
// 将以下内容 (boolean)$vx
$v1 = 10;  // true
$v2 = 0;   // 以下都为false
$v3 = '';
$v4 = '0';
$v5 = array();
$v6 = null;
$v7 = 0.0;
```

## 数据类型判断
```php
// 类型判断
is_int()
is_string()
is_bool()
is_float()
is_scalar() // 注意：判断是否为标量！！！
is_array()
is_object()
is_null()
is_resource()

// 其他
isset() // 判断变量是否有设置值(其实判断变量知否为NULL) -> 所以NULL返回FALSE
empty() // 只要是被强制转换为BOOL后位FALSE的，都为空，如果空返回TRUE，否则返回FALSE
```






<br/><br/>
# 运算符

## 自操作运算符

 - `++`(自增)
   - `$result = ++$i`(前自增)：先将内存中变量的值自增1，在获取变量的新值参与式子的运算。
   - `$result = $i++`(后自增)：先获取变量的新值参与式子的运算，在将内存中变量的值自增1。
 - `--`(自减)   以下同上

 ```php
$i+=2  // $i = $i + 2;
$i-=2  // $i = $i - 2;
$i*=2  // $i = $i * 2;
$i/=2  // $i = $i / 2;
$i%=2  // $i = $i % 2;
 ```

## 字符串运算符

 - `.`  对字符串进行拼接
 - `.=` 对字符串进行拼接

注意：如果字符串和整数进行拼接
```php
$str = 'num is:' . 10 // 数值和.之间必须加上空格，不然会认为是浮点数
```


## 比较运算符

需要注意`==`与`===`，前者只需要数值相同，后者不仅需要数值相同，还需要类型相同。


## 逻辑运算符
 - `&&`  (逻辑与)
 - `||`  (逻辑或)
 - `!`   (逻辑非)
 - `and` (逻辑与)
   - 和`&&`运算规则相同，但是`and`优先级低于`=`
 - `or`  (逻辑或)
   - 同上

### 逻辑与短路
 ```php
 $v2 = 10;
 $result = false && ++v2;
 var_dump($result); // 10
 // 因为逻辑与运算中有一个为FALSE，结果就为False，就没必要去计算第二个值的结果
 ```


### 逻辑或短路
```php
$v2 = 10;
$result = true || ++v2;
var_dump($v2); // 10
// 逻辑或运算中，只要有一个为True或能转换为True，结果就返回True，就没必要去计算第二个值的结果
```

### 逻辑非的一种用法
```php
!!$v; // 这样写的含义是将变量强制转换为false
```

### 条件运算符
三元运算符：(可以看做一个简单的IF语句)
1. 表达式?表达式A:表达式B;
2. 变量?表达式B;
使用要求php>=5.6

运算规则：
  先计算表达式是否成立，如果成立取表达式A的值，否则取B的值。
```php
isset($result)?$result:'';
```


<br/>
## 错误控制运算符
隐藏错误信息的方法有多种

### 错误抑制符
`@`符号为错误抑制符，当将`@`放置在一个 PHP 表达式之前，该表达式可能产生的任何错误信息都被忽略掉。
例如`mysql_connect('127.0.0.1','root','2222');`


### 更改php.ini
隐藏错误信息也可以在`php.ini`中进行隐藏`display_errors=Off`,这将隐藏使用该配置文件的站点所有错误信息

### 脚本级的错误控制
可以通过PHP函数来设置当前PHP脚本文件的错误控制被称为脚本级的错误控制
```php
ini_set('display_errors','Off') // 设置php.ini的配置项，只作用于当前脚本


ini_get() // 获取php.ini的配置项

```


<br/>
## 运算符优先级
在一个式子中可能会出现多种运算符，但是在运算符之间会有优先级
运算口诀：`单 算 关 逻 条 赋 逗`
单路运算符就是`++ -- ~ (int)`


<br/><br/>


# 流程控制
流程控制可分为三种
1. 顺序结构
程序自上而下的一个执行过程。
2. 分支结构
根据某一条件将程序转向不同的分支处执行。
3. 循环结构
解决重复性的问题。


## 分支结构

### if
```php
// 语法
if(条件1){

}else if(条件2){

}else{

}
```
在if中，如果语句体只有一句语句，此语句体的{}可以省略

### switch
```php
switch(){
  case 值1:
    语句体 1
    break;
  case 值2:
    语句体 2
    break;
  default:
    缺省语句体
}
```
如果在`case`中不带有`break`，会直接执行下一个`case`，直到遇到`break`或者到结尾为止。


## 循环结构

### for
```php
for(循环控制变量初始值;表达式;循环控制变量的更改){
  循环体
}
```
执行顺序：
1. 首先执行循环控制变量初始化，此步仅执行一次
2. 判断表达式是否成立，不成立跳出循环
3. 执行循环体
4. 执行循环控制变量的更改
5. 判断表达式是否成立，不成立跳出循环
6. 执行循环体
7. ...


### while
```php
while(表达式){
  循环体
}
```
与for的区别：while用于循环次数未知的循环

执行顺序：
1. 判断表达式是否成立，不成立执行
2. 执行循环体
3. 判断表达式是否成立，不成立


### do...while
```php
do{
  循环体
}while(表达式);
```
和while的区别：无论表达式是否成立，都会先执行一次循环体


### 循环的结束语退出

#### continue
```php
continue [n] // n为整数，如果缺省为1
```
结束当前循环结构的本次循环，继续上n层循环结构的下一次执行。

#### break
```php
break [n] // n为整数，缺省为1
```
直接结束上n层循环结构。


<br/><br/>
# PHP其他知识点
## PHP的输出语法
 - `echo`
   - 只能输出标量，对于任何数据都会转换为字符串输出
 - `print()`
   - 只能输出标量，对于任何数据都会转换为字符串输出
   - print 实际上不是函数（而是语言结构），所以可以不用圆括号包围参数列表。 
   - 和 echo 最主要的区别： print 仅支持一个参数，并总是返回 1。 
 - `print_r()`
   - 可以打印`integer string float array`类型的数据内容
 - `var_dump()`
   - 打印数据本身和数据类型
 - `sprintf()`
   - 格式化字符串中的占位符用于指明输出的参数值如何格式化
   - `echo sprintf('This is %s',$str)`
   - `echo sprintf('bin is %b',255)`
   - `echo sprintf('float is %.2f',10)`
   - 其他打印函数无法输出小数点等

## 使用PHP自带的服务器

## PHP命令行
```
php -r
php -f
php --ini
php -S
php -m
```

## PHP标签语法
当在HTML插入PHP代码的时候使用PHP标签语法
```php
<?php for($i=1;$i<=4;$i++){?>
    <span><?php echo $i?></span>
<?php }?> // 结尾 } 前面必须要加空格，不然报错

// if标签语法
// 标准语法
<?php if(...):?>
  // 语句体
<?php endif ?>

// 简化语法
<?php if(...){>
  // 语句体
<?php }?>

//for标签语法
// 标准语法
<?php for(...):?>
  // 循环体
<?php endif ?>

<?php for(...){?>
  // 循环体
<?php }?>

```

<br/><br/>
# 文件包含
```php
include(文件名)
include_once(文件名)

require(文件名)
require_once(文件名)

// once 会在每次引入文件时，检查该文件是否已经被引入，如果引入就不会在引入
// 重复包含可能会引发 函数重名等报错
```

如果在当前脚本，函数可以先引用后调用
但是如果使用文件包含去引入函数，必须先引入，在调用函数


一般是为了在PHP文件中获取数据，在HTML文件中展现数据

## 引入路径问题
在实现项目中，对于HTML文件，是不允许用户直接请求，而是指向一个PHP文件，让PHP文件来引入这个HTML文件
当一个PHP文件引入一个HTML文档时，html文件本身也会引入一些其他的文件，如图片、CSS、JS
在这个时候就会发生路径更改的问题

解决办法：
使用绝对路径或者使用正确的相对路径


## include和require区别
`include`如果引用文件出现问题，虽然会报错，但仍然会执行之后的代码。
`require`如果引用文件出现问题，会直接中断PHP程序执行。

## 魔术变量问题
```php
__DIR__
__FILE__
```
如果引入的文件中带有以上魔术常量，它不会随着文件引入而改变，而是存在于哪个文件，这个常量的值就是哪个文件


<br/><br/>
# 错误处理
## 错误的分类
 - 编译错误
编译的过程中发生的错误就是编译错误，编译错误最容易解决，很多时候是书写错误

 - 执行错误
在编译通过后，在执行阶段发生的错误。此种错误一旦发生，会根据错误的等级，来决定是否中断程序的执行。

 - 逻辑错误
由于程序的逻辑不严谨，而产生的错误。此种错误是最难排查的。程序可以正常的执行，但是结果并不是我们所期望的。

## 错误代码
每一种错误，都有一个错误标识
```php
echo '<pre>';
print_r(get_defined_constants());
echo '</pre>';

// 系统错误
[E_ERROR] => 1                   // 致命错误 发生中断程序
[E_WARNING] => 2                 // 警告错误 不会中断程序
[E_PARSE] => 4                   // 编译错误 发生中断程序
[E_NOTICE] => 8                  // 提示错误 不会中断程序
[E_STRICT] => 2048               
[E_RECOVERABLE_ERROR] => 4096    // 警告错误 不会中断程序

// 弃用函数的错误(函数还可以用，但是使用弃用函数会报错)
[E_DEPRECATED] = 8192

// 核心错误(编译器的错误)
[E_CORE_ERROR] => 16
[E_CORE_WARNING] => 32
[E_COMPILE_ERROR] => 64
[E_COMPILE_WARNING] => 128

// 用户错误(可以用户自定义)
[E_USER_ERROR] => 256
[E_USER_WARNING] => 512
[E_USER_NOTICE] => 1024
[E_ALL] => 6143
```


## 用户自定义错误
```php
trigger_error(msg, type);
// msg: 错误描述信息
// type：自定义错误的代码（E_USER_ERROR,E_USER_NOTICE...）

// 例子
is_arrays(1);
function is_arrays($arr){
	if(!is_array($arr)){
		trigger_error('not is array',E_USER_NOTICE);
	}
}

// 这样的错误信息能被日志所记录
Notice: not is array in D:\PhpStudyCode\php\test.php on line 9
```

## 控制错误的显示
控制是否显示错误
```
display_errors=On/Off
```
控制显示哪一类错误
```
error_reporting=E_ALL 
error_reporting=E_NOTICE | R_WARNING
error_reporting=E_ALL & ~E_NOTICE

注意：可以按照位去运算来控制
```

##  错误日志设置
错误日志就是将PHP程序的报错信息进行记录
```php
// 开启日志记录
log_errors=on

// 错误日志存放位置
error_log=syslog // 会记录到操作系统的日志中
error_log=文件名 // 会记录到自定义的位置
// 如果没有设置error_log默认是记录到apache的错误日志中\logs\error.log
```

<br/><br/>
# 函数
函数是模块化程序的产物。在实际开发过程中，会将项目划分为各个大的功能模块。同样将大的功能模块划分为小的功能。

**php中函数名是不区分大小写**。函数名的命名规则与变量的命名规则相同。只能包含字母、数字、下划线。而且以字母下划线开头。


## 可变函数
类型可变变量，函数名也可以使用变量 
```php
function showInfo(){
  echo 123;
}
$f = 'showInfo';
$f();
```

## 函数的参数
### 形参与实参
函数定义时的参数是形参
函数调用时的参数是实参

### 形参默认值
php在定义函数时，可以为形参赋值，这个值就是形参的默认值。
如果调用函数没有给具有默认值的形参传递数据，那么形参会使用默认值。
**一般具有默认值的形参，一般位于形参列表的最后**
```php
function showInfo($msg='hello'){
  echo $msg;
}
showInfo('123'); // 123
showInfo(); // hello
```

### 形参和实参之间的引用传值
形参和实参之间其实是一个赋值过程，那么我们也可以将这个赋值过程，变为一个引用传值过程。
```php
function showInfo(&$msg){
  $msg = 200;
}
$msg = 100;
showInfo($msg);
echo $msg;

// 低版本是在实参前加&符号，而高版本是在形参前加&符号
```

### 伪类型
PHP所规定的另外几种类型，我们在翻阅手册会经常碰到
```php
mixed       // 表示类型不确定
callback    // 表示函数
scalar      // 如果是int float string boolean
```

### 相关函数(系统函数)
```php
// 方法1
function showInfo(){
  echo func_num_args(); // 获取实参的个数
  echo func_get_arg(0); // 第一个实参
  print_r(func_get_args()); // 所有的实参
}
showInfo(1,2,3,4);

// 方法2 PHP>=5.6  
// 必须三个 . 号 
function showInfo(...$args){
	print_r($args); // Array ( [0] => 1 [1] => 2 [2] => 3 [3] => 4 )
}

showInfo(1,2,3,4); 
```
### 函数运行的内存原理
![函数运行的内存原理](2018-07-11-19-15-20.png)




## return
执行完函数需要返回数据需要使用`return`
函数内部可以有多个`return`，只要遇到一个`return`就会中断函数的执行，在函数内也可以忽略`return`

## 匿名函数
没有名字的函数就是匿名函数

```php
$fn = function(){
  echo 123;
};
$fn(); // 123
// 匿名函数没有名节，并且结尾必须加入分号!!!
// PHP中的匿名函数无法自调用
// 但是可以赋值给一个变量，还可以作为某个函数的参数
// 所以我们可以将它赋值给变量，并通过变量来调用。
```

### 回调函数callback
在开发过程中，我们使用别人的函数或者系统函数时，函数的参数需要你传递一个函数，作为参数传递的函数就是回调函数。
```php
function showInfo($fn){
	$fn();
}

function say(){
	echo 123;
}


showInfo('say');
// 使用匿名函数作为回调函数
showInfo(function(){
  echo 123;
});

```

## 作用域
PHP中分为两种作用域：
1. 全局作用域
2. 局部作用域

### 全局作用域与全局变量
在函数外部定义的变量，其作用域就是全局作用域，这个变量就是全局变量

### 局部作用域与局部变量
在函数内部定义的变量，其作用域就是局部作用域，这个变量就是局部变量


在PHP中只有相同作用域的才能互相访问

### global关键字
很多时候我们需要在局部变量中使用全局变量，或者全局变量中访问内部。
所以有以下方式：
```php
// 1. 使用引用赋值(内部访问外部)
function showInfo(&$msg){
  $msg+=100; 
}
$msg = 100;
showInfo($msg);
echo $msg;

// 2. 使用超全局变量$GLOBALS(内部访问外部)
// $GLOBALS用数组的形式记录了所有全局变量
function showInfo(){
  $GLOBALS['msg']+=100;
}
$msg = 100;
showInfo();
echo $msg; // 200

// 3. 使用global关键字(内外互相访问)
/*
global关键字背后所做的其实是
在函数内部创建一个与函数外部同名的变量的引用。如果外部没有同名变量，会自动创建一个。
*/
// 内部访问外部
function showInfo(){
  global $msg;
  $msg+=100;
}
$msg = 100;
showInfo();
echo $msg; //200

// 外部访问内部
function showInfo(){
  global $msg;
  $msg = 100;
}
showInfo();
echo $msg; // 100
```

### 常量的作用域
常量没有作用域限制，一个脚本的常量，在任何的位置都可以访问。

### 预定义变量(超全局变量)
超全局变量也不收作用域限制。



## 生命周期

### 作用域与生命周期
作用域表示的是一个变量的作用空间范围。

生命周期表示的是一个变量的作用时间范围。



### 全局变量的生命周期
全局变量的生命周期是从脚本执行开始到脚本执行结束。

### 局部变量的生命周期
局部变量的生命周期是从函数执行开始，到函数执行结束。

## 静态变量
静态变量需要是用`static`关键字来定义。它只会被初始化一次，如果有函数多次调用，它不会随着函数生命周期结束后空间的销毁而销毁。
```php
function add(){
  static $num = 1;
  $num+=1;
  echo $num;
}
add(); // 2
add(); // 3
add(); // 4
add(); // 5

// 静态变量的使用场景，如果想在同一个函数多次调用时，共享一份数据。那么就可以使用静态变量。
```
 
![静态变量的内存存储](2018-07-11-20-39-11.png)


## 系统函数
### 日期时间函数
```php
time(); // 获取时间戳。单位是秒，时间戳就是时间原点至今的一个秒数。
// 时间原点：1970年1月1日0时0分为时间原点。

microtime(); // 同上，但是会获得毫秒值

date(format [,time]); // 格式化返回时间信息
/*
format: Y-m-d H:i:s
data: 表示需要格式化的时间戳，默认当前时间
*/

mktime(时, 分, 秒, 月, 月, 日, 年) // 用于获取指定时间的时间戳

strtotime() // 运算时间戳，可以运算某时间点一周之后的时间戳
/*
使用它，可以快速算出例如用户密码更换策略的时间等等
*/
```




## 递归思想
递归思想其实就是函数自己调用自己，一般用于解决有规律性的重复
```php
/*
1 2 3 5 8 13 21 34 55 89 ...
*/

function func($pos){
	if($pos == 1){
		return 1;
	}elseif($pos == 2){
		return 2;
	}else{
		return func($pos-1) + func($pos-2);
	}
}

echo func(10);
```



# 字符串
字符串通常来说其实是由字符组成集合。
```php
$str = 'string';
echo $str[0]; // s
```


## 字符串定义
字符串有多种定义方式：
1. 单引号
```php
$str = 'string';
// 单引号定义的字符串能使用的转义字符有限
// 单引号中的变量无法被解析
```
2. 双引号
```php
$str = "string";
// 双引号定义的字符串能使用任意转义字符
// 双引号的变量能被解析
```
3. hereDoc
```php
$str = <<<str
"string" is string
str;

// 本质上就是使用双引号定义的大串文本，只是使用另一种方式书写
```
4. nowDoc
```php
$str = <<<'str'
"string" is string
str;

// 本质上就是使用单引号定义的大串文本，只是使用另一种方式书写
```
## 字符串长度

### strlen()
获取字符串的 **字节数**。
```php
$str = 'string';
echo strlen($str); // 6
```

### 多字节字符
前面提到字节数，默认字母在任何字符住在占据的一个字节保存一个字符。
例如：汉字，一个字符可能占据多个字节
```php
$str = '你好, Json';
echo strlen($str); // 12
```

所以PHP提供了对多字节字符的支持，需要在`php.ini`中开启多字节字符的支持
```php
extension=php_mbstring.dll // 支持扩展

// 实例
$str = '你好, Json';
echo mb_strlen($str,'utf-8'); // 8
``` 

## 字符串相关函数

### 输出函数
```php
echo
print
print_r
var_dump
```

### 查找函数
```php
strstr(str, substr);
/*
查找str首次出现substr的位置，并截取到最后
*/
$str = 'string';
echo strstr($str, 'r'); // ring



strrchr(str, substr);
/*
查找str最后一次出现substr的位置，并截取到最后
*/
$str = 'stringstring';
echo strrchr($str, 'r'); // ring



strpos(str, substr);
/*
查找str首次出现substr的位置
*/
$str = 'stringstring';
echo strpos($str, 'r') // 2  

strrpos(str, substr);


strrpos(str, substr)
/*
查找str最后一次出现substr的位置
*/
$str = 'stringstring';
echo strrpos($str, 'r') // 8
```
### 分割
```php
explode(分隔符, str)
/*
根据指定的分隔符，将字符串str进行分割，并将每一部分组织成数组，返回
*/
$strArr = explode('|', 's|t|r|i|n|g');
print_r($strArr); //Array ( [0] => s [1] => t [2] => r [3] => i [4] => n [5] => g )

```

### 替换
```php
str_replace(replace, search, str);
/*
在str中查找search部分，并替换成replace部分，并返回替换后的内容
*/
$str = 'Hello, Json';
echo str_replace('Json', 'myj' ,$str); // Hello, myj
```

### 大小写转换
```php
strtolower(str)
/*
转换为小写
*/
$str = 'String';
echo strtolower($str); // string



strtoupper(str)
/*
转换为大写
*/
$str = 'String';
echo strtoupper($str); // STRING
```


### 去除指定字符
```php
trim(str [,substr])
/*
将字符串两侧的子字符串去除，substr可以省略，省略为去除空格
*/
$str = 'AstringA';
echo trim($str, 'A'); // string



ltrim(str [,substr])
rtrim(str [,substr])
/*
区别于上为左/右
*/
```

### pathinfo
```php
pathinfo(path [,option])
/*
获取一个文件的路径信息
option: 获取执行的路径信息
  PATHINFO_DIRNAME
  PATHINFO_BASENAME
*/
echo '<pre>';
print_r(pathinfo('D:\Datas\test.php'));

/*
Array
(
    [dirname] => D:\Datas
    [basename] => test.php
    [extension] => php
    [filename] => test
)
*/
```

### htmlspecialchars()
```php
htmlspecialchars(str)
htmlspecialchars_decode(str)
/*
用于将str中的特殊符号转换为HTML字符实体。
*/
echo htmlspecialchars('<>'); // &lt;&gt;gt;

```

<br/><br/>
# 数组
数组是一种数据的集合，数组主要是用于存储具有行列特征(表格)的数据。


<br/>
## 数组分类
1. 索引数组
数据的下标是整数
2. 关联数组
数据的下标是字符串


<br/>
## 数组的创建
1. 索引数组的创建
```php
// 显示创建
$arr = array(10, 20, 30, 40);
$arr2 = [10, 20, 30, 40];

// 隐式创建
$arr3 = array();
$arr3[0] = 10;
$arr3[1] = 20;
$arr3[6] = 20;

注意：
  PHP中的数据下标可以不规连续
```

2. 关联数组的创建
```php
// 显示创建
// $arr = array(键名=>键值, 键名=>键值, ...);
// $arr2 = [键名=>键值, 键名=>键值, ...]
$arr = array('key'=>'value');
$arr2 = ['key'=>'value'];

// 隐式创建
$arr = array();
$arr['key'] = 'value';
```

<br/>
## 多维数组
PHP中支持多维数组，如果一个数组的元素又是数组，那么就是多维数组。(最多支持60维)

### 二维数组
```php
// 显示创建
$arr = array(
	'userInfo' => array('id'=>1, 'username'=>'Json')
);
$arr2 = [
	'userInfo' => array('id'=>1, 'username'=>'Json')
];

// 隐式创建
$arr3 = array();
$arr3['userInfo'] = array('id'=>1, 'username'=>'Json');
```

<br/>
## 数组元素的访问
```php
// 一维数组    $数组名[下标/键名]
echo $arr['userInfo'];

// 二维数组    $数组名[行下标/列下标]
echo $arr['userInfo']['id'];
```

<br/>
## 数组的长度
```php
echo count($arr); // 行数
echo count($arr['userInfo']); // 具体行的列数
```

<br/>
## 数组的指针
表示当前所获得的点
```php
current(arr)   // 获取当前指针所指向元素的键值
key(arr)       // 获取当前指针所指向元素的键名
next(arr)      // 将指针下移
prev(arr)      // 指针上移
reset(arr)     // 指针重置
end(arr)       // 将指针移到最后
```

<br/>
## 数组的遍历

### for
```php
$arr = [10,20,30,40];
for($i=0;$i<=count($arr);$i++){
  echo $arr[$i];
}

// 但只能作用于这种有规律的方式
```

### foreach
```php
foreach($arr as [$key=>]$value){
  // 循环体
}
/*
原理：
对数组的指针进行重置。
读取当前指针所指向的数组元素，
并将元素的键名赋值给$key,
键值赋值给$value，
同时数组下移一行
直接读取到数组最后(数组的最后一行其实为NULL，当foreach读到BNULL会自动停止)
*/
$arr = array('id'=>'1', 'username'=>'admin', 'password'=>'admin123');
foreach($arr as $key=>$value){
  echo $key.'=>'.$value, '<br/>';
}
/*
id=>1
username=>admin
password=>admin123
*/
```

### while-each-list遍历
```php
echo();
/*
获取当前指针所指向的元素键名和键值，并且返回以索引+关联的数组，并且将指针下移一行。
如果是数组的最后一行，它会自动
*/
$arr = ['key'=>'value'];
print_r(each($arr));
/*
Array
(
    [1] => value
    [value] => value
    [0] => key
    [key] => key
)
*/



list();
/*
将数组中的索引元素赋值给变量列表中的变量。
*/
$arr = [10,20,30,40];
list($a, $b, $c, $d) = $arr;
```
```php
$arr = array('id'=>'1', 'username'=>'admin', 'password'=>'admin123');
while(list($k,$v) = each($arr)){
  echo $k.'=>'.$v,'<br/>';
}
```


### foreach-list
```php
// php>5.6
// 注意：list只能使用索引数组
$arr = [
  [1,2,3,4]
];

foreach($arr as list($a, $b, $c, $d)){
  echo $a,$b,$c,$d;
}

```

<br/>
## 数组的常用操作元素

### 数组的长度
```php
count($arr);
```

### 获取数组的键名/值
```php
array_keys(); // 返回所有键名
array_values(); // 返回所有键值
```


### 判断键名/值是否存在
```php
array_key_exists(key, arr); // 判断键名是否存在
in_array(value, arr); // 判断键值是否存在
```


### 数组合并
```php
array_merge($arr, $arr2);// 合并两个数组
```

### 数组排序
```php
sort();   // 按键值升序
rsort();  // 按键值降序
asort();  // 按键值进行升序，但原下标不变
arsort(); // 按键值进行降序，但原下标不变
```


### 数组解压(extract)
```php
$arr = ['id'=>'1','username'=>'admin']; 
extract($arr); // 将我们关联元素转换为键名为名的变量
echo $id;
echo $username;
```

<br/>
## 数组排序算法
### 冒泡排序
```php
// 注意内循环的条件与外循环的条件
// 外循环中，由于是交换两个数，只需要循环数组数-1即可
// 内循环中，由于每一个循环都会把最大值放到最右边，所以最右边的界限需要不断改变
$arr = [11,5,123,23,64,224,56,1,3,99];
$len = count($arr);
for($i=1;$i<$len ;$i++){
	for($j=0;$j<$len-$i;$j++){
		if($arr[$j]>$arr[$j+1]){
			$temp = $arr[$j];
			$arr[$j] = $arr[$j+1];
			$arr[$j+1] = $temp;
		}
	}
}
foreach($arr as $value){
	echo '['.$value.'],';
}
```

### 插入排序
```php
/*
算法步骤描述：
1. 从第一个元素开始，假设第一个元素已经被排列
2. 在取下一个元素，将其和已经排序的元素中从后往前比较
3. 如果以排列元素大于新元素，那么交换两者位置，
4. 重复步骤3，直到以排序元素小于或者等于新元素
5. 重复2-4步骤
*/

$arr = [11,12,9,23,64,224,56,1,3,99];


for($i=1;$i<count($arr);$i++){
	$tmp = $arr[$i];
	for($j=$i-1;$j>=0;$j--){
		if($arr[$j]>$arr[$j+1]){
			$arr[$j+1] = $arr[$j];
			$arr[$j] = $tmp;
		}
	}
}


print_r($arr);
```

<br/>
## 数组查找算法

### 数组排序查找法
```php
function searchArr($arr, $search){
	for($i=0;$i<=count($arr);$i++){
		if($arr[$i]==$search){
			echo $search;
			break;
		}
	}
}

$arr = [11,12,9,23,64,224,56,1,3,99];
searchArr($arr, '224');

```

### 二分查找法
```php
/*
前提：数组有序并不重复
可以简单的把数组想象成一个轴，对轴进行二分
*/
$arr = [1,2,3,4,5,6,12,15,17,18,32,55,73,74,89];


function searchArr($arr, $search){
	$left = 0;
	$right = count($arr);
	while($left <= $right){
		$mid = ceil(($left + $right) / 2);
		if($search > $arr[$mid]){
			$left = $mid + 1;
		}elseif($search < $arr[$mid]){
			$right = $mid - 1;
		}else{
			return 'yes';
		}
	}
	return 'no,not exists';
}
echo searchArr($arr, 15);
```
