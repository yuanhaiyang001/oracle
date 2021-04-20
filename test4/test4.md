# 实验4：对象管理

### 实验目的：

了解Oracle表和视图的概念，学习使用SQL语句Create Table创建表，学习Select语句插入，修改，删除以及查询数据，学习使用SQL语句创建视图，学习部分存储过程和触发器的使用。

### 实验场景：

假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。

## 实验内容：

### 录入数据：

要求至少有1万个订单，每个订单至少有4个详单。至少有两个部门，每个部门至少有1个员工，其中只有一个人没有领导，一个领导至少有一个下属，并且它的下属是另一个人的领导（比如A领导B，B领导C）。

![2.创建部门表](image\2.创建部门表.png)

![3.创建员工表](image\3.创建员工表.png)

![6.创建order表](image\6.创建order表.png)

![5.创建临时表](image\5.创建临时表.png)

###  序列的应用

插入ORDERS和ORDER_DETAILS 两个表的数据时，主键ORDERS.ORDER_ID, ORDER_DETAILS.ID的值必须通过序列SEQ_ORDER_ID和SEQ_ORDER_ID取得，不能手工输入一个数字。

![9.创建序列](image\9.创建序列.png)

###  触发器的应用：

维护ORDER_DETAILS的数据时（insert,delete,update）要同步更新ORDERS表订单应收货款ORDERS.Trade_Receivable的值。

![1.创建触发器](image\1.创建触发器.png)

![8.创建行级触发器](image\8.创建行级触发器.png)

![11.批量插入订单数据](image\11.批量插入订单数据.png)

###  查询数据：

##### 1.查询某个员工信息

```sql
select * from employees where employee_id=11
```

![12.查询某个员工](image\12.查询某个员工.png)

##### 2.递归查询某个员工及其所有下属，子下属员工

```sql
WITH A (EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID) AS
  (SELECT EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID
    FROM employees WHERE employee_ID = 11
    UNION ALL
  SELECT B.EMPLOYEE_ID,B.NAME,B.EMAIL,B.PHONE_NUMBER,B.HIRE_DATE,B.SALARY,B.MANAGER_ID,B.DEPARTMENT_ID
    FROM A, employees B WHERE A.EMPLOYEE_ID = B.MANAGER_ID)
SELECT * FROM A;
```

![13.递归查询某个员工及其下属](image\13.递归查询某个员工及其下属.png)

##### 3.查询订单详表，要求显示订单的客户名称和客户电话，产品类型用汉字描述。

```sql
select orders.customer_name,orders.customer_tel,product_type from (
select products.product_type,order_id from order_details left join products on order_details.product_name=products.product_name)a
left join orders on orders.order_id=a.order_id
```

![14.查询订单详情表](image\14.查询订单详情表.png)

##### 4.查询出所有空订单，即没有订单详单的订单。

```sql
select orders.order_id from orders where order_id not in(
select order_id from order_details)
```

![15.空订单](image\15.空订单.png)

##### 5.查询部门表，统计每个部门的销售总金额。

```sql
select department_name,count(price) from departments left join(
select employees.department_id,order_id,price from employees left join(
select orders.employee_id, orders.order_id,price from orders left join( 
select order_id, product_num*product_price as price from order_details)a on a.order_id=orders.order_id)b on b.employee_id=employees.employee_id)c on c.department_id=departments.department_id
group by departments.department_name
```

![16.部门销售额](image\16.部门销售额.png)

### 实验总结

本次实验的主要内容是：创建表，触发器，创建和使用序列，序列在插入大量数据时可以对主键或者有序列号要求的赋值，在做实验时，做到插入大量数据时报错，由于时间问题就暂时搁置了这个实验，后面再做的时候发现是之前的数据插入有错误，然后就是之前创建的序列不知道什么原因没了，这个问题一直困扰着我。后面的就是查询了，在做部门销售总金额的查询时要连接4张表，就这个比较难，其他的没什么问题。本次实验收获很大，意识到了自己的不足之处和需要多加联系的地方。