# Mysql 命令详解

## 一、  登录数据库

> 格式 mysql -h 主机地址 -u 用户名 -p 用户密码
>
> mysql -u 用户名 –p
>
> 输入指令后使用密码登录
>
> ![image-20220905220349639](images/image-20220905220349639.png)

## 二、修改数据库密码

> 修改数据库本地账号密码  
>
> ```
> alter user 'root'@'localhost' identified by 'root';
> ```
>
> 或者
>
> ```
> set password = password(‘root’);
> ```
>
> 设置本地密码永不过期
>
> ```
> alter user 'root'@'localhost' password expire never;
> ```
>
>  使修改生效
>
> ```
> FLUSH PRIVILEGES;
> ```

## 三、数据库用户操作

### 1、查看现用用户

> 首先到mysql库中
>
> ```
> use mysql;
> ```
>
> 查询host,user 数据
>
> ```
>     select Host,User from user;
> ```
>
> 下面的列表就是查询user表后可以访问数据库的指定的Ip地址
>
> ![image-20220905220644229](images/image-20220905220644229.png)

### 2、增加新用户

> grant 权限 on 数据库.表名 to ‘用户名’@’登录主机’ identified by "密码";
>
> 例：增加一个用户user密码为password，让其可以在本机上登录， 并对所有数据库有查询、插入、修改、删除的权限。首先用以root用户连入mysql，然后键入以下命令：
>
> ```
> grant select,insert,update,delete on *.* to'user'@'localhost' identified by "password";
> ```
>
> 如果希望该用户能够在任何机器上登陆mysql，则将localhost改为"%"。
>
> ```
> grant all on *.* to 'wang'@'localhosl' identified by "password";
> ```
>
> - 库名:要远程访问的数据库名称,所有的数据库使用“*”
>
> - 表名:要远程访问的数据库下的表的名称，所有的表使用“*”
>
> - 用户名:要赋给远程访问权限的用户名称
>
> - 登录主机:可以远程访问的电脑的IP地址，所有的地址使用“%”
>
> - 密码:要赋给远程访问权限的用户对应使用的密码

### 3、删除用户

> drop user '用户名'@'登录主机';
>
> ```
> drop user 'wang'@'192.168.101.148';
> ```
>
> ![image-20220905220832843](images/image-20220905220832843.png)

### 4、查看用户权限

> show grants for ‘用户名’@’登录主机’;
>
> 添加一个本地登录只有查询和修改权限的user用户
>
> ```
> grant select, update on *.* to'user'@'localhost' identified by "password";
> ```
>
> 查看 user 用户权限
>
> ```
>   show grants for 'user'@'localhost';
> ```
>
> ![image-20220905220921897](images/image-20220905220921897.png)

### 5、删除权限

> 删除 user用户的update 权限
>
> ```
> revoke update on *.* from 'user'@'localhost';
> ```
>
> ![image-20220905221023652](images/image-20220905221023652.png)

### 6 、查看Mysql 当前登录用户

> ```
> select user();
> ```
>
> ![image-20220905221057181](images/image-20220905221057181.png)

## 四、数据库及表管理

### 1、查看数据库

> ```
> show databases;
> ```
>
> ![image-20220905221200970](images/image-20220905221200970.png)

### 2、查看数据中的表

> 首先进入数据库 use 数据库名;
>
> 显示库中的表结构
>
> ```
>  show tables;
> ```
>
> ![image-20220905221315549](images/image-20220905221315549.png)

### 3、显示数据表结构

> ```
> describe 表名;
> ```
>
> ![image-20220905221336493](images/image-20220905221336493.png)

### 4、查看表内容

> ```
> select 表字段 for 表名称 where 条件;
> ```
>
> ![image-20220905221410080](images/image-20220905221410080.png)

## 五、mysqldump 导出库表详细举例

> 在mysql中有个测试库，下面有两个表student表和teachers表
>
> ![image-20220905221453091](images/image-20220905221453091.png)

### 1、导出指定数据库

> mysqldump -u 用户名 -p数据库名 > 导出的文件名
>
> 导出test库下的所有表
>
> ![image-20220905221520032](images/image-20220905221520032.png)

### 2、导出整个数据库

> 查询数据库，并吧查询数据重定向出来
>
> mysqldump -u 用户名 -p数据库名 –e “select * from …” > ~/nohup.out

### 3、导出一个表

> mysqldump -u 用户名 -p数据库名 表名> 导出的文件名
>
>    导出 test 库中的student表
>
> ![image-20220905221708886](images/image-20220905221708886.png)

## 六 、sql查询命令

### 1、统计查询

> count() 括号中是*，是统计查询数据的所有行数，括号中是字段名，统计的是字段查询出来的行数。
>
> select count(*) from 表名 where 条件;

### 2、排序

> select * from 表名 where 条件 order by 表字段 asc; //asc 升序排列（默认） 
>
> select * from 表名 where 条件 order by 表字段 desc; //desc 降序排列 

### 4、分页

> select * from 表名 where 条件 limit 10; //在页面中展示10行数据 