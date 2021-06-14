

# 图书信息管理系统数据库设计

## 表设计：

#### 用户表：

|   字段名    | 数据类型 | 是否允许NULL |   备注   |
| :---------: | :------: | :----------: | :------: |
|   user_id   |  number  |   not null   |   主键   |
|  username   | varchar  |   not null   |  用户名  |
|   passwd    | varchar  |   not null   |   密码   |
|    uname    | varchar  |   not null   |   姓名   |
|     sex     | varchar  |   not null   |   性别   |
| book_quota  |  number  |   not null   | 借书限额 |
| overdue_num |  number  |   not null   | 逾期次数 |

```sql
CREATE TABLE users(
  user_id number NOT NULL,
  username varchar(20) NOT NULL,
  passwd varchar(20) NOT NULL,
  uname varchar(20) NOT NULL,
  sex varchar(5) NOT NULL,
  book_quota number NOT NULL,
  overdue_num number NOT NULL,
  PRIMARY KEY (user_id)
);
```



#### 管理员表：

|   字段名    | 数据类型 | 是否允许NULL |   备注   |
| :---------: | :------: | :----------: | :------: |
|  admin_id   |  number  |   not null   |   主键   |
|  username   | varchar  |   not null   |  用户名  |
|   passwd    | varchar  |   not null   |   密码   |
|    uname    | varchar  |   not null   |   姓名   |
|     sex     | varchar  |   not null   |   性别   |
| book_quota  |  number  |   not null   | 借书限额 |
| overdue_num |  number  |   not null   | 逾期次数 |

```sql
CREATE TABLE admin  (
  admin_id number NOT NULL,
  username varchar(16) NOT NULL,
  passwd varchar(16) NOT NULL,
  uname varchar(16) NOT NULL,
  sex varchar(5) NOT NULL,
  book_quota number NOT NULL,
  overdue_num number NOT NULL,
  PRIMARY KEY (admin_id)
);
```



#### 图书表：

|      字段名      | 数据类型 | 是否允许NULL |    备注    |
| :--------------: | :------: | :----------: | :--------: |
|     book_id      |  number  |   not null   |    主键    |
|       ISBN       | varchar  |   not null   | 书籍ISBN号 |
|    book_name     | varchar  |   not null   |    书名    |
|      author      | varchar  |   not null   |    作者    |
| publishing_house | varchar  |   not null   |   出版社   |
|     surplus      |  number  |   not null   |  可借余量  |

```sql
CREATE TABLE book(
  book_id number NOT NULL,
  ISBN varchar(20) UNIQUE NOT NULL,
  book_name varchar(50) NOT NULL,
  author varchar(50) NOT NULL,
  publishing_house varchar(50) NOT NULL,
  surplus number NOT NULL,
  PRIMARY KEY (book_id)
);
```



#### 借书记录表：

|         字段名         | 数据类型 | 是否允许NULL |        备注        |
| :--------------------: | :------: | :----------: | :----------------: |
| borrow_books_record_id |  number  |   not null   | 借书记录id（主键） |
|        user_id         |  number  |   not null   |   用户id（外键）   |
|          ISBN          | varchar  |   not null   | 书籍ISBN号（外键） |
|       lend_time        |   date   |   not null   |      借出日期      |
|       lend_days        |  number  |   not null   |      借出天数      |

```sql
CREATE TABLE borrow_record(
  borrow_books_record_id number NOT NULL,
  user_id number NOT NULL,
  ISBN varchar(50) NOT NULL,
  lend_time date NOT NULL,
  lend_days number NOT NULL,
  PRIMARY KEY (borrow_books_record_id),
  CONSTRAINT user_id FOREIGN KEY (user_id) REFERENCES users (user_id),
  CONSTRAINT ISBN FOREIGN KEY (ISBN) REFERENCES book (ISBN)
);
```



#### 还书记录表：

|         字段名         | 数据类型 | 是否允许NULL |        备注        |
| :--------------------: | :------: | :----------: | :----------------: |
|           id           |  number  |   not null   |        主键        |
| borrow_books_record_id |  number  |   not null   | 借书记录id（外键） |
|      return_time       | varchar  |   not null   |      归还日期      |
|       is_overdue       | varchar  |   not null   |      是否逾期      |

```sql
CREATE TABLE return_record(
  id number NOT NULL,
  borrow_books_record_id number NOT NULL,
  return_time date NOT NULL,
  is_overdue varchar(5) NOT NULL,
  PRIMARY KEY (id),
  CONSTRAINT borrow_books_record_id FOREIGN KEY (borrow_books_record_id) REFERENCES borrow_record (borrow_books_record_id)
)
```

## 索引设计：

```sql
-- 创建普通用户id序列
create sequence seq_newUserids increment by 1 start with 1 maxvalue 999999999;
-- 创建管理员id序列
create sequence seq_newAdminids increment by 1 start with 1 maxvalue 999999999;
-- 创建图书id序列
create sequence seq_newBookids increment by 1 start with 1 maxvalue 999999999;
-- 创建图书ISBN序列
create sequence seq_newISBNs increment by 1 start with 10000000 maxvalue 999999999;

-- 创建借书记录id序列
create sequence seq_newBorrow_books_recordids increment by 1 start with 1 maxvalue 999999999;
-- 创建还书记录id序列
create sequence seq_newids increment by 1 start with 1 maxvalue 999999999;
```

## 存储过程：

插入users表数据

```sql
create or replace
PROCEDURE insertdata as
flag number;
begin
    flag:=0;
    for i in 1..10000
        loop
            insert into users(user_id,username,passwd,uname,sex,book_quota,overdue_num)
            values(seq_newUserids.nextval,'testuser','000','张三','男',5,0);
            flag:=flag+1;
            if flag=10001 then 
                commit;
            end if;
        end loop;
end;
```

插入admin表数据

```sql
create or replace
PROCEDURE insertadmindata as
flag number;
begin
    flag:=0;
    for i in 1..10000
        loop
            insert into admin(admin_id,username,passwd,uname,sex,book_quota,overdue_num)
            values(seq_newAdminids.nextval,'testuser','111','李四','女',10,0);
            flag:=flag+1;
            commit;
        end loop;
end;
```

插入book表数据

```sql
create or replace
PROCEDURE insertbookdata as
flag number;
begin
    flag:=0;
    for i in 1..10000
        loop
            insert into book(book_id,isbn,book_name,author,publishing_house,surplus)
            values(SEQ_NEWBOOKIDS.nextval,to_char(SEQ_NEWISBNS.nextval),'西游记','吴承恩','新华出版社','5');
            flag:=flag+1;
            commit;
        end loop;
end;
```

插入borrow_record表数据

```sql
create or replace
PROCEDURE insertlendbookdata as
flag number;
begin
    flag:=0;
    for i in 1..10000
        loop
            insert into borrow_record(borrow_books_record_id,user_id,isbn,lend_time,LEND_DAYS)
            values(SEQ_NEWBORROW_BOOKS_RECORDIDS.nextval,getuserid(),getisbn(),SYSDATE(),5);
            flag:=flag+1;
            commit;
        end loop;
end;
```

插入return_record表数据

```sql
create or replace
PROCEDURE insertreturnbookdata as
flag number;
begin
    flag:=0;
    for i in 1..10000
        loop
            insert into return_record(id,borrow_books_record_id,return_time,is_overdue)
            values(SEQ_NEWIDS.nextval,getborrowid(),SYSDATE(),'否');
            flag:=flag+1;
            if flag=10001 then 
                commit;
            end if;
        end loop;
end;
```

## 函数：

随机获得一个用户id

```sql
create or replace function getUserid
return number
is
user_id number;
begin
    select user_id into User_id from (select * from users order by dbms_random.value) where rownum< 2;
    return user_id;
end;
```

随机获得一个ISBN

```sql
create or replace function getIsbn
return varchar2
as
isbn varchar2(20 byte);
begin
    select isbn into isbn from (select * from book order by dbms_random.value) where rownum <2;
    return isbn;
end;
```

随机获得一个借书记录id

```sql
create or replace function getborrowid
return number
as
borrowid number;
begin
    select borrow_books_record_id into borrowid from (select * from borrow_record order by dbms_random.value) where rownum <2;
    return borrowid;
end;
```

