

![](image/Pasted%20image%2020241221214258.png)

Ein **Theta-Join** (R⋈θSR ) ist eine spezielle Form des Joins, bei dem die Verknüpfung von zwei Relationen R und S auf der Basis einer Bedingung θ erfolgt. ==Wenn θ leer ist (d. h., es gibt keine Bedingung für den Join), reduziert sich der Theta-Join zum **kartesischen Produkt** (R×S).==

# 1 **Erklärung:**

- **Theta-Join Definition**:
    - ![](image/Pasted%20image%2020241221214339.png)
    - Wenn θ\thetaθ leer ist, wird **jede Kombination** von Tupeln aus RRR und SSS akzeptiert.
- **Kartesisches Produkt Definition**:
    - ![](image/Pasted%20image%2020241221214346.png)
    - Das kartesische Produkt kombiniert **alle Tupel** von RRR mit **allen Tupeln** von SSS, unabhängig von einer Bedingung.

Wenn der Theta-Join keine Bedingung (θ\thetaθ) spezifiziert, entspricht das Ergebnis dem **kartesischen Produkt** der beiden Relationen RRR und SSS.


# 2 Beispiel

**R**

|A|
|---|
|1|
|2|

**S**

|B|
|---|
|x|
|y|

**Kartesisches Produkt (R × S):**

|A|B|
|---|---|
|1|x|
|1|y|
|2|x|
|2|y|

Theta-Join (R⋈θSR \bowtie_\theta SR⋈θ​S) mit leerer Bedingung:
Das Ergebnis ist identisch mit dem kartesischen Produkt.




