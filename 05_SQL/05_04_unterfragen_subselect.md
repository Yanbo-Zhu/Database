


# 1 wo genau Unterabfragen in SQL-Statements vorkommen können 



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



# 2 einfache Unteranfrage und korrelierten Unteranfrage

Was unterscheidet eine einfache von einer korrelierten Unteranfrage?

Eine einfache Unteranfrage wird einmal ausgeführt.
Eine einfache Unteranfrage ist unabhängig von der äußeren Anfrage und kann deshalb separat getestet werden.

Eine korrelierte Unteranfrage wird mehrmals ausgeführt.
Eine korrelierte Unteranfrage ist abhängig von der äußeren Anfrage und kann deshalb nicht separat getestet werden.

![](image/Pasted%20image%2020250129210201.png)




# 3 All- oder Any-Bedingungen


## 3.1 All


```
select v.vertreter, v.name,v.prov  
from matinf.vert v  
where v.prov < all  
(select v.prov  
from matinf.vert v  
where v.plz = '12487');
```


```
select v.vertreter, v.name,v.prov  
from matinf.vert v  
where v.prov <    // 等效于 all 
(select min(v.prov)  
from matinf.vert v  
where v.plz = '12487');
```


---


Es sollen Vertreternummer, Name, Provision und Stadtteil derjenigen Vertreter ermittelt werden, deren Provision höher ist als die durchschnittliche Provision in seinem / ihrem Stadtteil.

```
select vl.name,vl.prov,vl.stadtteil
from matinf.vert vl
where vl.prov >
(select avg(v2.prov)
from matinf.vert v2
where vl.stadtteil=v2.stadtteil);
```

![](image/Pasted%20image%2020250129210958.png)



Es sollen Vertreternummer, Name, Provision und Stadtteil derjenigen Vertreter ermittelt werden, deren Provision höher ist als die durchschnittliche Provision in seinem / ihrem Stadtteil.

```
select vl.name,vl.prov,vl.stadtteil
from matinf.vert vl
where vl.prov >
(select avg(v2.prov)
from matinf.vert v2
where vi.stadtteil=v2.stadtteil);
```

![](image/Pasted%20image%2020250129211049.png)



## 3.2 Any

Any: (wie OR-Verknüpfung)
–Wird true, wenn Bedingung für ein Tupel erfüllt
–Wird false, wenn Bedingung für alle Tupel nicht erfüllt

---

Beispiel  1

Anzeige von name, prov und plz der Vertreter, deren Prov. kleiner ist als die Prov. eines Vertreters in PLZ 12487


```
select name,prov
from vert
where prov < any
(select prov
from vert
where plz = "12487");
```

6 Tupel als Ergebnis

![](image/Pasted%20image%2020250129101310.png)



---

Beispiel 2

Es sollen Vertreternummer, Name und Provision derjenigen Vertreter ermittelt werden, deren Provision kleiner ist als die Provision (irgend-) eines Vertreters aus dem Stadtbezirk mit der PLZ „12487".

```
select v.name,v.prov
from matinf.vert v
where v.prov < any
(select v. prov
from matinf.vert v
where v.plz = '12487')
```

![](image/Pasted%20image%2020250129210921.png)




# 4 korrelierten Unteranfrage


## 4.1 Beispiel 


1 
Anzeige von name und prov der Vertreter, deren Prov. höher ist als Durchschnitt ihres Stadtteils

![](image/Pasted%20image%2020250129133330.png)


```
select name, prov
from vert v1
where prov >
(select avg(prov)
from vert v2
where v1.stadtteil=v2.stadtteil)
order by name;
```

![](image/Pasted%20image%2020250129133353.png)


----

2

Erweitertes Beispiel:
Anzeige von name und prov der Vertreter, deren Prov. höher ist als Durchschnitt ihres Stadtteils
Zusätzlich: Anzeige des Stadtteils und der Durchschnittsprovision des Stadtteils

![](image/Pasted%20image%2020250129133522.png)

```
select name, prov, w.stadtteil, w.durchschnitt
from vert v1, ( select avg(prov) as durchschnitt, stadtteil from vert group by stadtteil) w
where prov >
(select avg(prov)
from vert v2
where v1.stadtteil=v2.stadtteil)
and v1.stadtteil = w.stadtteil
order by name;
```



## 4.2 Exists and not Exists-Operator:

exists 运行的很快, 提高查询效率 


Exists-Operator:
Anzeige von Kunden, die etwas bestellt haben, mit Nr., Name, Adressangaben

```sql
select k.kdnr,k.firma, k.plz,k.ort
from kdst k
where exists
(select *
from aufkopf a
where k.kdnr=a.kdnr);
```

![](image/Pasted%20image%2020250129133843.png)


Not Exists-Operator:
Anzeige von Kunden, die nichts bestellt haben, mit Nr., Name, Adressangaben

```sql
select k.kdnr,k.firma, k.plz,k.ort
from kdst k
where not exists
(select *
from aufkopf a
where k.kdnr=a.kdnr);
```

![](image/Pasted%20image%2020250129133922.png)


# 5 Einfache Unteranfrage


## 5.1 Not in

Not in - Operator:
Anzeige von Kunden, die nichts bestellt haben, mit Nr., Name, Adressangaben

```sql
select k.kdnr,k.firma, k.plz,k.ort
from kdst k
where k.kdnr not in
(select a.kdnr
from aufkopf a);
```

![](image/Pasted%20image%2020250129134339.png)



