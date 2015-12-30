---
layout: post
title: 在Linux上架设Perfect服务器
---

Swift是一门可以用来写服务端的语言，本文记录了搭建基于Swift的[Perfect](https://github.com/PerfectlySoft/Perfect)服务器的步骤。Swift暂时只支持Linux的Ubuntu，本文安装的环境是Ubuntu 14.04.3，使用OS X的Terminal远程到云服务器上安装完成。
### 一、SSH到远程Ubuntu服务器

### 二、安装Swift编译器，见[官网](https://swift.org/download/#linux)

1. 安装Swift编译器依赖

		sudo apt-get install clang libicu-dev
2. 下载最新的Swift编译器和sig符号文件
		
		wget https://swift.org/builds/ubuntu1404/swift-2.2-SNAPSHOT-2015-12-21-a/swift-2.2-SNAPSHOT-2015-12-21-a-ubuntu14.04.tar.gz
		wget https://swift.org/builds/ubuntu1404/swift-2.2-SNAPSHOT-2015-12-21-a/swift-2.2-SNAPSHOT-2015-12-21-a-ubuntu14.04.tar.gz.sig
	
3. 导入PGP密钥，把密钥更新到最新版本
		
		wget -q -O - https://swift.org/keys/all-keys.asc | gpg --import -
		gpg --keyserver hkp://pool.sks-keyservers.net --refresh-keys Swift
		
4. 到步骤2下载的.sig文件所在文件夹，验证sig文件

		gpg --verify swift-2.2-SNAPSHOT-2015-12-21-a-ubuntu14.04.tar.gz.sig
		
5. 到步骤2下载的.tar.gz文件所在的文件夹，解压文件

		tar xzf swift-2.2-SNAPSHOT-2015-12-21-a-ubuntu14.04.tar.gz

6. 将解夹之后的/swift-2.2-SNAPSHOT-2015-12-21-a-ubuntu14.04/usr/bin路径导出到系统PATH下

		export PATH=/path/to/usr/bin:"${PATH}"

### 三、使用apt-get命令安装三个Perfect依赖
	
	apt-get install libssl-dev
	apt-get install libevent-dev
	apt-get install libsqlite3-dev


### 四、下载Perfect项目源码
	git clone https://github.com/PerfectlySoft/Perfect.git
		
### 五、编译PerfectLib

	cd Perfect/PerfectLib
	make
	sudo make install
		
### 六、安装PerferctServer
1. 改造Perfect/PerfectServer的makefile文件，新建两个文件夹，PerfectLibraries放置各个服务编译好的.so，webroot放置静态html或mustache等模板文件

		@mkdir -p PerfectLibraries
    	@mkdir -p webroot


2. 以Examples中的Authenticator为例，改造Perfect/Examples/Authenticator的makefile文件，将编译好的so和mustache文件放到PerfectServer对应的位置。另两个Example可以自行修改make文件。
		
		PERFECT_SERVER = ../../PerfectServer

		@cp *.so ${PERFECT_SERVER}/PerfectLibraries/
        @cp Authenticator/*.mustache ${PERFECT_SERVER}/webroot/
	
3. 编译PerfectServer

		cd Perfect/PerfectServer
		make
		
4. 以Authenticator为例，进入例子文件夹并且编译

		cd ../../Examples/Authenticator
		make
	
4. 编译好后生成perfectserverhttp和perfectserverfcgi两个可运行文件，运行PerfectServer

		./perfectserverhttp

5. 在浏览器中打开ip:port加对应路径即可打开页面，如：http://120.27.xxx.xxx:8181/index

### 七、安装PerfectServer FastCGI服务

PerfectServer支持加载成Apache module模块，由FastCGI提供Apache与Server之间的联通。安装步骤如下：

1. 安装Apache，并安装rewrite模块

		sudo apt-get install apache2
		a2enmod rewrite
2. 编译mod_perfect

		cd Perfect/Connectors/mod_perfect
		make
3. 将Perfect工程放到/var/www目录下，否则Apache没有权限访问root用户目录下的文件
	
		mv /Perfect /var/www/
			
4. 查看webroot目录的权限，如果没有写权限则修改文件夹权限
		
		s -ld /var/www/Perfect/PerfectServer/webroot/
		chmod 755 /var/www/Perfect/PerfectServer/webroot/
		
5. 用vi打开Apache配置文件

		vi /etc/apache2/sites-enabled/000-default.conf
		
	除了000-default.conf中，也可以在/etc/apache2/apache2.conf中自定义配置。配置文件主要操作是将mod_perfect模块加入、配置重写规则和webroot静态页面路径。全文如下：
		
		<IfModule !perfect_module>
    		LoadModule perfect_module /var/www/Perfect/Connectors/mod_perfect/mod_perfect.so
		</IfModule>

		<IfModule !rewrite_module>
   			LoadModule rewrite_module libexec/apache2/mod_rewrite.so
		</IfModule>

		<VirtualHost *:80>
		        # The ServerName directive sets the request scheme, hostname and port that
		        # the server uses to identify itself. This is used when creating
		        # redirection URLs. In the context of virtual hosts, the ServerName
		        # specifies what hostname must appear in the request's Host: header to
		        # match this virtual host. For the default virtual host (this file) this
		        # value is not decisive as it is used as a last resort host regardless.
		        # However, you must set it for any further virtual host explicitly.
		        #ServerName www.example.com
		
		        ServerAdmin webmaster@localhost
		        ServerName my-server.local		        
		        DocumentRoot "/var/www/Perfect/PerfectServer/webroot"
		        <Directory "/var/www/Perfect/PerfectServer/webroot">
		                Options Indexes FollowSymLinks
		                AllowOverride None
		                Order allow,deny
		                Allow from all
		                Require all granted
		                DirectoryIndex index.mustache index.html
		        </Directory>
		
		
		        RewriteEngine on
		
		        # unless a directory, remove trailing slash
		        RewriteCond %{DOCUMENT_ROOT}%{REQUEST_FILENAME} !-d
		        RewriteRule ^(.*)/$ $1 [R=301,L]
		
		        # resolve .mustache file for extensionless mustache urls
		        RewriteCond %{DOCUMENT_ROOT}%{REQUEST_FILENAME} !-d
		        RewriteCond %{DOCUMENT_ROOT}%{REQUEST_FILENAME} !-f
		        RewriteCond %{DOCUMENT_ROOT}%{REQUEST_FILENAME}\.mustache -f
		        RewriteRule ^(.*)$ $1.mustache [NC,PT,L]
		
		        # redirect external .mustache requests to extensionless url
		        RewriteCond %{THE_REQUEST} ^[A-Z]+\ /([^/]+/)*[^.#?\ ]+\.mustache([#?][^\ ]*)?\ HTTP/
		        RewriteRule ^(([^/]+/)*[^.]+)\.mustache $1 [R=301,L]
		
		        <Location ~ "^.*\.[Mm][Uu][Ss][Tt][Aa][Cc][Hh][Ee]$">
		                SetHandler perfect-handler
		        </Location>
		        
		        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
		        # error, crit, alert, emerg.
		        # It is also possible to configure the loglevel for particular
		        # modules, e.g.
		        #LogLevel info ssl:warn
		
		        ErrorLog ${APACHE_LOG_DIR}/error.log
		        CustomLog ${APACHE_LOG_DIR}/access.log combined
		
		        # For most configuration files from conf-available/, which are
		        # enabled or disabled at a global level, it is possible to
		        # include a line for only one particular virtual host. For example the
		        # following line enables the CGI configuration for this host only
		        # after it has been globally disabled with "a2disconf".
		        #Include conf-available/serve-cgi-bin.conf
		</VirtualHost>

6. 重新启动Apache
		
		service apache2 restart
		
7. 启动Perfect的FastCGI服务

		cd /var/www/Perfect/PerfectServer/
		./perfectserverfcgi
		
8. 通过浏览器访问Authenticator模块，地址：http://120.27.xxx.xxx/index


	
	




	
	