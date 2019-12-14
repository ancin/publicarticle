# mysql乱码错误。

# 查看mysql 编码
 ## show varibables like '%char%';
 
 ## 显示
  character_set_database    latin1
  character_set_server     latin1
  character_set_client     utf8
  
  #session 范围
   show variables like '%char%';
   
   ## 修改
   set character_set_server=utf8;
   set character_set_database=utf8;
   show variables like '%char%';

# global范围
mysql设置变量的范围默认是session范围。如果设置多个会话的字符集那么需要设置global范围:Set [global|session] variables
set global character_set_database=utf8;
set global character_set_server=utf8;
show variables like '%char%';

# 重启mysql 服务器；又回去了。。。 不用怕
  service mysqld restart
mysql -uroot -pyourpassword
show variables like '%char%';

修改mysql配置文件/etc/my.cnf。

[mysqld]
character-set-server=utf8 
[client]
default-character-set=utf8 
[mysql]
default-character-set=utf8

