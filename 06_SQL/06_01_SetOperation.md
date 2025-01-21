

![](image/Pasted%20image%2020250121214734.png)

- Intersect
    - Schnittmenge zwischen zwei SELECT Anfragen
- Union: 
    - Beim Union: gleich Datensatz nicht zweimal auftauchen, Duplikat nicht vorzeigen  
- Unioin ALL
    - Duplikat kann vorzeigen  
- Minus/Except: 
    - Differenzmenge minus oder except


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



# 2 Minus/Except


Entwickeln Sie bitte eine Abfrage auf der Datenbank mat_inf, die die gleiche Ergebnistabelle liefert, wie die nachfolgend dargestellte Abfrage. Entwickeln Sie eine Lösung mit Unterabfrage! (5 Punkte)
```
select ort from kdst
except
select ort from vert;
```

SELECT ort FROM kdst WHERE ort NOT IN (SELECT ort FROM vert)

Die Orte von kdst, die nicht in vert vorkommen


