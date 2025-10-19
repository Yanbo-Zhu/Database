
# 1 Details zu **CHECK**:

Das **`CHECK`**-Constraint **`(modelljahr >= gruendungsjahr)`** wird verwendet, um sicherzustellen, dass die Spalte `modelljahr` immer einen Wert enthält, der **größer oder gleich** dem Wert in der Spalte `gruendungsjahr` ist. Dies dient dazu, Datenintegrität auf der Ebene der Tabelle sicherzustellen.

1. **Funktion**:    
    - Mit dem `CHECK`-Constraint können Bedingungen festgelegt werden, die jede Zeile der Tabelle erfüllen muss.
    - Wenn eine Zeile eingefügt oder aktualisiert wird und die Bedingung nicht erfüllt, wird die Operation mit einem Fehler abgelehnt.
2. **Anwendungsfall in diesem Beispiel**:
    - **`CHECK (modelljahr >= gruendungsjahr)`** bedeutet:
        - Das `modelljahr` eines Artikels darf nicht kleiner sein als das Gründungsjahr (`gruendungsjahr`) der Firma.
    - Beispiel:
        - `modelljahr = 2000` und `gruendungsjahr = 1995` → **erlaubt** (2000 ≥ 1995).
        - `modelljahr = 1990` und `gruendungsjahr = 2000` → **nicht erlaubt** (1990 < 2000).
3. **Einsatz im SQL-Statement**:
    - Der `CHECK`-Constraint wird mit dem Befehl **`ALTER TABLE`** oder beim Erstellen einer Tabelle mit **`CREATE TABLE`** hinzugefügt.


Beispiel: Hinzufügen eines `CHECK`-Constraints
```sql
ALTER TABLE artikelzubehoer
ADD CONSTRAINT chk_modelljahr
CHECK (modelljahr >= gruendungsjahr);
```


Beispiel: Test mit korrekten und fehlerhaften Daten
```sql
-- Korrekte Einfügung
INSERT INTO artikelzubehoer (ArtNr, Modell, Firma, Modelljahr, EAN, Gruendungsjahr, Pos, Zubehoerteil)
VALUES (1, 'ModellX', 'FirmaA', 2010, '1234567890123', 2000, 'Pos1', 'TeilA');

-- Fehlerhafte Einfügung (Verstoß gegen CHECK-Bedingung)
INSERT INTO artikelzubehoer (ArtNr, Modell, Firma, Modelljahr, EAN, Gruendungsjahr, Pos, Zubehoerteil)
VALUES (2, 'ModellY', 'FirmaB', 1990, '9876543210987', 2000, 'Pos2', 'TeilB');
-- Fehler: CHECK constraint "chk_modelljahr" violated

```


---



Beispiel für CHECK-Constraint in CREATE TABLE
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT CHECK (age >= 18),  -- Alter muss mindestens 18 sein
    salary DECIMAL(10,2) CHECK (salary >= 3000), -- Mindestgehalt 3000
    department VARCHAR(50) CHECK (department IN ('HR', 'IT', 'Finance')) -- Nur diese Werte erlaubt
);

```

CHECK-Constraint mit ALTER TABLE hinzufügen
```sql
ALTER TABLE artikelzubehoer
ADD CONSTRAINT chk_modelljahr
CHECK (modelljahr >= gruendungsjahr);
```


**`CHECK`-Constraint entfernen**
```sql
ALTER TABLE employees DROP CONSTRAINT check_salary;
```


## 1.1 Beispiel

### 1.1.1 

Gehen Sie von folgendem Relationenschema der Tabelle „Artikelzubehoer“ aus. Passen Sie die Definition der existierenden Tabelle mit einem SQL-Statement so an, dass das Modelljahr eines Artikels nicht vor dem Gründungsjahr der Herstellerfirma liegen darf. Artikelzubehoer(ArtNr, Modell, Firma, Modelljahr, EAN, Gruendungsjahr, Pos, Zubehoerteil)

alter table artikelzubehoer add constraint chk_jahr CHECK (modelljahr >= gruendungsjahr);

用 create 是不对的 
create table artikelzubehoer add constraint chk_jahr CHECK (modelljahr >= gruendungsjahr);


### 1.1.2 

Welche Möglichkeiten gibt es, die nachfolgende Integritätsbedingung im SQL-Standard umzusetzen: das Umsatzsoll der Kunden soll mindestens 10.000 € betragen.

答案

Die folgenden Optionen sind korrekt, um die Integritätsbedingung, dass das Umsatzsoll der Kunden mindestens 10.000 € betragen soll, im SQL-Standard umzusetzen:

1. **check (umssoll > 10000) - bei CREATE TABLE**  
    Dies ist eine einfache Möglichkeit, die Bedingung direkt in der Tabelle zu definieren, wenn sie erstellt wird.
    
2. **CHECK (VALUE > 10000); und Nutzung der Domäne umsatzsollbegrenzug bei CREATE oder ALTER TABLE als Datentyp**  
    Erstellen Sie eine Domäne (z.B. `umsatzsollbegrenzug`), die die Bedingung enthält, und wenden Sie sie auf die entsprechende Spalte an.
    
3. **durch einen BEFORE INSERT OR UPDATE OF UMSSOLL ON KDST - Trigger**  
    Ein Trigger vor dem Einfügen oder Aktualisieren kann die Bedingung überprüfen und sicherstellen, dass das Umsatzsoll die Mindestgrenze von 10.000 € nicht unterschreitet.
    
4. **check (umssoll > 10000) - bei ALTER TABLE**  
    Falls die Tabelle bereits existiert, kann die Bedingung auch später über `ALTER TABLE` hinzugefügt werden.
    

Die folgenden Optionen sind jedoch **nicht geeignet**:

- **durch einen Primärschlüssel**  
    Ein Primärschlüssel dient zur Identifizierung von Datensätzen, nicht zur Validierung von Bedingungen wie dem Mindestumsatz.
    
- **durch alleine einen BEFORE INSERT ON KDST - Trigger**  
    Ein `BEFORE INSERT`-Trigger alleine überprüft nur die eingefügten Werte, aber es könnte sein, dass auch ein Update erforderlich ist, um die Integrität zu wahren.
    
- **durch einen BEFORE UPDATE OF UMSSOLL ON KDST - Trigger**  
    Ein Trigger, der nur auf das Update der `UMSSOLL`-Spalte reagiert, könnte ebenfalls nicht ausreichen, wenn der Wert einmalig gesetzt wurde und später nicht geändert wird.
    
- **durch einen entsprechenden Fremdschlüssel auf Umsatzsoll in einer Umsatzsolltabelle**  
    Ein Fremdschlüssel stellt eine Beziehung zwischen zwei Tabellen her, aber er überprüft nicht direkt die Integrität von Werten in einer einzelnen Tabelle.




# 2 die referenzielle Integrität (foreign key)


## 2.1 Gefährdung der referenziellen Integrität durch schreibende  Operationen:


![](../04_DDL_DML/image/Pasted%20image%2020250128222930.png)


- Insert bei referenzierender Relation (FS Wert falsch), z.B.: insert into artst
- Delete bei referenzierter Relation, z.B. delete from artgru
- Update des PS Wertes bei referenzierter Relation
    - z.B. update artgru set gruppe = …
- Update des FS Wertes bei referenzierender Relation,
    - z.B. update artst set gruppe = …


----

例子 

Welche kritischen Operationen können die referenzielle Integrität in einem relationalen DBS gefährden? 

Erläutern Sie bitte 2 dieser Operationen anhand des Beispiels: Gegeben sind die Relationen 
AuftragsPosition (aufnr, posnr, menge, ArtikelStamm->artnr) und 
ArtikelStamm (artnr, artbez, ekpreis). (5 Punkte)


答案: 
der Fremdschlüssel ist ja eine integritätsbedingung und gehört zu diesen ja Modell Integritätsbedingungen

1 Delete operation
Löschen eines Primärschlüssels in der referenzierten Tabelle

2 Update operation
- Ändern eines Primärschlüssels in der referenzierten Tabelle
- Updaten des Fremdschlüssels auf einen nicht existierenden Primärschlüssel


3 Insert operation 
Erstellen von Auftragsposition mit Fremdschlüssel der keinen Datensatz in ArtikelStamm referenziert



