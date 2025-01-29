
# 1 DDL Create table 

```sql
create table mitgliedschaft
(
mitgliedsnummer int primary key,  
eintrittsdatum date not null,   default  'm', unique,  
nachname varchar(30),  
primary key (mitgliedsnummer, eintrittsdatum),  
foreign key (mitgliedsnummer) references mitglied(mitgliedsnummer)) on delete ... , 
foreign key (nachname) references xx(yy)) on update ... 
unique(usercode)
);
no action, restrict, cascade, set null, set default 
```



# 2 DDL Drop table 


Drop Table 表名字


# 3 DDL alter table

语法
alter table 表名 add|drop|modify|change column 列名 【列类型 约束】;

修改列名
ALTER TABLE book CHANGE COLUMN publishdate pubDate DATETIME;

修改列的类型或约束
ALTER TABLE book MODIFY COLUMN pubdate TIMESTAMP;

添加新列
ALTER TABLE author ADD COLUMN annual DOUBLE; 

删除列
ALTER TABLE book_author DROP COLUMN annual;

修改表名
ALTER TABLE author RENAME TO book_author;

# 4 DML 


## 4.1 insert


四种方式

1 全插入
insert into t_student(no,name,sex,classno,birth) values(1,'zhangsan','1','gaosan1ban', '1950-10-12');

2 全插入 但是顺序颠倒 -> t_student(name,sex,classno,birth,no) 中 顺序不是完全按照 表中原本的字段的顺序 
insert into t_student(name,sex,classno,birth,no) values('lisi','1','gaosan1ban', '1950-10-12',2);

3 部分插入 insert into t_student(name) values('wangwu'); // 会生成一行， 除name字段之外，剩下的所有字段自动插入NULL。

4 字段可以省略不写，但是后面的value对数量和顺序都有要求。不能少写， 不能颠倒
insert into t_student values(1,'jack','0','gaosan2ban','1986-10-23');

5 一次插入多行数据
```
insert into t_student(no,name,sex,classno,birth)
values (3,'rose','1','gaosi2ban','1952-12-14'),(4,'laotie','1','gaosi2ban','1955-12-14'); 注意中间的逗号
```


## 4.2 update

```
UPDATE boys SET boyname='张飞',usercp=10 WHERE id=2;


update emp set sal=sal+sal*0.1 where job='MANAGER';
```

## 4.3 delete 

```
delete from emp where comm=500;
```

# 5 Datenintegrität

## 5.1 Check 

 Der `CHECK`-Constraint wird mit dem Befehl **`ALTER TABLE`** oder beim Erstellen einer Tabelle mit **`CREATE TABLE`** hinzugefügt.

Beispiel für CHECK-Constraint in CREATE TABLE
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT CHECK (age >= 18),  -- Alter muss mindestens 18 sein
    salary DECIMAL(10,2) CHECK (salary >= 3000), -- Mindestgehalt 3000
    department VARCHAR(50) CHECK (department IN ('HR', 'IT', 'Finance')) -- Nur diese Werte erlaubt
);

```

CHECK-Constraint mit ALTER TABLE hinzufügen
```sql
ALTER TABLE artikelzubehoer
ADD CONSTRAINT chk_modelljahr
CHECK (modelljahr >= gruendungsjahr);
```


**`CHECK`-Constraint entfernen**
```sql
ALTER TABLE employees DROP CONSTRAINT check_salary;
```



# 6 SQL

## 6.1 Join 



Inner Join:
```
select firma,aufnr
from kdst k inner join aufkopf a
on k.kdnr = a.kdnr
order by aufnr;

select firma,aufnr
from kdst k , aufkopf a
where k.kdnr = a.kdnr
order by aufnr;
```



Left Outer Join:
```
select firma,aufnr
from kdst k 
left outer join aufkopf a
on k.kdnr = a.kdnr
order by aufnr;

select firma,aufnr
from kdst k , aufkopf a
where k.kdnr = a.kdnr (+)
order by aufnr;
```


## 6.2 Unteranfrage 

wo genau Unterabfragen in SQL-Statements vorkommen können
- Select-Closure: es liefert eine splate fur ergebnisstablle 
- From-Closure: liefert eine temporäre/ virtuelle/ abgeleitet Tabelle,
- Where-Closure: Filtern von Zeilen
- HAVING-Closure: Filtern von Zeilen

## 6.3 not in and not existes

```sql
select k.kdnr,k.firma, k.plz,k.ort
from kdst k
where k.kdnr not in
(select a.kdnr
from aufkopf a);
```

```sql
select k.kdnr,k.firma, k.plz,k.ort
from kdst k
where not exists
(select *
from aufkopf a
where k.kdnr=a.kdnr);
```



### 6.3.1 Any and all

```
select name,prov
from vert
where prov < any
(select prov
from vert
where plz = "12487");
```



```
select v.vertreter, v.name,v.prov  
from matinf.vert v  
where v.prov < all  
(select v.prov  
from matinf.vert v  
where v.plz = '12487');
```




```
select v.vertreter, v.name,v.prov  
from matinf.vert v  
where v.prov <    // 等效于 all 
(select min(v.prov)  
from matinf.vert v  
where v.plz = '12487');
```

