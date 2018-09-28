---
layout: post
title: Mysql 触发器和存储过程
date: 2018-9-28
categories: blog
tags: [mysql]
description: 文章金句。
---

# 1、首先建表： 
create table tababin( 
id int not null auto_increment, 
name varchar(100), 
constraint pk primary key(id) 
) 

# 2、拷贝一张相同的表： 
create table tababin1 like tababin; 

# 3.建立主键自增触发器： 
create trigger triabin before insert on tababin for each ROW 
begin 
set @new=new.id; 
end 

# 4、插入记录： 
insert into tababin (name) values ('abin1') 
insert into tababin (name) values ('abin2') 
insert into tababin (name) values ('abin3') 

# 5‘编写存储过程(带游标和LOOP循环的存储过程)： 
CREATE  PROCEDURE pabin() 
begin 
declare id,status int ; 
declare name varchar(100); 
declare mycur cursor for select * from tababin; 
declare continue handler for not found set status=1; 
open mycur; 
set status=0; 
loopLabel:loop 
fetch mycur into id,name; 
if status=0 then 
if id is not null then 
if name is not null then 
insert into tababin1 values (id,name); 
end if; 
end if; 
end if; 
if status =1 then 
leave loopLabel; 
end if; 
end loop; 
close mycur; 
end 

6、测试存储过程： 
call pabin() 


结果：tababin1表里面新增了数据。











