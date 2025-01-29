
Datenmodellierung
Begriffe und Definitionen
ER-Diagramme
(Designprinzipien)
Erweitertes ER-Modell


# 1 Einleitung: Was sind Modelle und wozu braucht man sie

Was ist ein Modell?
- Ein Modell ist eine abstrakte Darstellung der Wirklichkeit
- Ein Modell entspricht damit nie exakt der Realität
- Modellieren bedeutet immer Hervorheben und Weglassen
    - Hervorheben des Wesentlichen
    - Weglassen unwichtiger Details
- Was aber ist wesentlich, was unwichtig?
    - Keine allgemeine Beantwortung möglich, sondern abhängig davon,
    - was für Ziele mit dem Modell verfolgt werden und
    - wer die Leser des Modells sind (Wissensstand, Sichtweise, Versiertheit, Entscheidungsbefugnis)
- Modelle werden erstellt, um existierende oder zukünftige IT- und/oder Geschäftssysteme (z.B. Prozesse, Organisationen, Strukturen) besser verstehen zu können

Warum braucht man Modelle?
- Ermöglichen einer formalen Kommunikationsbasis zwischen allen Beteiligten
- Kompakte Visualisierung der Fakten für die Auftraggeber, Fachexperten, Entwickler und Anwender
- Überprüfung der Fakten bezüglich Vollständigkeit, Widerspruchsfreiheit und Korrektheit
- In der Softwaretechnik: zur Generierung von Code ->  Model Driven Architecture (MDA), Model Driven Design (MDD)


# 2 Phasenmodell des DB-Entwurf

![](image/Pasted%20image%2020241119085015.png)


1 Anforderungsanalyse
Anforderungsanalyse (Informationsbedarf )
informale Beschreibung

2 Konzeptioneller Entwurf : 得到 erm
 (Externe Schemata, Sichtenintegration, Konzeptuelles Schema (KS))
1-th Formale Beschreibung
Datenmodellierung mit konzeptuellem Datenmodell (ERM / UML)

3 Logischer Entwurf (Logisches Schema (LS))  推导出 RDM 
2-th Formale Beschreibung
Schritte:
Ableitung des LS (RDM) aus dem KS (ERM)
Normalisierung

4 Datendefinition  使用 DDL 产生DBMS
Datendefinition (Umsetzung „abstrakte“ Schemata in „konkrete“ Schemata eines DBMS unter Verwendung der DB-Sprachen)

Definition des LS (logische Schema) unter Verwendung der DDL
create table Kunden
(KDNR char(10) primary key, ...) ;

Definition Sichten unter Verwendung VDL
create view grossKunden as select kdnr,firma,umssoll from Kunden where umssoll > 100.000;


5  Physischer Entwurf  产生 Schema and table
Physischer Entwurf (Entwurf internes Schema (IS))
Definition IS unter Verwendung von SSL

create index KundenName on Kunden (firma asc) ;
create cluster Kunden_Und_Auftraege
(KDNR char(10));

![](image/Pasted%20image%2020241119090236.png)

5  Implementierung und Wartung
(Einsatz in Zielumgebung, Wartung)


# 3 Grundlegende Elemente des ERM zur Datenmodellierung


## 3.1 Modellelemente 


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


Kardinalität (Cardinality)
Die Kardinalität (auch Multiplizität) beschreibt die mögliche Anzahl (Min/Max) der an einer Beziehung beteiligten Entitäten

## 3.2 Darstellung von Entitätentypen und Attributen


Entitäten werden als Rechtecke dargestellt, die in ihrer Mitte die Bezeichnung tragen
Attribute werden als Ovale mit einer Linie mit dem entsprechenden Rechteck verbunden

![](image/Pasted%20image%2020241119090429.png)


## 3.3 Darstellung von Beziehungstypen
Beziehungen zwischen zwei Entitätentypen werden über eine Linie zwischen diesen Entitäten symbolisiert
Die Linie wird von einer Raute dekoriert in die optional eine klassifizierende Bezeichnung für den Beziehungstyp eingetragen werden kann

![](image/Pasted%20image%2020241119090502.png)


## 3.4 Darstellung von attributierten Beziehungstypen
Weist ein Beziehungstyp zusätzliche Eigenschaften (als nur das Herstellen der Beziehung) auf, so können diese durch Attribute notiert werden
Attribute werden (auch hier) als Ovale mit einer Linie mit der entsprechenden Raute verbunden

![](image/Pasted%20image%2020241119090537.png)



## 3.5 Darstellung von Kardinalitäten

![](image/Pasted%20image%2020250129233410.png)

![](image/Pasted%20image%2020241117204252.png)

### 3.5.1 (1,M,N) Notation

Die mögliche Anzahl der mit einer Entität in Beziehung stehenden Entitäten kann mit einer Maximalanzahl ausgedrückt werden. z.B. (Zu einem Lieferanten kann eine Adresse hinterlegt sein

Dabei wird die Anzahl immer an das Ende der Assoziationslinie notiert


![](image/Pasted%20image%2020241027173947.png)



1 Lieferant steht mit 1 addresse in Beziehung, 1 Lieferant hat nur 1 Addresse 
1 addresse  kann N Lieferenten zugeordnet sein 



### 3.5.2 (Min, Max) Notation

Die mögliche Anzahl der an einer Beziehung beteiligten Entitäten kann mit einer Minimal- und einer Maximalanzahl ausgedrückt werden.
z.B. (Zu einem Lieferanten kann keine oder genau eine Adresse hinterlegt sein  (0,1)

Dabei wird die Anzahl immer an den Beginn der Assoziationslinie notiert 
(im Gegensatz zur UML, wo es am Ende der Assoziation notiert wird)

![](image/Pasted%20image%2020241119090638.png)



![](image/Pasted%20image%2020241027181133.png)




![](image/Pasted%20image%2020241027174512.png)

1 
1 lieferant nimmt in der Beziehung mindestens einmal teil und kann oft teilnehmen 
Der lieferant kann viele Artikel  lieferen muss aber mindesten einen artikel liefern , ansonsten Lieferent 就不能成为 lieferent 

Ein Artikel muss  einen Beziehung lieferung eingehen oder kann auch viele Beziehung eingehen
Ein Artikel kann von vielen lieferenten geliefert werden . er kann von einem lieferent geliedert werden oder muss nicht gar nicht geliefert werden. 
Der selbst produzierte Artikel in unserer Daten kan gar nicht von lieferant geliefert werden 


2 
Ein Lieferant der muss keine Beziehung zu Adresse eingehen, aber kann auch maximal eine Beziehung eingehen.  就是一个lieferant 不能被分配多个地址  
Ein Lieferant kann maximale einen  einzige adresse haben, aber muss nicht addresse haben 
In unsere Datenbank mochte wir, ohne dass wir sofort eine Adresse hinterlegen müssen.
in unsere datenbauer ermöglichen, dass wir Lieferanten eintragen können, ohne dass wir sofort eine Adresse hinterlegen


Ein addresse  kann N Lieferenten  und muss nicht Lieferent zugeordnet sein 
Z.B eine Addressebuch, wenn wir eben hier auch Adressen haben, die momentan keine Zuordnungen haben . 
Die Adressen die wir für andere Zweck gebrauchen, weil wir vielleicht noch Kunden in unserer Datenbank haben oder andere Geschäftspartner. Solche Address muss nicht Lieferent zugeordnet werden 

## 3.6 Existenzabhängigkeiten

Existenzabhängigkeit und Optionalität 


![](image/Pasted%20image%2020241027183159.png)


只看 minimum  的值 ,决定 existenzabhangigkeit 
Existenzabhangigkeit:  删除了一个Objekt, 和他有Existenzabhangigkeit 的Objekt 也会跟着删除 


图二:
(1,1 ) represents , dass ein lieferent nicht ohne adresse existieren darf 

图三 
`(1,*)  `决定 这个特性



## 3.7 Arten von Beziehungstypen

Nach der Anzahl der beteiligten Entitätentypen (auch Grad der Beziehung) werden unterschieden:
- Binäre Beziehungstypen (2 Entitätentypen)
    - Siehe bisherige Beispiele
- Rekursive / unäre Beziehungstypen (1 Entitätentyp)
    - An einem rekursiven Beziehungstyp nimmt derselbe Entitätentyp mehrfach teil. Eine Entität nimmt dabei verschiedene Rollen ein, Bsp.:
        - Abteilung ist übergeordnete Abteilung zu anderer (untergeordneter) Abt.
        - Abteilung ist untergeordnete Abteilung zu anderer (übergeordneter) Abt.
- N-äre Beziehungstypen (n Entitätentypen)
    - An einem n-ären Beziehungstyp nehmen n Entitätentypen teil.


## 3.8 rekursiver Beziehungstyp
![](image/Pasted%20image%2020250127172645.png)



## 3.9 n-ärer Beziehungstyp
Beispiel: Artikel werden für Projekte in einer bestimmten Menge benötigt und können von Lieferanten geliefert werden:
- Ein Lieferant muss mindestens einen Artikel liefern: `(1, *)`.
- Ein Artikel muß nicht für ein Projekt geliefert werden (Eigenproduktion), kann aber für viele Projekte geliefert werden: `(0, *)`.
- Ein Projekt muss mindestens einen Artikel verwenden: `(1, *)`.

![](image/Pasted%20image%2020250127172922.png)

- Ein n-ärer Beziehungstyp kann nicht verlustfrei in binäre Beziehungstypen aufgebrochen werden!
- Modell 1 ist nicht gleich Modell 2 ! (Bei Modell 2 können falsche Beziehungen aus der Kombination der Beziehungstypen abgeleitet werden.)

![](image/Pasted%20image%2020250127173017.png)


  
# 4 Erweiterungen des ERM 

## 4.1 Aggregation


ERM: Aggregation / Komposition
Zusammenfassung verschiedener Elemente eines Schemas zu einem zusammengesetzten Element

![](image/Pasted%20image%2020250127173106.png)

将一个大的模型
(有很多小的element组成) 编程 只有若干个 中等element 组成的模型 
因为外界 根本不关心很多小点 

aggregated Kunde  <> aggregated Auftrag <> aggregated Artiktel 


Aggregat  名词 集成体

## 4.2 Generalisierung

2 个 Objekttypen , die enthalten viele aehnliche Attribute sie scheinen zu sein. 

Zusammenfassung ähnlicher Elemente eines Schemas zu einem generischen Element
![](image/Pasted%20image%2020250127173122.png)


Mengenbeziehungen von Subtypen (nach Zehnder)
1. Betrachtung: Überschneidung der Objektmengen der Subtypobjekte? (Existieren Entitäten, die 2 Subtypen zugeordnet werden können ?)
    1. Überschneidung (ja)
    2. Disjunktheit (nein)
2. Betrachtung: vollständige Überdeckung der Vereinigungsmenge der Subtypobjekte über die Menge der Supertypobjekte (Existieren Entitäten, die keinem Subtyp zugeordnet werden können?
    1. Vollständige Überdeckung: Alle Entitäten können einem Subtyp zugeordnet werden.
    2. Unvollständige Überdeckung: Nicht alle Entitäten können einem Subtyp zugeordnet werden.


![](image/Pasted%20image%2020250127173138.png)

---

Nicht disjunkt, unvollständig
![](image/Pasted%20image%2020250127173232.png)


有些 Geschaftspartner   ist weder Kunde oder Lieferant , 
有些 Geschaftspartner   ist nicht nur Kunde sondern auch Lieferant

---

disjunkt, vollständig
![](image/Pasted%20image%2020250127173241.png)

 Geschaftspartner  ist sowohl Kunde als auch Lieferant . Weiter keine anderen geschaeftspartnern 

---

disjunkt, unvollständig
![](image/Pasted%20image%2020250127173258.png)

---


Nicht disjunkt, vollständig
![](image/Pasted%20image%2020250127174105.png)

 有些Geschaftspartner  ist sowohl Kunde als auch Lieferant . 
 有些 Geschaftspartner   ist nicht nur Kunde sondern auch Lieferant 
 Weiter keine anderen geschaeftspartnern 

