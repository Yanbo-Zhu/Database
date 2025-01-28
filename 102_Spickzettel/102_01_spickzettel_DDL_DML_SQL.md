
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

## 1.1 DML 



# 2 Datenintegrität

## 2.1 Check 

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



