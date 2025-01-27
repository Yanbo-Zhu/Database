
# 1 

![](image/Pasted%20image%2020250127175219.png)

Ein namhaftes Versandhaus bietet im Rahmen seiner verschiedenen Kataloge (young fashion, modern fashion, woman’s fashion) Produkte häufig als sogenannte Produktvariationen an, die der Kunde bestellen kann. Das heißt, dass ein bestimmtes Produkt, z.B. ein Shirt eines bestimmten Herstellers aus einem bestimmten Material, in verschiedenen Produktvariationen angeboten werden kann. Ein Basisprodukt besteht dabei aus genau einem Material, wie im Beispiel „Baumwolle“.
Eine Produktvariation zu einem Basisprodukt besteht aus einer gewählten Farbe, einer Größe und einem Druckmotiv. Die Preisbildung orientiert sich an der Größe. Abgebildet wird nachfolgend beispielhaft ein mögliches Basisprodukt mit einigen möglichen Produktvariationen:

![](image/Pasted%20image%2020250127175506.png)

Je Druckmotiv sollen, neben dessen Kurzbezeichnung, in der Datenbank auch eine Motivbeschreibung abgelegt werden. Je Größe werden Maßangaben und Toleranzen (als Text) sowie eine Kurzbeschreibung abgespeichert. Je Farbe werden der Farbcode, ein Farbkürzel sowie eine Beschreibung verwendet.
Zur Bestellung von Produkten aus den Katalogen muss die Produktvariation abgebildet werden, wie z.B.

![](image/Pasted%20image%2020250127175515.png)

Aufgabenstellung:
Erstellen Sie zu dem dargestellten Sachverhalt bitte ein ER-Diagramm mit (min,max)-Notation. Definieren Sie bitte zu den Objekttypen die Primärschlüssel- sowie ca. 3-5 Nichtschlüsselattribute (gemäß der Aufgabenstellung sowie evtl. weitere).

Umsetzung der Aufgabenstellung:
Diagramme erstellen Sie bitte mit einem Datenmodellierungstool wie z.B. dem DBDesignerFork (vgl. Moodle) oder der MySQLWorkbench. Laden Sie Ihre Lösungen als PDF- oder PPT-Dokument mit integrierten Abbildungen hoch.


---


解答 

![](image/Pasted%20image%2020250127175304.png)

这里面 有 redundancy, 因为 dritte Normale Form 没有别 被meeted
Preis 是仅仅由 Grosse 确定的 , 而现在 他们都是 Produktvariation 的 Attribute , 这两个 attribute 完全是一对一相关的

类似于这个例子

![](image/Pasted%20image%2020250127175812.png)


---

![](image/Pasted%20image%2020250127175312.png)


---


![](image/Pasted%20image%2020250127175328.png)


对上面不同的entitaettype 之间的 kaninalitat (min -max notation) 的解释

|                                 |                                                                                                                               |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Bestellung und Produktvariation | Ein bestellung muss ein  zumindenst Produktvariation  enthalen<br><br>Produktvariation  muss nicht bestellt werden            |
| Katalog  and Produktvariation   | Ein Katalog muss ein  zumindenst Produktvariation  enthalen<br><br>Produktvariation   muss zumindesnt zu 1 katalog zugeordent |
| Material und basisprodukt       | 有的 Materiall kann null mal in basisprodukt  auftauchen , 就是说 允许这个 material 从来没有在任何 Basisprodukt , 就没被用过                       |
| Motive und Produktvariation     | motiv muss nicht bei Produktvariation  benutzt                                                                                |

---




![](image/Pasted%20image%2020250127175337.png)


# 2 

From prasenz 01 

## 2.1 Beschreibung einer Anwendungsdomäne

- Ein Unternehmen verkauft Artikel über ein Netz von Vertretern.
- Vertreter sind hierarchisch jeweils einem Hauptvertreter unterstellt (Hauptvertreter müssen nicht unterstellt sein.).
- Jeder Vertreter betreut einen ihm zugewiesenen Kreis von Kunden. Ein Kunde Wird genau von einem Vertreter betreut.
- Kunden gehören zu einer Branche (z.B. EDV, Chemie, etc.)
- und werden in Abhängigkeit vom Auftragsvolumen einer Kundengruppe zugeordnet (Großabnehmer, Normalkunde, etc.).
- Von den Kunden werden Aufträge entgegengenommen, die jeweils ein bis mehrere Artikel des Sortiments des Unternehmens in bestimmten Mengen umfassen.
- Einzelne Auftragspositionen können mit Teillieferungen beliefert werden, so dass die noch zu liefernde Restmenge jederzeit ablesbar sein muss. Pro Position werden ein Liefertermin und ein positionsspezifischer Preis vereinbart.
- Das Artikelsortiment des Unternehmens ist in verschiedene Produktgruppen unterteilt (Fahrzeuge, Ersatzteile, etc.).
- Die Kundenaufträge werden je nach Bearbeitungsstand in 4 Status-Kategorien eingeordnet (erfasst, bestätigt, geliefert, fakturiert). Jeder Auftrag kann zu einem Zeitpunkt nur genau einen Status annehmen.


## 2.2 逐条分析 


1 
Jeder Vertreter betreut einen ihm zugewiesenen Kreis von Kunden. Ein Kunde Wird genau von einem Vertreter betreut.

![](image/Pasted%20image%2020250127181924.png)


![](image/Pasted%20image%2020250127181932.png)

---
2 

Von den Kunden werden Aufträge entgegengenommen, die jeweils ein bis mehrere Artikel de Sortiments des Unternehmens in bestimmten Mengen umfassen.

![](image/Pasted%20image%2020250127182207.png)


![](image/Pasted%20image%2020250127182217.png)


---
3 
Vertreter sind hierarchisch jeweils einem Hauptvertreter unterstellt (Hauptvertreter müssen
nicht unterstellt sein.).

![](image/Pasted%20image%2020250127182156.png)


---

4
Die Kundenaufträge werden je nach Bearbeitungsstand in 4 Status-Kategorien eingeordnet (erfasst, bestätigt, geliefert, fakturiert). Jeder Auftrag kann zu einem Zeitpunkt nur genau einer Status annehmen 

Die alten Statusangaben sollen bei Statusänderung erhalten bleiben.


![](image/Pasted%20image%2020250127182322.png)



5

Einzelne Auftragspositionen können mit Teillieferungen beliefert werden, so dass die noch zu
liefernde Restmenge jederzeit ablesbar sein muss. Pro Position werden ein Liefertermin und ein ositionss ezifischer Preis vereinbart.

![](image/Pasted%20image%2020250127182353.png)


![](image/Pasted%20image%2020250127182426.png)


