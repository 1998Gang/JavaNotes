* 检索出数据不同的行（排除重复的）`select distinct`
* 检索出指定的行数：`select * from tablename limit 5`  给出检索出的数据的前5行
* 指定开始位置，以及所需行数：`select * from tablename limit 3,5`从符合条件的数据的第3行开始，列出后面5行。第4、5、6、7、8行数据。
* 排序数据 `select * from students order by studen_id` 查询所有学生，按学号升序排序。如果要降序，在`student_id`后面加` DESC`就行。
* 通配符 `select * from students where name like '彭%' `查询所有姓彭的同学，
* 通配符 `select * from students where name like '彭_'` 查询所有姓彭，并且名字是两个字的同学
* 拼接字段 `select concat(name,'(',student_id,')') as massage` 将字段name 和学号进行拼接。实例：彭渝刚（2017214033）,这个拼接后的字段叫 massage。

#### DISTINCT

* 作用：输入不同的数据。  
* 语法： `DISTINCT 列名` 注意，它不能作用在某一列，它的作用范围是它之后所有的列。

```SQL
//输出商品表里有那些价格
SELECT DISTINCT prod_price FROM Products;   
```

#### LIMIT

* 作用：输出结果的前多少行
* 语法：`LIMIT 4` 输出结果的前4行、`LIMIT 3,4`从结果的第3行开始，输出4行，等价于`LIMIT 4 OFFSET 3`

```sql
//输出学生表里 第4位到第7位学生
SELECT * FROM USER LIMIT 3,4;
```

#### ORDER BY

* 作用：对输出结果进行排序
* 语法：`ORDER BY 列名` 将数据结果，按某一列进行排序，默认是升序，在列名后使用`DESC`选择降序。如果想对多个列都使用降序，必须在每个列名后面加`DESC`关键字。

```sql
//输出成绩前十名的学生
SELECT * FROM students ORDER BY grades LIMIT 10;
//输出成绩后10名的学生
SELECT * FROM students ORDER BY grades DESC LIMIT 10;
```

#### WHERE

* 作用：过滤指定条件的数据
* 语法：`WHERE 条件` 
* 相关操作符：
  * __=__                         等于
  * __< > 、!=__             不等于
  * __BETWEEN __       介于两个值之间
  * __IS NULL__            范围某一字段为空的数据。



--------

###  MySQL函数

#### Trim(列名)

* 作用：去掉该列数据的左右空格



#### UPPER(列名)

* 作用：将文本转换为大写



#### DATE(代表日期的列名)

* 作用：对于时间的匹配操作，更加合理

```sql
//查询2005年9月所有订单
SELECT * FROM orders WHERE DATE(order_date)='2005-09';
//等价于
SELECT * FROM orders WHERE YEAR(order_date)='2005' AND MONTH(order_date) ='09';
```



--------

### 汇总数据

#### GRUOP BY

* 作用：按某一列进行分组。	

```sql
//查询各个供应商供应了多少物品,按供应商id进行分组
select count(*),vend_id from products group by vend_id; 
//查询各个供应商供应了多少物品，并且返回所有供应商供应物品的总数，WITH ROLLUP。就是返回所有分组的和数据 在最后一行。
select count(*),vend_id from products group by vend_id with rollup;

```

#### HAVING

* 作用：过滤分组数据，对数据进行分组查询后，并不需要所有的分组数据，就使用`HAVING`进行过滤。相当于普通查询的`WHERE`

```sql
//查询供应产品在3钟以上的供应商id 以及他们供应物品数量。
SELECT COUNT(*) AS num,vend_id FROM products GROUP BY vend_id HAVING num>=3;
```



-------------



### 特殊语句

```sql
//更新学生表数据，将男生该为女生  女生改为男生
update student set sex = if(sex='男'，'女','男');

update student set sex =(case sex when '男' then '女' else '男');
```

 ```sql
-- 查询最新一条数据
SELECT * FROM apple ORDER BY id DESC LIMIT 1;
-- 今天  
select * from apple where to_days(data_time) = to_days(now()); 
-- 昨天  
select * from apple where to_days(NOW()) - TO_DAYS(data_time) <= 1;  
-- 近7天  
select * from apple where date_sub(CURDATE(),INTERVAL 7 DAY) <= DATE(data_time);  
-- 近30天  
SELECT * FROM apple where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= date(data_time);
-- 本月  
SELECT * FROM apple WHERE DATE_FORMAT( data_time, '%Y%m' ) = DATE_FORMAT( CURDATE() , '%Y%m' );
-- 上一月  
SELECT * FROM apple WHERE PERIOD_DIFF( date_format( now( ) , '%Y%m' ) , date_format( data_time, '%Y%m' ) ) =1; 
-- 查询本季度数据  
select * FROM apple where QUARTER(data_time)=QUARTER(now()); 
-- 查询上季度数据  
select * FROM apple where QUARTER(data_time)=QUARTER(DATE_SUB(now(),interval 1 QUARTER));  
-- 查询本年数据  
select * FROM apple where YEAR(data_time)=YEAR(NOW());  
 ```



