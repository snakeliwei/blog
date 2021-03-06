#SQL HELP Facility Installation

#写在前面的话

Unix/Linux下可以用`man`、`info`查询命令的具体用法，十分方便，既不需要联网要求，也不需要额外打开所谓的命令大全，对于系统管理员来说，确实是一个不可或缺的好帮手。

在数据库方面，MySQL、SQLite、MongoDB对SQL、Utility等都提供十分友好的help输出；Oracle Database也有，但仅有SQL*Plus，SQL的帮助需要额外安装。

查了下MOS，找到一篇比较古老的文档，最后更新时间是在2003年，那应该是针对Oracle 8i的，用SQL*Loader导入。用AvaFind查了下硬盘，发现在我的电脑上还真有8i的help。后来，也在网络上找了下，走了一些弯路，故写下备忘，当然也算是一种吐槽。

#Doc [1054547.6]

##Problem Description
 
You try to  use the HELP facility in SQL*Plus and you receive the following 
error: 
 
    'HELP not accessible.'  

##Solution Description: 

The SQL*Plus HELP facility has not been enabled. 

The SQL*Plus HELP facility can be created by performing the following steps: 
 
1. Go to the following directory: 
         
	% cd $ORACLE_HOME/sqlplus/admin/help 
 
2. Login to SQL*Plus as the user SYSTEM. 
 
	% sqlplus system/<password> 
  
3. Run the "helptbl.sql" script. 
 
	SQL> @helptbl 
 
4. Exit SQL*Plus and execute the following SQL*Loader commands:  
 
	% sqlldr system/manager control=plushelp.ctl	
	
	% sqlldr system/manager control=sqlhelp.ctl 
	
	% sqlldr system/manager control=plshelp.ctl 
	 
5. Log back into SQL*Plus as the user SYSTEM and run the "helpindx.sql" 
   script.  
 
   SQL> @helpindx 
 
 
##Solution Explanation: 

Performing these steps will build and load the SQL*Plus HELP table.


#Install SQL HELP on Oracle 11g

安装前，仅有`SQL*Pus`的help（安装数据库时一般会自动安装）：

	SQL> help index
	help index
	
	Enter Help [topic] for help.
	
	 @             COPY         PAUSE                    SHUTDOWN
	 @@            DEFINE       PRINT                    SPOOL
	 /             DEL          PROMPT                   SQLPLUS
	 ACCEPT        DESCRIBE     QUIT                     START
	 APPEND        DISCONNECT   RECOVER                  STARTUP
	 ARCHIVE LOG   EDIT         REMARK                   STORE
	 ATTRIBUTE     EXECUTE      REPFOOTER                TIMING
	 BREAK         EXIT         REPHEADER                TTITLE
	 BTITLE        GET          RESERVED WORDS (SQL)     UNDEFINE	
	 CHANGE        HELP         RESERVED WORDS (PL/SQL)  VARIABLE	
	 CLEAR         HOST         RUN                      WHENEVER OSERROR	
	 COLUMN        INPUT        SAVE                     WHENEVER SQLERROR	
	 COMPUTE       LIST         SET                      XQUERY	
	 CONNECT       PASSWORD     SHOW


	SQL> help topics
	help topics
	Help is available on the following topics:
	/
	@
	@@
	ACCEPT
	APPEND
	ARCHIVE LOG
	ATTRIBUTE
	BREAK
	BTITLE
	CHANGE
	... ...
	XQUERY


HELP表`system.help`结构：
	
	SQL> desc system.help
	desc system.help

	 Name			  Null?     Type	
	 ---------------- --------- ------------------
	 TOPIC		      NOT NULL  VARCHAR2(50)
	 SEQ			  NOT NULL  NUMBER
	 INFO					    VARCHAR2(80)

查看当前Help数：

	select count(1) from system.help;
	  COUNT(*)
	----------
	       919

`help create`相关主题：

	SQL> help create
	SP2-0172: No HELP matching this topic was found.

SQL*Plus帮助文件路径：

	[oracle@oradb ~]$ cd /db/oracle/product/11.2.0/db_1/sqlplus/admin/help/
	[oracle@oradb help]$ ll
	total 80
	-rw-r--r--. 1 oracle oinstall   265 Feb 17  2003 helpbld.sql
	-rw-r--r--. 1 oracle oinstall   337 Jun 28  2000 helpdrop.sql
	-rw-r--r--. 1 oracle oinstall 65975 Jun 29  2009 helpus.sql
	-rw-r--r--. 1 oracle oinstall  2086 Jan  6  2009 hlpbld.sql

以上脚本简介：

* helpbld.sql：Invoke and execute the script to loads the `SQL*Plus` HELP system and upon completion, exit the `SQL*Plus` connection. Code: @@&1/hlpbld.sql &2
* helpdrop.sql：Drops the `SQL*Plus` HELP table
* helpus.sql：Inserts `SQL*Plus` HELP text in English. This script is called from helpbld.sql
* hlpbld.sql：Builds the `SQL*Plus` HELP table and loads the HELP data from a data file. The data file must exist before this script is run.

在安装Oracle数据库时默认已经安装一些Help，不过主要是针对`SQL*Plus`，对Oracle SQL的Help没有。查MOS，暂且仅找到8i自带的SQL Help。在CSDN上找了下，有一个help.sql脚本，针对SQL语法的，对DBA日常管理应该已经够用。下载链接见文末[参考](#reference)。

上传help.sql文件：

	[oracle@oradb help]$ rz
	rz waiting to receive.**B0100000023be50
	?

	[oracle@oradb help]$ ll
	total 596
	-rw-r--r--. 1 oracle oinstall    265 Feb 17  2003 helpbld.sql
	-rw-r--r--. 1 oracle oinstall    337 Jun 28  2000 helpdrop.sql
	-rw-r--r--. 1 oracle oinstall 527523 May 11  2005 help.sql
	-rw-r--r--. 1 oracle oinstall  65975 Jun 29  2009 helpus.sql
	-rw-r--r--. 1 oracle oinstall   2086 Jan  6  2009 hlpbld.sql

安装help.sql：

	[oracle@oradb help]$ pwd
	/db/oracle/product/11.2.0/db_1/sqlplus/admin/help
	[oracle@oradb help]$ sqlplus system/oracle
	
	SQL*Plus: Release 11.2.0.1.0 Production on Tue Jan 22 17:15:58 2013
	
	Copyright (c) 1982, 2009, Oracle.  All rights reserved.
	
	Connected to:
	Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
	With the Partitioning, OLAP, Data Mining and Real Application Testing options
	
	SQL> @helpbld
	Enter value for 1: /db/oracle/product/11.2.0/db_1/sqlplus/admin/help
	Enter value for 2: /db/oracle/product/11.2.0/db_1/sqlplus/admin/help/help.sql
	
	PL/SQL procedure successfully completed.


安装后，查看help：

	oracle@oradb help]$ sqlplus  /nolog
	
	SQL*Plus: Release 11.2.0.1.0 Production on Tue Jan 22 17:17:54 2013
	
	Copyright (c) 1982, 2009, Oracle.  All rights reserved.
	
	SQL> conn /as sysdba
	Connected.
	SQL> select count(1) from system.help;
	
	  COUNT(1)
	----------
	      5085
	
	SQL> help index
	 Use the HELP TOPIC command for a list of help topics.
	
	SQL> help topic
	 Help is available on the following topics:
	
	%ROWTYPE ATTRIBUTE
	%TYPE ATTRIBUTE
	/
	@
	@@
	ACCEPT
	ALLOCATE (EMBEDDED SQL)
	ALTER CLUSTER
	ALTER DATABASE
	ALTER FUNCTION
	ALTER INDEX
	ALTER PACKAGE
	ALTER PROCEDURE
	ALTER PROFILE
	ALTER RESOURCE COST
	ALTER ROLE
	ALTER ROLLBACK SEGMENT
	ALTER SEQUENCE
	ALTER SESSION
	... ...

`help create database`示例：

	SQL> help create database
	
	 CREATE DATABASE
	 ---------------
	
	 Use this command to create a database, making it available for
	 general use, with the following options:
	
	   *  to establish a maximum number of instances, data files, redo
	      log files groups, or redo log file members
	   *  to specify names and sizes of data files and redo log files
	   *  to choose a mode of use for the redo log
	   *  to specify the national and database character sets
	
	 Warning: This command prepares a database for initial use and erases
	 any data currently in the specified files. Only use this command
	 when you understand its ramifications.
	
	 CREATE DATABASE [database]
	   { CONTROLFILE REUSE
	   | LOGFILE [GROUP integer] filespec
	           [,[GROUP integer] filespec] ...
	   | MAXLOGFILES integer
	   | MAXLOGMEMBERS integer
	   | MAXLOGHISTORY integer
	   | MAXDATAFILES integer
	   | MAXINSTANCES integer
	   | {ARCHIVELOG | NOARCHIVELOG}
	   | EXCLUSIVE
	   | CHARACTER SET charset
	   | NATIONAL CHARACTER SET charset
	   | DATAFILE filespec [AUTOEXTEND {OFF | ON [NEXT integer [K | M] ]
	                       [MAXSIZE { UNLIMITED | integer [K | M]} ] } ]
	           [, filespec [AUTOEXTEND {OFF | ON [NEXT integer [K | M] ]
	                       [MAXSIZE { UNLIMITED | integer [K | M]} ] } ] ] ...} ...
	
	 For detailed information on this command, see the Oracle8 Server SQL
	 Reference.
	
	
	 CREATE DATABASE LINK
	 --------------------
	
	 Use this command to create a database link. A database link is a
	 schema object in the local database that allows you to access
	 objects on a remote database or to mount a secondary database in
	 read-only mode. The remote database can be either an Oracle or a
	 non-Oracle system.
	
	 CREATE [PUBLIC] DATABASE LINK dblink
	   [CONNECT TO user IDENTIFIED BY password]
	   [USING 'connect_string']
	
	 For detailed information on this command, see the Oracle8 Server SQL
	 Reference.


#<h1 id="reference">Reference</h1>
* SQL*Plus HELP Message 'HELP NOT ACCESSIBLE' [ID 1054547.6]]
* [Alter Oracle's SQL*Plus Help Facility](http://it.toolbox.com/blogs/database-solutions/altering-oracles-sqlplus-help-facility-7697)
* [Oracle之常用FAQ V1.0](http://www.itpub.net/thread-180363-1-1.html)
* [Google help.sql](http://www.google.fr/search?hl=zh-CN&newwindow=1&safe=strict&tbo=d&biw=1440&bih=762&noj=1&q=site%3Adownload.csdn.net+help.sql&btnG=Google+%E6%90%9C%E7%B4%A2)