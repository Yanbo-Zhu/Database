


# 1 ACID Prinzip 

见 200_10_ConcurrencyControl 



# 2 Concurrency Anomalien

见 200_10_ConcurrencyControl 

# 3 Transitionsvariable NEW

Welche Werte liegen bei der Transitionsvariable NEW bei folgenden Operationen (nacheinander) vor:
- bevor ein DELETE ausgeführt wird,
- bevor ein UPDATE ausgeführt wird,
- bevor ein INSERT ausgeführt wird.

NULL, neuer Wert, neuer Wert

---

Die Transitionsvariable **NEW** repräsentiert die neuen Werte, die in einer Datenzeile nach einer DML-Operation (INSERT, UPDATE, DELETE) vorliegen. Die Antwort auf diese Frage hängt davon ab, welche Art von DML-Operation ausgeführt wird.

Vor einem DELETE
- Beim **DELETE** gibt es keinen "neuen Wert" für die Zeile, die gelöscht wird. Stattdessen wird der Wert durch **NULL** ersetzt, da die Zeile gelöscht wird und daher keine Daten mehr existieren.
- **NEW** zeigt daher für ein DELETE-Statement **NULL** an.

Vor einem UPDATE
- Beim **UPDATE** gibt es sowohl den alten Wert der Zeile als auch den neuen Wert, der nach der Aktualisierung gesetzt wird. **NEW** zeigt den neuen Wert an, der nach dem Update in der Zeile vorhanden sein wird.

Vor einem INSERT
- Beim **INSERT** wird eine neue Zeile eingefügt, daher gibt es keinen "alten" Wert. **NEW** zeigt den neuen Wert an, der in die Zeile eingefügt wird.

Zusammengefasst:
- **Vor einem DELETE**: **NULL** (es gibt keinen neuen Wert).
- **Vor einem UPDATE**: **neuer Wert** (der Wert nach der Aktualisierung).
- **Vor einem INSERT**: **neuer Wert** (der Wert, der eingefügt wird).


