
### 1.查看数据库引擎
mysql->show variables like '%storage_engine%';
此命令可以查看数据库默认引擎。
如果默认引擎为My|Sam，请修改为InnoDB。

----
### 2.在my.ini添加默认引擎
打开my.ini,在[mysqld]中添加
;设置INNODB引擎模式、

```sql
default-storage-engine=INNODB
```

## 区别：
1.InnoDB支持事务，MyISAM不支持，对于InnoDB每一条SQL语言都默认封装成事务，自动提交，这样会影响速度，所以最好把多条SQL语言放在begin和commit之间，组成一个事务； <br>

2. InnoDB支持外键，而MyISAM不支持。对一个包含外键的InnoDB表转为MYISAM会失败。<br>
3.InnoDB是聚集索引，使用B+Tree作为索引结构，数据文件是和（主键）索引绑在一起的（表数据文件本身就是按B+Tree组织的一个索引结构），必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。 <br>
4. InnoDB不保存表的具体行数，执行select count(*) from table时需要全表扫描。而MyISAM用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快（注意不能加有任何WHERE条件）；<br>
5. Innodb不支持全文索引，而MyISAM支持全文索引，在涉及全文索引领域的查询效率上MyISAM速度更快高；PS：5.7以后的InnoDB支持全文索引了.	  <br>
6. MyISAM表格可以被压缩后进行查询操作			<br>
7. InnoDB支持表、行(默认)级锁，而MyISAM支持表级锁 <br>
8、InnoDB表必须有主键（用户没有指定的话会自己找或生产一个主键），而Myisam可以没有。 <br>
9、Innodb存储文件有frm、ibd，而Myisam是frm、MYD、MYI<br>
        Innodb：frm是表定义文件，ibd是数据文件<br>
        Myisam：frm是表定义文件，myd是数据文件，myi是索引文件<br>

