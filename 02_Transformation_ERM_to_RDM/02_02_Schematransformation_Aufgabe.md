

# 1 


![](image/Pasted%20image%2020250127213512.png)

![](image/Pasted%20image%2020250127213648.png)

Resultierende Relationenschemata:
vertreter (vertnr, name, kunde->kdnr ?, chef->vertreter.vertnr)
(vertreterhierarchie (mitarbeiter ->vertreter.vertnr,chef ->vertreter.vertnr)
 -> Standardtransformationsregel für rekursive Beziehungstypen

kunde (kdnr, vertnr ->vertreter.vertnr, branche ->branche.branchennr, kdgruppe -> 

kdgruppe.kdgruppennr, …)

branche (branchennr, … )

kdgruppe (kdgruppennr,… )

auftrag (aufnr, kunde ->kunde.kdnr, status -> auftragsstatus.statusnr)

auftragsstatus (statusnr, …)

artikel (artnr, produktgruppe ->produktgruppe.prodgrunr )
->  Nachfolgend 3 Varianten der Bildung des Primärschlüssels bei auftragsposition (Farben / Unterstreichung)

auftragsposition (artnr->artikel.artnr,aufnr -> auftrag.aufnr, positionsnr, positionsnr, 

bestellmenge,restmenge,termin,preis)


produktgruppe (prodgrunr, …)



