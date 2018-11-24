#开启sql自动化审计
```
docker run --name inception -v /Users/jim/aa/inc1.cnf:/etc/inc.cnf  -p 6669:6669 -dti registry.cn-hangzhou.aliyuncs.com/lihuanhuan/inception
```
#调用采用python3 mysqldb语言包
返回格式中有详细的定义参数
具体这个是最简单的调用.
```
#!/usr/bin/python
#-\*-coding: utf-8-\*-
import MySQLdb
sql='/*--user=root;--password=my-secret-pw;--host=demo.p88health.com;--execute=1;--port=33061;*/\
inception_magic_start;\
use mysql;\
CREATE TABLE adaptive_office(id int);\
inception_magic_commit;'
try:
    conn=MySQLdb.connect(host='127.0.0.1',user='',passwd='',db='',port=6669)
    cur=conn.cursor()
    ret=cur.execute(sql)
    result=cur.fetchall()
    num_fields = len(cur.description)
    field_names = [i[0] for i in cur.description]
    print (field_names)
    for row in result:
        print (row[0], "|",row[1],"|",row[2],"|",row[3],"|",row[4],"|",row[5],"|",row[6],"|",row[7],"|",row[8],"|",row[9],"|",row[10])
    cur.close()
    conn.close()
except MySQLdb.Error as e:
     print ("Mysql Error %d: %s" % (e.args[0], e.args[1]))
```

#inc.cnf
```
[inception]
general_log=1
general_log_file=inception.log
port=6669
socket=/tmp/inc.socket
character-set-client-handshake=0
character-set-server=utf8
inception_remote_system_password=my-secret
inception_remote_system_user=root
inception_remote_backup_port=3306
inception_remote_backup_host=demo.p88health.com
inception_support_charset=utf8,utf8mb4
inception_enable_nullable=0
inception_check_primary_key=1
inception_check_column_comment=1
inception_check_table_comment=1
inception_osc_on=OFF
inception_osc_bin_dir=/usr/bin
inception_osc_min_table_size=1
inception_osc_chunk_time=0.1
inception_enable_blob_type=1
inception_check_column_default_value=1
```
#inc1.cnf
```
[inception]
######inception基本设置######
#MySQL原生参数,设置inception程序打开全日志
general_log=ON
#MySQL原生参数，定义inception程序的全日志位置
general_log_file=/data/w/inception/data/inception.log
#MySQL原生参数,定义inception的端口
port=6669
#MySQL原生参数,定义inception的socket
socket=/tmp/inc.socket
#MySQL原生参数,跳过mysql程序起动时的字符参数设置,使用服务器端字符集设置
character-set-client-handshake=OFF
#MySQL原生参数,设置默认的服务器端字符集
character-set-server=utf8mb4
#设置当前Inception服务器是不是只读的，这是为了防止一些人具有修改权限的帐号时，通过Inception误修改一些数据，如果inception_read_only设置为ON，则即使开了enable-execute，同时又有执行权限，也不会去执行，审核完成即返回
inception_read_only=OFF
#打开与关闭Inception对SQL语句中各种名字的检查，如果设置为ON，则如果发现名字中存在除数字、字母、下划线之外的字符时，会报Identifier "invalidname" is invalid, valid options: [a-z,A-Z,0-9,_]
inception_check_identifier=ON
#inception的统计功能,设置是不是支持统计Inception执行过的语句中，各种语句分别占多大比例，如果打开这个参数，则每次执行的情况都会在备份数据库实例中的inception库的statistic表中以一条记录存储这次操作的统计情况，每次操作对
应一条记录，这条记录中含有的信息是各种类型的语句执行次数情况
inception_enable_sql_statistic=ON
######inception的语法校验######
#检查insert语句的列属性，insert table (xx,xx..) values(xx,xx..)
inception_check_insert_field=ON
#在DML语句中没有WHERE条件时，报错
inception_check_dml_where=ON
#在DML语句中使用了LIMIT时，报错
inception_check_dml_limit=ON
#在DML语句中使用了Order By时，报错
inception_check_dml_orderby=ON
#Select * 时，报错
inception_enable_select_star=ON
#order by rand时，报错
inception_enable_orderby_rand=ON
#创建或者新增列时如果列为NULL，报错
inception_enable_nullable=OFF
#不支持外键
inception_enable_foreign_key=OFF
#一个索引中，列的最大个数，超过这个数目则报错（可设置1-64）
inception_max_key_parts=5
#在一个修改语句中，预计影响的最大行数，超过这个数就报错
inception_max_update_rows=5000
#一个表中，最大的索引数目，超过这个数则报错（1-1024）
inception_max_keys=16
#建表指定的存储引擎不为Innodb，报错
inception_enable_not_innodb=ON
#表示在建表或者建库时支持的字符集
inception_support_charset=utf8mb4
#建表时，表没有注释时报错
inception_check_table_comment=ON
#建表时，列没有注释时报错
inception_check_column_comment=ON
#建表时，如果没有主键，则报错
inception_check_primary_key=ON
#不支持分区表
inception_enable_partition_table=OFF
#不支持enum,set,bit数据类型
inception_enable_enum_set_bit=OFF
#检查索引名字前缀是否为"idx_"，检查唯一索引前缀是不是"uniq_"，不是的话，报错
inception_check_index_prefix=ON
#自增列为无符号型
inception_enable_autoincrement_unsigned=ON
#当char类型的长度大于这个值时，就提示将其转换为VARCHAR
inception_max_char_length=16
#当建表时自增列的值指定的不为1，则报错
inception_check_autoincrement_init_value=ON
#当建表时自增列的类型不为int或者bigint时报错
inception_check_autoincrement_datatype=ON
#建表时，如果没有为timestamp类型指定默认值，则报错
inception_check_timestamp_default=ON
#不允许自己设置列的字符集
inception_enable_column_charset=OFF
#建表时，如果指定的自增列的名字不为ID，则报错，说明是有意义的，给提示
inception_check_autoincrement_name=ON
#在多个改同一个表的语句出现是，报错，提示合成一个
inception_merge_alter_table=ON
#检查在建表、修改列、新增列时，新的列属性要有默认值
inception_check_column_default_value=ON
#检查支持BLOB字段，包括建表、修改列、新增列操作
inception_enable_blob_type=ON
#检查在SQL语句中，是不是有标识符被写成MySQL的关键字，默认值为不允许。
inception_enable_identifer_keyword=OFF
######inception备份设置（可动态在连接命令中传入）######
#远程备份库的host
inception_remote_backup_host=YOURHOST
#远程备份库的port
inception_remote_backup_port=YOURPORT
#远程备份库的一个用户
inception_remote_system_user=YOURUSER
#用户的密码
inception_remote_system_password=YOUREPASWORD
######inception的osc设置######
#指定pt-online-schema-change脚本的位置
inception_osc_bin_dir=/usr/bin/
#对应OSC参数--recursion-method
inception_osc_recursion_method=processlist
#对应OSC参数--[no]drop-old-table
inception_osc_drop_old_table=1
#对应OSC参数--[no]check-replication-filters
inception_osc_check_replication_filters=1
#对应OSC参数--check-interval
inception_osc_check_interval=5
#对应OSC参数--max-lag
inception_osc_max_lag=1
#对应OSC参数--chunk-time
inception_osc_chunk_time=0.05
#对应OSC参数--max-load中的Threads_connected部分
inception_osc_max_thread_connected=1300
#对应OSC参数--max-load中的Threads_running部分
inception_osc_max_thread_running=64
#这个参数实际上是一个OSC的开关，如果设置为0，则全部ALTER语句都走OSC，如果设置为非0，则当这个表占用空间大小大于这个值时才使用OSC方式。单位为M，
inception_osc_min_table_size=16
#一个全局的OSC开关，默认是打开的
inception_osc_on=1
#无法支持--set-vars
######权限认证######
#MySQL原生参数，因为Incpetion没有权限验证过程，那么为了实现更安全的访问，可以给Inception服务器的这个参数设置某台机器（Inception上层的应用程序）不地址，这样其它非法程序是不可访问的，那么再加上Inception执行的选项中的用
户名密码，对MySQL就更加安全
#bind_address=*
#这个用户名在配置之后，在连接Inception的选项中可以不指定user，这样线上数据库的用户名及密码就可以不暴露了，可以做为临时使用的一种方式，但这个用户现在只能是用来审核，也就是说，即使在选项中指定--enable-execute，也不能执
行，这个是只能用来审核的帐号。
#inception_user=''
#与上面的参数是一对，这个参数对应的是选项中的password，设置这个参数之后，可以在选项中不指定password
#inception_password=''
```
