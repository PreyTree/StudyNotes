# MySQL基础

## DQL-执行顺序

> - SELECT 字段列表 5
> - FROM 表名列表 1
> - WHERE 条件列表 2
> - GROUP BY 分组列表 3
> - HAVING 分组后条件列表 4
> - ORDER BY 排序字段列表 6
> - LIMIT 分页参数 7

```sql
select e.name ename,e.age eage from emp e where e.age>15 order by eage asc;
```



## 基础函数用例

### 字符串函数

|           函数           |                       功能                        |
| :----------------------: | :-----------------------------------------------: |
|    concat(s1,s2,s3,…)    |                    字符串拼接                     |
|        lower(str)        |                   字符串转大写                    |
|        upper(str)        |                   字符串转小写                    |
|     lpad(str,n,pad)      | 左填充，用pad对str左边进行填充，达到n个字符串长度 |
|     rpad(str,n,pad)      | 右填充，用pad对str左边进行填充，达到n个字符串长度 |
|        trim(str)         |                   去除头尾空格                    |
| substring(str,start,len) |    返回字符串str从start位置起len个长度的字符串    |

企业员工的工号统一为5位数，目前不为五位数的全部在前面补0，比如：1号员工应为000001

```sql
update emp set workno = lpad(workno,5,'0');
```

### 数值函数

|    函数    |                功能                |
| :--------: | :--------------------------------: |
|  ceil(x)   |              向上取整              |
|  floor(x)  |              向下取整              |
|  mod(x,y)  |            返回x/y的模             |
|   rand()   |         返回0~1内的随机数          |
| round(x,y) | 返回参数x四舍五入的值，保留y位小数 |

通过数据库的函数，生成一个六位数的随机验证码

```sql
select lpad(round(rand()*1000000,0),6,0);
```

### 日期函数

|               函数                |                       功能                        |
| :-------------------------------: | :-----------------------------------------------: |
|             curdate()             |                   返回当前日期                    |
|             curtime()             |                   返回当前时间                    |
|               now()               |                返回当前日期和时间                 |
|            year(date)             |                 获取指定date年份                  |
|            month(date)            |                 获取指定date月份                  |
|             day(date)             |                 获取指定date日期                  |
| date_add(date,INTERVAL expr type) | 返回一个日期/时间值加上一个时间间隔expr后的时间值 |
|       datediff(date1,date2)       |    返回结束时间date1和起始时间date2时间的天数     |

查询所有员工的入职天数，并根据入职天数倒序排序

```sql
select name,datediff(curdate(),entrydate) as 'entrydays' from emp order by entrydays desc;
```

### 流程函数

|                          函数                          |                         功能                         |
| :----------------------------------------------------: | :--------------------------------------------------: |
|                     if(value,t,f)                      |          如果value为true,则返回t，否则返回f          |
|                 ifnull(value1,value2)                  |     如果value1不为空，返回value1,否则返回value2      |
|    case when [val1] then [res1] …else[default] end     |    如果val1为true,返回res1,…否则返回default默认值    |
| case [expr] when [val1] then [res1] …else[default] end | 如果expr的值等于val1,返回res1,…否则返回default默认值 |

统计各个学员成绩

```sql
select
	id,
	name,
	(case when math >=85 then '优秀' when math >=60 then '及格' else '不及格' end) math,
from score;
```

查询emp表的员工姓名和工作地址(北京/上海 —> 一线城市，其他—> 二线城市)

```sql
select
	name,
	(case workaddress when '北京' then '一线城市' when '上海' then '一线城市' else '二线城市' end) as '工作地址'
from emp;
```

## 约束

### 概述

| 约束                     | 描述                                                     | 关键字      |
| ------------------------ | -------------------------------------------------------- | ----------- |
| 非空约束                 | 限制该字段的数据不为null                                 | NOT NULL    |
| 唯一约束                 | 保证该字段的所有数据都是唯一，不重复                     | UNIQUE      |
| 主键约束                 | 主键是一行数据的唯一标识，要求非空且唯一                 | PRIMARY KEY |
| 默认约束                 | 保存数据时，如果未指定该字段值，则采用默认值             | DEFAULT     |
| 检查约束(8.0.16版本之后) | 保证字段值满足某一个条件                                 | CHECK       |
| 外键约束                 | 用来让两张表的数据之间建立连接，保证数据的一致性和完整性 | FOREIGN KEY |

### 外键删除跟新行为

| 行为        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| no action   | 当在父表中删除/跟新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/跟新(与restrict一致) |
| restrict    | 当在父表中删除/跟新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/跟新(与no action一致) |
| cascade     | 当在父表中删除/跟新对应记录时，首先检查该记录是否有对应外键，如果有则也删除/跟新外键在子表中的记录 |
| set null    | 当在父表中删除对应记录时，首先检查该记录是否有对应外键，如果有则设置子表中该外键值为null(这就要求该外键允许取null) |
| set default | 父表有变更时，子表将外键设置成一个默认值(Innodb不支持)       |

### 添加外键

给emp表添加名为fk_emp_dept_id的外键约束并设置外键删除跟新行为,外键字段为emp表的dept_id字段，关联字段为dept表的id字段

```sql
alter table emp add constraint fk_emp_dept_id foreign key (dept_id) references dept(id) on update cascade on delete cascade;
```

### 删除外键

```sql
alter table emp drop foreign key fk_emp_dept_id;
```

## 多表查询

### 多表关系

#### 一对多

> - 案例：部门和员工的关系
> - 关系：一个部门对应多个员工，一个员工对应多个部门
> - 实现：在多的一方建立外键，指向一的一方的主键

#### 多对多

> - 案例：学生与课程的关系
> - 关系：一个学生可以选修多门课程，一门课程也可以供多个学生选择
> - 实现：建立第三张中间表，中间表至少包含两个外键，分别关联两方主键

```sql
create table student_course(
	id int auto_increment comment '主键' primary key,
    studentid int not null comment '学生ID',
    courseid int not null comment '课程ID',
    constraint fk_courseid foreign key (courseid) references course (id),
    constraint fk_studentid foreign key (studentid) references student (id),
)comment '学生课程中间表'；
```

### sd

>- dfdf



