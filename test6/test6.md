# 基于Oracle的```购物商城```数据库设计 

- 期末考核要求

  - 自行设计一个信息系统的数据库项目，自拟```某项目```名称。
  - 设计项目涉及的表及表空间使用方案。至少5张表和5万条数据，两个表空间。
  - 设计权限及用户分配方案。至少两类角色，两个用户。
  - 在数据库中建立一个程序包，在包中用PL/SQL语言设计一些存储过程和函数，实现比较复杂的业务逻辑，用模拟数据进行执行计划分析。
  
  - 设计自动备份方案或则手工备份方案。
  - 设计容灾方案。使用两台主机，通过DataGuard实现数据库整体的异地备份(可选)。

## 实验注意事项，完成时间： 2021-6-15日前上交

- 实验在自己的计算机上完成。
- 文档`必须提交`到你的Oracle项目中的test6目录中。文档必须上传两套，因此你的test6目录中必须至少有两个文件：
  - 一个是Markdown格式的，文件名称是test6.md。
  - 另一个是Word格式的，文件名称是test6_design.docx。参见[test6_design.docx](./test6_design.docx)，文件不用打印。
- 上交后，通过这个地址应该可以打开你的源码：https://github.com/你的用户名/oracle/tree/master/test6
- 文档中所有设计和数据都必须是你自己独立完成的真实实验结果。不得抄袭，杜撰。

## 评分标准

| 评分项     | 评分标准                     | 满分 |
| :--------- | :--------------------------- | :--- |
| 文档整体   | 文档内容详实、规范，美观大方 | 10   |
| 表设计     | 表，表空间设计合理，数据合理 | 20   |
| 用户管理   | 权限及用户分配方案设计正确   | 20   |
| PL/SQL设计 | 存储过程和函数设计正确       | 30   |
| 备份方案   | 备份方案设计正确             | 20   |

sqlplus sys/123@202.115.82.8/pdborcl as sysdba

## 1、建表

```mysql
-- 商家表
create table "business" (
  "business_id" NUMBER(6,0), 
	"bus_name" VARCHAR2(10 BYTE), 
	"bus_gender" VARCHAR2(4 BYTE), 
  "bus_age" NUMBER(2,0),
  CONSTRAINT ORDERS_PK PRIMARY KEY (business_id)
) SEGMENT CREATION IMMEDIATE 	
PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
NOCOMPRESS NOLOGGING
STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
TABLESPACE "GWH_02";
```

![image-20210616222614365](E:\School\大三\下期\oracle\课件\期末\img\image-20210616222614365.png)



```mysql
-- 商品表
create table "product" (
  "product_id" NUMBER(6,0), 
  "product_name" VARCHAR2(40 BYTE),
  "product_content" VARCHAR2(255 BYTE),
	"start_date" DATE, 
	"business_id" NUMBER(6,0)
) SEGMENT CREATION IMMEDIATE 	
PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
NOCOMPRESS NOLOGGING
STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
TABLESPACE "GWH_02";

-- 商品详情表
create table "product_details" (
  "pro_detail_id" NUMBER(6,0), 
  "pro_type" VARCHAR2(40 BYTE),
  "discount" NUMBER(8,2),
  "number" NUMBER(8,0),
  "price" NUMBER(8,0),
	"start_date" DATE, 
	"product_id" NUMBER(6,0)
  ) SEGMENT CREATION IMMEDIATE 	
PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
NOCOMPRESS NOLOGGING
STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
TABLESPACE "GWH_02";

-- 数据插入
--create sequence user_sequence
--    minvalue 1
--    maxvalue 30000
--    increment by 1
--    start with 1
--    nocache;
declare
    dt date;
    bus_id number;
    pro_name varchar2(40);
    pro_content varchar2(40);
    p number;
    pro_type varchar2(40);
    discount number;
    num number;
begin
--  delete from order_details;
--  delete from orders;
--  isbn := 1;

  for i in 1..10000
  loop
    if i mod 6 =0 then
      dt:=to_date('2015-3-2','yyyy-mm-dd')+(i mod 60);
    elsif i mod 6 =1 then
      dt:=to_date('2016-3-2','yyyy-mm-dd')+(i mod 60);
    elsif i mod 6 =2 then
      dt:=to_date('2017-3-2','yyyy-mm-dd')+(i mod 60);
    elsif i mod 6 =3 then
      dt:=to_date('2018-3-2','yyyy-mm-dd')+(i mod 60);
    elsif i mod 6 =4 then
      dt:=to_date('2019-3-2','yyyy-mm-dd')+(i mod 60);
    else
      dt:=to_date('2020-3-2','yyyy-mm-dd')+(i mod 60);
    end if;
    
    -- 商品归属商家id
    bus_id:=CASE i MOD 6 
    WHEN 0 THEN 1 
    WHEN 1 THEN 2 
    WHEN 2 THEN 3
    WHEN 3 THEN 4 
    WHEN 4 THEN 5 END;

--    select isbn into isbn from (select * from book order by dbms_random.value) where rownum <2;
    
    pro_name := CASE i MOD 6 
    WHEN 0 THEN concat('电脑_', i)
    WHEN 1 THEN concat('手机_', i)
    WHEN 2 THEN concat('冰箱_', i)
    WHEN 3 THEN concat('打火机_', i)
    WHEN 4 THEN concat('耳机_', i)
    else concat('电视_', i) END;
    
    pro_content := concat('这是', pro_name);
    
    insert into PRODUCT(PRODUCT_ID, PRODUCT_NAME, PRODUCT_CONTENT, START_DATE, business_id)
      values (i, pro_name, pro_content, dt, bus_id);

--    插入一个商品包括3个商品种类
    for j in 1..3
    loop
        p:=dbms_random.value(10000,4000);
        pro_type := pro_name|| '_' ||(j mod 3 + 1);
        discount := dbms_random.value(0,1);
        num := dbms_random.value(100,1000);
        INSERT INTO PRODUCT_DETAILS(PRO_DETAIL_ID, PRO_TYPE ,DISCOUNT, PRO_NUMBER ,PRICE, START_DATE, PRODUCT_ID)
          values (user_sequence.nextval, pro_type, discount, num, p, dt, i);
    end loop;

    IF i MOD 1000 =0 THEN
      commit; --每次提交会加快插入数据的速度
    END IF;
  end loop;
end;
/

select count(*) as "商品" from product;
select count(*) as "商品种类" from product_details;
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210616221025440.png" alt="image-20210616221025440" style="zoom:67%;" />

![image-20210616221055144](E:\School\大三\下期\oracle\课件\期末\img\image-20210616221055144.png)

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210616221315287.png" alt="image-20210616221315287" style="zoom:67%;" />

```mysql
-- 用户表
create table "consumers" (
  "consumer_id" NUMBER(6,0), 
	"consumer_name" varchar2(40 byte), 
  "consumer_gender" varchar2(2 byte), 
  "consumer_age" NUMBER(6,0),
  "consumer_tel" varchar2(11 byte),
) SEGMENT CREATION IMMEDIATE 	
PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
NOCOMPRESS NOLOGGING
STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
TABLESPACE "GWH_02";


-- 订单表
create table "pro_order" (
  "order_id" NUMBER(6,0), 
	"create_date" DATE, 
	"states" VARCHAR2(10 BYTE), 
	"customer_address" VARCHAR2(40 BYTE), 
  "customer_name" VARCHAR2(20 BYTE), 
  "customer_tel" VARCHAR2(20 BYTE), 
	"pro_detail_id" NUMBER(6,0),
  "user_id" NUMBER(6,0)
) SEGMENT CREATION IMMEDIATE 	
PCTFREE 10 PCTUSED 40 INITRANS 1 MAXTRANS 255 
NOCOMPRESS NOLOGGING
STORAGE(INITIAL 65536 NEXT 1048576 MINEXTENTS 1 MAXEXTENTS 2147483645
PCTINCREASE 0 FREELISTS 1 FREELIST GROUPS 1
BUFFER_POOL DEFAULT FLASH_CACHE DEFAULT CELL_FLASH_CACHE DEFAULT)
TABLESPACE "GWH_02";

-- 数据插入
declare
    create_date date;
    s varchar2(10);
    address varchar2(40);
    m varchar2(40);
    tel varchar2(40);
    detail_id number;
    consumer_id number;
    consumer_tel varchar2(40);
    consumer_name varchar2(40);
begin
  for i in 1..10000
  loop
    -- 商品归属商家id
    s:=CASE i MOD 6 
    WHEN 0 THEN '已付款'
    WHEN 1 THEN '已发货'
    WHEN 2 THEN '已收货'
    WHEN 3 THEN '已退货' 
    WHEN 4 THEN '待付款' END;

    select pro_detail_id,start_date into detail_id,create_date from (select * from PRODUCT_DETAILS order by dbms_random.value) where rownum <2;
    select consumer_id,consumer_tel,consumer_name into consumer_id,consumer_tel,consumer_name from (select * from consumers order by dbms_random.value) where rownum <2;
    
    address := CASE i MOD 6 
    WHEN 0 THEN concat('四川省乐山市', i)
    WHEN 1 THEN concat('四川省成都市', i)
    WHEN 2 THEN concat('重庆市', i)
    WHEN 3 THEN concat('北京市', i)
    WHEN 4 THEN concat('河南市', i)
    else concat('老挝', i) END;
    
    insert into PRO_ORDER(order_id, create_date, states, customer_address, customer_name, customer_tel, pro_detail_id, user_id)
      values (i, create_date, s, address, consumer_name, consumer_tel, detail_id, consumer_id);

    IF i MOD 1000 =0 THEN
      commit; --每次提交会加快插入数据的速度
    END IF;
  end loop;
end;
/
```

![image-20210616222740656](E:\School\大三\下期\oracle\课件\期末\img\image-20210616222740656.png)

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210616223905091.png" alt="image-20210616223905091" style="zoom:67%;" />

1万条订单表数据

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210616223957935.png" alt="image-20210616223957935" style="zoom:67%;" />



## 2、角色用户创建

1、表空间建立

```mysql
create tablespace gwh_01
datafile './gwh_01.dbf ' size 50M

create tablespace gwh_02
datafile './gwh_02.dbf ' size 50M
```

![image-20210605124132734](E:\School\大三\下期\oracle\课件\期末\img\image-20210605124132734.png)



2、创建con_temp_view 角色，只拥有连接查看数据库的权限。

![image-20210605123214388](E:\School\大三\下期\oracle\课件\期末\img\image-20210605123214388.png)

![image-20210605124727506](E:\School\大三\下期\oracle\课件\期末\img\image-20210605124727506.png)

![image-20210605125004351](E:\School\大三\下期\oracle\课件\期末\img\image-20210605125004351.png)



3、创建管理员角色，拥有创建表、视图、存储过程、存储函数等权限，将表空间gwh_02赋予角色

![image-20210605125253385](E:\School\大三\下期\oracle\课件\期末\img\image-20210605125253385.png)



## 3、PL/SQL  存储过程和函数

创建包Exam。

```mysql
create or replace PACKAGE Exam IS
  PROCEDURE bus_pros;
  function hot RETURN SYS_REFCURSOR;
  FUNCTION recommend RETURN SYS_REFCURSOR;
  PROCEDURE Expenditure_ranking(startDate date,endDate date);
  FUNCTION popular_product(startDate date,endDate date) 
  RETURN SYS_REFCURSOR;
END Exam;
/
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210609233853549.png" alt="image-20210609233853549" style="zoom: 50%;" />



1、商家拥有总商品数排名，过程。

计算每个商家拥有的各种的商品总数，进行从大到小的排名。

```mysql
create or replace PACKAGE BODY Exam IS
	PROCEDURE bus_pros
AS
    LEFTSPACE VARCHAR(200);
    begin
      LEFTSPACE:=' ';
      DBMS_OUTPUT.PUT_LINE('商家  商品总数');
      DBMS_OUTPUT.PUT_LINE('=======================');
      for v in
      (select b.bus_name,count(*) as num from business b left join
      product p on p.business_id = b.business_id
      left join product_details pd on p.product_id = pd.product_id
      group by b.bus_name order by count(*) desc)
      LOOP
        DBMS_OUTPUT.PUT_LINE(LPAD(LEFTSPACE,v.bus_name||' '||v.num));
      END LOOP;
    END;
END Exam;
/
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210610002012870.png" alt="image-20210610002012870" style="zoom:50%;" />



2、最热门商品 、最推荐商品展示，函数

根据订单表数据，统计用户下单购买的商品（status不为3、4，表示用户已付款购买），展示为最热门的商品，对用户收藏最多的商品（status为0），展示为最推荐的商品。

```mysql
create or replace PACKAGE BODY Exam IS
  FUNCTION hot
  RETURN SYS_REFCURSOR is
    type_cur SYS_REFCURSOR;
    BEGIN
    	open type_cur for select p.product_name,pd.pro_type,o."次数" from
    	(select pro_detail_id,count(*) as "次数" from pro_order 
       where status != '已退款' and status != '未付款' group by pro_detail_id
    	order by count(*) desc ) o left join product_details pd
    	on pd.pro_detail_id = o.pro_detail_id
    	left join product p on p.product_id = pd.product_id
        where rownum < 2;
    	
    	return type_cur;
    END;
 
  FUNCTION recommend
  RETURN SYS_REFCURSOR is
    type_cur SYS_REFCURSOR;
  AS
    N NUMBER(20,2); 
    BEGIN
    	open type_cur for select p.product_name,pd.pro_type,o."次数" from
    	(select pro_detail_id,count(*) as "次数" from pro_order 
       where status = '待付款' group by pro_detail_id
    	order by count(*) desc ) o left join product_details pd
    	on pd.pro_detail_id = o.pro_detail_id
    	left join product p on p.product_id = pd.product_id
        where rownum < 2;
    	
    	return type_cur;
    END;
END Exam;
/
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210610002225256.png" alt="image-20210610002225256" style="zoom:50%;" />

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210610002303928.png" alt="image-20210610002303928" style="zoom:50%;" />



3、一段时间内用户购买商品花销排行，过程

创建存储过程，在存储过程中使用游标，根据传入参数startDate和endDate，开始和结束时间。统计每个用户在该段时间内购买商品的花销，进行排名。信息用dbms_output包中的put_line函数输出。

```mysql
create or replace PACKAGE BODY Exam IS
	PROCEDURE Expenditure_ranking(startDate date,endDate date)
  AS
    LEFTSPACE VARCHAR(2000);
    begin
      LEFTSPACE:=' ';
      DBMS_OUTPUT.PUT_LINE(' 起始时间：' || startDate || ',结束时间' || endDate);
      DBMS_OUTPUT.PUT_LINE('用户   花费金额');
      DECLARE cursor cur_info is
      select consumer_name,n.total from consumers u left join
      (select user_id,count(price*discount) as total from pro_order po left join
      product_details pd on po.pro_detail_id = pd.pro_detail_id
      where po.create_date >= startDate and po.create_date <= endDate
      group by user_id) n
      on u.consumer_id = n.user_id;

			v_info cur_info%ROWTYPE;
      begin open cur_info;
      loop
        fetch cur_info into v_info;
    	exit when cur_info %notfound;
    	dbms_output.put_line(LPAD(' ',(v_info.user_name)||' '||v_info.total));
      end loop;
      close cur_info;
      end;
    END;
END Exam;
/
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210610002909276.png" alt="image-20210610002909276" style="zoom:50%;" />



4、一段时间内最受欢迎的商品，函数

创建存储函数，查询最受欢迎的商品，根据传入参数startDate和endDate，开始和结束时间。统计并输出在该段时间内被购买次数最最多的商品及其被购买次数。

```mysql
create or replace PACKAGE BODY Exam IS
  FUNCTION popular_product(startDate date,endDate date) 
  RETURN SYS_REFCURSOR is
    type_cur SYS_REFCURSOR;
  AS
    N NUMBER(20,2); 
    BEGIN
    	select p.product_name,pd.pro_type,o."次数" from
    	(select pro_detail_id,count(*) as "次数" from pro_order 
       where status != '已退款' and status != '未付款' 
       and create_date >= startDate and create_date <= endDate
       group by pro_detail_id
    	order by count(*) desc ) o left join product_details pd
    	on pd.pro_detail_id = o.pro_detail_id
    	left join product p on p.product_id = pd.product_id
        where rownum < 2;
    	
    	return type_cur;
    END;
END Exam;
/
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210610003052174.png" alt="image-	" style="zoom:50%;" />



## 4、设计自动备份方案或则手工备份方案

1、从pdborcl创建可插接数据库

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210611131633326.png" alt="image-20210611131633326" style="zoom:50%;" />

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210611131655039.png" alt="image-20210611131655039" style="zoom:50%;" />



```

CREATE PLUGGABLE DATABASE gwh FROM pdborcl file_name_convert=(
'/home/oracle/app/oracle/oradata/orcl/pdborcl/temp01.dbf','/home/student/pdb/db3/temp01.dbf',
'/home/oracle/app/oracle/oradata/orcl/pdborcl/system01.dbf','/home/student/pdb/db3/system01.dbf',
'/home/oracle/app/oracle/oradata/orcl/pdborcl/sysaux01.dbf','/home/student/pdb/db3/sysaux01.dbf',
'/home/oracle/app/oracle/product/12.2.0/dbhome_1/dbs/gwh_02.dbf ','/home/student/pdb_gwh/gwh_02.dbf '
);
```

2、查询需要备份的文件

```mysql
SELECT NAME FROM v$datafile
UNION ALL
SELECT MEMBER AS NAME FROM v$logfile
UNION ALL
SELECT NAME FROM v$controlfile;
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210610103329790.png" alt="image-20210610103329790" style="zoom:50%;" />

3、查询自己创建的表空间的大小

```mysql
SELECT SUM(bytes) / (1024 * 1024) AS free_space, tablespace_name FROM dba_free_space WHERE tablespace_name='GWH_01' GROUP BY tablespace_name;

SELECT SUM(bytes) / (1024 * 1024) AS free_space, tablespace_name  FROM dba_free_space WHERE tablespace_name='GWH_02' GROUP BY tablespace_name;
```

![image-20210610105453012](E:\School\大三\下期\oracle\课件\期末\img\image-20210610105453012.png)



4、手工文件备份

**全库0级备份(只作一次)**

```
run{
configure retention policy to redundancy 1;
configure controlfile autobackup on;
configure controlfile autobackup format for device type disk to '/home/student/rman_backup/%F';
configure default device type to disk;
crosscheck backup;
crosscheck archivelog all;
allocate channel c1 device type disk;
allocate channel c2 device type disk;
allocate channel c3 device type disk;
backup incremental level 0 database format '/home/student/rman_backup/level0_%d_%T_%U.bak';
report obsolete;
delete noprompt obsolete;
delete noprompt expired backup;
delete noprompt expired archivelog all;
release channel c1;
release channel c2;
release channel c3;
}
```

**全库1级增量备份**

```
run{
configure retention policy to redundancy 1;
configure controlfile autobackup on;
configure controlfile autobackup format for device type disk to '/home/student/rman_backup/%F';
configure default device type to disk;
crosscheck backup;
crosscheck archivelog all;
allocate channel c1 device type disk;
allocate channel c2 device type disk;
allocate channel c3 device type disk;
backup incremental level 1 database format '/home/student/rman_backup/level1_%d_%T_%U.bak';
report obsolete;
delete noprompt obsolete;
delete noprompt expired backup;
delete noprompt expired archivelog all;
release channel c1;
release channel c2;
release channel c3;
}
```



5、修改数据

查看数据文件

```
select file_name from dba_data_files;
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210611130149759.png" alt="image-20210611130149759" style="zoom:50%;" />

查看表数据

```
select * from gwh_user.consumers;

select to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') as currentdate from dual;
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210614152359654.png" alt="image-20210614152359654" style="zoom: 67%;" />

![image-20210614152427523](E:\School\大三\下期\oracle\课件\期末\img\image-20210614152427523.png)



更新数据

```
update gwh_user.consumers set consumer_age=consumer_age+1 where consumer_id = 1;
```

<img src="E:\School\大三\下期\oracle\课件\期末\img\image-20210611132732657.png" alt="image-20210611132732657" style="zoom:67%;" />



![image-20210614152701293](E:\School\大三\下期\oracle\课件\期末\img\image-20210614152701293.png)

修改文件名，模拟文件损失

```
$mv -f /home/student/pdb_gwh/gwh_02.dbf  /home/student/pdb_gwh/gwh_02_1.dbf2
```



**选项1：单库完全恢复**

```
$rman target sys/123@202.115.82.8/orcl:dedicated
RMAN> restore pluggable database gwh;
RMAN> recover pluggable database gwh;
RMAN> alter pluggable database gwh open;
RMAN> exit;

- 完全恢复成功后，hr用户登录gwh，
$ sqlplus system/123@202.115.82.8/gwh
SQL> select * from consumers;

可见，完全恢复成功，数据是最新的（即2021-04-27 08:03:01），无损失。
```

![image-20210614153623866](E:\School\大三\下期\oracle\课件\期末\img\image-20210614153623866.png)

**选项2：单库不完全恢复,恢复到update语句之前的状态**

```
$rman target sys/123@202.115.82.8/orcl:dedicated
RMAN> restore pluggable database gwh;
RMAN> recover pluggable database gwh until time "to_date('2021-04-27 08:02:24','yyyy-mm-dd hh24:mi:ss')" AUXILIARY DESTINATION '/home/student/zwd';
正在开始介质的恢复
线程 1 序列 1624 的归档日志已作为文件 /home/oracle/app/oracle/product/12.2.0/dbhome_1/dbs/arch1_1624_1064951903.dbf 存在于磁盘上
线程 1 序列 1625 的归档日志已作为文件 /home/oracle/app/oracle/product/12.2.0/dbhome_1/dbs/arch1_1625_1064951903.dbf 存在于磁盘上
RMAN> alter pluggable database ly open resetlogs;
已处理语句
RMAN>exit

- 不完全恢复成功后，hr用户登录gwh，
$ sqlplus hr/123@202.115.82.8/gwh
SQL> select * from consumers;

可见，不完全恢复成功，数据回到了修改前的状态（即：2021-04-27 08:02:24）。
```

![image-20210614154125944](E:\School\大三\下期\oracle\课件\期末\img\image-20210614154125944.png)

