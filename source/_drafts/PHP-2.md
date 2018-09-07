---
title: PHP备忘录(2)
date: 2018-07-13 20:10:36
categories: Notes
tags:
description:
---

PHP核心编程
<!-- more -->

<br/>
# 前台后台数据的传递

## 前台数据的提交 

### 表单
```php
/*
 * GET
点击提交按钮后，数据会作为URL后面的参数进行传递
login.php?username=1&password=2&submit=%E7%99%BB%E5%BD%95
*/
<form method="get" action="login.php">
	用户名：<input type="text" name="username" id="username" /><br />
	密码：  <input type="text" name="password" id="password" /><br />
	<input type="submit" name="submit" value="登录" />
</form>


/*
 * POST
 点击提交按钮，数据会放置在HTTP报文的请求体重进行传递
 username=2&password=2&submit=%E7%99%BB%E5%BD%95
*/
<form method="post" action="login.php">
	用户名：<input type="text" name="username" id="username" /><br />
	密码：  <input type="text" name="password" id="password" /><br />
	<input type="submit" name="submit" value="登录" />
</form>
```

### 模拟表单
我们可以直接在URL上修改参数，直接想后台提交数据


<br/>

## 后台数据的接受
```php
$_GET       // 接受前台使用GET方式提交的数据
$_POST      // 接受前台使用POST方式提交的数据
// $_GET/$_POST其实就是一个由'name'为键名,'value'为键值的关联数组

$_REQUEST   // 接受前台使用GET或者POST方式的数据
// 如果GET和POST键名重复，会取POST值
```

<br/>

## 特殊表单的提交
我们在提交表单的时候，可以给name参数值加上`[]`，对于name属性来说，它仅仅代表字符串， **但是在PHP接受该参数，会自动转换为数组。**
```php
// 复选框例子
<form method="get" action="login.php">
	用户名：<input type="text" name="username" id="username" /><br />
	密码：  <input type="text" name="password" id="password" /><br />
	<input type="checkbox" name="color[]" value="red" />红色<br />
	<input type="checkbox" name="color[]" value="blue" />蓝色<br />
	<input type="submit" name="submit" value="登录" />
</form>

/*
Array
(
    [username] => 
    [password] => 
    [color] => Array
        (
            [0] => red
            [1] => blue
        )
    [submit] => 登录
)
*/

```


## 课堂案例
```php
/*
网页版计算器
*/

<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
<?php 
$op = '';
$num = '';
$num2 = '';
$result = '';
if(isset($_POST['submit'])){
	$op = $_POST['op'];
	$num = $_POST['num'];
	$num2 = $_POST['num2'];
	$result = 0;
	switch($op){
		case 'plus':
		$result = $num + $num2;
		break;
		case 'minus':
		$result = $num - $num2;
		break;
		case 'times':
		$result = $num * $num2;
		break;
		case 'divide':
		$result = $num / $num2;
		break;
	}
}

 ?>
<h4>网页版计算器</h4>
<form action="cail.php" method="post">
<input type="input" name="num" value="<?php echo $num; ?>" />
<select name="op">
	<option value="plus" <?php echo $op=='plus'?'selected':''; ?>>+</option>
	<option value="minus" <?php echo $op=='minus'?'selected':''; ?>>-</option>
	<option value="times" <?php echo $op=='times'?'selected':''; ?>>*</option>
	<option value="divide" <?php echo $op=='divide'?'selected':''; ?>>/</option>
</select>
<input type="input" name="num2" value="<?php echo $num2; ?>" />
<input type="submit" name="submit" value="=" />

<?php echo $result; ?>




</body>
</html>
```

# 文件上传

## 前台部分
```php
/* 
form 表单设置注意！！！
1. action属性必须指向PHP文件
2. method属性必须设置POST
3. enctype属性
    application/x-www/encoded  上传普遍文本数据
    multipart/form-data        上传多种类型的数据(文件上传)
*/
<form action="fileupload.php" method="post" enctype="multipart/form-data">
	<input type="file" name="file" />
	<input type="submit" name="submit" value="submit" />
</form>

```

## 后台数据接受部分
上传文件的相关信息，被保存在PHP中`$_FILES`这个预定义变量中。
```php
print_r($_FILES); // 接受文件数据后
/*
Array
(
    [file] => Array   表单中name值
        (
            [name] => php.gif   文件名
            [type] => image/gif 文件MIME类型
            [tmp_name] => C:\Windows\php1A11.tmp   文件上传到服务器后存放的临时文件  
            [error] => 0    文件错误信息
            [size] => 28   文件大小(单位字节)
        )
)

临时文件注意：
    临时文件在PHP脚本执行结束后会被删除，所以我们需要使用 
    move_uploaded_file(tmp,dest) 函数去将临时文件移动我们制定的上传目录
*/
```

## 文件上传功能
### 文件上传简单实现
```php
// 上传文件基本信息
$file_name = $_FILES['file']['name'];
$file_mime = $_FILES['file']['type'];
$file_size = $_FILES['file']['size'];
$file_tmp = $_FILES['file']['tmp_name'];
$file_error = $_FILES['file']['error'];

// 判断文件错误
switch($file_error){
	case 1:
		exit('文件大小超出限制');
		break;
	case 2:
		exit('文件大小超出HTML限制');
		break;
	case 3:
		exit('文件上传不完整');
		break;
	case 4:
		exit('没有选择文件');
		break;
	case 6:
		exit('服务器内部错误');
		break;
	case 7:
		exit('服务器内部错误');
		break;
}

// 判断文件大小
$max_size = 1024*1024*3; // 3M
if($file_size > $max_size){
	exit('文件过大');
}

// 判断MIME类型
$mimes = ['image/jpeg', 'image/jpg', 'image/pjpeg', 'image/png', 'image/gif'];
if(!in_array($file_mime, $mimes)){
	exit('上传的文件类型不合法！');
}

// 移动文件
$file_ext = pathinfo($file_name, PATHINFO_EXTENSION);
$new_file_name = getRandName().'.'.$file_ext;
$dest = 'upload/'.$new_file_name;
move_uploaded_file($file_tmp, $dest);



/*
生成随机文件名   (当前时间+6位由数字大小写字母组成的随机字符)
mt_rand(m, n)  生成m与n之间的随机整数
chr(code)      将ASICC码转换为对应字符
*/
function getRandName(){
	$str = date('YmdHis');
	for ($i=0; $i < 6; $i++) { 
		switch(mt_rand(0, 2)){
			case 0:
			$str .= chr(mt_rand(97, 122));
			break;
			case 1:
			$str .= chr(mt_rand(65, 90));
			break;
			case 2:
			$str .= mt_rand(0, 9);
			break;
		}
	}
	return $str;
}
```
### 文件上传功能封装
```php
$file = $_FILES['file'];
$mimes = ['image/jpeg', 'image/jpg', 'image/pjpeg', 'image/png', 'image/gif'];
$dir = 'upload/';
$max_size = 1024*1024*3; // 3M
echo upload($file, $dir, $max_size, $mimes);
/*
文件的参数
文件上传目录
文件大小
需要判断的MIME类型
*/
$errorInfo = [
	'1001'=>'文件大小超出限制',
	'1002'=>'文件大小超出HTML限制',
	'1003'=>'文件上传不完整',
	'1004'=>'没有选择文件',
	'1006'=>'服务器内部错误',
	'1007'=>'服务器内部错误',
	'1010'=>'上传文件超过限制',
	'1020'=>'上传文件MIME类型不符合要求',
];
function upload($file, $dir, $max_size, $mimes){
	// 上传文件基本信息
	$file_name = $file['name'];
	$file_mime = $file['type'];
	$file_size = $file['size'];
	$file_tmp = $file['tmp_name'];
	$file_error = $file['error'];

	// 判断文件错误
	switch($file_error){
		case 1:
			return 1001;
			break;
		case 2:
			return 1002;
			break;
		case 3:
			return 1003;
			break;
		case 4:
			return 1004;
			break;
		case 6:
			return 1006;
			break;
		case 7:
			return 1007;
			break;
	}

	// 判断文件大小
	if($file_size > $max_size){
		return 1010;
	}

	// 判断MIME类型
	if(!in_array($file_mime, $mimes)){
		return 1020;
	}

	// 移动文件
	$file_ext = pathinfo($file_name, PATHINFO_EXTENSION);
	$new_file_name = getRandName().'.'.$file_ext;
	$dest = $dir.$new_file_name;
	if(move_uploaded_file($file_tmp, $dest)){
		// 上传成功函数会返回1
		return $new_file_name;
	}
}


/*
生成随机文件名   (当前时间+6位由数字大小写字母组成的随机字符)
mt_rand(m, n)  生成m与n之间的随机整数
chr(code)      将ASICC码转换为对应字符
*/
function getRandName(){
	$str = date('YmdHis');
	for ($i=0; $i < 6; $i++) { 
		switch(mt_rand(0, 2)){
			case 0:
			$str .= chr(mt_rand(97, 122));
			break;
			case 1:
			$str .= chr(mt_rand(65, 90));
			break;
			case 2:
			$str .= mt_rand(0, 9);
			break;
		}
	}
	return $str;
}
```


## php.ini 文件上传设置
```php
file_uploads = On        // 允许文件上传
upload_tmp_dir = d:/     // 设置文件上传临时目录
max_file_uploads = 20    // 一次性上传多少个文件
upload_max_filesize = 2M // 文件上传大小
post_max_size = 2M       // POST数据最大字节长度
max_input_time = -1      // 表单提交的最大时间(秒)  -1为无限制
max_execution_time = 0   // 脚本最大允许执行时间(秒) 0为无限制
memory_limit = 128M      // 一个PHP文件所占用
```


# PHP操作MYSQL数据库(mysqli)

## 基本函数
```php
// 面向过程写法
/*
1. 连接数据库
如果连接成功会返回M连接对象，如果失败返回的是是false。
*/
$mysqli = mysqli_connect('127.0.0.1','root','root'); 
/*
2. 设置客户端字符集
mysqli_query(link, query) 
    link 为mysqli_connect()返回的连接字符串
    query 是sql语句
*/
$charset_sql = 'set names utf8';
mysqli_query($mysqli, $charset_sql);
/*
3. 选择数据库
*/
$database_sql = 'use news';
mysqli_query($mysqli, $database_sql);

/*
4. 开始执行业务的SQL语句
*/

/*
如果查询的SELECT语句,查询成功会返回一个mysqli的资源结果集，我们不能直接操作资源，
但是可以用PHP提供给我们从资源结果集中获取数据的函数
mysqli_fetch_row(mysqli_result);       从结果集中读取一条数据，返回索引数组，并且数组指针下移
mysqli_fetch_assoc(mysqli_result);     从结果集中读取一条数据，返回关联数组，并且数组指针下移
mysqli_fetch_array(mysqli_result);     从结果集中读取一条数据，返回索引+关联数组，并且数组指针下移
mysqli_result 是mysqli_query 执行SELECT语句之后返回的资源结果集。
*/
$sql = "SELECT * FROM news order by sort desc";
$result = mysqli_query($mysqli, $sql);


/*
扩展： mysql错误信息
mysqli_error(mysqli_result)    获取当前执行的SQL错误信息,如果有错误返回错误描述字符串 
mysqli_errno(mysqli_result)    获取当前执行的SQL错误编码,例如1061,1064
*/
if(mysqli_errno($mysqli)){
	echo 'sql语句执行错误,错误信息如下:<br/>';
	echo mysqli_error($mysqli);
}

/*
扩展： 获取最新插入一条记录的ID
mysqli_insert_id(mysqli_result)
*/
```



# 文件操作
文件操作指利用PHP代码针对文件（文件夹）进行增删改查操作。
文件分为两种，一种是文件夹，一种是文件

## 目录操作(文件夹)

### 基本操作
```php
// 创建文件夹
mkdir("dir3"); // 当前路径

// 删除文件夹
rmdir("dir3");

// 读取目录
// 1.打开资源
$r = opendir("./");
var_dump($r); //resource(2) of type (stream)


// 2.阅读目录
echo readdir($r); // 从资源中读取指针所在位置的文件名字，然后指针下移，直到指针移出资源

while($file = readdir($r)){
	echo $file.'<br>';
}

// 3.关闭目录
closedir($r);

```

### 其他操作
```php
$dir = dirname(__FILE__); //得到的是路径的上一层路径
var_dump($dir); // string(15) "D:\PhpStudyCode"


$dir = realpath(__FILE__); // 得到真实路径（目录路径），如果是文件那么得到的结果是false
var_dump($dir);


$dir = is_dir(__FILE__); // 判断指定路径是否是一个目录
var_dump($dir);

$dir = scandir('./'); // 封装版的opendir\readdir\closedir，获取一个指定路径下的所有文件信息，以数组形式返回
var_dump($dir);
```

### 遍历指定目录案例
```php
currDir('./');


function currDir($path, $level=0){
	$r = opendir($path);
	while($file = readdir($r)){
		// 不对.和..做处理
		if($file!='.' || $file!='..'){
			echo str_repeat('&nbsp;&nbsp;&nbsp;&nbsp;',$level).$file.'<br>';
			// 构建完整路径
			$filePath = $path.'/'.$file;
			// 判断是否为目录
			if(is_dir($filePath)){
				// 递归遍历
				currDir($filePath, $level+1);
			}
		}

	}
	closedir($r);
}
```

```
currDir('./');


function currDir($path, $level=0){
	$files = scandir($path);
	foreach ($files as $file) {
		if($file=='.' || $file=='..'){
			continue;
		}
		echo str_repeat('&nbsp;&nbsp;&nbsp;&nbsp;',$level).$file.'<br>';
		// 构建完整路径
		$filePath = $path.'/'.$file;
		// 判断是否为目录
		if(is_dir($filePath)){
			// 递归遍历
			currDir($filePath, $level+1);
		}
	}
}
```

## 文件操作
### PHP5常见文件操作函数
```php
file_put_contents('./file/test.txt','test'); // 将指定内容写入到指定文件内：如果当前路径下不存在指定的文件，函数会自动创建（如果路径不存在，不会创建路径）
echo file_get_contents('./file/test.txt'); // 获取指定文件的所有内容，如果路径不存在最好做安全处理
```

### PHP4常见文件操作函数
```php
$file = fopen('./file/test.txt','a'); // 打开一个文件资源，限定打开模式
fwrite($file,'aaa'); // 向打开的资源中写入指定的内容
fclose($file); // 关闭资源
$file = fopen('./file/test.txt','r');
echo fread($file,10); // 从打开的资源中读取指定长度的内容（字节）
fclose($file); // 关闭资源
```

### 其他常用函数
```php
is_file(); // 判断文件是否正确（不识别路径）
filesize(); // 获取文件大小
file_exists(); // 判断文件是否存在（识别路径）
unLink(); // 取消文件名字与磁盘地址的连接（删除文件）
filemtime(); // 获取文件最后一次修改的时间
fseek(); // 设定fopen打开的文件的指针位置
fgetc(); // 一次获取一个字符
fgets(); // 一次获取一个字符串（默认行）
file(); // 读取整个文件，类似file_get_contents，区别是按行读取，返回一个数组
```

## 文件下载
从服务器将文件通过HTTP协议传输到浏览器，浏览器不解析保存成相应的文件。
提供下载方式可以使用HTML中的a标签：`<a href=”互联网绝对文件路径”>点击下载</a>`
1. 缺点1：a标签能够让浏览器自动下载的内容有限：浏览器是发现如果解析不了才会启用下载
2. 缺点2：a标签下载的文件存储路径会需要通过href属性写出来，这样会暴露服务器存储数据的位置（不安全）

**PHP中的下载**
读取文件内容，以文件流的形式传递给浏览器：在响应头中告知浏览器不要解析


```php
// 简易版本
// 1. 通过协议头告诉浏览器，将要发送给你的数据，而应该作为一个应用数据数据
header('Content-type:application/octet-stream;');
// 2.通过协议头告诉浏览器，将要发送给你的数据，作为附件下载
header('Content-Disposition:attachment;filename="test.jpg');
// 3.读取要发送的文件内容，并发送给客户端
echo file_get_contents('test.jpg');

// 如果是小文件，使用`php_get_contents`
// 如果是大文件，使用`fopen`
```





# CURL
CURL是一个非常强大的开源库，支持很多协议，包括HTTP、FTP、TELNET等，我们使用它来发送HTTP请求。

```php
// 1. 开启浏览器
$ci = curl_init();

// 2. 设置浏览器参数（输入URL）
curl_setopt($ci,CURLOPT_URL, 'http://byxs0x0.cn');

// 3. 请求（回车）
echo curl_exec($ci);
```


# 会话技术
由于HTTP协议是无连接无状态的，所以HTTP协议无法记住客户端的信息。为了弥补HTTP协议这两点的“不足”，所以出现了会话技术。

## Cookie
服务器将数据通过HTTP响应存储到浏览器上，浏览器可以在以后携带对应的COOKIE数据访问服务器。

### Cookie设置流程
1.	第一次请求时，PHP通过setcookie函数将数据通过http协议响应头传输给浏览器
2.	浏览器在第一次响应的时候将Cookie数据保存到浏览器
3.	浏览器后续请求同一个网站的时候，会自动检测是否存在Cookie数据，如果存在将在请求头中将数据携带到服务器
4.	PHP执行的时候会自动判断浏览器请求中是否携带Cookie，如果写到，自动保存到$_COOKIE中
5.	利用$_COOKIE访问Cookie数据

### Cookie操作
```php
// 设置Cookie
setcookie('username','admin',time() + 7*24*60*60); // 7天后过期
// 注意：cookie的值只能是整数或者字符串
setcookie('users[username]','admin');
setcookie('users[password]','admin');
// 读取的时候$_COOKIE会以数组的形式显示users

// 读取Cookie
var_dump($_COOKIE);
```

### Cookie的生命周期
COOKIE在浏览器生存时间（浏览器在下次访问服务器的时候是否携带对应的COOKIE）
1. 默认（不设定）时的生命周期：不设定周期默认是关闭浏览器（会话结束）
2. 设定为一个常规日期戳的周期：通过setcookie第三个参数可以限定生命周期，是用时间戳来管理，从格林威治时间开始
3. 设定为“0”的周期：在第三个参数设定生命周期的时候，用0代替时间戳：表示就是普通设置，会话结束过期`setcookie('username','admin',0)`
4. 删除一个cookie的做法：服务器没有权限去操作浏览器上的内容（不可能删除）。
 - 可以通过设定生命周期来让浏览器自动判定COOKIE是否有效：无效就清除`setcookie('username','admin',time())`
 - 设置cookie为空,`setcookie('username','')` 


### Cookie的作用范围（路径）
设置的Cookie后访问该网站下任何文件路径都会带上Cookie吗？

如果`sercookie('username','admin',time()+30,'/')`，那么Cookie的作用范围是网站根目录。
不设定，则默认是设置Cookie的PHP文件路径。

### Cookie的域
在同一级别域名下，myitcast.com（一级域名），可以有多个子域名（www.myitcast.com和gz.myitcast.com），他们之间是搭建在不同的服务器上（不同文件夹：E:/server/apache/htdocs和E:/web），但是可以通过COOKIE设置实现对应的COOKIE共享访问。但是默认是不允许跨域名访问的。

可以通过`setcookie(名字,值,生命周期,作用范围,有效域名)`来设置。
不设定时的默认有效域名

跨子域的设定方法：在设定域名访问的时候用设定上级域名即可：myitcast.com，这个是有所有以myitcast.com结尾的网站都可以共享COOKIE
`setcookie('username','admin',time()+30,'/','myitcast.com')`

### secure
`setcookie('username','admin',time()+30,'/','myitcast.com','true')`
如果secure参数被设为 true，那么当客户端使用了HTTPS时，才会将cookie带给服务器端。


### httponly
`setcookie('username','admin',time()+30,'/','myitcast.com','true','true')`
当httponly参数设置为true时候，那么cookie只能被http协议访问，不能被脚本语言，比如js去读取。

## Session
Session与浏览器无关，但是与Cookie有关。Session是以Cookie为基础，将重要的数据保存在服务器端，同时将能够唯一表示这份数据的数据以cookie的形式保存在客户端。
会话技术的本质是为了实现跨脚本共享数据：在一个脚本中定义数据，在另外一个脚本中保存数据

### Session流程
1.	PHP碰到session_start()时开启session会话，会自动检测sessionID
 - 如果Cookie中存在，使用现成的
 - 如果Cookie中不存在，创建一个sessionID，并通过响应头以Cookie形式保存到浏览器上
2.	初始化超全局变量$_SESSION为一个空数组
3.	PHP通过sessionID去指定位置（session文件存储位置）匹配对应的文件
 - 不存在该文件：创建一个sessionID命名文件
 - 存在该文件：读取文件内容（反序列化），将数据存储到$_SESSION中
4.	脚本执行结束，将$_SESSION中保存的所有数据序列化存储到sessionID对应的文件中

#### 客户端第一次访问流程
1. 创建SessionId
2. 初始化超全局变量$_SESSION
3. 在php指定目录创建以SESSIONId为名的文件
4. 脚本执行结束，将$_SESSION中保存的所有数据序列化存储到SessionID对应的文件中。并将SESSIONID以SETCOOKIE的形式返回给客户端

#### 客户端第二次访问流程
1. 检测到客户端的COOKIE中的SESSIONID
2. 初始化超全局变量$_SESSION
3. 找到文件，反序列化读取数据存储到$_SESSION中。
4. 脚本执行结束，将$_SESSION中保存的所有数据序列化存储到SESSIONID对应的文件中。

### 操作Session
```php
// 开启SESSIONID检测,如果客户端Cookie存在Sessionid，直接使用SessionID
// 如果不存在，创建新的SessionID，并set Cookie该SessionID
session_start();

// 设置Session值，可以是数组
$_SESSION['abc'] = 123;

// 读取Session值，其实就是将硬盘的Session文件中的数据读取到内存中。
print_r($_SESSION['abc']);

// 删除Session
unset($_SESSION['abc']); // 删除指定
$_SESSION = []; // 清空

// 销毁Session
Session删除是指删除session数据，$_SESSION中看不到而已；销毁session是指删除session对应的session文件。
session_destroy(); // 自动根据session_start得到的sessionID去找到指定的session文件，并把其删除。
```

### Session中的配置
在php.ini中关于SESSION的配置
```php
// 基本配置
session.name：session名字，保存到COOKIE中sessionID对应的名字
session.auto_start：是否自动开启session（无需手动session_start()），默认是关闭的
session.save_handler：session数据的保存方式，默认是文件形式
session.save_path：session文件默认存储的位置

// 常用配置
session.cookie_lifetime：PHPsessionID在浏览器端对应COOKIE的生命周期，默认是会话结束
session.cookie_path：sessionID在浏览器存储之后允许服务器访问的路径（COOKIE有作用范围）
session.cookie_domain：COOKIE允许访问的子域（COOKIE可以跨子域）
```

### Session分目录存储
```php
// 表示在e:/tmp目录下需要创建a-z,0-9为文件夹名的文件夹36个
session.save_path = "1;e:/tmp" 
```

### Session的垃圾回收机制
session会话技术后，session文件并不会自动清除，如果每天有大量session文件产生但是又都是失效的，会增加服务器的压力和影响session效率。
垃圾回收，是指session机制提供了一种解决垃圾session文件的方式：给session文件指定周期，通过session文件最后更改时间与生命周期进行结合判定，如果已经过期则删除对应的session文件，如果没有过期则保留。这样就可以及时清理无效的僵尸文件，从而提升空间利用率和session工作效率。

1.	任何一次session开启（session_start），session都会尝试去读取session文件
2.	读取session文件后，有可能触发垃圾回收机制（在session系统中也是一个函数：自己有一定几率调用）
3.	垃圾回收机制会自动读取所有session文件的最后编辑时间，然后加上生命周期（配置文件）与当前时间进行比较（所有session文件）
 - 过期：删除
 - 有效：保留


#### 垃圾回收参数设置
```php
session.gc_maxlifetime = 1440 // 规定的session文件最大的生命周期是1440秒，24分钟
session.gc_probability = 1 // 垃圾回收概率因子（分子）
session.gc_divisor = 1000 // 垃圾回收概率分母
```


### 禁用COOKIE后使用SESSION
```php
// 不使用Cookie
session.use_only_cookies=0
// 使用transid保存sessionid
session.use_trans_sid=1
```
完成以上配置后，只要在脚本中开启`session_start()`，就会给该页面中所有URL增加一个参数`url?PHPSESSION=sdf98a78f78a78adas`


# GD库
....

# 面向对象

## 面向过程与面向对象
面向过程侧重于 **过程(步骤)**为中心的编程思想。
面向对象侧重于 **事物(对象)**为中心的编程思想。
前者侧重 **做什么**，后者侧重 **谁来做**

## 类和对象
类就是分类、类别、概念、理论、思想，无形的、看不见、摸不着、不存在的。
可以理解为：“汽车模型”、“图纸”。
类是由相同属性和方法构成。


对象就是一个一个的实体。有形的、看得见、摸得着、存在的。
对象也是属性(特征)和方法(行为)构成的。

在现实中，先有对象，后有类。
在计算机中，先有类，后有对象。


## 类语法
```php
// 声明类的语法格式
class ClassName{
	attr1;
	attr2;
	method1;
	method2;
}

1. class是声明类的关键字，不区分大小写;
2. 类名、函数名、关键字，不区分大小写;
3. ClassName是类的名称，类名的命名规则与变量一样，但不带$符号;
4. 类名可以由数字、字母、下划线构成;
5. 类名不能以数字开头，但可以以字母或下划线开头;
6. 类名尽量使用驼峰式命名
7. 类名后不跟小括号
8. 大括号中{}定义的是类的成员属性和成员方法
```

### 成员属性
成员属性和普通变量的区别：
 - 成员属性一定要有前提，就是“谁的属性”，普通变量一般都是全局变量。
 - 成员属性一定要加权限控制符，而普通变量不需要。
```php
权限控制符 变量名 = 变量值;
public $name = 'tony';
public $age;

// 访问修饰符  -》  作用：主要用来保护数据的安全。
public(公共权限)：在任何地方都可以访问，主要指类内部、类外部、子类中都可以访问。
protected(受保护的权限)：只能在本类中、子类中被访问，在类外不能访问。
private(私有的权限)：只能在本类中被访问，在类外、子类中都无权访问。
```



### 成员方法
成员方法与普通函数的区别：
 - 成员方法，一定是哪个对象的方法，不能单独存在。
 - 成员方法要加权限控制符，普通函数不需要加；
 - 成员方法可以省略权限控制符，默认为public，建议不要省略。

```php
class Dog{
	public function eat($thing){
		return "dog eating $thing";
	}
}
```


### 类的实例对象
 - 类可以产生N多个对象；
 - 类几乎不占内存，但每个对象都要占用内存空间。
 - 平常只有对象才可以帮我们做工作，不是类。
 - 在JS中，创建类的对象的方法。例如：var obj = new Date()
 - 在PHP中，创建类的对象的方法。例如：$obj = new Student()
 - 使用new关键字来创建类的对象。

```php
$dog1 = new Dog;          // 无参数无括号
$dog2 = new Dog();        // 无参数有括号
$dog3 = new Dog('123');   // 有参数有括号
```

### 对象的属性和方法
在PHP中，访问对象的属性和方法，使用 `->` 来访问
```php
$obj->name;
$obj->eat();
```

#### 属性基本操作
```php
class Dog{
	public $name;
}

// 实例对象
$tony = new Dog;
// 修改属性
$tony->name = 'tony';
// 增加属性
$tony->age = '10';
// 删除属性
unset($tony->age);
// 读取属性
echo "$tony->name dog is tony";
```

#### 方法基本操作
```php
class Dog{
	public $name;
	public function eat($thing){
		return "i am eating $thing";
	}
}


// 实例对象
$tony = new Dog;
// 修改属性
$tony->name = 'tony';
echo $tony->eat('apple');
```

#### 伪变量$this
 - PHP中$this变量代表当前对象。
 - $this代表当前对象，用来调用对象的属性和方法。
 - $this只能在成员方法中存在，其它地方都不能使用。
 - $this对象是怎么来的？当使用$obj对象调用成员方法时(`$obj->eat()`)，自动将当前对象$obj传递到成员方法中(`$obj == $this`)，在成员方法中，使用$this变量来代替传递过来的$obj变量

```php
class Dog{
	public $name;
	public function eat($thing){
		return "$this->name eating $thing";
	}
}
```

### 类的常量
 - 常量：值就是值永远不变的量，常量不能修改，常量也不能删除。
 - 提示：在一次HTTP请求过程中，常量不能修改。
 - 类常量定义使用const关键字。define()定义的常量为全局常量。
 - 类常量，就是类的常量，与对象无关。
 - 类常量，只能通过类名来调用(类名::常量)；成员的东西，只能通过对象来调用。
 - 访问类常量，是通过范围解析符(::)来访问类的常量。例如：Student::TITLE
 - 访问对象的内容，是通过箭头(->)来访问的。例如：$obj->name、$obj->show()
 - 类常量在内存中只有一份，不会随着对象的增加而增加。类常量可以被所有对象共享。
 - 好处：节省内存。例如：班级名称(全栈3期)、ICP备案号、公司名称等。

```php
const 常量名 = 常量值

// 语法说明
常量没有权限访问符;
const定义的常量，一般认为是局部常量;
常量名不加$符号，尽量全大写;
常量的值，必须是一个固定的值;
```

```php
class Dog{
	const DOG_HEAD = 1;
	public function get_dogHead_num(){
		return "dog head is".Dog::DOG_HEAD;
	}
}

$dog = new Dog;
echo $dog->get_dogHead_num();
```


### 静态属性和静态方法
 - static关键字修饰的属性，就是静态属性；
 - static关键字修饰的方法，就是静态方法；
 - 静态属性，就是类的属性，与类相关，与对象无关；
 - 静态方法，就是类的方法，与类相关，与对象无关；
 - 静态属性和静态方法，是通过“类名::静态属性或静态方法”方式来访问的。
 - 静态属性和静态方法，在内存中只有一份，不会随着对象的增加而增加。
 - 好处：节省内存。可以被所有对象去共享。
 - 静态属性的值是可以改变的，可以被所有对象共享。
 - 静态属性和静态方法，是有权限限制的。

 ```php
class Dog{
	static public $kind = 'dog'; // 两者顺序可以调换
	public $name;
	private static function showLine(){
		return '<hr/>';
	}
	public function say(){
		$str = 'i am '.Dog::$kind;
		$str .= Dog::showLine();
		$str .= 'wang!wang!wang!';
		return $str;
	}
}

$tony = new Dog;
echo $tony->say();
 ```

#### 区分常量和静态属性
两者都可被所有对象共享，但是类常量永远不变。


### seif关键字
 - $this代表当前对象，self代表当前类；
 - $this用来调用对象的东西：成员属性、成员方法；
 - self用来调用类的东西：类常量、静态属性、静态方法；
 - $this使用箭头(->)来调用成员属性、成员方法；
 - self使用(::)来调用类常量、静态属性、静态方法；
 - $this只能用在成员方法中；self可以用在成员方法、静态方法中；


```php
class Dog{
	static public $kind = 'dog';
	public $name;
	private static function showLine(){
		return '<hr/>';
	}
	public function say(){
		$str = 'i am '.self::$kind;
		$str .= self::showLine();
		$str .= 'wang!wang!wang!';
		return $str;
	}
}

$tony = new Dog;
echo $tony->say();
```

### 构造方法
 - 当使用new关键字，创建一个类的对象时，第1个自动调用的方法，就是构造方法。
 - 构造方法的名称是固定的：__construct()
 - 构造方法可以有参数，也可以没有参数；
 - 当new一个类时，类名后跟的小括号的参数，就是传给构造方法的。例如：new Student(‘张三’,34)
 - 构造方法的作用：对象初始化。例如：给私有属性赋值、数据库对象初始化(连通、选择数据库)
 - 提示：构造方法只能定义一个；构造方法可有可无。
 - 构造方法必须是成员方法。
 - 构造方法一定没有返回值，不要使用return语句。

```php
class Dog{
	public $name;
	public function __construct($name){
		$this->name = $name;
	}
	public function say(){
		$str = 'i am '.$this->name;
		$str .= '   wang!wang!wang!';
		return $str;
	}
}

$tony = new Dog('tony');
echo $tony->say();
```


### 析构方法
 - 当销毁一个对象前，自动调用的方法，就是析构方法；
 - 析构方法的名称是固定的：__destruct()
 - 析构方法一定没有参数，析构方法一定是成员中方法；
 - 析构方法的作用：垃圾回收。例如：可以断开数据库的连接、在线人数等。

```php
class Dog{
	public function __destruct(){
		echo 'i am die';
	}
}

$tony = new Dog;
echo 2333; //2333i am die
```

**对象会在什么时候销毁?**
1. 网页执行完毕，所有变量自动销毁，包含对象变量；
2. 手动销毁变量，使用unset()函数



### 统计在线人数实例
```php
class Student{
	static private $count = 0;
	public function __construct(){
		self::$count++;
	}
	public function __destruct(){
		self::$count--;
	}
	static public function studentNum(){
		return self::$count;
	}
}

$student = new Student;
$student2 = new Student;
$student3 = new Student;
$student4 = new Student;
$student5 = new Student;

echo Student::studentNum();
```




## OOP中内存分配情况
![OOP 中内存的分配情况](2018-08-13-21-42-09.png)
![OOP 中内存的分配情况2](2018-08-13-21-50-06.png)

### 值传递
**什么是值传递**
将一个变量的“值”，复制一份，传递给另一个变量；两个变量之间没有任何关系；修改其中一个变量的值，另一个变量不会改变。
字符串型、整型、浮点型、布尔型、数组，默认都是“值传递”。

**值传递在内存中如何表现**
标量数据类型的变量，在内存中如何存储？将变量名和变量的值，都存在“栈内存”中。
“栈内存”速度比较快，但不能存储太多内容。


### 引用传递
“引用传地址”将一个变量的“数据地址”，复制一份，传递给另一个变量；两个变量指向了“同一数据”；修改其中一个变量的数据，另一个变量也会一起变。

PHP中默认的“引用传地址”的数据类型是：对象和资源。


![引用传递](2018-08-14-10-45-36.png)


## 类的三大特性
类的三大特性：封装性、继承性、多态性、抽象性。

### 类的封装性
 - 类的封装性：将敏感数据保护起来，不被外界访问。
 - 类的封装性再次理解：将一个功能的方方面面，封装成一个类。例如：数据库工具类，把数据库操作的所有方面全面封装到类中，因此，在该类外，不能再使用”mysql_*”开头的函数。
 - 类的封装性实现，就是通过权限控制符来实现。
 - 在项目中，所有成员属性，一般都是private、protected权限。

#### 访问修饰符
 - public(公共权限)：在任何地方都可以被访问，主要是：类内、类外、子类中。
 - protected(受保护的权限)：只能在本类、子类中被访问。在类外禁止访问。
 - private(私有权限)：只能在本类中被访问。
 - 成员属性、静态属性必须要加权限控制符，不能省略。
 - 成员方法、静态方法可以不加权限控制符，默认为public。建议都要加权限。


### 类的继承性
 - CSS继承：将上层标签定义的样式，继承到子标签来使用。多个标签如果具有相同的样式，只需要在父标签定义，再继承到子标签来使用。相同的样式只需要定义一次。
 - 继承：如果一个B类拥有了A类的所有特征信息，则我们就认为B类继承了A类。
 - A类：父类、上层类、基础类(最顶层的类)
 - B类：子类、下层类、派生类
 - 为什么继承？**继承是为了实现功能的升级和扩展。如果一个项目不需要升级和扩展，则不用继承。**
 - 功能的升级：原来有的功能，对现在的功能进行更加完善的处理；
 - 功能的扩展：原来没有的功能，增加一个新功能；
 - 如果项目需要升级和扩展功能，**不能直接修改原类**，需要创建一个子类，并继承父类。


```php
class SubClass extends ParentClass{
	// 子类的功能代码
}

语法说明:
1. SubClass代表要创建的子类的名称
2. extends是继承关键字，不区分大小写
3. ParentClass代表已经存在的父类或上层类
```

#### 单继承和多继承
 - 单继承：只能从一个父类来继承功能，例如：PHP、Java等主流的程序语言。
 - 多继承：可以同时从多个父类来继承功能，例如：C++

继承可以理解为：把父类 **引用传递** 给子类，而不是 **值传递**给子类。

简单实例：
```php
class Dog{
	private $name;
	private $age;
	public function __construct($name, $age){
		$this->name = $name;
		$this->age = $age;
	}
	public function showInfo(){
		return "i am {$this->name},I's age is {$this->age}";
	}
}

class Corgi extends Dog{

}

$tony = new Corgi('tony', '20');
echo $tony->showInfo();
```

#### 构造方法与析构方法的继承
**如果权限足够,就能继承。**
```php
class Dog{
	private $name;
	private $age;
	public function __construct($name, $age){
		$this->name = $name;
		$this->age = $age;
	}
	public function showInfo(){
		return "i am {$this->name},I's age is {$this->age}";
	}
}

class Corgi extends Dog{
	public function __construct($name, $age){
		$this->name = $name;
		$this->age = $age;
	}
	public function subShowInfo(){
		return "{$this->name}";
	}
}

$tony = new Corgi('tony', '20');
// echo $tony->showInfo();
echo $tony->subShowInfo();
var_dump($tony);


object(Corgi)#1 (4) {
  ["name":"Dog":private]=>
  NULL
  ["age":"Dog":private]=>
  NULL
  ["name"]=>
  string(4) "tony"
  ["age"]=>
  string(2) "20"
}
```


#### parent关键字
 - self代表当前类，parent代表父类；
 - self可以调用本类的内容：类常量、静态属性、静态方法、**成员方法**;
 - parent可以调用父类的内容：类常量、静态属性、静态方法、**成员方法**;



### 类的多态
 - 类的多态，就是的类的多种形态；
 - 类的多态，主要指方法重载和方法重写；
 - 函数重载：在一个脚本文件中，定义两个同名函数；PHP不支持。
 - 方法重载：在同一个类中，定义两个同名方法；PHP不支持。
 - 方法重写：父类有一个方法，在子类用同样的名称再定义一次。
 - 功能升级：父类有的功能，子类的功能比它更完善、更详尽。通过方法重写来实现。
 - 如果不需要升级，也不需要扩展，继承就没有意义。


#### 方法重写
 - 子类中重写的方法名称，要与父类方法名称一致；
 - 子类中重写的方法的参数个数，必须要与父类方法的参数个数一致；
 - 子类中重写的方法的类型，必须要与父类方法的类型一致；父类是成员的方法，子类必须是成员的方法；父类是静态方法，子类也必须是静态方法；
 - 子类中重写的方法的权限，不能低于父类方法的权限。
	 - 如果父类方法权限为public，则重写方法必须是public；
	 - 如果父类方法权限为protected，则重写方法必须是public、protected；
	 - 如果父类方法权限为private，则子类无法继承，无法继承。

下面例子中，重写了父类的方法，并且在父类的方法上做了扩展和升级。
```php
class Dog{
	private $name;
	private $age;
	public function showInfo(){
		return "i am {$this->name},I's age is {$this->age}";
	}
}

class Corgi extends Dog{
	public function subShowInfo($name, $age){
		$str = "wangwangwang!!!<hr>";
		$str.= parent::showInfo($name, $age);
		return $str;
	}
}

$tony = new Corgi;
echo $tony->subShowInfo('tony','20');
```


#### 构造方法重写
所有方法都可重写，但是，构造方法重写，没有参数个数要求，也就是参数个数可以不对等。
重写构造方法，可以引用父类的构造方法！！！
```php
class Dog{
	private $name;
	private $age;
	public function __construct($name, $age){
		$this->name = $name;
		$this->age = $age;
	}
	public function showInfo(){
		return "i am {$this->name},I's age is {$this->age}";
	}
}

class Corgi extends Dog{
	public function __construct($name, $age, $color){
		parent::__construct($name, $age);
		$this->color = $color;
	}
	public function subShowInfo(){
		$str = parent::showInfo();
		$str.= ",I's color is {$this->color}!!!";
		return $str;
	}
}

$tony = new Corgi('tony', '20', 'red');
echo $tony->subShowInfo();
```

#### 最终类和最终方法
 - final关键字修饰的类，就是最终类；
 - final关键字修饰的方法，就是最终方法；
 - 最终类：该类不能被继承，直接实例化。该类已经十分完善了，不需要升级和扩展。
 - 最终方法：该方法不能被重写，直接调用即可。该方法已经十分完善了，不需要升级了。
 - 例如：数据库操作类，也可定义最终类。
 - 结论：最终类和最终方法，不能同时加final关键字。


#### 抽象类和抽象方法
**抽象类**
 - Abstract关键字修饰的类，就是抽象类；
 - 抽象类：该类只能被继承，不能直接实例化。常用于“基础类”。
 - 抽象类中，也可能有其它元素：成员属性、成员方法、静态属性、静态方法、常量。


**抽象方法**
 - Abstract关键字修饰的方法，就是抽象方法；
 - 抽象方法：该方法没有方法体，抽象方法必须先继承，后重写。
 - 如果一个类中有一个抽象方法，该类必须声明为抽象类；
 - 抽象方法作用：方法的命名规范，是一种 **监督的机制**。
 - 抽象方法不能是静态方法，只能是成员方法。
 - 抽象方法可以有参数，也可以没有。


```php
abstract class Dog{
	private $name;
	private $age;
	abstract public function eat($thing);
}

class Corgi extends Dog{
	public function eat($thing){
		return "i am eating $thing";
	}
}

$tony = new Corgi();
echo $tony->eat('apple');
```


### 接口技术
PHP只支持单继承，只能从一个父类来继承功能；如果PHP同时想从多个父类来继承功能，怎么办？可以使用接口来实现；接口也是子类中方法的命名规范。

 - 接口就是特殊的抽象类；
 - interface关键字，用来声明一个接口。接口是一种特殊类。
 - implements关键字，创建一个子类，来实现接口。
 - 同类的东西，使用extends关键字；不同类的东西，使用implements关键字。
 - 例如：子类继承父类、接口继承接口、类实现接口。
 - 接口中只能存在两样东西：**类常量、抽象方法**。
 - 接口中的方法，默认都是抽象方法，因此，**不加abstract关键字**。
 - 接口中方法的权限，**必须是public**；
 - 接口中方法，可以是 **成员方法，也可以是静态方法**；
 - 接口中所有的抽象方法，在子类中必须要重写；
 - 接口中的常量不能重写，只能继承。
 - **PHP中的“重写”，不一定是方法重写，还可以是常量重写、静态属性重写、静态方法重写；**

**接口实例**
![接口实例](2018-08-14-16-13-41.png)

**接口实例2**
![接口实例](2018-08-14-16-33-38.png)
![接口实例](2018-08-14-16-33-52.png)
![接口实例](2018-08-14-16-33-58.png)

### 类的自动加载
程序员在做开发时，会把每一个功能，都定义成一个独立的类文件，类文件是以”.class.php”结尾。一个大的项目有50个功能，就需要定义50个类文件，在应用页面，就需要把相关的类文件require()、require_once()包含进当前页面。因此，在页面的开头就会有50个reuire_once()语句。如果这样的话：极大浪费内存空间，不需要的类也包含进来。

如何解决类文件加载的问题？**按需加载**。

#### 常规的自动加载类函数(__autoload())

##### 类文件的命名规则
 - 将每一个功能，单独定义成一个类文件；
 - 每一个类文件，尽量以”.class.php”结尾；例如：Student.class.php
 - 类文件的主名，要与类名一致；例如：class Student{}
 - 类名命名方式，尽量使用“驼峰式”命名，每个单词首字母大写；例如：class ConnMySQL
 - 方法命名方式，尽量使用“驼峰式”命名，第1个单词全小写，后面每个单词首字母大写；例如：getCode()
 - 属性命名方式，尽量使用“驼峰式”命令，与方法命名一致。


##### __autoload()
 - __autoload()是系统函数，不是方法，名称是固定的；
 - 我们需要定义该函数的内容；
 - 该函数有一个唯一的参数，就是类名参数；
 - 当使用一个不存在的类时，__autoload($className)会自动调用；
   - 当使用new关键字，创建一个不存在的类对象时，__autoload()自动调用；例如：$obj = new Student()
   - 当使用静态化方式调用不存在类的属性或方法时，__autoload()自动调用；例如：Student::getCode()
   - 当继承一个不存在的父类时，__autoload()自动调用；例如：class ItcastStudent extends Student{}
   - 当实现一个不存在的接口类时，__autoload()自动调用；例如：class Student implements Inter{}
 - 函数的内容包含两方面：
   - 构建类文件的真实路径；
   - 判断并包含类文件代码；

```php
// 自动加载
function __autoload($className){
	$filename = "./class/{$className}.class.php";
	if(file_exists($filename)) require_once($filename);
}

$db_config = array(
	'db_host' => '127.0.0.1',
	'db_user' => 'root',
	'db_pass' => '123123',
	'db_name' => 'news',
	'charset' => 'utf8'
);

$mysql = new Db($db_config);
var_dump($mysql);
```


#### 自定义类文件加载函数(spl_autoload_register)
 - __autoload()有点局限性，如果类文件位于不同的目录，类文件名命名方式也不尽相同。
 - 自定义类文件加载函数：spl_autoload_register()，主要应用在项目中，可以应对不同的情况。
 - spl_autoload_register()注册多个类加载函数，形成了一个类文件的队列，按照注册时的顺序，依次执行。哪个类文件存在，就包含哪个类文件。
 - 每个函数就是一种类文件的加载规则。

**实例1**
```php
/*
参数:函数，有两种情况
1. 字符串的函数名
2. 匿名函数
*/
spl_autoload_register('classDir');
spl_autoload_register('libDir');

function classDir($className){
	$filename = "./class/{$className}.class.php";
	if(file_exists($filename)) require_once($filename);
}

function libDir($className){
	$filename = "./lib/{$className}.class.php";
	if(file_exists($filename)) require_once($filename);
}

$db_config = array(
	'db_host' => '127.0.0.1',
	'db_user' => 'root',
	'db_pass' => '123123',
	'db_name' => 'news',
	'charset' => 'utf8'
);

$mysql = new Db($db_config);
var_dump($mysql);
```

**传参匿名函数**

```php
spl_autoload_register(function($className){
	$dir = array(
		"./class/{$className}.class.php",
		"./lib/{$className}.class.php"
	);
	foreach($dir as $filename){
		if(file_exists($filename)) require_once($filename);
	}
});

$db_config = array(
	'db_host' => '127.0.0.1',
	'db_user' => 'root',
	'db_pass' => '123123',
	'db_name' => 'news',
	'charset' => 'utf8'
);

$mysql = new Db($db_config);
var_dump($mysql);

```

### 对象克隆(__clone())

```php
class Dog{
	public $name;
	// 当对象被克隆，魔术方法__clone()会自动调用
	public function __clone(){
		$this->name = 'fuzhiren1';
	}
}
$tony = new Dog;
$tony->name = 'tony';
// clone 克隆新对象
$fuzhiren = clone $tony;
var_dump($tony);
var_dump($fuzhiren);
```

### 对象遍历
![对象遍历](2018-08-15-09-31-20.png)


### 魔术方法
#### __tostring()
将对象转成字符时，魔术方法__toString()会自动调用

```php
class Dog{
	public $name;
	public function __toString(){
		return "my name is {$this->name}";
	}
}
$tony = new Dog;
$tony->name = 'tony';
echo $tony;
```

#### __invoke()
当把对象当成函数调用时，魔术方法__invoke()会自动调用。

```php
class Dog{
	public $name;
	public function __invoke(){
		return "nonono,this object can's as a function";
	}
}
$tony = new Dog;
echo $tony();
```

### 面向对象设计模式
 - 设计模式，就是面向对象代码设计经验的总结。
 - 设计模式，可以实现代码重用、节省时间、对于后期维护十分方便。


#### 常用的设计模式
 - 单例模式：一个类只能创建一个对象，不管用什么办法，都无法创建第2个对象。例如：数据库对象。
 - 工厂模式：根据传递的不同类名，来创建不同类的对象的工厂。


#### 单例模式
三私一公
 - 一私：私有的静态的保存对象的属性；
 - 一私：私有的构造方法，阻止类外new对象；
 - 一私：私有的克隆方法，阻止类外clone对象；
 - 一公：公共的静态的创建对象的方法。

```php
class Dog{
	// 私有静态存储对象
	private static $obj;
	// 私有构造方法
	private function __construct(){}
	// 私有克隆方法
	private function __clone(){}
	// 共有的静态对象创建方法
	public static function getInstance(){
		// 判断当前对象是否存在
		if(!self::$obj instanceof self){
			self::$obj = new self;
		}
		return self::$obj;
	}
}
$tony = Dog::getInstance();
```

#### instanceof 关键字
 - 描述：判断一个对象是不是某个类产生的对象。
 - 语法：$obj instanceof ClassName
 - 返回：如果$obj是ClassName的对象，则返回TRUE，否则，返回FALSE。


#### 用户信息展示实例
```php
// Db.class.php

class Db{
	// 存储对象
	private static $obj = NULL;
	// 私有成员属性
	private $db_host;
	private $db_user;
	private $db_pass;
	private $db_name;
	private $charset;
	// 私有的构造方法
	private function __construct($db_config){
		$this->db_host = $db_config['db_host'];
		$this->db_user = $db_config['db_user'];
		$this->db_pass = $db_config['db_pass'];
		$this->db_name = $db_config['db_name'];
		$this->charset = $db_config['charset'];
		$this->connectDb();
		$this->selectDb();
		$this->setCharset();
	}

	// 私有的克隆方法
	private function __clone(){}
	
	// 共有的创建对象方法
	public static function getInstance($db_config){
		// 对象不存在
		if(!self::$obj instanceof self){
			self::$obj = new self($db_config);
		}
		// 存在直接返回对象
		return self::$obj;
	}

	// 连接数据库
	private function connectDb(){
		if(!@mysql_connect($this->db_host, $this->db_user, $this->db_pass)){
			die("mysql connect fail");
		}
	}

	// 选择数据库
	private function selectDb(){
		if(!@mysql_select_db($this->db_name)){
			die("mysql database select fail");
		}
	}

	// 设置语言
	private function setCharset(){
		$this->exec("set charset {$this->charset}");
	}

	// 执行insert、delete等sql语句，返回bool
	private function exec($sql){
		return mysql_query($sql);
	}

	// 执行select语句，返回结果集
	private function query($sql){
		return mysql_query($sql);
	}

	// 获得单行数据内容
	public function fetchOne($sql, $type=2){
		$result = $this->query($sql);
		$typeArr = array(MYSQL_NUM, MYSQL_ASSOC, MYSQL_BOTH);
		return mysql_fetch_array($result, $typeArr[$type]);
	}

	// 获得多行数据内容
	public function fetchAll($sql, $type=2){
		$result = $this->query($sql);
		$typeArr = array(MYSQL_NUM, MYSQL_ASSOC, MYSQL_BOTH);
		while($row = mysql_fetch_array($result, $typeArr[$type])){
			$arrs[] = $row;
		}
		return $arrs;
	}

	// 获取数据数量
	public function rowCount($sql){
		$result = $this->query($sql);
		return mysql_num_rows($result);
	}
}
```
```php
// conn.php
function __autoload($className){
	$filename = "./libs/{$className}.class.php";
	if(!file_exists($filename)) require_once($filename);
}
require_once("./libs/Db.class.php");

$db_config = array(
	'db_host' => '127.0.0.1',
	'db_user' => 'root',
	'db_pass' => '123123',
	'db_name' => 'news',
	'charset' => 'utf8'
);

$db = Db::getInstance($db_config);
```
```php
// list.php

<?php
	header('content-type:text/html;charset=utf-8');
	require_once('./conn.php');
	$sql = "SELECT * FROM users";

	
	// $str = $db->rowCount($sql);

	// $row = $db->fetchOne($sql);
	// $str = '';
	// $str .= "<tr><th>{$row['id']}</th>";	
	// $str .= "<th>{$row['username']}</th>";	
	// $str .= "<th>{$row['password']}</th></tr>";	

	$result = $db->fetchAll($sql);
	$str = '';
	foreach ($result as $row) {
		$str .= "<tr><th>{$row['id']}</th>";	
		$str .= "<th>{$row['username']}</th>";	
		$str .= "<th>{$row['password']}</th></tr>";	
	}
?>
<!DOCTYPE html>
<html>
<head>
	<title>用户展示</title>
</head>
<body>
	<table border=1>
		<tr>
			<th>id</th>
			<th>username</th>
			<th>password</th>
		</tr>
		<?php echo $str; ?>
	</table>
</body>
</html>
```

```php
// 表单获取方式
foreach($POST as $k => $v){
	$$k = $v;
}
```

### 工厂模式
 - 工厂模式：根据传递的不同类名，创建不同类的对象；
 - 工厂模式，就是生产不同类对象的工厂，避免使用new关键字。
 - 还可以理解为：改变了创建对象的方式。


#### 设计要求
 - 工厂模式，可以设计一个工厂类；
 - 工厂类中有一个私有的静态的属性，用来存储不同类的对象；
 - 工厂类中有一个公共的静态的创建对象的方法。
 - 静态方法的功能代码：判断当前类的对象是否存在，如果存在，直接返回；如果不存在，则创建一个对象，再返回。
 - 注意：该工厂类不创建对象。工厂类本身不是单例的。

**实例**
```php
final class Factory{
	//存储对象的类
	private static $arr = array();
	//创建对象
	public static function getInstance($className){
		// 判断对象是否不存在，使用iset，否则$arr[$className]不存在，会优先报错,isset避免了这个错误
		if(!isset(self::$arr[$className])){
			self::$arr[$className] = new $className;
		}
		return self::$arr[$className];
	}
}

class Dog{
	public $name = 'tony';
}

class Cat{
	public $name = 'cat';
}

$dog = Factory::getInstance('Dog');
$cat = Factory::getInstance('Cat');
var_dump($dog,$cat);
```


## 重载
 - PHP中的“重载”与其它绝大多数面向对象语言不同。传统的“重载”是用于提供多个同名的类方法，但各方法的参数类型和个数不同。 
 - PHP所提供的"重载"（overloading）是指动态地"创建"类属性和方法。我们是通过魔术方法来实现的。 
 - 当调用当前环境下未定义或不可见的类属性或方法时，重载方法会被调用。
 - 所有的重载方法都必须被声明为 public。 
 - 属性重载只能在对象中进行。在静态方式中，这些魔术方法将不会被调用。所以这些方法都不能被 声明为 static。 
 - 这些魔术方法的参数都不能通过引用传递。


### 属性重载
#### __set()
 - 描述：在给不可访问属性赋值时，__set() 会被调用。
 - 语法：public void __set ( string $name , mixed $value )



#### __get()
 - 描述：读取不可访问属性的值时，__get() 会被调用。
 - 语法：public mixed __get ( string $name )

#### __isset()
 - 描述：当对不可访问属性调用 isset() 或 empty() 时，__isset()会被调用。
 - 语法：public bool __isset ( string $name )


#### unset()
 - 描述：当对不可访问属性调用 unset() 时，__unset()会被调用。
 - 语法：public void __unset ( string $name )



### 方法重载
#### __call()
 - 描述：在对象中调用一个不可访问方法时，__call() 会被调用。
 - 语法：public mixed __call ( string $name , array $arguments )


#### __callStatic()
 - 描述：用静态方式中调用一个不可访问方法时，__callStatic() 会被调用。
 - 语法：public static mixed __callStatic ( string $name , array $arguments )



## 静态延时绑定
 - 自 PHP 5.3.0 起，PHP 增加了一个叫做后期静态绑定的功能，用于在继承范围内引用静态调用的类。"后期绑定"的意思是说，static::不再被解析为定义当前方法所在的类，而是在实际运行时计算的。也可以称之为"静态绑定"，因为它可以用于（但不限于）静态方法的调用。 
 - 我们需要一个在调用执行时才确定当前类的一个特征，就是说将static关键字对某个类的绑定推迟到调用执行时，就叫静态延迟绑定！
 - 语法：static::静态属性，静态方法，成员方法，类常量

static在哪个类，它就代表哪个类，

![静态延时绑定](2018-08-15-16-09-40.png)




## 类型约束
 - Java属于强类型语言，变量在程序运行过程中，类型不可以改变。
 - PHP和JS属于弱类型语言，变量在程序运行过程中，类型是可以改变的。
 - PHP5.3以后，还有了类型约束。PHP的类型约束，主要指方法参数或函数参数，其它地方还不行。
 - PHP的类型约束有三种：数组约束、对象约束、接口约束。

![类型约束](2018-08-15-16-13-40.png)




## 魔术常量的应用
 - __LINE__，当前行号
 - __FILE__，当前文件名
 - __DIR__，当前目录名
 - __FUNCTION__，当前函数名
 - __CLASS__，当前类名
 - __METHOD__，当前方法
 - __NAMESPACE__，当前空间名(在PDO中讲)


## 序列化
- 变量序列化：将变量转成可存储或传输的字符串的过程，会保留变量的类型和结构。
- 变量反序化：将序列化的字符串，再还原成原始变量。
- **除了资源变量外，其它变量都可以序列化。**

### serialize()
 - 描述：产生一个可存储的值的表示
 - 语法：string serialize ( mixed $value )

### unserialize()
 - 描述：从已存储的表示中创建 PHP 的值
 - 语法：mixed unserialize ( string $str )


### 对象序列化
 - 对象的序列化过程，与其它变量数据一样；
 - 当序列化对象时，PHP 将试图在序列化动作之前调用该对象的成员函数 __sleep()。这样就允许对象在被序列化之前做任何清除操作。
 - __sleep()魔术方法功能：此功能可以用于清理对象，并返回一个包含对象中所有应被序列化的变量名称的数组。
 - 对象序列化的内容只能包含成员属性，不能包含常量、静态属性、成员方法、静态方法。

![对象序列化](2018-08-15-16-22-28.png)


### 对象反序列化
 - 对象的反序列化过程，与其它变量数据一样；
 - 若反序列化的变量是一个对象，在成功地重新构造对象之后，PHP 会自动地试图去调用 __wakeup()成员函数（如果存在的话）。
 - unserialize() 会检查是否存在一个 __wakeup()方法。如果存在，则会先调用 __wakeup 方法，预先准备对象需要的资源。 
 - __wakeup() 经常用在反序列化操作中，进行一些初始化操作，例如重新建立数据库连接，或执行其它初始化操作。



![对象反序列化](2018-08-15-16-22-43.png)




## 常用的类和对象的操作函数

### 判断类、接口、方法、属性是否存在
 - class_exists()判断类是否存在
 - interface_exists()判断接口是否存在
 - method_exists()判断方法是否存在
 - property_exists()判断属性是否存在


### 获取类名
 - get_class()根据对象返回类名
 - get_parent_class()返回父类名称


检查一个变量是不是对象is_object()



## 命名空间
 - 命名空间是一种封装事物的方法；例如：函数、类、方法等。
 - 命名空间作用：用来解决类名或应用程序名冲突问题；
 - 举例：项目中会用到第三方类，第三方类加载进来后，可能与项目中的类名冲突。


### 命令空间的要求
 - 使用namespace关键字，来声明一个命名空间；
 - 所有代码都可以存在于命名空间中，但是，只有三种代码会受到空间影响：类、常量(const)、函数。
 - 除了类、常量、函数代码外，其它代码可以写在空间中，但不受空间影响。其它代码相当于”全局代码”。全局代码可以在”任何地方”都能直接使用。
 - 全局代码所在的空间，称为”根空间”、”全局空间”，相当于windows系统的”桌面”。
 - 声明命名空间的语法，是PHP脚本的第1行代码；空格、空行都不可以；
 - 命名空间是虚拟的空间。

![命名空间的语法](2018-08-15-16-29-14.png)


### 命名空间实例
![命名空间使用实例](2018-08-15-16-30-08.png)


### 子命名空间
文件夹可以有子目录的情况，命名空间也有子空间情况。
目录的分割符号正斜杠(/)，空间路径的分割符是反斜杠(\)。
举例：App\Home\Controller。


![子命名空间使用](2018-08-15-16-37-12.png)



### 同一文件下定义多命名空间

#### 简单组合方法
![简单组合方法](2018-08-15-16-40-13.png)


#### 大括号法
![大括号法](2018-08-15-16-40-37.png)


#### 不包含在命名空间中的代码
如果一个文件用大括号语法定义多个命名空间，如果要写全局代码，还不想把全局代码加到某个命名空间中，该怎么写？
![不包含在命名空间中的代码](2018-08-15-16-41-57.png)


#### 访问命名空间中元素的方式
 - 非限定名称(不含前缀)。如果访问 $obj = new Student()，它的完整路径是：$obj = new curSpace\Student()
 - 限定名称(含有相对前缀)。如果访问 $obj=new Home\Student()，
   - 它的完整路径是：$obj = new curSpace\Home\Student()
 - 完全限定名称(含有绝对前缀)。
   - 如果访问 $obj = new \App\Home\Controller\Student()
   - 它的完整访问路径：$obj = new \App\Home\Controller\Student()



![访问命名空间中元素的方式](2018-08-15-16-50-50.png)



### namespace关键字和魔术常量__NAMESPACE__

#### namespace关键字
 - namespace含义之一：声明空间关键字；
 - namespace含义之二：可以用来直接代码当前空间名，相当于self关键字。

![namespace关键字](2018-08-15-16-53-47.png)

#### 魔术常量__NAMESPACE__
获取当前命名空间的字符串名称。

![魔术常量__NAMESPACE__](2018-08-15-16-54-10.png)




### 命名空间的别名/导入
 - 首先导入空间中的类，常量和函数不能导入。
 - 使用use关键字来导入空间中的类。例如：use App\Home\Controller\Student
 - 使用use关键字来导入空间名。例如：use App\Home\Controller;
 - 使用as关键字，可以给空间或类起个别名。
   - 给空间起别名：use App\Home\Controller as Controller
   - 给空间中的类起别名：use App\Home\Controller\Student as Student2

![命名空间的别名/导入](2018-08-15-16-56-13.png)


### 命名空间在项目中的使用
![命名空间在项目中的使用](2018-08-15-17-05-29.png)




# PDO
 - PDO就是PHP Data Object的简称。
 - PDO主要用来代替数据库操作类。
 - PDO就是一系统类。
 - PHP同时可以操作多个数据库。例如：MySQL、SQL Server、Oracle、Db2等。
 - PDO扩展为PHP访问数据库定义了一个轻量级的、一致性的接口，无论使用什么数据库，都可以通过一致的函数(方法)来执行查询和获取数据。
 - PDO是一个数据库访问抽象层，作用是统一各种数据库的访问接口，与MYSQL和MSSQL函数库相比，PDO让跨数据库的使用更具亲和力，与ADODB和MDB2相比，PDO更高效.
 - 有了PDO，您不必再使用mysql_*函数、oci_*函数或者mssql_*函数，也不必再为它们封装数据库操作类，只需要使用PDO接口中的方法就可以对各种数据库进行操作。在选择不同的数据库时，只修改PDO的DSN即可。

![PDO访问流程](2018-08-15-17-16-13.png)




## PDO连接MySQL
 - 描述：创建一个表示数据库连接的 PDO 对象
 - 语法：PDO::__construct ( string $dsn [, string $username [, string $password ]] )
 - 参数：
  - $dsn数据源名称，包含了连接数据库的基本信息；
	- 格式：$dsn = “dbtype: host=主机名; port=端口号; dbname=数据库名; charset=字符集”
	- dbtype参数：代表要连接的数据库的类型，例如：mysql、mssql、oracle等。
	- host参数：数据库服务器地址，可以域名，也可以是IP地址；
	- port参数：数据库端口号，mysql的端口号3306；
	- dbname参数：数据库名称；
	- charset参数：字符集。
  - $username数据库的用户名；
  - $password数据库的用户密码。


![PDO连接MySQL](2018-08-15-17-21-59.png)



### PDO对象常用方法
#### exec()
 - 描述：执行一条 SQL 语句，并返回受影响的行数
 - 语法：int PDO::exec ( string $statement )
 - 注意：执行SELECT语句，将返加0。


#### query()
 - 描述：执行SELECT、SHOW语句，并返回一个结果集对象(PDOStatement)
 - 语法：public PDOStatement PDO::query ( string $statement )

#### lastInsertId()
 - 描述：获取插入记录的ID号
 - 语法：string PDO::lastInsertId ( void )

#### setAttribute()
 - 描述：设置属性
 - 语法：bool PDO::setAttribute ( int $attribute , mixed $value )
 - 参数：
   - PDO::ATTR_CASE：强制列名为指定的大小写。 
	 - PDO::CASE_LOWER：强制列名小写。 
	 - PDO::CASE_NATURAL：保留数据库驱动返回的列名。 
	 - PDO::CASE_UPPER：强制列名大写。 
   - PDO::ATTR_ERRMODE：错误报告。 
	 - PDO::ERRMODE_SILENT： 仅设置错误代码。
	 - PDO::ERRMODE_WARNING: 引发 E_WARNING 错误
	 - PDO::ERRMODE_EXCEPTION: 抛出 exceptions 异常。
   - PDO::ATTR_DEFAULT_FETCH_MODE： 设置默认的提取模式。
	 - PDO::FETCH_ASSOC：返回一个索引为结果集列名的数组 
	 - PDO::FETCH_BOTH（默认）：返回一个索引为结果集列名和以0开始的列号的数组 
	 - PDO::FETCH_NUM：返回一个索引为以0开始的结果集列号的数组

![setAttribute](2018-08-15-17-24-39.png)


### PDOStatement结果集对象常用方法
#### fetch

 - 描述：从结果集中获取一行，并将指针下移。
 - 语法：mixed PDOStatement::fetch ([ int $fetch_style ] )
 - 参数：
   - PDO::FETCH_ASSOC：返回一个索引为结果集列名的数组 
   - PDO::FETCH_BOTH（默认）：返回一个索引为结果集列名和以0开始的列号的数组 
   - PDO::FETCH_NUM：返回一个索引为以0开始的结果集列号的数组

![fetch](2018-08-15-17-27-58.png)




#### fetchAll
 - 描述：返回一个包含结果集中所有行的数组
 - 语法：array PDOStatement::fetchAll ([ int $fetch_style] )

#### fetchColumn
 - 描述：从结果集中返回单独的一列。
 - 语法：string PDOStatement::fetchColumn ([ int $column_number = 0 ] )
 - 参数：$column_number是列的索引值，默认为0。

#### rowCount
 - 描述：返回受上一个 SQL 语句影响的行数
 - 语法：int PDOStatement::rowCount ( void )




### PDO错误处理模式
PDO的错误报告模式有三种：静默模式、警告模式、异常模式。
 - 静默模式(silent)：当PDO执行SQL语句有错时，不显示任何错误(默认)。
 - 警告模式(warning)：当PDO执行SQL语句有错时，用PHP的错误等级来报告信息。
 - 异常模式(exception)：当PDO执行SQL语句有错时，先抛出异常，再捕获异常。


#### 静默模式(silent)
 - PDO::errorCode()：获取错误状态码。如果状态码为为”00000”，说明没有错误。
 - PDO::errorInfo()：获取描述性的信息。

![静默模式](2018-08-15-17-32-45.png)

#### 警告模式
**要想报告”警告模式错误”，必须先设置错误的报告模式。**

![警告模式](2018-08-15-17-33-28.png)

#### 异常模式
**要想报告”异常模式错误”，必须先设置错误的报告模式。**

![异常模式](2018-08-15-17-34-09.png)


### SQL语句预处理

![SQL语句执行](2018-08-15-17-36-34.png)

 - SQL语句的执行，分成两个阶段：编译和执行。
 - 如果SQL语句，是第1次执行，先编译再执行。编译过程十分复杂，耗用系统资源，相对不太安全；
 - 如果SQL语句(即相同的SQL语句)，是第2次执行，直接从缓存中读取，无疑执行效率是最高的，也是比较安全的，可以有效避免SQL注入等安全问题；


#### PDO预处理的步骤
 - 先提取相同结构的sql部分！(将数据部分，可变的部分去掉)
 - 编译这个相同的结构！将编译结果保存！
 - 再将不同的数据部分进行替换！
 - 执行即可！


```php
$dsn = "mysql:host=127.0.0.1;port=3306;dbname=news;charset=utf8";
$db_user = 'root';
$db_pass = '123123';
$pdo = new PDO($dsn, $db_user, $db_pass);

// 1. 制作相同结构的SQL语句
// 使用占位符":value"来代替真正的数据
// $sql = "SELECT * FROM users WHERE id =:id";
// 使用占位符"?"来代替真正的数据
$sql = "SELECT * FROM users WHERE id =?";

// 2. 预编译相同结构的SQL语句
$PDOStatement = $pdo->prepare($sql);

// 3. 给占位符绑定真正的数据
// $PDOStatment->bindValue(":id","1");
$PDOStatement->bindValue("1","1");

// 4. 执行预编译的SQL语句
// 4. 执行预编译的SQL语句
$PDOStatement->execute();

var_dump($row = $PDOStatement->fetch(PDO::FETCH_ASSOC));
// array(3) { ["id"]=> string(1) "1" ["username"]=> string(3) "123" ["password"]=> string(4) "csrf" }
```



# Smarty

## 模版引擎工作原理
 - HTML代码和PHP代码分离，其实就是前端人员和程序员分离。
 - 前端人员喜欢HTML标记形式的代码：{$name}
 - PHP只能解释的代码：<?php echo $name?>
 - 如何将{$name}转成<?php echo $name?>？思路就是：查找替换。

![模版引擎工作原理](2018-08-15-20-52-15.png)



## Smarty简单案例
![简单案例](2018-08-15-22-06-42.png)




## Smarty常用配置
### 左右定界符
 - 左定界符：$smarty->left_delimiter = “<{”
 - 右定界符：$smarty->right_delimiter = “}>”

![左右定界符](2018-08-17-09-53-00.png)


### 设置或读取视图文件目录
 - 设置视图文件目录：$smarty->setTemplateDir(新目录路径)
 - 读取视图文件目录：$smarty->getTemplateDir()


### 其它目录的设置和读取方法
 - 编译目录的设置：$smarty->setCompileDir()
 - 编译目录的读取：$smarty->getCompileDir()
 - 配置目录的设置：$smarty->setConfigDir()
 - 配置目录的设置：$smarty->getConfigDir()


## Smarty中的变量
### 普通变量
 - 所有的PHP的变量，都可以传递到视图文件来使用。
 - 但是，在视图中，对象和资源变量，不常用。

![普通变量](2018-08-17-10-00-14.png)


### 保留变量
所有的超全局数组变量，可以在视图文件中直接使用。

#### 访问页面请求变量
![访问页面请求变量](2018-08-17-10-04-28.png)
![访问页面请求变量](2018-08-17-10-04-37.png)


#### 访问PHP预定义常量

![访问PHP预定义常量](2018-08-17-10-05-08.png)

#### Smarty时间戳
![Smarty时间戳](2018-08-17-10-05-57.png)


#### Smarty配置文件

## Smarty-foreach
![Smarty-foreach](2018-08-17-10-10-51.png)

![Smarty-foreach](2018-08-17-10-11-23.png)

### foreach的常用属性应用
 - @key：输出当前值的索引，可能是整型索引，也可能是字符索引；
 - @index：当前数组索引，从0开始计算；
 - @iteration，当前循环的次数，从1开始计算；
 - @first：当首次循环时，值为true；
 - @last：当最后一次循环时，值为true；
 - @total：是整个循环的次数，可以在foreach内部或外部使用；


![foreach常用属性](2018-08-17-10-12-22.png)

## Smarty-section
 - section循环，与PHP的for循环相似。
 - for循环可以指定循环起点。
 - for循环可以指定步长值。
 - for循环可以计算最大循环次数。
 - for只能遍历枚举数组，数组下标必须是从0开始的正整数。
 - for不能遍历关联数组，数组下标是字符串。


![Smarty-section](2018-08-17-10-15-11.png)

name和loop属性是必须的。
start、step、max是可选属性。


![Smarty-secion-shili](2018-08-17-10-15-33.png)


![Smarty-section-shili2](2018-08-17-10-15-52.png)

![Smarty-section-shili3](2018-08-17-10-16-05.png)


## Smarty-if
Smarty中的if与PHP中的if很像。PHP中的运算符在Smarty中都可以使用。

![Smarty-if](2018-08-17-10-16-38.png)

### if中的运算符
![if中的运算符](2018-08-17-10-17-23.png)



![smarty-if-shili](2018-08-17-10-17-48.png)

![smarty-if-shili2](2018-08-17-10-18-06.png)



## Smarty中的变量调节器
变量调节器，就是对变量进行格式的函数，对变量进行格式化输出。

![变量调节器](2018-08-17-10-29-20.png)



### Smarty中常用变量调节器
![Smarty中常用变量调节器](2018-08-17-10-30-26.png)

 - upper：转成全大写，对应PHP的strtoupper()
 - lower：转成全小写，对应PHP的strtolower()
 - nl2br：将\n换行符，转成`<br />`，对应PHP的nl2br()
 - replace：查找替换，对应于PHP的str_replace()
 - date_format：时间戳格式化，对应于PHP的date()
 - truncate：截取字符串，对应于PHP的substr()或mb_substr()

![smarty中常用变量调节器-实例](2018-08-17-10-33-56.png)

### date_format调节器的参数及应用
![date_format调节器的参数及应用](2018-08-17-10-34-38.png)


![date_format调节器的参数及应用-实例](2018-08-17-10-35-13.png)

### truncate调节器的应用
![truncate调节器的应用](2018-08-17-10-35-43.png)

![truncate调节器的应用-实例](2018-08-17-10-36-03.png)


**问题：为什么有时截取时，会出现中文乱码？**
出现中文乱码，说明是按字节截取。在UTF-8下，一个汉字是三个字节。
截取字符串函数substr()，是按字节截取。
截取字符串函数mb_substr()，是按字符截取。这个需要在PHP配置中开启。



# MVC
 - MVC就是一种编程思想，是一种软件设计的典范。
 - 在MVC框架中，没有任何新的知识点。每种语言，都有MVC框架。
 - MVC是由Model(数据模型)、View(数据展示)、Controller(调度中心)三个模块构成。
 - 在MVC中，每个模块只做自己的事情，不是自己的事情一定不做。
 - 举例：一家饭店，只买饭，如果没有面条了，打电话给卖面条的送即可；如果没有食用油了，打电话给卖油的送即可。等等。
 - MVC专人做块事，不是你自己的活可以不干。
 - 在一次HTTP请求过程中，Controller负责与客户交互，Controller找Model来获取数据，View负责展示或格式化数据。


## MVC各部分功能
 - Controller(控制器)：负责与客户打交道，包括：获取客户请求(GET和POST)、返回结果给客户、逻辑处理、调用Model来获取数据、调用View来格式化数据。理解为“调度中心”、“控制中心”。
 - Model(数据模型)：负责数据处理，与MySQL直接打交道。数据的所有操作，都由Model来处理。数据获取到，再交给控制器。
 - View(视图)：负责数据的展示、格式化。主要涉及到前端相关技术：HTML、CSS、JS、AJAX、jQuery、Flash等。
 - MVC适合大项目、适合多人合作开发。



## MVC示意图
![MVC示意图](2018-08-17-10-42-46.png)


 - 一个项目由若干个功能模块构成：新闻管理、学生管理、产品管理、文章管理、分类管理等。
 - 一个功能只对应一个控制器，例如：NewsController、StudentController、ProductController
 - 一个控制器，只对应一个模型类，例如：NewsModel、StudentModel、ProductModel
 - 一个模型类，对应一个数据表的操作，例如：news、student、product
 - 一个控制器，可以对应多个视图文件，例如：StudentView.html、StudentAddView.html等





## MVC流程图
![MVC流程图](2018-08-17-16-48-20.png)

![MVC流程图](2018-08-19-19-49-21.png)


## 最终MVC的目录结构
![最终MVC的目录结构](2018-08-17-16-53-51.png)








