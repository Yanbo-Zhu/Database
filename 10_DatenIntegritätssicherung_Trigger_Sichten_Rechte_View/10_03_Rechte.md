
# 1 Grant privileges

Es ist möglich, einer Rolle B andere Rollen (z.B. Rolle A) zuzuweisen, so dass Privilegien hierarchisch vererbt werden können, z.B. von Rolle A -> Rolle B.

Ja, das ist korrekt. In relationalen Datenbanksystemen mit rollenbasierter Zugriffskontrolle (Role-Based Access Control, RBAC) ist es möglich, Rollen hierarchisch zu organisieren, sodass Privilegien von einer Rolle an eine andere Rolle vererbt werden können.

 Hierarchische Vererbung von Rollen
1. **Rollenbeziehungen**:  
    Eine Rolle kann einer anderen Rolle zugewiesen werden, wodurch die zweite Rolle automatisch die Berechtigungen (Privilegien) der ersten Rolle erbt.  
    Beispiel: Wenn Rolle A bestimmte Rechte hat und Rolle B Rolle A zugewiesen wird, besitzt Rolle B sowohl ihre eigenen Rechte als auch die Rechte von Rolle A.
2. **Vorteile**:
    - **Wiederverwendbarkeit**: Gemeinsame Privilegien können in einer Basissrolle (z. B. Rolle A) definiert werden, die dann von anderen Rollen genutzt wird.
    - **Flexibilität**: Änderungen an der Basissrolle wirken sich automatisch auf alle abhängigen Rollen aus.
    - **Bessere Organisation**: Hierarchien erleichtern das Management komplexer Berechtigungsstrukturen.


SQL-Beispiel: Rollen mit Vererbung
In SQL (z. B. PostgreSQL) können Rollen wie folgt definiert und zugewiesen werden:
```sql
-- Erstellen von Rolle A mit spezifischen Privilegien
CREATE ROLE rolle_a;
GRANT SELECT ON tabelle_x TO rolle_a;

-- Erstellen von Rolle B und Zuweisung von Rolle A an Rolle B
CREATE ROLE rolle_b;
GRANT rolle_a TO rolle_b;

-- Zuweisung von Rolle B an einen Benutzer
GRANT rolle_b TO benutzer1;

```

Funktionsweise
1. **Rolle A** hat das Privileg, Daten in `tabelle_x` zu lesen.
2. **Rolle B** wird Rolle A zugewiesen. Dadurch erbt Rolle B die Berechtigungen von Rolle A.
3. Ein Benutzer, der Rolle B zugewiesen bekommt (z. B. `benutzer1`), erhält:
    - Die Berechtigungen von Rolle B.
    - Die geerbten Berechtigungen von Rolle A.

Praktische Anwendungen
- **Hierarchische Berechtigungsstruktur**: Zum Beispiel kann eine Rolle "Mitarbeiter" Basissrechte definieren, die von Rollen wie "Manager" oder "Administrator" geerbt werden.
- **Vereinfachung der Verwaltung**: Änderungen an der Basissrolle "Mitarbeiter" wirken sich auf alle übergeordneten Rollen aus.

Einschränkungen
- Die hierarchische Vererbung sollte sorgfältig geplant werden, um **ungewollte Eskalation von Privilegien** zu vermeiden.
- Überprüfen Sie regelmäßig die Rollenstruktur, um sicherzustellen, dass nur berechtigte Benutzer Zugriff auf sensible Daten oder Funktionen haben.


# 2 revoke privileges


To revoke privileges in reverse order step by step:

1. **Start with the most specific privilege**: First, revoke any specific privileges granted on individual database objects, such as tables, views, or procedures.
    
2. **Revoke object-level privileges**: For example, revoke **SELECT**, **INSERT**, **UPDATE**, and **DELETE** privileges on specific tables or views.
    
3. **Revoke role-specific privileges**: If any roles were granted with specific privileges, revoke the roles granted to the user.
    
4. **Revoke system-level privileges**: Finally, revoke broader system-level privileges such as **CREATE**, **ALTER**, or **DROP** privileges that might have been granted at the database or server level.
    

This method ensures that the revocation follows a logical flow from the most specific permissions to the broader ones.


## 2.1 Beispiel 


Wie sehen die korrekten SQL-Anweisungen in korrekter Reihenfolge aus, mit denen Sie die nachfolgend vergebenen  
Privilegien in umgekehrter Reihenfolge Schritt für Schritt wieder entziehen. Bitte löschen Sie auch die beiden erstellten Rollen.

```
CREATE ROLE controlling; 

CREATE ROLE inventur;

GRANT SELECT ON Artikel, Halle TO PUBLIC;  
GRANT SELECT ON Lagerbestand TO controlling;  
GRANT SELECT, INSERT, UPDATE, DELETE ON Lagerbestand TO inventur;
```

 
 Wählen Sie eine oder mehrere Antworten:

```
drop role inventur;

revoke select, insert, update, delete on lagerbestand from inventur;  
revoke select on artikel, halle from public;

revoke select on lagerbestand from controlling;  
drop role controlling;
```

```
revoke select, insert, update, delete on lagerbestand from inventur;  
drop role inventur;

revoke select on lagerbestand from controlling;  
drop role controlling;

revoke select on artikel, halle from public;
```

```
drop role inventur;

drop role controlling;

revoke select, insert, update, delete on lagerbestand from inventur;

revoke select on artikel, halle from public;

revoke select on lagerbestand from controlling;
```

```
revoke select, insert, update, delete on lagerbestand from inventur;  
drop role inventur;

revoke select on artikel, halle from public;

revoke select on lagerbestand from controlling;  
drop role controlling;
```

```
revoke select on artikel, halle from public;

revoke select, insert, update, delete on lagerbestand from inventur;  
drop role inventur;

revoke select on lagerbestand from controlling;  
drop role controlling;
```

```
revoke select on lagerbestand from inventur;

drop role inventur;

revoke insert, update, delete on lagerbestand from inventur;

revoke select on artikel, halle from public;

revoke select on lagerbestand from controlling;  
drop role controlling;
```

```
revoke select, insert, update, delete on lagerbestand from inventur;  
revoke select on lagerbestand from controlling;  
revoke select on artikel, halle from public;  
drop role inventur;  
drop role controlling;
```

```
revoke select on lagerbestand from inventur;

revoke insert, update, delete on lagerbestand from inventur;

revoke select on lagerbestand from controlling;

drop role inventur;

drop role controlling;
```


只有 这些是对的 其他的都不对 


```
revoke select on lagerbestand from inventur;
revoke insert, update, delete on lagerbestand from inventur;
revoke select on lagerbestand from controlling;

drop role inventur;
drop role controlling;
```

```
revoke select, insert, update, delete on lagerbestand from inventur;
drop role inventur;
revoke select on lagerbestand from controlling;
drop role controlling;
revoke select on artikel, halle from public;
```


```
revoke select on artikel, halle from public;
revoke select, insert, update, delete on lagerbestand from inventur;
drop role inventur;
revoke select on lagerbestand from controlling;
drop role controlling;
```

```
revoke select, insert, update, delete on lagerbestand from inventur;
drop role inventur;
revoke select on artikel, halle from public;
revoke select on lagerbestand from controlling;
drop role controlling;
```




9

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



10


