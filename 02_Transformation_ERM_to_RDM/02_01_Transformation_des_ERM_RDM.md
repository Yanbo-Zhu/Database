



# 1 Grundlagen_der_Transformation_des_ERM_RDM


- Es sollte eine minimale Anzahl an Relationen erzeugt werden, da viele Relationen das Anfrageverhalten und die Übersicht negativ beeinflussen können
- Möglichst die komplette Information aus dem konzeptuellen Schema soll im logischen Schema nutzbar gemacht werden.
- Die Abbildung der Modellelemente des ERM auf die Modellelemente des Relationalen Datenmodells sollte verständlich und nachvollziehbar sein.
- Die richtigen Antworten sind: Möglichst die komplette Information aus dem konzeptuellen Schema soll im logischen Schema nutzbar gemacht werden.



Phasenmodell 
![](image/Pasted%20image%2020241113104639.png)

![](image/Pasted%20image%2020241113103317.png)





 konzeptuelle Schema (ER Modelle) -> Logsiche Schema -> Datenmodell 

满足了 3NF , 可以实现 minimale Redundanz 
![](image/Pasted%20image%2020241113103723.png)


![](image/Pasted%20image%2020241113104002.png)


# 2 Transformation der grundlegenden Modellelemente des ERM ins RDM 


![](image/Pasted%20image%2020241113104756.png)


## 2.1 Transformation von Attributen 

![](image/Pasted%20image%2020241113105400.png)


## 2.2 Transformation von Entitätebtypen 

- Transformation von Entitätentypen:
    - Ein Entitätentyp und seine elementaren Attribute wet-den auf eine Relation und deren Attribute abgebildet.

![](image/Pasted%20image%2020241113110142.png)


## 2.3 Transformation von Beziehungstypen 


### 2.3.1 Ein Beziehungstyp der Kardinalität 1:1

- 不需要额外的table for Bezihung 
- FS zu beliebig Seite


Ein Beziehungstyp der Kardinalität 1:1 Wird über einen ==Fremdschlüssel== abgebildet: Der Primärschlüssel eines (beliebigen) Entitätentyps, der an der Beziehung teilnimmt, Wird als Fremdschlüssel in der Relation ==des beliebige anderen Entitätentyps== verwendet.

Franchbereich 必须至少有有一个 dozent
反映到 zielschema 中 就是 Fachbereich 中必须有一个 attribute Dekan 系主任

![](image/Pasted%20image%2020241113110325.png)


### 2.3.2 Ein Beziehungstyp der Kardinalität 1:N oder N:1

不需要一个Schema table ,  
FS 放到N seite 

---
那边是 N Seite 




---

Ein Beziehungstyp der Kardinalität I:N Wird über einen Fremdschlüssel abgebildet: Der Primärschlüssel des Entitätentyps der 1-Seite, der an der Beziehung teilnimmt, Wird als Fremdschlüssel in der Relation des Entitätentyps der ==N-Seite== verwendet.

> Key point ist toc chose the suitable Entitattyp as intermediary 
> Der Schlüssel für eine Redundanzarm 

![](image/Pasted%20image%2020241113111508.png)

### 2.3.3 Beziehungstyp der Kardinalität M:N

需要一个table,  FS von linker Seite und rechte Seite 都填到这个 新的 
0 Tabelle 中 

`(0, *) ` 的关系就是 对应的是1kardinalität M:N 

Ein Beziehungstyp der Kardinalität M:N Wird auf eine Relation abgebildet: 
Die Primärschlüssel der Entitätentypen, die an der Beziehung teilnehmen, werden als Fremdschlüssel in diese Relation übergeben und bilden i.d.R. deren zusammengesetzten Primärschlüssel. Attribute des Beziehungstyps werden ebenso dieser Relation zugeordnet.

这种情况下的 Beziehungstyp 需要自成一个table 
![](image/Pasted%20image%2020241113111925.png)


### 2.3.4 Rekursive Beziehungstyp 

Ein rekursiver Beziehungstyp Wird auf eine Relation abgebildet: 
Der Primärschlüssel des Entitätentyps Wird als Fremdschlüssel zweimal in diese Relation übergeben, wobei durch Umbenennung eindeutige Namen zu gewährleisten Sind. Attribute des Beziehungstyps werden ebenso dieser Relation zugeordnet.

这种情况下的 Beziehungstyp 需要自成一个table 
![](image/Pasted%20image%2020241113112626.png)


### 2.3.5 Ein n-närer Beiyhungstyp 

Ein n-ärer Beziehungstyp Wird auf eine Relation abgebildet: 
Die Primärschlüssel der Entitätentypen, die an der Beziehung teilnehmen, werden als Fremdschlüssel in diese Relation übergeben und bilden i.d.R. deren zusammengesetzten Primärschlüssel. Attribute des Beziehungstyps werden ebenso dieser Relation zugeordnet.

这种情况下的 Beziehungstyp 需要自成一个table 
1 lieferent liefert Artikel fuer 1 Projekt 
1 lieferant liefert die Menge  fur projekt 
![](image/Pasted%20image%2020241113113232.png)




# 3 Transformation von Generalisierungshierarchien ins RDM 


从几个 Entitattyp 中  抽象出 一个Supertyp 
Supertyp  中的 attibute   就不需要出现在 subtyp 中了, 减少冗余 


Eine Generalisierungshierarchie Wird derart abgebildet, dass für den Supertyp und alle Subtypen eigene Relationen angelegt werden. Die Relation des Supertyps enthält generalisierbare Attribute, einen Primärschlüssel und evtl. ein Typkennzeichen. Die Relationen der Subtypen beinhalten die speziellen Attribute und erben den Primärschlüssel des Supertyps.

1 General 

GeschaftsPatner 中的Typ  指的是 他是 Kunde 还是 Lieferant 
![](image/Pasted%20image%2020241113114119.png)


2  Variant

![](image/Pasted%20image%2020241113115156.png)


Standardvariante: 分开两张表存数据 ,   两张表的数据不重复 
Horizontale Partitionierung:   Supertype , 也设置两个 type, 但是这三张表中的数据有可能重复, 有可能不重复 
Typisierte Partitionierung:  只设置Supertype 的table, 将两个 type 的数据 都存到这一张表里面 

## 3.1 Horizontale Partitionierung: ER-Still

有的电影 可能同时属于 三个 type: Film, Zeichentrickfileme, Krimis,  这三个 schema 都有他的 entry,   造成了 Redundancy 



![](image/Pasted%20image%2020241113120840.png) 


![](image/Pasted%20image%2020241113122613.png)

## 3.2 Horizontale Partitionierung: Objekt-orientierter Stil 

![](image/Pasted%20image%2020241113122639.png)


FilmeZ: Film Zeichenkrickfilme 
FilmeK: Krimis
FlimeZK: film 同时属于 Zeichenkrickfilme and Krimis


![](image/Pasted%20image%2020241113123108.png)



## 3.3 Typisierte Partitionierung: Mit Nullwerte



![](image/Pasted%20image%2020241113123349.png)







