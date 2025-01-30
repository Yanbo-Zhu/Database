
![](image/Pasted%20image%2020250121214734.png)


- Intersect
    - Schnittmenge zwischen zwei SELECT Anfragen
- Union: 
    - Beim Union: gleich Datensatz nicht zweimal auftauchen, Duplikat nicht vorzeigen  
- Unioin ALL
    - Duplikat kann vorzeigen  
- Minus/Except: 
    - Differenzmenge minus oder except
    - Gibt **alle Zeilen aus der ersten Abfrage zurück, die NICHT in der zweiten Abfrage enthalten sind.**


Unter welchen (zwei) Voraussetzungen kann dieser Operator eingesetzt werden? 
-  Gleiche Anzahl Spalten
-  kompatible Datentypen: Zweiten müssen die korrespondierenden Spalten den kompatible Datentypen aufweisen insofern also immer genau aufpassen .




# 1 Intersect


Welche Aussage produziert die folgende SQL-Anweisung, wenn gilt, dass die Namen aller Geschäftspartner eindeutig sind? (5 Punkte)

```
Select name from kunde
Intersect
Select name from lieferant;
```

**"Gibt die Namen aller Personen oder Unternehmen zurück, die sowohl Kunden als auch Lieferanten sind."**

Das ein Geschäftspartner sowohl Lieferant als auch Kunde ist


---

Gibt nur die Zeilen zurück, die in beiden Abfragen vorkommen (gemeinsame Elemente).

```
🔹 **Ergebnis:** Alle Personen, die **sowohl Kunden als auch Mitarbeiter** sind.

SELECT name FROM customers
INTERSECT
SELECT name FROM employees;

```


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




# 2 Minus/Except


Entwickeln Sie bitte eine Abfrage auf der Datenbank mat_inf, die die gleiche Ergebnistabelle liefert, wie die nachfolgend dargestellte Abfrage. Entwickeln Sie eine Lösung mit Unterabfrage! (5 Punkte)
```
select ort from kdst
except
select ort from vert;
```

SELECT ort FROM kdst WHERE ort NOT IN (SELECT ort FROM vert)

Die Orte von kdst, die nicht in vert vorkommen

----


Gibt **alle Zeilen aus der ersten Abfrage zurück, die NICHT in der zweiten Abfrage enthalten sind.**

```
🔹 Ergebnis: Alle Kunden (customers), die keine Mitarbeiter (employees) sind.

SELECT name FROM customers
EXCEPT
SELECT name FROM employees;
```



# 3 Union

1 
Anzeige von Kunden, die etwas bestellt haben, mit Nr., Name, Adressangaben, Auftragsnr. Sowie Von Vertretern mit Nr., Name, Adresse, Kdnr

Frage: können vollkommen unabhängige Ergebnisse verknüpft werden? -> ja


```sql
select k.kdnr as nummer,k.firma as name,k.plz,k.ort,a.aufnr as "aufnr/kdnr"
from kdst k inner join aufkopf a on k.kdnr = a.kdnr
union
select v.vertreter,v.name,v.plz,v.ort,k.kdnr
from vert v inner join kdst k on v.vertreter=k.vertreter
;
```

![](image/Pasted%20image%2020250129205335.png)



2

Anzeige von Kunden, die etwas bestellt haben, mit Nr., Name, Adressangaben, Auftragsnr. Sowie Der Anzahl an Aufträgen

Frage: können detaillierte und aggregierte Ergebnisse verknüpft werden? -> ja

```
select k.kdnr as nummer,k.firma as name,k.plz,k.ort,a.aufnr
from kdst k inner join aufkopf a on k.kdnr = a.kdnr
union
select null,null,null,'Anzahl Aufträge: ', count(a.aufnr)
from aufkopf a ;

```

![](image/Pasted%20image%2020250129205505.png)


3 

```
select k.kdnr as nummer,k.firma as name,k.plz,k.ort,a.aufnr
from kdst k inner join aufkopf a on k.kdnr = a.kdnr

union

select v.vertreter,v.name,v.plz,v.ort,k.kdnr
from vert v inner join kdst k on v.vertreter=k.vertreter

union
select null,null,null,'Anzahl Zeilen',count(u.nummer)
from (
select k.kdnr as nummer,k.firma as name,k.plz,k.ort,a.aufnr
from kdst k inner join aufkopf a on k.kdnr = a.kdnr
union
select v.vertreter,v.name,v.plz,v.ort,k.kdnr
from vert v inner join kdst k on v.vertreter=k.vertreter
) u
```


![](image/Pasted%20image%2020250129205619.png)



