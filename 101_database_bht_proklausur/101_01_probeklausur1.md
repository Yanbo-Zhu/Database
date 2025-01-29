
# 1 Theoretische Fragen


## 1.1 unterabfragen, subselects

1 Nachfolgende Fragestellung beschäftigt sich mit Unterabfragen (Subselects) in SQL: Erklären Sie zunächst kurz, wo genau Unterabfragen in SQL-Statements vorkommen können und welche Funktion sie haben. Stellen Sie außerdem bitte kurz einfache und korrelierte Unterabfragen einander gegenüber. Nennen und beschreiben Sie 2 Unterschiede. (5 Punkte)

### 1.1.1 wo genau 

生成每个 的 sql 的例子 

Unterabfragen (auch _Subqueries_ genannt) sind SQL-Anfragen, die innerhalb einer anderen SQL-Anfrage eingebettet sind. Sie dienen dazu, komplexe Datenabfragen zu vereinfachen und ermöglichen eine verschachtelte Analyse von Daten. Unterabfragen können in den folgenden Bereichen von SQL-Statements vorkommen:


**SELECT-Statement**: Unterabfragen können in der `SELECT`-Klausel verwendet werden, um aggregierte Werte oder Berechnungen zu ermitteln, die nicht direkt in der Hauptabfrage enthalten sind.
- unterabfragen in select : es liefert eine splate fur ergebnisstablle 

Die Unterabfrage berechnet den durchschnittlichen Preis aller Artikel, und dieser Wert wird als Spalte in der Ergebnistabelle angezeigt.
Ergebnis: Eine Spalte wird hinzugefügt, die den Durchschnittspreis aller Artikel für jede Zeile anzeigt.
```sql
SELECT 
    Artikel_ID, 
    Artikelname, 
    Preis, 
    (SELECT AVG(Preis) FROM Artikel) AS Durchschnittspreis
FROM Artikel;

```




**FROM-Klausel**: Unterabfragen können als Tabellenquellen innerhalb der `FROM`-Klausel verwendet werden. Dies wird als "Inline-View" oder "Derived Table" bezeichnet.
- liefert eine temporäre/ virtuelle/ abgeleitet Tabelle, die mit Joinsbedingung mit weiter Trabllen verknüpft werden muss 

Die Unterabfrage erstellt eine temporäre Tabelle (abgeleitete Tabelle), die dann in der Hauptabfrage verwendet wird.
Ergebnis: Nur Artikel mit einem Preis über 50 werden in der temporären Tabelle "TeuerArtikel" aufgenommen und weiterverarbeitet.
```sql 
SELECT 
    Artikelname, 
    Preis
FROM (
    SELECT 
        Artikel_ID, 
        Artikelname, 
        Preis 
    FROM Artikel 
    WHERE Preis > 50
) AS TeuerArtikel;
```




**WHERE-Klausel**: Unterabfragen können als Filter in der `WHERE`-Klausel verwendet werden, um spezifische Bedingungen zu erfüllen. Hier kommen sie häufig in Kombination mit Operatoren wie `IN`, `EXISTS`, `ANY`, `ALL` vor.
- Filtern von Zeilen
- die Zeilen nur für die Ergebnisse Tabelle ausgewählt , die die Bedingungen die sie in der Bedingung erfullen 

Die Unterabfrage filtert die Hauptabfrage basierend auf bestimmten Bedingungen.
Ergebnis: Nur Artikel, die von Kunde mit Kunden_ID = 1 bestellt wurden, werden angezeigt.
```sql
SELECT 
    Artikelname, 
    Preis
FROM Artikel
WHERE Artikel_ID IN (
    SELECT Artikel_ID 
    FROM Bestellungen 
    WHERE Kunden_ID = 1
);

```



**HAVING-Klausel**: Ähnlich wie in der `WHERE`-Klausel können Unterabfragen auch in der `HAVING`-Klausel verwendet werden, um aggregierte Bedingungen zu überprüfen.
    - Filtern von Zeilen

Die Unterabfrage wird verwendet, um Gruppenbedingungen zu prüfen.
Ergebnis: Zeigt Kunden an, deren Gesamtbestellwert über dem durchschnittlichen Gesamtwert aller Kunden liegt.
```sql
SELECT 
    Kunden_ID, 
    SUM(Bestellwert) AS Gesamtwert
FROM Bestellungen
GROUP BY Kunden_ID
HAVING SUM(Bestellwert) > (
    SELECT AVG(SUM(Bestellwert)) 
    FROM Bestellungen
    GROUP BY Kunden_ID
);
```



### 1.1.2 Funktion von Unterabfragen

Unterabfragen ermöglichen es, in einer einzigen SQL-Anweisung komplexe Berechnungen, Vergleiche oder Datenabrufe durchzuführen. Sie bieten die Möglichkeit, Ergebnisse einer Abfrage als Input für eine andere Abfrage zu verwenden, was die Lesbarkeit und Wartbarkeit des Codes verbessert.

### 1.1.3 Einfache Unterabfragen und korrelierten Unterabfragen
 
 **Einfache Unterabfragen**:
- **Definition**: Eine einfache Unterabfrage ist eine Unterabfrage, die unabhängig von der äußeren Abfrage ist. Sie wird einmal ausgeführt und liefert ein Ergebnis, das dann in der äußeren Abfrage verwendet wird.
```sql
SELECT name
FROM employees
WHERE salary > (
    SELECT AVG(salary) 
    FROM employees
);

```
Erklärung: Hier wird die einfache Unterabfrage verwendet, um das durchschnittliche Gehalt zu berechnen, und dieses Ergebnis wird dann mit dem Gehalt der Mitarbeiter verglichen. Die Unterabfrage wird nur einmal ausgeführt.


**Korrelierte Unterabfragen**
- **Definition**: Eine korrelierte Unterabfrage ist eine Unterabfrage, die sich auf Spalten der äußeren Abfrage bezieht. Sie wird für jede Zeile der äußeren Abfrage ausgeführt und liefert ein Ergebnis, das von den Daten der äußeren Abfrage abhängt.
```sql
SELECT name
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e1.department = e2.department
);

```
Erklärung: Hier bezieht sich die Unterabfrage auf die Spalte department der äußeren Abfrage. Für jede Zeile der äußeren Abfrage wird die Unterabfrage erneut ausgeführt und das durchschnittliche Gehalt für die entsprechende Abteilung berechnet.


 **Unterschiede zwischen einfachen und korrelierten Unterabfragen**
 一共有三个 用 chatgpt 找出来 
1. **Abhängigkeit von der äußeren Abfrage**:
    - **Einfache Unterabfrage**: Unabhängig von der äußeren Abfrage. Sie wird ==einmal== ausgeführt und liefert ein festes Ergebnis für die äußere Abfrage.
        - der äußeren Abfrage ist  abhängig nicht unterabfragen 
    - **Korrelierte Unterabfrage**: Hängt von der äußeren Abfrage ab. Sie wird ==für jede Zeile== der äußeren Abfrage ausgeführt und berechnet ein Ergebnis basierend auf den jeweiligen Werten der äußeren Abfrage.
2. **Leistung**:
    - **Einfache Unterabfrage**: Da sie nur einmal ausgeführt wird, kann die Leistung besser sein, besonders wenn es sich um große Datenmengen handelt.
    - **Korrelierte Unterabfrage**: Da sie für jede Zeile der äußeren Abfrage ausgeführt wird, kann sie langsamer sein und mehr Ressourcen benötigen, besonders bei großen Tabellen oder komplexen Berechnungen.
3.  **Komplexität und Anwendungsfall**:
    - **Einfache Unterabfrage**: Wird häufig verwendet, wenn eine feste Berechnung oder ein fester Vergleichswert benötigt wird, der nicht von der äußeren Abfrage abhängt.
    - **Korrelierte Unterabfrage**: Wird verwendet, wenn das Ergebnis der Unterabfrage für jede Zeile der äußeren Abfrage unterschiedlich ist und sich dynamisch ändern kann.





## 1.2 Nullwert?

2 Was ist ein Nullwert? Worin besteht der Unterschied zu „0“ und „ „? Woraus resultiert ein Nullwert? Wie können Sie das Auftreten von Nullwerten unterbinden? Erläutern Sie dies bitte kurz an einem selbstgewählten Beispiel! (5 Punkte)


NULL = Wert ist nicht definiert, NULL ist ungleich "0"
Leerzeichen ist Ist ein string oder Varchar, nicht NULL

 Woraus resultiert ein Nullwert
 - keine eingabe, nicht nullwert, nicht Default-wert 


Wie können Sie das Auftreten von Nullwerten unterbinden
- Insert into ... Values ('xyz', NULL)
- insert into kunde (kdnr,ort) values (100,'Berlin'),   kunde 还有其他的 attribute, 这里只写了 kdnr,ort, 其他没写的 attribute 都自动 置成 NULL 

Wie schaffe ich es bei der Kundennummer dass dort ein Nullwert nicht erlaubt ist
- beim primärschlüssel dann PRIMARY KEY  , Kundennummer als   PRIMARY KEY  ,    PRIMARY KEY   darf nicht als Null gestezt werden 
- oder beim Definition der Tabelle  durch   NOT NULL 


wo setze ich denn Primärschlüssel oder normal in welchen Statements
- beim Create Table  und Alter tabel


## 1.3 rekursiver (unärer) Relationshiptyp


3 . Wie kann ein rekursiver (unärer) Relationshiptyp grundsätzlich auf das Relationale Datenmodell abgebildet werden? Beschreiben Sie dies bitte kurz und verwenden Sie ein eigenes Beispiel für die Darstellung. (5 Punkte)

Ein **rekursiver (unärer) Relationshiptyp** beschreibt eine Beziehung zwischen Entitäten derselben Entitätsmenge. Um diese in ein relationales Datenmodell zu überführen, wird eine **Fremdschlüsselspalte** in der Tabelle eingefügt, die auf den Primärschlüssel derselben Tabelle verweist.

Vorgehen zur Abbildung**
1. **Erstellung der Tabelle**: Die Entität wird als Relation (Tabelle) modelliert.
2. **Primärschlüssel (PK)**: Ein eindeutiger Identifikator für jede Zeile.
3. **Fremdschlüssel (FK)**: Eine zusätzliche Spalte verweist auf den Primärschlüssel derselben Tabelle, um die Beziehung zu speichern.
4. **Zusätzliche Attribute**: Weitere relevante Spalten der Entität können hinzugefügt werden.


### 1.3.1 Beispiel: Mitarbeiter-Hierarchie (Vorgesetzte & Untergebene)**

Ein Unternehmen speichert Informationen über seine **Mitarbeiter**. Jeder Mitarbeiter kann einen **Vorgesetzten** haben, der ebenfalls ein Mitarbeiter ist. Dies stellt eine **rekursive 1:N-Beziehung** dar (ein Vorgesetzter hat mehrere Untergebene, ein Mitarbeiter hat maximal einen Vorgesetzten).

![](image/Pasted%20image%2020250130000009.png)

- `MitarbeiterID` ist der Primärschlüssel.
- `VorgesetzterID` ist ein **Fremdschlüssel**, der auf `MitarbeiterID` verweist.
- Alice (CEO) hat keinen Vorgesetzten (`NULL`).
- Bob berichtet an Alice, Carol an Bob, David an Carol.

# 2 SQL

Notieren Sie bitte die korrekten SQL-Befehle für die folgenden Aufgabenstellungen, die sich auf die Datenbanken mat_inf und company beziehen:


## 2.1 

Entwickeln Sie bitte eine Abfrage in Bezug auf die Datenbank mat_inf, die folgende Information enthält: 
Es soll je Auftrag die insgesamt zu entrichtende Mehrwertsteuer ausgegeben werden (Gehen Sie dazu von konstant 16% aus). Für den Auftrag genügt die Anzeige der Auftragsnummer. Die Anzeige soll nach der Mehrwertsteuer absteigend sortiert werden. Die Spalteninhalte sollen aus den Spaltenbezeichnungen eindeutig hervorgehen. (5 Punkte)



## 2.2 

Entwickeln Sie bitte eine Abfrage in Bezug auf die Datenbank mat_inf, die folgende Information enthält: 
Zu ermitteln sind der Name, die Vertreternummer und die Provision der Vertreter, deren Provision niedriger ist als die Provision irgendeines Vertreters aus dem Stadtbezirk mit der PLZ „12487“. (5 Punkte)


## 2.3 

Entwickeln Sie bitte eine Abfrage in Bezug auf die Datenbank company, die folgende Information enthält: 
Es sollen die Nummern aller Teile angezeigt werden, die von mehr als einem Lieferanten geliefert worden sind. (5 Punkte)


## 2.4 

Geben Sie das Ergebnis der folgenden Abfrage in der Datenbank company an

```
SELECT suppname
FROM supplier
WHERE suppno IN
    (SELECT suppno
    FROM supp_part
    WHERE partno IN
        (SELECT partno
        FROM part
        WHERE color = 'Red'));
```


# 3 Datenbankentwurf


Gegenstand von Aufgabe 9 ist die Verwaltung von Materialien für ein Fernstudium Informatik an einer Hochschule:
Ausgegangen werden soll von Lehreinheiten als größeren, inhaltlich orientierten Studienkomplexen in einem Hochschulfernstudium. Als Lehreinheiten existieren beispielsweise „Datenbanksysteme“, „Internettechnologien“ etc.

Zu jeder Lehreinheit werden momentan jeweils 8 Kurseinheiten angeboten sowie ein Praktikum als weitere Kurseinheit. Kurseinheiten der Lehreinheit „Datenbanksysteme“ sind beispielsweise „Einführung in SQL“ oder „XML und Datenbanken“.

Zu jeder Kurseinheit, mit Ausnahme des Praktikums, wird jeweils ein Lehrheft angeboten. Ein Lehrheft gehört genau zu einer Kurseinheit. Ein Lehrheft umfasst verschiedene Abschnitte mit jeweils einer Hauptüberschrift und dem eigentlichen Text. Je Abschnitt werden in variabler Anzahl Übungsaufgaben und zu diesen wiederum Beispiellösungen in variabler Anzahl angeboten. Abschnitte sind hierarchisch gegliedert, d.h. sie können auch Unterabschnitte enthalten. Ein Unterabschnitt gehört genau zu einem übergeordneten (Haupt-)Abschnitt. Die Übungsaufgaben sind eindeutig einem entsprechenden Lehrheftabschnitt zugeordnet.



Gehen Sie bitte von nachfolgend dargestelltem Sachverhalt aus und beantworten Sie die Fragen:
9.1 Bitte entwickeln Sie das Datenmodell für die Abbildung des Sachverhaltes. Stellen Sie das Datenmodell in Form eines ER-Diagramms mit (min,max)-Notation dar. Die Attribute können in dieser Darstellung weggelassen werden. (12 Punkte)

9.2 Leiten Sie aus dem ER-Diagramm aus 9.1 normalisierte Relationen ab. Definieren Sie dazu die entsprechenden Primär- und Fremdschlüssel. Ordnen Sie bitte alle angesprochenen Attribute zu und entwickeln sie, wenn nötig, eigene Attribute. Je Relation soll mindestens ein Nichtschlüsselattribut existieren. Notieren Sie die Relationen in Relationenschreibweise. (12 Punkte)

9.3 Entwickeln Sie bitte für die aus 9.2. resultierende(n) Relation(en) für die Abbildung von Lehrheften mit entsprechenden Abschnitten den (die) entsprechenden „create table“-Befehl(e), mit Abbildung der Primär- und Fremdschlüssel. (10 Punkte)

9.4 Es soll nachträglich die Relation für die Abbildung von Lehrheften ergänzt werden um die Angabe eines Autorennamens über eine zusätzliche Spalte. Diese soll eine variabel lange Zeichenkette von max. 30 Zeichen aufnehmen können. Alle existierenden Lehrhefte wurden bisher von dem Autor „Max Meier“ publiziert. Diese Angabe soll in alle verfügbaren Zeilen eingetragen werden. Setzen Sie dies bitte mit SQL um. (6 Punkte)




