

# 1 Freitext Aufgabe 

Bitte beachten Sie: Diese Aufgabe muss **offline** und **handschriftlich** (möglichst auf weißem Papier) bearbeitet werden. Ihre Lösung geben Sie bitte am Ende der Klausur ab.

Die Aufgabe bezieht sich mit den genannten Teilaufgaben auf den nachfolgend dargestellten Sachverhalt.

**FT1.** Entwickeln Sie bitte das Datenmodell für die Abbildung des Sachverhaltes. Stellen Sie das Datenmodell in Form eines ER-Diagramms dar, welches sinnvoll normalisiert ist. Machen Sie die Beziehungen entsprechend der in der Aufgabenstellung geforderten Kardinalitäten deutlich. Die Attribute können in dieser Darstellung weggelassen werden.  (**12 Punkte**).

**FT2.** Leiten Sie aus dem ER-Diagramm (aus **FT1**) Relationen gemäß der Ihnen bekannten Transformationsregeln ab. Notieren Sie die Relationen gerne in Tupelschreibweise. Definieren Sie zu jeder Relation Primär- und Fremdschlüssel. Machen Sie die Primär- und Fremdschlüssel kenntlich (z.B. Unterstreichung oder Hervorhebung). Ordnen Sie bitte alle angesprochenen Attribute zu (**12 Punkte**).

**FT3.** Entwickeln Sie bitte für die aus **FT2** resultierende(n) Relation(en) für die Abbildung von Lehrheften mit entsprechenden Abschnitten den (die) entsprechenden „create table“-Befehl(e), mit Abbildung der Primär- und Fremdschlüssel. **(10 Punkte)**

**FT4**. Herausgeber und Autor sind Personen. Konsequenterweise sollte hier eine Generalisierung verwendet werden. Stellen Sie die Generalisierung mit den Mitteln des EERM dar und begründen Sie kurz, für welche Art von Generalisierungshierarchie Sie sich entscheiden. (6 Punkte). **(6 Punkte)**

-----------------------------------

Gegenstand der vorliegenden Aufgabe ist die Verwaltung von Lehrmaterialien für ein Fernstudium Informatik an einer Hochschule:

Ausgegangen werden soll von Lehreinheiten als größeren, inhaltlich orientierten Studienkomplexen in einem Hochschulfernstudium. Als Lehreinheiten existieren beispielsweise „Datenbanksysteme“, „Internettechnologien“ etc. Lehreinheiten werden von verantwortlichen Herausgebern betreut.

Zu jeder Lehreinheit werden momentan jeweils 8 Kurseinheiten angeboten sowie ein Praktikum als weitere Kurseinheit. Kurseinheiten der Lehreinheit „Datenbanksysteme“ sind beispielsweise „Einführung in SQL“ oder „XML und Datenbanken“.  
Zu jeder Kurseinheit, mit Ausnahme des Praktikums, wird jeweils ein Lehrheft angeboten. Ein Lehrheft gehört genau zu einer Kurseinheit. Lehrhefte werden von Autoren entwickelt, die nicht mit dem Herausgeber der Lehreinheit übereinstimmen müssen (andere Personen sind möglich). Ein Lehrheft umfasst verschiedene Abschnitte mit jeweils einer Hauptüberschrift und dem eigentlichen Text. Abschnitte sind hierarchisch gegliedert, d.h. sie können auch Unterabschnitte enthalten. Ein Unterabschnitt gehört genau zu einem übergeordneten (Haupt-) Abschnitt.

Für die Autoren und Herausgeber sind neben den persönlichen Adressdaten auch die Koordinaten der Institution, bei der sie beschäftigt sind, zu speichern.



# 2 Choice Aufgabe 

## 2.1 Create Table DDL 

![](image/Pasted%20image%2020250121200150.png)

Gegeben sind die folgenden Relationen. Mit welchen Befehlen werden korrekt Tabellen dafür angelegt?   



Mitglied(Mitgliedsnummer, Nachname, Vorname)  
Mitgliedschaft(Mitgliedsnummer->Mitglied.Mitgliedsnummer, Eintrittsdatum, Austrittsdatum)


Wählen Sie eine oder mehrere Antworten:

```
create table mitglied  
(mitgliedsnummer int,  
nachname varchar(30),  
vorname varchar(30),
primary key (mitgliedsnummer));

create table mitgliedschaft
(mitgliedsnummer int,  
eintrittsdatum date,  
austrittsdatum date,  
primary key (mitgliedsnummer, eintrittsdatum),  
foreign key (mitgliedsnummer) references mitglied(mitgliedsnummer));
```


```
create table mitgliedschaft
(mitgliedsnummer int,
eintrittsdatum date,
austrittsdatum date,
primary key (mitgliedsnummer, eintrittsdatum));


create table mitglied
(mitgliedsnummer int primary key,
nachname varchar(30),
vorname varchar(30)
foreign key (mitgliedsnummer) references mitgliedschaft(mitgliedsnummer));
```



```
create table mitglied  
(mitgliedsnummer int primary key,  
nachname varchar(30),  
vorname varchar(30));


create table mitgliedschaft
(mitgliedsnummer int,  
eintrittsdatum date,  
austrittsdatum date,  
primary key (mitgliedsnummer, eintrittsdatum),  
foreign key (mitgliedsnummer) references mitglied(mitgliedsnummer));
```


```
create table mitgliedschaft
(mitgliedsnummer int,
eintrittsdatum date,
austrittsdatum date,
primary key (mitgliedsnummer, eintrittsdatum),
foreign key (mitgliedsnummer) references mitglied(mitgliedsnummer));


create table mitglied
(mitgliedsnummer int primary key,
nachname varchar(30),
vorname varchar(30));

```



```
create table mitglied
(mitgliedsnummer int primary key,
nachname varchar(30),
vorname varchar(30));

create table mitgliedschaft
(mitgliedsnummer int primary key,
eintrittsdatum date primary key,
austrittsdatum date,
foreign key (mitgliedsnummer) references mitglied(mitgliedsnummer));
```

---

Der **kombinierte Primärschlüssel** (`primary key (mitgliedsnummer, eintrittsdatum)`) stellt sicher, dass kein Mitglied für das gleiche Eintrittsdatum mehrfach erfasst werden kann.
上面不可以写为mitgliedsnummer int primary key, eintrittsdatum date primary key  因为这样就变成了 定义出来了两个 primary key 不是 kombierte key 了 

当只有一个 Primärschlüssel 的时候 可以写为
可以写成 `primary key (mitgliedsnummer)` 等价于 mitgliedsnummer int primary key



正确选项为 


```
create table mitglied  
(mitgliedsnummer int,  
nachname varchar(30),  
vorname varchar(30),
primary key (mitgliedsnummer));

create table mitgliedschaft
(mitgliedsnummer int,  
eintrittsdatum date,  
austrittsdatum date,  
primary key (mitgliedsnummer, eintrittsdatum),  
foreign key (mitgliedsnummer) references mitglied(mitgliedsnummer));
```

和 


```
create table mitglied  
(mitgliedsnummer int primary key,  
nachname varchar(30),  
vorname varchar(30));


create table mitgliedschaft
(mitgliedsnummer int,  
eintrittsdatum date,  
austrittsdatum date,  
primary key (mitgliedsnummer, eintrittsdatum),  
foreign key (mitgliedsnummer) references mitglied(mitgliedsnummer));
```


---
下面是不正确的 

```
create table mitglied
(mitgliedsnummer int primary key,
nachname varchar(30),
vorname varchar(30));

create table mitgliedschaft
(mitgliedsnummer int primary key,
eintrittsdatum date primary key,
austrittsdatum date,
foreign key (mitgliedsnummer) references mitglied(mitgliedsnummer));

```






## 2.2 Revolke Privilegien

Wie sehen die korrekten SQL-Anweisungen in korrekter Reihenfolge aus, mit denen Sie die nachfolgend vergebenen  
Privilegien in umgekehrter Reihenfolge Schritt für Schritt wieder entziehen. Bitte löschen Sie auch die beiden erstellten Rollen.

```
CREATE ROLE controlling; 

CREATE ROLE inventur;

GRANT SELECT ON Artikel, Halle TO PUBLIC;  
GRANT SELECT ON Lagerbestand TO controlling;  
GRANT SELECT, INSERT, UPDATE, DELETE ON Lagerbestand TO inventur;
```


Wählen Sie eine oder mehrere Antworten:

```
revoke select on lagerbestand from inventur;
revoke insert, update, delete on lagerbestand from inventur;
revoke select on lagerbestand from controlling;

drop role inventur;
drop role controlling;
```


```
revoke select, insert, update, delete on lagerbestand from inventur;  
revoke select on lagerbestand from controlling;  
revoke select on artikel, halle from public;  
drop role inventur;  
drop role controlling;
```


```
revoke select on artikel, halle from public;

revoke select, insert, update, delete on lagerbestand from inventur;  
drop role inventur;

revoke select on lagerbestand from controlling;  
drop role controlling;
```


```
revoke select on lagerbestand from inventur;
drop role inventur;

revoke insert, update, delete on lagerbestand from inventur;
revoke select on artikel, halle from public;

revoke select on lagerbestand from controlling;  
drop role controlling;
```


```
revoke select, insert, update, delete on lagerbestand from inventur;  
drop role inventur;

revoke select on artikel, halle from public;

revoke select on lagerbestand from controlling;  
drop role controlling;
```


```
drop role inventur;

revoke select, insert, update, delete on lagerbestand from inventur;  
revoke select on artikel, halle from public;

revoke select on lagerbestand from controlling;  
drop role controlling;
```

```
drop role inventur;
drop role controlling;

revoke select, insert, update, delete on lagerbestand from inventur;
revoke select on artikel, halle from public;
revoke select on lagerbestand from controlling;
```

```
revoke select, insert, update, delete on lagerbestand from inventur;  
drop role inventur;

revoke select on lagerbestand from controlling;  
drop role controlling;

revoke select on artikel, halle from public;
```


## 2.3 Trigger 

Der nachfolgende Trigger bewirkt beim Einfügen eines Auftrags in die Tabelle aufkopf, dass der Wert der Spalte kdnr, wenn er nicht 99 ist, auf das Vorhandensein in der Tabelle kdst in der gleichnamigen Spalte geprüft wird. Ist der Wert der Spalte kdnr dort nicht vorhanden, wird eine No-Data-Found-Exception geworfen.

```
create or replace trigger checkAuftrag  

after insert or update of kdnr on aufkopf  

for each row  
when (new.kdnr <> 99)  
declare  nr number(3);  

begin  
select kdnr into nr from kdst k where k.kdnr=:new.kdnr;  
exception  
when no_data_found then  
    raise_application_error(-20000,'Integrität verletzt');  
end;
```

Frage 5 Bitte wählen Sie eine Antwort:

Wahr
Falsch


正确答案是 wahr 


## 2.4 Arten von Sichten

Welche Arten von Sichten kennen Sie?

Frage 6 Wählen Sie eine oder mehrere Antworten:

- Selektionssicht, Projektionssicht, Verbundsicht
- Selektionssicht, Produktionssicht, Verbundsicht
- Verbundsicht, Unionsicht, Generierungssicht
- Verbundsicht, Zentrierungssicht, Aggregierungssicht
- Verbundsicht, Unionsicht, Aggregierungssicht

选择 
Selektionssicht, Projektionssicht, Verbundsicht
Verbundsicht, Unionsicht, Aggregierungssicht

---

正确的 

**Selektionssicht:**
- Eine Sicht, die eine Teilmenge von Daten aus einer Tabelle oder mehreren Tabellen enthält, basierend auf bestimmten Filterkriterien (z. B. mit `WHERE`-Klauseln).
```sql
CREATE VIEW aktive_kunden AS
SELECT * FROM kunden
WHERE status = 'aktiv';

```


**Projektionssicht:**
- Eine Sicht, die nur bestimmte Spalten (Attribute) einer Tabelle oder mehreren Tabellen enthält. Sie wird verwendet, um überflüssige Informationen auszublenden.

```sql
CREATE VIEW kunden_name AS
SELECT vorname, nachname FROM kunden;
```

**Verbundsicht (Join-Sicht):**
- Eine Sicht, die Daten aus mehreren Tabellen kombiniert, typischerweise über einen Join.

```sql
CREATE VIEW bestellungen_kunden AS
SELECT b.bestellnummer, k.vorname, k.nachname
FROM bestellungen b
JOIN kunden k ON b.kundennummer = k.kundennummer;
```

**Unionsicht:** Die Kombination von Daten mittels `UNION`

Aggregierungssicht

```sql
CREATE VIEW Branchen AS
SELECT branche, COUNT(*) as count
FROM kdst
WHERE ort = 'Berlin'
GROUP BY branche;
```

---



 Falsche Optionen:
- **Produktionssicht:** Dieser Begriff existiert nicht in der Datenbankterminologie.
- **Unionsicht:** Die Kombination von Daten mittels `UNION`
- **Generierungssicht, Zentrierungssicht:** Diese Begriffe sind nicht gebräuchlich in der Datenbanktheorie. Aggregationen werden zwar in Sichten verwendet, aber nicht als eigenständige Sichtart bezeichnet.


## 2.5 CRUD-Operationen


Worauf beziehen sich CRUD-Operationen?
Datensätze
das Data Dictionary
die Datenbank als Ganzes
Dateien
Karteikarten


----

答案见 selbertest 10, 或者 这个 note 中的某个 笔记中, 一搜就知道了 

Die korrekte Antwort lautet: **Datensätze**

Erklärung:
CRUD steht für die grundlegenden Operationen, die auf **Datensätzen** in einer Datenbank ausgeführt werden können:

- **C**reate: Erstellen eines neuen Datensatzes (z. B. `INSERT` in SQL).
- **R**ead: Lesen oder Abfragen eines bestehenden Datensatzes (z. B. `SELECT` in SQL).
- **U**pdate: Aktualisieren eines bestehenden Datensatzes (z. B. `UPDATE` in SQL).
- **D**elete: Löschen eines bestehenden Datensatzes (z. B. `DELETE` in SQL).



## 2.6 Transaktion

Eine Transaktion kann nur aus einer Operation bestehen. Sie ist entweder vollständig auszuführen oder muss im Fehlerfall zurückgerollt werden.

Frage 8 Bitte wählen Sie eine Antwort:
Wahr
Falsch

---



Die korrekte Antwort lautet: **Wahr**

Erklärung:

Eine Transaktion ist eine logische Einheit von Datenbankoperationen, die als **untrennbare Einheit** ausgeführt werden. Auch wenn die Transaktion nur aus einer einzigen Operation besteht, gelten die Eigenschaften von Transaktionen, insbesondere die **ACID-Eigenschaften**:
1. **Atomicity (Atomarität):** Die Transaktion wird entweder vollständig ausgeführt oder im Fehlerfall zurückgerollt.
2. **Consistency (Konsistenz):** Nach der Transaktion bleibt die Datenbank in einem konsistenten Zustand.
3. **Isolation (Isolation):** Transaktionen beeinflussen sich gegenseitig nicht.
4. **Durability (Dauerhaftigkeit):** Nach Abschluss der Transaktion bleiben die Änderungen dauerhaft gespeichert.

Selbst eine einzelne Operation, wie das Einfügen eines Datensatzes (`INSERT`), zählt als Transaktion, wenn sie von einem **COMMIT** abgeschlossen oder bei Fehlern **zurückgerollt (ROLLBACK)** wird.



## 2.7 Integritätsbedingung einfügen

Welche Möglichkeiten gibt es, die nachfolgende Integritätsbedingung im SQL-Standard umzusetzen: das Umsatzsoll der Kunden soll mindestens 10.000 € betragen.

Frage 9 Wählen Sie eine oder mehrere Antworten:

- check (umssoll > 10000) - bei CREATE TABLE
- durch alleine einen BEFORE UPDATE OF UMSSOLL ON KDST - Trigger
- durch einen Primärschlüssel
- durch einen BEFORE INSERT OR UPDATE OF UMSSOLL ON KDST - Trigger
- durch einen entsprechenden Fremdschlüssel auf Umsatzsoll in einer Umsatzsolltabelle
- durch alleine einen BEFORE INSERT ON KDST - Trigger
- check (umssoll > 10000) - bei ALTER TABLE

最后一个选项是 
```
CREATE DOMAIN umsatzsollbegrenzug AS INTEGER
CHECK (VALUE > 10000); 
und Nutzung der Domäne umsatzsollbegrenzug  bei CREATE oder ALTER TABLE als Datentyp
```


答案见  10_01_Daten_Integritätssicherung


## 2.8 Trigger


Wie oft wird der nachfolgende Trigger ausgeführt, wenn ein UPDATE in der Tabelle STATUS_LOG 10.000 Zeilen betrifft?

```
CREATE OR REPLACE TRIGGER Status_Check

AFTER INSERT OR UPDATE OF S_STATUS ON AUFKOPF

FOR EACH ROW

when (new.aufnr != 99)  
declare  
status_alt aufkopf.s_status%TYPE;  
BEGIN  
if inserting then  
status_alt := null;  
else  
status_alt := :old.s_status;  
end if;  
insert into status_log  
values(:new.aufnr,status_alt,:new.s_status,sysdate);  
dbms_output.put_line('statusänderung protokolliert von auftrag: '|| :new.aufnr);  
END;
```

Frage 10 Wählen Sie eine oder mehrere Antworten:
0
10.001
1
10.000
9.999

---

答案是 0 ,

答案可以见 selblerntest 10 或者 这个 note 中 tigger 部分的笔记 

## 2.9 Art von Sicht 

Um welche Art von Sicht handelt es sich hier?

```
CREATE VIEW Branchen AS
SELECT branche, COUNT(*) as count
FROM kdst
WHERE ort = 'Berlin'
GROUP BY branche;
```

Frage 11 Wählen Sie eine oder mehrere Antworten:

Aggregierung,Projektion,Selektion, Union
Aggregierung,Projektion,Verbund
Projektion,Selektion, Verbund
Aggregierung, Projektion, Selektion

---

正确答案是 Aggregierung, Projektion, Selektion

- **Aggregierung:**  
    Die Verwendung von `COUNT(*)` und die Gruppierung der Ergebnisse mit `GROUP BY branche` zeigt, dass hier eine Aggregation erfolgt. Daten werden zusammengefasst, z. B. die Anzahl der Einträge pro Branche.
    
- **Projektion:**  
    Im `SELECT`-Statement werden nur bestimmte Spalten ausgewählt (`branche` und `COUNT(*) as count`). Dies entspricht der Projektion, da nur relevante Daten aus der Tabelle ausgewählt werden.
    
- **Selektion:**  
    Die Bedingung `WHERE ort = 'Berlin'` filtert die Datensätze, sodass nur die Einträge mit dem Ort "Berlin" in die Sicht einfließen. Dies ist eine Selektion.


Warum **Union** oder **Verbund** nicht korrekt sind:
- **Union** kombiniert die Ergebnisse von zwei oder mehr Abfragen. Das ist hier nicht der Fall.
- **Verbund** (Join) wird verwendet, um Daten aus mehreren Tabellen zu verknüpfen. Hier wird nur eine Tabelle (`kdst`) verwendet.
