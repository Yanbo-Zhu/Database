
# 1 Nested Loop Join und Hash Join

Ein Gleichverbund (Equi-Join) kann sowohl mit einem **Nested Loop Join** als auch mit einem **Hash Join** umgesetzt werden. Beide Methoden sind gebräuchliche Techniken zur Ausführung von Joins in relationalen Datenbanken.

- **Nested Loop Join**
    - Vergleicht jede Zeile der ersten Tabelle mit jeder Zeile der zweiten Tabelle.
    - Beim Gleichverbund wird die Vergleichsbedingung (`=`) genutzt, um passende Zeilen zu identifizieren.
- **Hash Join**:
    - Baut einen Hash-Table für eine der Tabellen basierend auf dem Join-Schlüssel (Gleichheitsbedingung).
    - Überprüft dann, welche Werte aus der zweiten Tabelle in der Hash-Table vorhanden sind, um die passenden Zeilen zu finden.



# 2 NULL Vergleich 
Es soll ein Gleichverbund zweier Relationen R1 und R2 über das gemeinsame Attribut A ausgeführt werden. Was lässt sich über die Ergebnisrelation aussagen, wenn zwei Tupel t1 - Element von R1 - und t2 - Element von R2 - existieren, die beide im Attribut A den Wert NULL aufweisen?

In relational algebra and SQL, **NULL** is treated as "unknown," and it does not equate to itself. This means that a comparison like `NULL = NULL` is not true; instead, it evaluates to **unknown**. In the context of an **equi-join** (Gleichverbund), this behavior has implications for how tuples with `NULL` in the join attribute are handled.

Wenn nur die beiden Tupel t1​∈R1​ und t2​∈R2​ existieren, dann ist die Ergebnisrelation leer.
- die aussage is richtig 
- If both t1 (from R1​) and t2​ (from R2​) have `NULL` in attribute A, the equi-join condition (`R_1.A = R_2.A`) cannot be satisfied because `NULL = NULL` evaluates to **unknown**.
- Therefore, the result of the equi-join is **empty**.


- **"The result relation is equal to the Cartesian product."**
    - This is **false** because the equi-join does not produce a Cartesian product. Instead, it filters the Cartesian product based on the equality condition, which is not satisfied for `NULL`.
- "If t1 with A=NULL in R1​ and t2​ with A=NULL in R2 exist, as well as other tuples in R1​ and R2 that do not have matching values in A, then the result relation is empty."
    - This is ture .  
- "If t1 with A=NULL in R1 and t2​ with A=NULL in R2​ exist, as well as other tuples in R1 and R2 that have matching values in A, then the result relation is empty."
    - This is **false** because the join will include tuples with non-`NULL` matching values in A but the tuples with `NULL` will not contribute to the result.




