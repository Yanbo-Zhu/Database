

# 1 Grundlagen der abstrakten Datenmodellierung


## 1.1 Phasenmodell des DB-Entwurf

![](image/Pasted%20image%2020241119085015.png)

## 1.2 Grundlegende Elemente des ERM zur Datenmodellierung

### 1.2.1 Modellelement 

![](image/Pasted%20image%2020241027171743.png)

Entität (Entity)
Abgrenzbarer Gegenstand / Begriff / „Ding“ / Person / Ereignis in der Realsphäre.

Entitätentyp (Entity Type)
Gleichartige Entitäten werden klassifiziert. Sie werden zu einem Entitätentyp bzw. einer Klasse zusammengefasst

Beziehung (Relationship)
Eine Beziehung stellt die logische Verknüpfung von zwei oder mehr Entitäten/Objekten.

Beziehungstyp (Relationship Type)
Gleichartige Beziehungen werden klassifiziert. Sie werden zu einem Beziehungstypen zusammengefasst Anmerkung: In der UML spricht man ebenfalls von einer (Klassen-)Assoziation

Attribut (Attribute)
Attribute werden Objekten und Beziehungen als deren Eigenschaften zugeordnet und erlauben deren Charakterisierung und Identifizierung. Die Ausprägungen der Attribute sind Werte. Alle (gültigen) Werte ergeben zusammen einen Wertebereich.


### 1.2.2 Darstellung von Kardinalitäten

(1,M,N) Notation

 (Min, Max) Notation

![](image/Pasted%20image%2020241117204252.png)


z.B. (Zu einem Lieferanten kann keine oder genau eine Adresse hinterlegt sein  (0,1)
![](image/Pasted%20image%2020250127170817.png)

### 1.2.3 Existenzabhängigkeiten


![](image/Pasted%20image%2020241027183159.png)



# 2 Transformation_des_ERM_zu_RDM


## 2.1 Modellelemente von ERM und RDM 

Ausgangsschema:
ERM-Beispiel: UoD Hochschule
Modellelemente:
- Attribut
- Entitytyp
- Beziehungstyp
- Generalisierung
Für UML genauso anwendbar!

Zielschema:
RDM
Modellelemente:
- Attribut
- Relation
- Integritätsbedingung
    - Primärschlüssel
    - Fremdschlüssel

## 2.2 Transformation der grundlegenden Modellelemente des ERM ins RDM

Transformation von Entitätentypen:
    - Ein Entitätentyp und seine elementaren Attribute wet-den auf eine Relation und deren Attribute abgebildet.
### 2.2.1 Transformation von Beziehungstypen 

Ein Beziehungstyp der Kardinalität 1:1
- 不需要额外的table for Bezihung 
- FS zu beliebig Seite
![](image/Pasted%20image%2020241113110142.png)



Ein Beziehungstyp der Kardinalität 1:N oder N:1
- 不需要一个Schema table ,  
- FS 放到N seite 
![](image/Pasted%20image%2020241113111508.png)



Beziehungstyp der Kardinalität M:N
- 需要一个table,  FS von linker Seite und rechte Seite 都填到这个 新的  0 Tabelle 中 
- `(0, *) ` 的关系就是 对应的是1kardinalität M:N 
- ==切记新的Relation中是kombinierte Pirmiarschlussel, 不是单一的schlussel==

![](image/Pasted%20image%2020241113111925.png)





Ein rekursiver Beziehungstyp
需要自成一格表格 
![](image/Pasted%20image%2020250127212350.png)


Ein n-närer Beiyhungstyp
- 这种情况下的 Beziehungstyp 需要自成一个table 
- 1 lieferent liefert Artikel fuer 1 Projekt 
- 1 lieferant liefert die Menge  fur projekt 
![](image/Pasted%20image%2020241113113232.png)


### 2.2.2 Transformation von Generalisierungshierarchien ins RDM

从几个 Entitattyp 中  抽象出 一个Supertyp 
Supertyp  中的 attibute   就不需要出现在 subtyp 中了, 减少冗余 

GeschaftsPatner 中的Typ  指的是 他是 Kunde 还是 Lieferant 
![](image/Pasted%20image%2020241113114119.png)


---

Standardvariante: 分开两张表存数据 ,   两张表的数据不重复 
Horizontale Partitionierung:   Supertype , 也设置两个 type, 但是这三张表中的数据有可能重复, 有可能不重复 
Typisierte Partitionierung:  只设置Supertype 的table, 将两个 type 的数据 都存到这一张表里面 


![](image/Pasted%20image%2020241113115156.png)



## 2.3 Primärschlüssel

写入便条 
Ein Primärschlüssel ist ein Attribut, welches

- in Bezug auf den Fremdschlüssel einer anderen Relation definiert ist
- als künstlicher Primärschlüssel definiert sein kann 
- keinen Nullwert aufweisen darf 
- eindeutig ist 


除了 "in Bezug auf den Fremdschlüssel einer anderen Relation definiert ist" 其他都是正确的 

# 3 Normal Form 


Welche Ziele hat die Normalisierung?
Ziel hinsichtlich Zugriffsoperationen: → ==Optimierung schreibende Zugriffe==
Ziel hinsichtlich Redundanz: Redundanzminimierung

Ein Ziel der Normalisierung ist die Beseitigung von ==ANOMALIEN==

First Normal Form
- 数据库表中不能出现重复记录 （每一行必须唯一），每个字段是原子性 atomar 的不能再分
- Alle Werte in einer Relation sind atomar (es gibt keine Mehrfachwerte oder verschachtelte Tabellen).  atomare Attribut


Second Normal Form
- 第二范式是建立在第一范式基础上的，另外要求所有非主键字段完全依赖主键，不能产生部分依赖
- Eine Relation ist in 2NF, wenn sie in 1NF ist und jedes Nicht-Schlüssel-Attribut **voll funktional abhängig** vom gesamten Primärschlüssel ist (d.h. keine Teilabhängigkeiten).
- Relation erfüllt die Regeln der 1NF und alle Nichtschlüsselattribute (NSA) sind vom gesamten Primärschlüssel voll funktional abhängig

Third Normal Form
建立在第二范式基础上的，非主键字段不能传递依赖于主键字段。（不要产生传递依赖）
Eine Relation ist in der **3. Normalform (3NF)**, wenn sie:
1. In **2NF** ist, und
2. Kein Nicht-Schlüssel-Attribut transitiv abhängig vom Primärschlüssel ist.

从上表可以看出，班级名称字段存在冗余，因为班级名称字段没有直接依赖于这个表本身的主键，班级名称字段依赖于班级编号，班级编号依赖于学生编号，那么这就是传递依赖，解决的办法是将冗余字段单独拿出来建立表


Daumenregeln
- Relationen in INF, die einen aus nur einem Attribut bestehenden Primärschlüssel besitzen sind automatisch in 2NF (da alle NSA automatisch voll funktional abhängig vom PK Sind)
- Relationen in 2NF, die nur Schlüsselattribute und/oder maximal ein NSA enthalten, sind automatisch in 3NF



We have to focus on some basic rules that are for BCNF:
1. Table must be in Third Normal Form.  
2. In relation X->Y, X must be a superkey in a relation.




# 4 Relationenalgebra 

## 4.1 符号 
- relation: 一个 realtion 1就是一个 table 
- tuple: Element 的组合 von eine Relation ( table )  
    - t1 ={ name, number, ... , }  这些都是 attribute von table 
- **π = Projection**
    - The projection operator selects specific attributes (columns) from a relation (table).
    - `π[A.SiteNo, B.SiteNo]`:   select a.siteno, b.siteno
- **σ = Selection**
    - The selection operator filters tuples (rows) from a relation based on a given condition.
    - ` σ [A.LinksTo = B.SiteNo](A x B)`: where a.linksto = b.siteno
- ρ = Renaming
    - The renaming operator allows you to change the name of a relation or its attributes.
    - `ρ[A](Students)`: 将 Students 命名为 A
- Χ = Cartesian Product
    - The Cartesian product combines two relations by pairing every tuple of the first relation with every tuple of the second relation.
    - `TableA x TableB` : from TableA, TableB,
- γ: Grouping and Aggregation 
    - `γ [(Nr), sum(Anzahl)]` : Group by `Nr` and compute `sum(Anzahl)` for each article.

Join 
- `Artikel ⨝ [Artikel.Nr = Lagerbestand.ArtNr] Lagerbestand`
    - Perform a natural join between `Artikel` and `Lagerbestand` on the condition `Artikel.Nr = Lagerbestand.ArtNr`.

## 4.2 Theta-Join


Ein **Theta-Join** (R⋈θSR ) ist eine spezielle Form des Joins, bei dem die Verknüpfung von zwei Relationen R und S auf der Basis einer Bedingung θ erfolgt. ==Wenn θ leer ist (d. h., es gibt keine Bedingung für den Join), reduziert sich der Theta-Join zum **kartesischen Produkt** (R×S).==

![](image/Pasted%20image%2020241221214339.png)
## 4.3 Natural Join

```
SELECT *
FROM Students
JOIN Courses USING (CourseID);
```


## 4.4 Operator

Differenz (-) und **Vereinigung** (∪) sowie **Schnitt** (∩)
- R1-R2: Die Tupel, die nur in R1 sind, aber nicht in R2.


# 5 Sichten/View

Art von View:   Selektionssicht, Projektionssicht, Verbundsicht, Unionsicht, Aggregierungssicht

**Selektionssicht:**
- Eine Sicht, die eine Teilmenge von Daten aus einer Tabelle oder mehreren Tabellen enthält, basierend auf bestimmten Filterkriterien (z. B. mit `WHERE`-Klauseln).
```
CREATE VIEW aktive_kunden AS
SELECT * FROM kunden
WHERE status = 'aktiv';

```



# 6 Rechte 

## 6.1 Grant privileges
```sql
-- Erstellen von Rolle A mit spezifischen Privilegien
CREATE ROLE rolle_a;
GRANT SELECT ON tabelle_x TO rolle_a;

-- Erstellen von Rolle B und Zuweisung von Rolle A an Rolle B
CREATE ROLE rolle_b;
GRANT rolle_a TO rolle_b;

-- Zuweisung von Rolle B an einen Benutzer
GRANT rolle_b TO benutzer1;

```


## 6.2 revoke privileges



To revoke privileges in reverse order step by step:
1. **Start with the most specific privilege**: First, revoke any specific privileges granted on individual database objects, such as tables, views, or procedures.
2. **Revoke object-level privileges**: For example, revoke **SELECT**, **INSERT**, **UPDATE**, and **DELETE** privileges on specific tables or views.
3. **Revoke role-specific privileges**: If any roles were granted with specific privileges, revoke the roles granted to the user.    
4. **Revoke system-level privileges**: Finally, revoke broader system-level privileges such as **CREATE**, **ALTER**, or **DROP** privileges that might have been granted at the database or server level.


This method ensures that the revocation follows a logical flow from the most specific permissions to the broader ones.


```
revoke select on lagerbestand from inventur;
revoke insert, update, delete on lagerbestand from inventur;
revoke select on lagerbestand from controlling;

drop role inventur;
drop role controlling;
```


# 7 Transaktionen

ACID Prinzip 
见 200_10_ConcurrencyControl 
• Atomicity: Either all operations of a transaction complete or none of them complete (“all or nothing”) — Recovery
• Consistency: A transaction that is applied to a consistent database produces a consistent database — Integrity constraints
• Isolation: A transaction executes as if it is the only transaction running in the system — Concurrency control
• Durability: The effects of committed transactions are reflected in the database even after failures — Recovery

Concurrency Anomalien
见 200_10_ConcurrencyControl 
- **Dirty Read**: A **Dirty Read** occurs when one transaction reads data that has been written by another transaction, but the second transaction has not yet committed. If the second transaction rolls back, the data read by the first transaction becomes invalid or "dirty."
- **Lost Update**: In a **Lost Update** scenario, two transactions concurrently modify the same data, and the update made by one transaction is overwritten by the other transaction, causing one of the updates to be lost. This happens when the transactions read and modify the same value without proper isolation.
- A **Non-Repeatable Read** occurs when a transaction reads the same data multiple times, but the value has been changed by another transaction in between those reads. This creates an inconsistency where the same data is read twice with different results.
    - 每次transcation 读取同一个参数的值不一样, 因为中途被改变了
- A **Phantom Read** occurs when a transaction reads a set of rows that match a certain condition, but another transaction concurrently inserts, deletes, or updates rows that cause the result set of the first transaction to change unexpectedly before it completes.

## 7.1 Transitionsvariable NEW

Welche Werte liegen bei der Transitionsvariable NEW bei folgenden Operationen (nacheinander) vor:
- bevor ein DELETE ausgeführt wird,
- bevor ein UPDATE ausgeführt wird,
- bevor ein INSERT ausgeführt wird.

Die Transitionsvariable **NEW** repräsentiert die neuen Werte, die in einer Datenzeile nach einer DML-Operation (INSERT, UPDATE, DELETE) vorliegen. Die Antwort auf diese Frage hängt davon ab, welche Art von DML-Operation ausgeführt wird.


Zusammengefasst:
- **Vor einem DELETE**: **NULL** (es gibt keinen neuen Wert).
- **Vor einem UPDATE**: **neuer Wert** (der Wert nach der Aktualisierung).
- **Vor einem INSERT**: **neuer Wert** (der Wert, der eingefügt wird).


# 8 Trigger 



