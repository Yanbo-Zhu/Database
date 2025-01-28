
# 1 Theoretische Fragen


## 1.1 

Durch welche Eigenschaften ist ein Primärschlüssel im relationalen Datenmodell charakterisiert? Bestimmen Sie den Primärschlüssel für die Relation Auftrag, wenn folgendes gilt:


Auftrag(AID, APosition, KdID, KdName, KdStrasse, KdPLZ, KdOrt, Taetigkeit, Arbeitszeit, Preis)
Funktionale Abhängigkeiten:
{AID} -> {KdID}
{KdID} -> {KdName, KdStrasse, KdPLZ, KdOrt }
{AID, APosition} ->  {Taetigkeit, Arbeitszeit }
{Taetigkeit, Arbeitszeit}  ->  {Preis}

Die Relation soll in der ersten Normalform vorliegen!

--- 

解答

{AID} -> {KdID}
AID ist primar key 

{KdID} -> {KdName, KdStrasse, KdPLZ, KdOrt }
KdID muss nicht primar key , falls uns nur 1 NF folgen.  wenn 2 nf KdID muss primar key sein


{AID, APosition} ->  {Taetigkeit, Arbeitszeit }
APosition muss auch primar key .  as kombinierte primar keys, 

{Taetigkeit, Arbeitszeit}  ->  {Preis}
不需要 {Taetigkeit, Arbeitszeit}  als primar key, 因为 {AID, APosition} -> Preis 

---
wenn 2 NF vorliegen 

Auftrag table und Kunde table 
Auftragposition  table 

Auftrag table {AID, KdID}

Kunde table  {KdID, KdName, KdStrasse, KdPLZ, KdOrt }

auftragsposition
{AID, APosition , Taetigkeit, Arbeitszeit }

当我们 3 NF einhalten  
Die Relation soll in der ersten Normalform vorliegen!
还需要一个 table Preis 
{Taetigkeit, Arbeitszeit, Preis}


## 1.2 

Wie sieht die Ergebnismenge dieser beiden Tabellen nach einer „union all“-Operation aus? Zeichnen Sie die Ergebnistabelle.

![](image/Pasted%20image%2020250128164641.png)


union : Duplikate Tupel wird eliminiert.  keine dopplung




## 1.3 

Was wird durch den folgenden Trigger realisiert und wie sieht die Tabelle status_log aus? 
Beschreiben Sie bitte, wie die Tabelle mit SQL angelegt wird! Primär- und Fremdschlüssel müssen Sie dabei nicht definieren!

```
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

spalten: auftragsnummer, alter status, neuer status, aktuelles datum?
Reihnfolge 必须尊重,   这四个 Spalte 之后 原来的 table 有更多的 spalte  是可以的 



---



Zusatzaufgabe: Wie kann der Trigger ohne Verlust seiner Funktionalität vereinfacht werden? (2 Zusatzpunkte)




## 1.4 

Warum normalisiert man Datenbankschemata? Bitte nennen Sie mindestens drei Gründe für die Durchführung der Normalisierung. Erläutern Sie kurz eine der Normalformen mit ihren Worten.


# 2 Abfragen mit SQL / Relationenalgebra


Die folgenden Aufgaben beziehen sich auf die Datenbanken mat_inf oder company. Notieren Sie bitte die korrekten SQL-Befehle. Beachten Sie bitte, dass Ihnen nur die in der jeweiligen Aufgabenstellung gegebenen Informationen vorliegen:

## 2.1 
Von denjenigen Aufträgen, deren gelieferte Menge kleiner als die Bestellmenge ist, sollen die Auftragsnummer, ein konstanter Text "noch offen" als Spalte "MengenStatus" sowie der Text des Auftragsstatus angezeigt werden. Diese Anzeige soll je Auftrag nur einmal erfolgen, selbst wenn die o.g. Selektionsbedingung für mehrere Positionen des Auftrags zutrifft (vgl. nachfolgende Ergebnistabelle):

![](image/Pasted%20image%2020250128165009.png)



## 2.2 

Angezeigt werden sollen (DB mat_inf) der Vertretername, die Stadt, der Stadtteil und der durchschnittliche Umsatz im Stadtteil Johannisthal, auf 2
Nachkommastellen (NK) gerundet. (Hinweis: Funktion `round(<wert>,<anzahlNKStellen>)`) .

![](image/Pasted%20image%2020250128165037.png)


## 2.3 ##

Entwickeln Sie bitte eine SQL-Abfrage auf der DB company, die folgende Information liefert: Es sollen alle Mitarbeiter, deren Job Manager ist, ausgegeben werden, sortiert nach dem Namen, die im RESEARCH-Department angestellt sind.


![](image/Pasted%20image%2020250128165048.png)


## 2.4 ##

In welchen Berufen (DB company) erhält ein Mitarbeiter keine Provision (Anzeige als Wert 0)? Jeder Beruf soll nur einmal gelistet werden. Hinweis: Funktion zur Nullwertbehandlung `nvl(<wert>,<Anzeigewert bei Nullwert>)`

![](image/Pasted%20image%2020250128165110.png)

## 2.5 ##

Zusatzaufgabe: Zusätzlich soll angezeigt werden, in welchen Berufen erhalten die Mitarbeiter Provision? 
Jeder Beruf soll nur einmal gelistet werden und je Beruf soll die durchschnittliche Provision angezeigt werden. (3 Punkte)

![](image/Pasted%20image%2020250128165123.png)


# 3 DB-Entwicklungsaufgabe

Gehen Sie bitte von nachfolgend dargestelltem Sachverhalt aus und beantworten Sie die Fragen:

9.1 Bitte entwickeln Sie das Datenmodell für die Abbildung des Sachverhaltes. Beziehen Sie dabei auch den Katalog als Objekttyp ein. Stellen Sie das Datenmodell in Form eines ER-Diagramms dar. Machen Sie die Beziehungen entsprechend der in der Aufgabenstellung geforderten Kardinalitäten deutlich. Die Attribute können in dieser Darstellung weggelassen werden. Beachten Sie, dass die eingefügten Beispieltabellen 1 und 2 nur zur Illustration dienen und keine verwendbaren Relationen darstellen. (13 Punkte)

9.2 Leiten Sie aus dem ER-Diagramm (aus 9.1) Relationen gemäß der Ihnen bekannten Transformationsregeln ab. Notieren Sie die Relationen in Relationenschreibweise. Definieren Sie zu jeder Relation Primär- und Fremdschlüssel. Machen Sie die Fremdschlüssel kenntlich (z.B. Unterstreichung oder Hervorhebung). Ordnen Sie bitte alle angesprochenen Attribute zu. (17 Punkte)

9.3 Entwickeln Sie bitte für die Relationen „Produktvariation“ und "Motiv" die entsprechenden „create table“-Befehle, mit Abbildung der Primär- und Fremdschlüssel. Die weiteren (auch referenzierten) Relationen sind nicht zu entwickeln, aber zu referenzieren! Entwickeln Sie für einen ausgewählten Fremdschlüssel die Option für „on delete“ und begründen Sie kurz. (12 Punkte)

9.4 Entwickeln Sie bitte den SQL-Befehl für das Anlegen eines Datensatzes für die in 9.3 erzeugte Tabelle „Motiv“. Verwenden Sie dazu Daten aus den Beispieltabellen. (5 Punkte)
muss man da wirklich nur das 1 INSERT für das eine Motiv screiben? : 答案是 ja , 就是说要写多个 insert 

9.5 Nachträglich soll in die Relation „Motiv“ noch das eigentliche Bild aufgenommen werden. Wie muss das SQL-Statement dafür aussehen, wenn diese Spalte noch nicht existiert? (5 Punkte)

---

Gegenstand der vorliegenden Aufgabe ist ein Produktkatalog mit -varianten:
Ein namhaftes Versandhaus bietet im Rahmen seiner verschiedenen Kataloge (young fashion, modern fashion, woman’s fashion) Produkte häufig als sogenannte Produktvariationen an, die der Kunde bestellen kann. Das heißt, dass ein bestimmtes Produkt, z.B. ein Shirt eines bestimmten Herstellers aus einem bestimmten Material, in verschiedenen Produktvariationen angeboten werden kann. Ein Basisprodukt besteht dabei aus genau einem Material, wie im Beispiel „Baumwolle“.
Eine Produktvariation zu einem Basisprodukt besteht aus einer gewählten Farbe, einer Größe und einem Druckmotiv. Die Preisbildung orientiert sich – neben dem Produkt - aussschließlich an der Größe. Abgebildet wird nachfolgend beispielhaft ein mögliches Basisprodukt mit einigen möglichen Produktvariationen:
![](image/Pasted%20image%2020250128165218.png)

Je Druckmotiv sollen, neben dessen Kurzbezeichnung, in der Datenbank auch eine Motivbeschreibung abgelegt werden. Je Größe werden Maßangaben und Toleranzen (als Text) sowie eine Kurzbeschreibung abgespeichert. Je Farbe werden der Farbcode, ein Farbkürzel sowie eine Beschreibung verwendet.

Zur Bestellung von Produkten aus den Katalogen muss die Produktvariation abgebildet werden, wie z.B.

![](image/Pasted%20image%2020250128165226.png)


