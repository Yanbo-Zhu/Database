
# 1 Theoretische Fragen


## 1.1 die referenzielle Integrität

Welche kritischen Operationen können die referenzielle Integrität in einem relationalen DBS gefährden? 

Erläutern Sie bitte 2 dieser Operationen anhand des Beispiels: Gegeben sind die Relationen AuftragsPosition (aufnr, posnr, menge, ArtikelStamm->artnr) und ArtikelStamm (artnr, artbez, ekpreis). (5 Punkte)


---

答案: 
der Fremdschlüssel ist ja eine integritätsbedingung und gehört zu diesen ja Modell Integritätsbedingungen

1 Delete operation
Löschen eines Primärschlüssels in der referenzierten Tabelle

2 Update operation
- Ändern eines Primärschlüssels in der referenzierten Tabelle
- Updaten des Fremdschlüssels auf einen nicht existierenden Primärschlüssel


3 Insert operation 
Erstellen von Auftragsposition mit Fremdschlüssel, der keinen Datensatz in ArtikelStamm referenziert


## 1.2 intersect


Welche Aussage ist mit dem SQL-Operator „intersect“ generell möglich? Unter welchen (zwei) Voraussetzungen kann dieser Operator eingesetzt werden? Welche Aussage produziert die folgende SQL-Anweisung, wenn gilt, dass die Namen aller Geschäftspartner eindeutig sind? (5 Punkte)
```sql
Select name from kunde
Intersect
Select name from lieferant;
```




Intersect" Schnittmenge zwischen zwei SELECT Anfragen
Union: Beim Union: gleich Datensatz nicht zweimal auftauchen, Duplikat nicht vorzeigen  
Unioin ALL : Duplikat kann vorzeigen  
Minus/Except:  Differenzmenge minus oder except

---

Welche Aussage ist mit dem SQL-Operator „intersect“ generell möglich? 
- Schnittmenge zwischen zwei SELECT Anfragen

---

Unter welchen (zwei) Voraussetzungen kann dieser Operator eingesetzt werden? 
-  Gleiche Anzahl Spalten
-  kompatible Datentypen: Zweiten müssen die korrespondierenden Spalten den kompatible Datentypen aufweisen insofern also immer genau aufpassen .

---


Welche Aussage produziert die folgende SQL-Anweisung, wenn gilt, dass die Namen aller Geschäftspartner eindeutig sind? (5 Punkte)

```
Select name from kunde
Intersect
Select name from lieferant;
```

Gibt die Namen aller Personen zurück, die sowohl Kunden als auch Lieferanten sind



# 2 SQL

Notieren Sie bitte die korrekten SQL-Befehle für die folgenden Aufgaben-stellungen, die sich auf die Datenbanken mat_inf und company beziehen:

## 2.1 
Entwickeln Sie bitte eine Abfrage auf der Datenbank mat_inf, die die gleiche Ergebnistabelle liefert, wie die nachfolgend dargestellte Abfrage. Entwickeln Sie eine Lösung mit Unterabfrage! (5 Punkte)
```
select ort from kdst
except
select ort from vert;
```

SELECT ort FROM kdst WHERE ort NOT IN (SELECT ort FROM vert)

Die Orte von kdst, die nicht in vert vorkommen


## 2.2 

Entwickeln Sie bitte eine Abfrage auf der Datenbank mat_inf, die folgende Information enthält: 

Es sollen die Firmenbezeichnung und die Kundennummer derjenigen Kunden gefunden werden, von denen keine Aufträge stammen. (5 Punkte)


## 2.3 

Geben Sie das Ergebnis der folgenden Abfrage in der Datenbank company an (5 Punkte):
```
SELECT suppname
FROM supplier
WHERE suppno IN
    (SELECT suppno
    FROM supp_part
    WHERE partno IN
        (SELECT partno
        FROM part
        WHERE color = 'Blue'));
```

## 2.4 ##

Entwickeln Sie bitte eine Abfrage auf der Datenbank company, die folgende Information enthält: 
Es sollen die Abteilungsnummer, das mittlere Gehalt und die mittlere Kommission für alle Abteilungen aufgelistet werden, in denen die mittlere Kommission größer ist als 25% des mittleren Gehalts. Nehmen Sie bitte eine Bezeichnung der Kalkulationsspalten vor. (5 Punkte)




# 3 Datenbankentwurf


Gegenstand von Aufgabe 9 ist die Verwaltung von Materialien für ein Fernstudium Informatik an einer Hochschule:

Ausgegangen werden soll von Lehreinheiten als größeren, inhaltlich orientierten Studienkomplexen in einem Hochschulfernstudium. Als Lehreinheiten existieren beispielsweise „Datenbanksysteme“, „Internettechnologien“ etc. Lehreinheiten werden von verantwortlichen Herausgebern betreut.

Zu jeder Lehreinheit werden momentan jeweils 8 Kurseinheiten angeboten sowie ein Praktikum als weitere Kurseinheit. Kurseinheiten der Lehreinheit „Datenbanksysteme“ sind beispielsweise „Einführung in SQL“ oder „XML und Datenbanken“.

Zu jeder Kurseinheit, mit Ausnahme des Praktikums, wird jeweils ein Lehrheft angeboten. Ein Lehrheft gehört genau zu einer Kurseinheit. 
Lehrhefte werden von Autoren entwickelt, die nicht mit dem Herausgeber der Lehreinheit übereinstimmen müssen (andere Personen sind möglich).

Ein Lehrheft umfasst verschiedene Abschnitte mit jeweils einer Hauptüberschrift und dem eigentlichen Text. Abschnitte sind hierarchisch gegliedert, d.h. sie können auch Unterabschnitte enthalten. Ein Unterabschnitt gehört genau zu einem übergeordneten (Haupt-) Abschnitt.

Für die Autoren und Herausgeber 发行人 sind neben den persönlichen Adressdaten auch die Koordinaten der Institution, bei der sie beschäftigt sind, zu speichern.


----


Gehen Sie bitte von nachfolgend dargestelltem Sachverhalt aus und beantworten Sie die unten angegebenen Fragen:
9.1 Bitte entwickeln Sie das Datenmodell für die Abbildung des Sachverhaltes. Stellen Sie das Datenmodell in Form eines ER-Diagramms mit (min,max)-Notation dar. Die Attribute können in dieser Darstellung weggelassen werden. (12 Punkte)

----


9.2 Leiten Sie aus dem ER-Diagramm aus 9.1 normalisierte Relationen ab. Definieren Sie dazu die entsprechenden Primär- und Fremdschlüssel. Ordnen Sie bitte alle angesprochenen Attribute zu und entwickeln sie, wenn nötig, eigene Attribute. Je Relation soll mindestens ein Nichtschlüsselattribut existieren. Notieren Sie die Relationen in Relationenschreibweise. (12 Punkte)

Lehreinheit
(idlehreinheit, bezeichnung, idherausgeber - > herausgeber.idherausgeber)

kurseinheit
(idkurseinheit, idlehreinheit -> lehreinheit.idlehreinheit, bezeichnung)

lehrheft
(idlehrheft, idkurseinheit -> kurseinheit.idkurseinheit, bezeichnung, idautor -> autor.idautor)

abschnitt
(idabschnitt, idlehrheft -> lehrheft.idlehrheft,  übergeordnete_abschnitt -> abschnitt.idabschnitt , hauptüberschrift, text)

übungsaufgabe
(iduebungsaufgabe, idabschnitt-> abschnitt.idabschnitt,  text )

beispiellösunge
(idbeispielloesung, iduebungsaufgabe-> übungsaufgabe.iduebungsaufgabe,  text )


herausgeber
(idherausgeber, idperson - > person.idperson, ... )

autor
(idherautor, idperson - > person.idperson, ... )


person
(idperson, address, idinsititut - > insititut.idinsititut )

insititut
(idinsititut, ... )

---

9.3 Entwickeln Sie bitte für die aus 9.2. resultierende(n) Relation(en) für die Abbildung von Lehrheften mit entsprechenden Abschnitten den (die) entsprechenden „create table“-Befehl(e), mit Abbildung der Primär- und Fremdschlüssel. (10 Punkte)

```sql
CREATE TABLE Institution (
    InstID INT PRIMARY KEY,
    Name VARCHAR(100) NOT NULL,
    Adresse VARCHAR(255),
    PLZ VARCHAR(10),
    Stadt VARCHAR(100),
    Koordinaten VARCHAR(50)
);

CREATE TABLE Person (
    PID INT PRIMARY KEY,
    Name VARCHAR(100) NOT NULL,
    Adresse VARCHAR(255),
    PLZ VARCHAR(10),
    Stadt VARCHAR(100),
    InstitutionID INT,
    Rolle ENUM('Autor', 'Herausgeber'),
    FOREIGN KEY (InstitutionID) REFERENCES Institution(InstID)
);

herausgebe ()

autor ()


CREATE TABLE Lehreinheit (
    LEID INT PRIMARY KEY,
    Titel VARCHAR(255) NOT NULL,
    HerausgeberID INT NOT NULL,
    FOREIGN KEY (HerausgeberID) REFERENCES Person(PID)
);

CREATE TABLE Kurseinheit (
    KEID INT PRIMARY KEY,
    Titel VARCHAR(255) NOT NULL,
    LEID INT NOT NULL,
    Typ ENUM('Theorie', 'Praktikum'),
    FOREIGN KEY (LEID) REFERENCES Lehreinheit(LEID)
);

CREATE TABLE Lehrheft (
    LHID INT PRIMARY KEY,
    Titel VARCHAR(255) NOT NULL,
    KEID INT NOT NULL,
    AutorID INT NOT NULL,
    FOREIGN KEY (KEID) REFERENCES Kurseinheit(KEID),
    FOREIGN KEY (AutorID) REFERENCES Person(PID)
);

CREATE TABLE Abschnitt (
    AID INT PRIMARY KEY,
    Titel VARCHAR(255) NOT NULL,
    Text TEXT NOT NULL,
    LHID INT NOT NULL,
    ParentAID INT,
    FOREIGN KEY (LHID) REFERENCES Lehrheft(LHID),
    FOREIGN KEY (ParentAID) REFERENCES Abschnitt(AID)
);

```


9.4 Herausgeber und Autor sind Personen. Konsequenterweise sollte hier eine Generalisierung verwendet werden. Stellen Sie die Generalisierung mit den Mitteln des EERM dar und begründen Sie kurz, für welche Art von Generalisierungshierarchie Sie sich entscheiden. (6 Punkte)


**Begründung:**
- **Herausgeber und Autoren sind Spezialisierungen der Entität "Person".**
- **Gemeinsame Attribute** (Name, Adresse, Institution) werden in `Person` gespeichert.
- **Die Generalisierung ist eine vollständige und überlappende Hierarchie,** weil eine Person gleichzeitig Herausgeber & Autor sein kann.

**EERM-Darstellung:**

- `Person` (Generalisiert) → `{Herausgeber, Autor}` (Spezialisierungen).
- Generalisierungstyp: **Erweitert (overlapping)**


?? 
