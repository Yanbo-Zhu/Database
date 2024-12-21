

# 1 Mengenoperationen
Differenz (-) und **Vereinigung** (∪) sowie **Schnitt** (∩)

∪: \cup
∩: \cap


# 2 symmetrische Differenz

Die **symmetrische Differenz** zweier Relationen R1 und R2 ​ kann ohne einen eigenen Operator durch die Kombination der **Mengenoperationen** **Differenz** (−-−) und **Vereinigung** (∪) sowie **Schnitt** (∩) gebildet werden. Die symmetrische Differenz besteht aus den Tupeln, die entweder in R1 oder in R2 sind, aber nicht in beiden.


Definition der symmetrischen Differenz: 
Die symmetrische Differenz R1ΔR2​ ist definiert als:
![](image/Pasted%20image%2020241221220437.png)

Das bedeutet:
- Die Tupel, die nur in R1 sind, aber nicht in R2.
- Die Tupel, die nur in R2​ sind, aber nicht in R1​.

## 2.1 **Analyse der gegebenen Optionen:**

1. **(R1 ∩ R2) - (R1 U R2)**
    - **Falsch**: Dies ist eine leere Menge, weil (R1∩R2)(R1 \cap R2)(R1∩R2) ein Teil von (R1∪R2)(R1 \cup R2)(R1∪R2) ist, und eine Differenz zwischen einer Teilmenge und einer Obermenge ergibt eine leere Menge.
2. **((R1 - R2) U (R2 - R1))**
    - **Korrekt**: Dies entspricht genau der Definition der symmetrischen Differenz. Es ist die Vereinigung der Tupel, die nur in R1R_1R1​ sind (also R1−R2R_1 - R_2R1​−R2​) und der Tupel, die nur in R2R_2R2​ sind (also R2−R1R_2 - R_1R2​−R1​).
3. **(R1 ∩ R2) U (R1 U R2)**
    - **Falsch**: Dies ist die Vereinigung der Schnittmenge und der Vereinigungsmenge, was nicht der Definition der symmetrischen Differenz entspricht.
4. **((R2 - R1) U (R1 - R2))**
    - **Korrekt**: Dies entspricht ebenfalls der symmetrischen Differenz, da es die Tupel aus R2R_2R2​ und R1R_1R1​ enthält, die nicht in der jeweils anderen Relation vorhanden sind.
5. **(R1 U R2) - (R1 ∩ R2)**
    - **Korrekt**: Diese Formulierung beschreibt ebenfalls die symmetrische Differenz. Es ist die Vereinigung von R1R_1R1​ und R2R_2R2​ minus der Schnittmenge von R1R_1R1​ und R2R_2R2​, also die Tupel, die entweder nur in R1R_1R1​ oder nur in R2R_2R2​ sind.








