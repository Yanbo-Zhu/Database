

# 1 mat_Inf


## 1.1 Aufgabe 1

Aufgabestelllung: 
Was haben Kunden (Firmen) von Vertreter Fischer bestellt?
Geben Sie bitte den Vertreternamen, den Kundennamen, den Artikelnamen, die Gesamtbestellmenge, den Durchschnitts- Auftragspreis und den Gesamtbestellwert (preis * menge) des Artikels an. Gehen Sie davon aus, dass ein Artikel von einem Kunden auch mehrfach bestellt worden sein kann.

----

Vertreternamen stammt aus der Tabelle vert . Kundennamen stammt aus der Tabelle kdst. Artikelnamen stammt aus der Tabelle artst . 
Die Tabelle vert korreliert mit der Tabelle kdst direkt. 
Die Tabelle artst korreliert mit der Tabelle kdst und vert indirekt. Deshalbe müssen wir die Tabelle aufpos und aufkopf butzen, um die Verbidnung zwischen Tabelle artst, kdst und vert zu bauen. 

```sql
SELECT 
    v.NAME AS VertreterName, 
    k.FIRMA AS KundeName, 
    ats.ARTBEZ AS ArtikelName 
FROM 
    vert v
JOIN 
    kdst k ON v.VERTRETER = k.VERTRETER
JOIN 
    aufkopf ak ON k.KDNR = ak.KDNR
JOIN 
    aufpos ap ON ak.AUFNR = ap.AUFNR
JOIN 
    artst ats ON ap.ARTNR = ats.ARTNR
WHERE 
    v.NAME = 'Fischer'
```

![](image/Pasted%20image%2020241207110817.png)


----

Die Daten von der Gesamtbestellmenge, dem DurchschnittsAuftragspreis und dem Gesamtbestellwert (preis * menge) des Artikels sind in Tabelle aufpos abgespeichert.
Um die Aggregation function AVG und SUM zu benuzten, muss GROUP BY v.NAME, k.FIRMA, a.ARTBEZ verwendet.


```sql
SELECT 
    v.NAME AS VertreterName, 
    k.FIRMA AS KundeName, 
    ats.ARTBEZ AS ArtikelName, 
    SUM(ap.MENGE) AS Gesamtbestellmenge, 
    AVG(ap.PREIS) AS Durchschnittspreis, 
    SUM(ap.MENGE * ap.PREIS) AS Gesamtbestellwert
FROM 
    vert v
JOIN 
    kdst k ON v.VERTRETER = k.VERTRETER
JOIN 
    aufkopf ak ON k.KDNR = ak.KDNR
JOIN 
    aufpos ap ON ak.AUFNR = ap.AUFNR
JOIN 
    artst ats ON ap.ARTNR = ats.ARTNR
WHERE 
    v.NAME = 'Fischer'
GROUP BY 
    v.NAME, k.FIRMA, ats.ARTBEZ
ORDER BY 
    v.NAME, k.FIRMA, ats.ARTBEZ;
```


![](image/Pasted%20image%2020241207110855.png)


## 1.2 Aufgabe 2

Aufgabestellung: 
Es sollen alle Informationen zu denjenigen Artikeln angezeigt werden, die genau zweimal bestellt wurden, d.h. die in genau 2 Bestellungen vorkommen. Von diesen Artikeln sollen nur diejenigen mit dem größten Lagerbestand angezeigt werden.

---

Um zu zeigen, welche Artikeln genue zweimal bestellt wurden

```sql
SELECT 
	ap.ARTNR
FROM 
	aufpos ap
JOIN 
	artst ats
ON	
	ap.ARTNR = ats.ARTNR
GROUP 
	BY ap.ARTNR
HAVING 
	COUNT(ap.AUFNR) = 2
```

![](image/Pasted%20image%2020241207110951.png)

----

Alle Informationen zu denjenigen Artikeln anzuzeigen : select arts.* benuzten 
Nuzten Exists in Where Clause, um die Artikeln zu filtern, die genue zweimal bestellt wurden.
Um diesen Artikeln sollen nur diejenigen mit dem größten Lagerbestand angezeigt werden zu lassen, einfach das Ergebnisse nach BESTAND Attribute Descending sortieren und den obersten Entität anzeigen lassen.

```sql
SELECT 
    ats.*
FROM 
    artst ats
WHERE 
    EXISTS (
        SELECT ap.ARTNR
        FROM aufpos ap
        WHERE ap.ARTNR = ats.ARTNR
        GROUP BY ap.ARTNR
        HAVING COUNT(ap.AUFNR) = 2
    )
ORDER BY 
    ats.BESTAND DESC
LIMIT 1;
```

![](image/Pasted%20image%2020241207111024.png)

## 1.3 Aufgabe 3

Aufgabestellung

Es soll eine Übersicht über die Auftragslage und den zu erwartenden Gewinn für alle Artikel erstellt werden. Sie soll enthalten (vgl. Tabelle unten): den Artikelnamen, die Bestellmenge, den Auftragspreis, den Auftragswert (menge * preis), den Lagerbestand, den Einkaufspreis, den Lagerwert (bestand * ekpreis), die Restmenge nach Abzug der Bestellmenge, den Gewinn, der zu bilden ist zwischen Einkaufs- und Auftragspreis. Auch der insgesamt erzielte Gewinn soll in einer zusätzlichen Zeile angezeigt werden.

-----

Warum LEFT JOIN: Ein LEFT JOIN wird verwendet, um sicherzustellen, dass alle Artikel aus der Tabelle artst in der Query enthalten sind, auch wenn  keine Bestellungen bei einem Artikel in der Tabelle aufpos exisitieren.
Die Attritube stammt aus der Tabelle artst: Artikelname, Lagerbestand, Einkaufspreis, 
Die Attritube stammt aus der Tabelle aufpos: Bestellmenge, Auftragspreis
Differenzmenge/Restmenge = Lagerbestand - Bestellmenge
Gewinn = (Auftragspreis- Einkaufspreis)* Bestellmenge

```sql
SELECT 
    ats.ARTBEZ AS Artikelname,
    SUM(ap.MENGE) AS Bestellmenge,
    MAX(ap.PREIS) AS Auftragspreis,
    SUM(ap.MENGE * ap.PREIS) AS Auftragswert,
    ats.EKPREIS AS Einkaufspreis,
    ats.BESTAND AS Lagerbestand,
    ats.BESTAND * ats.EKPREIS AS Lagerwert,
    ats.BESTAND - SUM(ap.MENGE) AS Differenzmenge,
    SUM((ap.PREIS - ats.EKPREIS) * ap.MENGE) AS Gewinn
FROM 
    artst ats
LEFT JOIN 
    aufpos ap ON ats.ARTNR = ap.ARTNR
GROUP BY 
    ats.ARTBEZ, ats.BESTAND, ats.EKPREIS
```

![](image/Pasted%20image%2020241207111243.png)

---

die Gesamtsummen für Auftragswert und Gewinn zu berechnen

```sql
SELECT 
    'Gesamt' AS Artikelname,
    NULL AS Bestellmenge,
    NULL AS Auftragspreis,
    NULL AS Auftragswert,
    NULL AS Einkaufspreis,
    NULL AS Lagerbestand,
    NULL AS Lagerwert,
    NULL AS Differenzmenge,
    SUM((ap.PREIS - ats.EKPREIS) * ap.MENGE) AS Gewinn
FROM 
    artst ats
LEFT JOIN 
    aufpos ap ON ats.ARTNR = ap.ARTNR;
```

![](image/Pasted%20image%2020241207111349.png)

---

UNION ALL: die Gesamtsummen für Auftragswert und Gewinn hinzufügen und in einer zusätzlichen Zeile anzeigen

```sql
SELECT 
    ats.ARTBEZ AS Artikelname,
    SUM(ap.MENGE) AS Bestellmenge,
    MAX(ap.PREIS) AS Auftragspreis,
    SUM(ap.MENGE * ap.PREIS) AS Auftragswert,
    ats.EKPREIS AS Einkaufspreis,
    ats.BESTAND AS Lagerbestand,
    ats.BESTAND * ats.EKPREIS AS Lagerwert,
    ats.BESTAND - SUM(ap.MENGE) AS Differenzmenge,
    SUM((ap.PREIS - ats.EKPREIS) * ap.MENGE) AS Gewinn
FROM 
    artst ats
LEFT JOIN 
    aufpos ap ON ats.ARTNR = ap.ARTNR
GROUP BY 
    ats.ARTBEZ, ats.BESTAND, ats.EKPREIS
UNION ALL
SELECT 
    'Gesamt' AS Artikelname,
    NULL AS Bestellmenge,
    NULL AS Auftragspreis,
    NULL AS Auftragswert,
    NULL AS Einkaufspreis,
    NULL AS Lagerbestand,
    NULL AS Lagerwert,
    NULL AS Differenzmenge,
    SUM((ap.PREIS - ats.EKPREIS) * ap.MENGE) AS Gewinn
FROM 
    artst ats
LEFT JOIN 
    aufpos ap ON ats.ARTNR = ap.ARTNR;
```

![](image/Pasted%20image%2020241207111412.png)


## 1.4 Aufgabe 4 

Es sollen alle Vertreter mit ihrem Namen und der dazugehörigen Firma angezeigt werden, die Großabnehmer betreuen Oder einen Kunden betreuen, der einen fakturierten Kundenauftrag hat. Zeigen Sie bitte auch die Kundengruppe und den Auftragsstatus an.

---

Um die dazugehörigen Firma anzuzeigen, muss die Tabelle kdst join gemacht werden
Der Vertreter muss die Großabnehmer betreuen: die Tabelle kdgru join müseen
Der Vertreter muss einen Kunden betreuen, der einen fakturierten Kundenauftrag hat: die Tabelle aufkopf und aufstat join müseen

```sql
SELECT 
    v.NAME AS VertreterName,
    k.FIRMA AS VertreterFirma,
    kg.grup_txt AS Kundengruppe,
    afs.STAT_TXT AS Auftragsstatus
FROM 
    vert v
JOIN 
    kdst k ON v.VERTRETER = k.VERTRETER
JOIN 
	kdgru kg ON k.KDGRUPPE = kg.kdgruppe 
JOIN 
    aufkopf ak ON k.KDNR = ak.KDNR
JOIN 
	aufstat afs ON ak.STATUS = afs.STATUS 
WHERE 
    kg.grup_txt LIKE '%Großabnehmer%' 
    OR afs.STAT_TXT LIKE '%fakturiert%';  -- Fakturierter Auftrag```

```



![](image/Pasted%20image%2020241207111457.png)

# 2 company 

## 2.1 Aufgabe5 

Aufgabestellung: 
Wie viele Teile muss jeder Zulieferer durchschnittlich liefern? Bitte zeigen Sie den Zulieferernamen, die Nummer und die durchschnittliche Anzahl an Teilen sowie die insgesamt durchschnittlich gelieferte Anzahl an Teilen.

Der Zulieferernamen und die Nummer stammt aus der Tabelle supplier
Die Daten von Anzahl an Teilen sind in der Tabelle supp_part abgespeichert.

```sql
SELECT 
    s.SUPPNAME, 
    s.SUPPNO, 
    AVG(sp.QUANTITY) AS durchschnittliche_anzahl_teile
FROM 
    supplier s
JOIN 
    supp_part sp ON s.SUPPNO = sp.SUPPNO
GROUP BY 
    s.SUPPNO
```

![](image/Pasted%20image%2020241207111730.png)

---

Die insgesamt durchschnittlich gelieferte Anzahl an alle Teilen   

```sql
SELECT 
    NULL AS SUPPNO, 
    'Durchschnitt insgesamt' AS SUPPNAME, 
    AVG(sp.QUANTITY) AS durchschnittliche_anzahl_teile
FROM 
    supp_part sp; 
```

![](image/Pasted%20image%2020241207111740.png)

---

UNION ALL: die insgesamt durchschnittlich gelieferte Anzahl an Teilen hinzufügen und in einer zusätzlichen Zeile anzeigen

```sql
SELECT 
    s.SUPPNAME, 
    s.SUPPNO, 
    AVG(sp.QUANTITY) AS durchschnittliche_anzahl_teile
FROM 
    supplier s
JOIN 
    supp_part sp ON s.SUPPNO = sp.SUPPNO
GROUP BY 
    s.SUPPNO
UNION ALL
SELECT 
    NULL AS SUPPNO, 
    'Durchschnitt insgesamt' AS SUPPNAME, 
    AVG(sp.QUANTITY) AS durchschnittliche_anzahl_teile
FROM 
    supp_part sp;
    
```

![](image/Pasted%20image%2020241207111748.png)

## 2.2 Aufgabe 6 

Aufgabestellung: 
Wer liefert von welchem Teil wie viel? Geben Sie bitte den Namen des Teils, den Namen des Lieferanten und die gelieferte Menge für das Teil an. Die Anzeige soll so geordnet sein, dass die Teile (alphabetisch sortiert) mit der größten Liefermenge vor den anderen Teilen gelistet werden.

---

Wer: SUPPNAME aus der Tabelle supplier
Welchem Teil: PARTNAME aus der Tabelle part
Wie viel: QUANTITY aus der Tabelle supp_part. Um die Summer von supp_part.QUANTITY anzureichen, muss der Entitäten nach part.PARTNAME und supplier.SUPPNAME grouppiert werden 

Sortieren
- die Teile (alphabetisch sortiert):    ORDER BY part.PARTNAME
- der größten Liefermenge vor den anderen Teilen gelistet werden: ORDER BY Menge DESC  

```sql
SELECT 
    part.PARTNAME, 
    supplier.SUPPNAME, 
    SUM(supp_part.QUANTITY) AS Menge
FROM 
    part
JOIN 
    supp_part ON part.PARTNO = supp_part.PARTNO
JOIN 
    supplier ON supp_part.SUPPNO = supplier.SUPPNO
GROUP BY 
    part.PARTNAME, supplier.SUPPNAME
ORDER BY 
    part.PARTNAME ASC, Menge DESC;
```


![](image/Pasted%20image%2020241207111852.png)

## 2.3 Aufgabe 7 

Aufgabestellung: 
Suchen Sie bitte aus jeder Abteilung den Namen, das Gehalt und die Abteilungsnummer derjenigen Mitarbeiter, deren Gehalt geringer ist als das Durchschnittsgehalt in ihrer Abteilung. listen Sie zusätzlich immer auch das Durchschnittsgehalt der zugehörigen Abteilung auf (bitte auf 2 Nachkommastellen runden)

---

Das Durchschnittsgehalt der zugehörigen Abteilung: SELECT AVG(SALARY) FROM emp WHERE DEPTNO = e.DEPTNO
Das Durchschnittsgehalt auf 2 Nachkommastellen runden: ROUND(AVG(SALARY), 2)

```sql
SELECT 
    e.EMPNAME, 
    e.SALARY, 
    e.DEPTNO, 
    (SELECT ROUND(AVG(SALARY), 2) FROM emp WHERE DEPTNO = e.DEPTNO) AS Durchschnittsgehalt
FROM 
    emp e
WHERE 
    e.SALARY < (SELECT AVG(SALARY) FROM emp WHERE DEPTNO = e.DEPTNO)
ORDER BY 
    e.DEPTNO;
```

![](image/Pasted%20image%2020241210203646.png)

## 2.4 Aufgabe 8 

Aufgabestellung: 
Zeigen Sie bitte alle Produktnamen (JOBNAME) und die zugehörigen Produktionsorte (CITY) an, für die (Zuliefer-)Teile von den Lieferanten "Jones" und "Clark" benötigt werden.

---

Der Name des Suppliers wird in der Tabelle supplier abgespeichert. Um der Tabelle job mit der Tabelle supplier zu korrelieren, muss supp_part_job benuztet werden


```sql
SELECT 
	j.CITY,
    j.JOBNAME
FROM 
    job j
JOIN 
    supp_part_job spj ON j.JOBNO = spj.JOBNO
JOIN 
    supplier s ON spj.SUPPNO = s.SUPPNO
WHERE 
    s.SUPPNAME IN ('Jones', 'Clark')
```


![](image/Pasted%20image%2020241207112020.png)

# 3 Eigene Aufgabenstellung 


Entwickeln Sie bitte eine eigene Aufgabenstellung und die entsprechende Lösung. Dokumentieren Sie die Aufgabenstellung! Die Übungsdatenbank kann mat_inf Oder company sein. Bedingung: es sollen mindestens eine Unteranfrage (Subquery) und ein join enthalten sein.

---

Aufgabestellung zum Company Schema: 
Zeigen Sie bitte den Namen der Abteilung, den Namen des Mitarbeiters und das Gehalt der Mitarbeiter an, die in einer Abteilung arbeiten, deren Durchschnittsgehalt über 2000 liegt.

Durch unterliegende Query ergibt es sich, welche Abteilung deren Durchschnittsgehalt über 2000 liegt
```sql
SELECT DEPTNO 
FROM emp 
GROUP BY DEPTNO 
HAVING AVG(SALARY) > 2000
```

![](image/Pasted%20image%2020241207112152.png)


Dann: 
```sql
SELECT 
    d.DEPTNAME, 
    e.EMPNAME, 
    e.SALARY
FROM 
    emp e
JOIN 
    dept d ON e.DEPTNO = d.DEPTNO
WHERE 
    d.DEPTNO IN (
        SELECT DEPTNO 
        FROM emp 
        GROUP BY DEPTNO 
        HAVING AVG(SALARY) > 2000
    );

```

![](image/Pasted%20image%2020241207112227.png)