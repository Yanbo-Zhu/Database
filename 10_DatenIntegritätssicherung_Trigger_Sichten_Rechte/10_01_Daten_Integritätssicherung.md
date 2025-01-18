
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

# 2 Beispiel

## 2.1 

Gehen Sie von folgendem Relationenschema der Tabelle „Artikelzubehoer“ aus. Passen Sie die Definition der existierenden Tabelle mit einem SQL-Statement so an, dass das Modelljahr eines Artikels nicht vor dem Gründungsjahr der Herstellerfirma liegen darf. Artikelzubehoer(ArtNr, Modell, Firma, Modelljahr, EAN, Gruendungsjahr, Pos, Zubehoerteil)

alter table artikelzubehoer add constraint chk_jahr CHECK (modelljahr >= gruendungsjahr);

用 create 是不对的 
create table artikelzubehoer add constraint chk_jahr CHECK (modelljahr >= gruendungsjahr);



## 2.2 


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