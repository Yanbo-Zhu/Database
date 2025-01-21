# 1 Freitext Aufgabe 



Gegenstand der vorliegenden Aufgabe ist die Verwaltung von Lehrmaterialien für ein Fernstudium Informatik an einer Hochschule:

Ausgegangen werden soll von Lehreinheiten als größeren, inhaltlich orientierten Studienkomplexen in einem Hochschulfernstudium. Als Lehreinheiten existieren beispielsweise „Datenbanksysteme“, „Internettechnologien“ etc.

Zu jeder Lehreinheit werden momentan jeweils 8 Kurseinheiten angeboten sowie ein Praktikum als weitere Kurseinheit. Kurseinheiten der Lehreinheit „Datenbanksysteme“ sind beispielsweise „Einführung in SQL“ oder „XML und Datenbanken“.

Zu jeder Kurseinheit, mit Ausnahme des Praktikums, wird jeweils ein Lehrheft angeboten. Ein Lehrheft gehört genau zu einer Kurseinheit. Ein Lehrheft umfasst verschiedene Abschnitte mit jeweils einer Hauptüberschrift und dem eigentlichen Text. Je Abschnitt werden in variabler Anzahl Übungsaufgaben und zu diesen wiederum Beispiellösungen in variabler Anzahl angeboten. Abschnitte sind hierarchisch gegliedert, d.h. sie können auch Unterabschnitte enthalten. Ein Unterabschnitt gehört genau zu einem übergeordneten (Haupt-)Abschnitt. Die Übungsaufgaben sind eindeutig einem entsprechenden Lehrheftabschnitt zugeordnet.

----



Bitte beachten Sie: Diese Aufgabe muss offline bearbeitet werden (Blatt Papier).

Sie bezieht sich mit den genannten Teilaufgaben auf den nachfolgend dargestellten Sachverhalt.

**FT1.** Entwickeln Sie bitte das Datenmodell für die Abbildung des Sachverhaltes. Stellen Sie das Datenmodell in Form eines ER-Diagramms dar, welches sinnvoll normalisiert ist. Machen Sie die Beziehungen entsprechend der in der Aufgabenstellung geforderten Kardinalitäten deutlich. Die Attribute können in dieser Darstellung weggelassen werden.  (**12 Punkte**).

**FT2.** Leiten Sie aus dem ER-Diagramm (aus **FT1**) Relationen gemäß der Ihnen bekannten Transformationsregeln ab. Notieren Sie die Relationen gerne in Tupelschreibweise. Definieren Sie zu jeder Relation Primär- und Fremdschlüssel. Machen Sie die Primär- und Fremdschlüssel kenntlich (z.B. Unterstreichung oder Hervorhebung). Ordnen Sie bitte alle angesprochenen Attribute zu (**12 Punkte**).

**FT3.** Entwickeln Sie bitte für die aus **FT2** resultierende(n) Relation(en) für die Abbildung von Lehrheften mit entsprechenden Abschnitten den (die) entsprechenden „create table“-Befehl(e), mit Abbildung der Primär- und Fremdschlüssel. **(10 Punkte)**

**FT4**. Es soll nachträglich die Relation für die Abbildung von Lehrheften ergänzt werden um die Angabe eines Autorennamens über eine zusätzliche Spalte. Diese soll eine variabel lange Zeichenkette von max. 30 Zeichen aufnehmen können. Alle existierenden Lehrhefte wurden bisher von dem Autor „Max Meier“ publiziert. Diese Angabe soll in alle verfügbaren Zeilen eingetragen werden. Setzen Sie dies bitte mit SQL um. **(6 Punkte)**


# 2 Choice


Was sind die Modellelemente des Relationalen Datenmodells?

Wählen Sie eine oder mehrere Antworten:
- Relation
- Relationshiptyp
- Adresse
- Primärschlüssel
- Fremdschlüssel
- Attribut

Korrekte Antworten:
- **Relation**  
    (Eine Tabelle, die Daten in Zeilen und Spalten organisiert.)

- **Attribut**  
    (Eine Spalte in einer Tabelle, die einen bestimmten Datentyp darstellt und eine Eigenschaft eines Objekts beschreibt.)


Falsche Antworten
- **Relationshiptyp:**  
    Dies ist ein Konzept des Entity-Relationship-Modells (ERM), nicht des relationalen Datenmodells.
- **Adresse:**  
    Dies ist ein Beispiel für ein Attribut, aber kein Modellelement des relationalen Datenmodells.
- **Primärschlüssel**  
    (Ein Attribut oder eine Kombination von Attributen, das jede Zeile in einer Relation eindeutig identifiziert.)
- **Fremdschlüssel**  
    (Ein Attribut oder eine Kombination von Attributen, das auf den Primärschlüssel einer anderen Relation verweist und Beziehungen zwischen Tabellen darstellt.)

----

Welche Speichermedien sind hinsichtlich der Zugriffsgeschwindigkeit die Schnellsten ? Beginnen Sie mit dem Schnellsten links

![](image/Pasted%20image%2020250115093414.png)

选对了 就这么选

---

Welche Aufgaben hat ein Datenbankmanagementsystem?
- Datensätze dauerhaft speichern
- nur diejenigen Operationen erlauben, die nicht die Integrität der Datenbank gefährden
- Buchführung über die verwendeten Karteikarten vornehmen
- einen optimalen Entwurf des Schemas garantieren
- dafür sorgen, dass nur berechtigte Nutzer die Daten anfragen dürfen
- das passende Speichermedium für besonders schnelle Anfragen finden


Korrekte Antworten
1. **Datensätze dauerhaft speichern**
    - Das DBMS ist dafür zuständig, Daten persistent zu speichern, sodass sie auch nach einem Systemabsturz erhalten bleiben.
2. **Nur diejenigen Operationen erlauben, die nicht die Integrität der Datenbank gefährden**
    - Ein DBMS stellt sicher, dass Integritätsbedingungen (z. B. Primär- und Fremdschlüssel) eingehalten werden, um die Konsistenz der Datenbank zu bewahren.
3. **Dafür sorgen, dass nur berechtigte Nutzer die Daten anfragen dürfen**
    - Das DBMS verwaltet Zugriffskontrollen und Berechtigungen, um Daten vor unbefugtem Zugriff zu schützen.

Falsche Antworten
1. **Buchführung über die verwendeten Karteikarten vornehmen**
    - Das ist ein veralteter Ansatz zur Datenverwaltung und keine Aufgabe eines modernen DBMS.
2. **Einen optimalen Entwurf des Schemas garantieren**
    - Ein DBMS kann den Entwurf des Schemas unterstützen, aber es liegt in der Verantwortung des Datenbankdesigners, ein gutes Schema zu erstellen.
3. **Das passende Speichermedium für besonders schnelle Anfragen finden**
    - Ein DBMS nutzt vorhandene Hardware effizient, wählt jedoch nicht selbstständig Speichermedien aus. Diese Entscheidung liegt bei der Systemkonfiguration.

----

Wie heißen die vier Operationen, die den Umgang mit Daten in Datenbanken beschreiben?
- Create, Read, Unlock, Delete
- Create, Read, Update, Delete
- Create, Reset, Update, Delete
- Compile, Read, Update, Delete



---

Welche Schemata kommen in einer schichtenorientierten Datenbank-Architektur vor? 
- physisches oder internes Schema
- adressorientiertes Schema
- anwenderseitiges oder externes Schema
- konzeptuelles Schema
- logistisches Schema

In einer schichtenorientierten Datenbank-Architektur, wie sie z. B. im Drei-Schema-Modell definiert ist, kommen die folgenden Schemata vor:

1. **Physisches oder internes Schema** 
    - Beschreibt, wie die Daten physisch auf den Speichermedien organisiert sind. Es umfasst Details wie Speicherstrukturen, Dateiorganisation und Indexierung.
2. **Anwenderseitiges oder externes Schema**
    - Definiert die Sicht eines Benutzers oder einer Benutzergruppe auf die Daten. Es stellt sicher, dass Benutzer nur die für sie relevanten Daten sehen.
3. **Konzeptuelles Schema**
    - Beschreibt die gesamte logische Struktur der Datenbank aus einer globalen Perspektive. Es ist unabhängig von der physischen Speicherung und stellt die Daten so dar, wie sie für die Organisation sinnvoll sind.


----

Eine Transaktion kann nur aus einer Operation bestehen. Sie ist entweder vollständig auszuführen oder muss im Fehlerfall zurückgerollt werden.

Frage 7 Bitte wählen Sie eine Antwort:
Wahr
Falsch  选这个 

Die Aussage beschreibt das grundlegende Prinzip der **Atomarität** (A) im Zusammenhang mit Transaktionen in Datenbanksystemen. Eine Transaktion kann durchaus aus **mehreren Operationen** bestehen, die zusammen eine logische Einheit bilden. Daher ist die Aussage **falsch**.


**Erklärung:**
- Eine **Transaktion** ist eine logische Einheit von Datenbankoperationen, die zusammen ausgeführt werden müssen, um eine Konsistenzregel zu erfüllen.
- Die **ACID-Eigenschaften** einer Transaktion sind:
    - **A**tomarität: Entweder wird die gesamte Transaktion ausgeführt (commit), oder keine der Operationen (rollback).
    - **C**onsistency: Der Zustand der Datenbank bleibt konsistent.
    - **I**solation: Parallel ablaufende Transaktionen beeinflussen sich nicht.
    - **D**urability: Nach einem erfolgreichen Abschluss bleiben die Änderungen persistent.

---


Welche Komponenten der Datenbank-Architektur werden bei einer lesenden Anfrage eingesetzt? 

- Query-Prozessor
- Optimierer
- Integritätsprüfung
- Transaktionsmanager
- Logbuch
- Ein- / Ausgabe-Prozessor
- Update-Prozessor
- Autorisierungskontrolle

Korrekte Antworten
1. **Query-Prozessor**
    - Aufgabe: Interpretiert und verarbeitet die Abfrage, wandelt sie in ausführbare Operationen um und übermittelt sie an die Datenbank.
    - Begründung: Essenziell für jede Anfrage, da ohne den Query-Prozessor keine Abfrage ausgeführt werden kann.
2. **Optimierer**
    - Aufgabe: Optimiert die Anfrage, um die effizienteste Ausführungsstrategie zu finden (z. B. Wahl des besten Index oder der optimalen Join-Reihenfolge).
    - Begründung: Auch bei lesenden Anfragen wichtig, um die Leistung zu verbessern.
3. **Ein-/Ausgabe-Prozessor**
    - Aufgabe: Stellt sicher, dass die erforderlichen Daten aus dem physischen Speicher (z. B. Festplatte) abgerufen und an den Query-Prozessor übergeben werden.
    - Begründung: Notwendig, um auf die physischen Daten zuzugreifen.
4. **Autorisierungskontrolle**
    - Aufgabe: Überprüft, ob der Benutzer berechtigt ist, die angeforderten Daten zu lesen.
    - Begründung: Stellt sicher, dass nur autorisierte Benutzer Zugriff auf die Daten haben.
- 2. **Transaktionsmanager**
    - Aufgabe: Verwalten von Transaktionen, um ACID-Eigenschaften sicherzustellen.
    - Begründung: Wird  benötigt, wenn eine Transaktion eine Konsistenzgarantie erfordert (z. B. bei lesende oder konkurrierenden Zugriffen oder schreibenden Operationen).
3. **Logbuch**
    - Aufgabe: Protokolliert Änderungen an der Datenbank für Wiederherstellung und Fehlerbehebung.
    - Begründung: Wird  bei  lesende oder schreibenden Anfragen oder Transaktionen benötigt, nicht bei lesenden Anfragen.



Falsche Antworten
1. **Integritätsprüfung**
    - Aufgabe: Prüft die Einhaltung von Integritätsregeln (z. B. Fremdschlüssel oder Constraints).
    - Begründung: Wird primär bei schreibenden Anfragen (z. B. Insert, Update) benötigt, nicht bei lesenden.

4. **Update-Prozessor**
    - Aufgabe: Verarbeitet Änderungen an der Datenbank (z. B. Insert, Update, Delete).
    - Begründung: Ist für lesende Anfragen irrelevant, da keine Änderungen vorgenommen werden.


----

Worauf beziehen sich CRUD-Operationen?
- die Datenbank als Ganzes
- Dateien
- das Data Dictionary
- Karteikarten
- Datensätze  选这个 

Diese Operationen beziehen sich spezifisch auf die Manipulation und Verwaltung von **Datensätzen** innerhalb einer Datenbanktabelle.

Falsche Antworten:
1. **Die Datenbank als Ganzes:** CRUD bezieht sich nicht auf die gesamte Datenbank, sondern auf einzelne Datensätze in Tabellen.
2. **Dateien:** Dateien werden in der Regel nicht direkt durch CRUD-Operationen manipuliert, es sei denn, sie werden in der Datenbank gespeichert.
3. **Das Data Dictionary:** Das Data Dictionary ist eine Metadatenstruktur der Datenbank, die Informationen über Tabellen, Spalten usw. enthält. CRUD-Operationen beziehen sich nicht auf das Data Dictionary selbst.
4. **Karteikarten:** Karteikarten sind ein analoges Konzept und stehen nicht in direktem Zusammenhang mit CRUD-Operationen.


----

Redundanz bei Daten ist heute kein Problem mehr, denn die Speichermedien kosten sehr wenig und durch Dienste wie DropBox oder OwnCloud kann jederzeit die Synchronisation erfolgen.

Wahr
Falsch

选 Falsch

Erklärung
**Redundanz** in Datenbanken bedeutet, dass dieselben Daten mehrfach gespeichert werden. Dies kann zwar durch kostengünstige Speichermedien und Cloud-Dienste einfacher gehandhabt werden, bleibt aber aus folgenden Gründen problematisch:

1. **Dateninkonsistenz:**
    - Wenn redundante Daten nicht korrekt synchronisiert werden, können unterschiedliche Versionen derselben Information entstehen, was zu Fehlern führt.
2. **Wartungsaufwand:**
    - Die Verwaltung und Pflege redundanter Daten erhöhen den Aufwand, da alle Kopien konsistent gehalten werden müssen.
3. **Effizienz:**
    - Redundante Daten belasten Systeme, da mehr Speicherplatz benötigt wird und Abfragen durch unnötig große Datenmengen verlangsamt werden können.
4. **Integritätsprobleme:**
    - Redundanz erschwert die Einhaltung von Datenintegrität, insbesondere wenn Änderungen an mehreren Stellen vorgenommen werden müssen.


---


Bei welchen der folgenden Vorgänge handelt es sich um eine horizontale Verteilung bzw. Partitionierung der Datensätze?
- Die Kundendaten werden in einer Datenbank gehalten und die Auftragsdaten in einer anderen Datenbank. 
- Kunden, die 3 Jahre nicht bestellt haben, werden archiviert.
- Die Kundendaten werden in häufig genutzte Daten wie Name, Adresse und selten genutzte Daten wie Video- oder Bilddaten in verschiedenen Datenbanken verteilt.
- Die Kunden werden nach der Zuordnung zu Europa und USA in Datenbanken verteilt.


Analyse der Vorgänge:
1. **Die Kundendaten werden in einer Datenbank gehalten und die Auftragsdaten in einer anderen Datenbank.**
    - **Nicht horizontal:** Dies ist eine **vertikale Partitionierung**, da unterschiedliche Datenarten (Kunden vs. Aufträge) getrennt werden.
2. **Kunden, die 3 Jahre nicht bestellt haben, werden archiviert.**
    - **Horizontal:** Hier wird basierend auf einem zeitlichen Kriterium (Bestelldatum) eine horizontale Partitionierung durchgeführt. Aktive und archivierte Kunden werden getrennt.
3. **Die Kundendaten werden in häufig genutzte Daten wie Name, Adresse und selten genutzte Daten wie Video- oder Bilddaten in verschiedenen Datenbanken verteilt.**
    - **Nicht horizontal:** Dies ist eine **vertikale Partitionierung**, da verschiedene Attribute (häufig vs. selten genutzte Daten) getrennt werden.
4. **Die Kunden werden nach der Zuordnung zu Europa und USA in Datenbanken verteilt.**
    - **Horizontal:** Die Aufteilung der Kunden nach geografischen Kriterien ist ein klassisches Beispiel für horizontale Partitionierung.


