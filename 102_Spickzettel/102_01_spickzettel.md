

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
