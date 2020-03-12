# 使用Mysql

## 安装
[下载](https://dev.mysql.com/downloads/mysql/)

## 配置
+ 编辑`my.ini`文件
在Mysql安装文件夹下新建`my.ini`文件
```ini
[client]
# 设置mysql客户端默认字符集
default-character-set=utf8
 
[mysqld]
# 设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=C:/Software/IDE/mysql-8.0.18
# 设置数据存放位置
datadir=C:/Software/IDE/mysql-8.0.18/data
# 设置初始密码, 好像没有什么用
default_authentication_plugin=mysql_native_password
# 允许最大连接数
max_connections=20
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=UTF8MB4		
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```
1. 其中`datadir`不能有任何文件, 否则启动mysql服务会报错:`--initialize specified but the data directory has files in it. Aborting.`
2. 配置完成后进入安装路径的`.\bin`目录, 用管理员身份打开`CMD`键入:`mysqld --initialize --console`进行初始化得到密码
3. 输入mysql指令发现` You must reset your password using ALTER USER statement before executing this statement.`, 需要通过`alter user user() identified by "mypwd";`修改密码即可.


## mysql常用指令
+ [WINDOWS]`mysqld install`: 安装mysql服务
+ [WINDOWS管理员]`net start mysql`: 启动mysql服务
+ `mysql -u root -p`: 进入mysql
+ `alter user user() identified by "passwrod";`: 修改mysql密码

## mysql可视化
+ [heidi sql](https://www.heidisql.com/)

## REF:
[安装mysql](https://www.runoob.com/mysql/mysql-install.html)

[设置mysql密码](https://blog.csdn.net/hj7jay/article/details/65626766)

