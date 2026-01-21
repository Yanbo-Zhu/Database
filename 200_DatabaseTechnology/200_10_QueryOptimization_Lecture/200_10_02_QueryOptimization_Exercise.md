
# 1 Introduction
## 1.1 What is Query Optimization in the context of DBMSs?
solution

 The process of transforming a query plan into a semantically equivalent, but more efficient query plan.

## 1.2 What is the difference between a logical and a physical query plan?
solution

Logical Query Plan 
 uses relational algebra operations, describes "what" should be computed, agnostic of DBMS implementation

Physical Query Plan 
 uses internal DBMS operators, describes "how" something should be computed, DBMS implementation-specific

## 1.3 Which of the following are valid algebraic laws?


![](image/Pasted%20image%2020260121171258.png)

solution

Not generally valid, projection may remove attributes necessary for selection predicate

Valid, push predicates to the correct relations

# 2 Logical Plan Optimization / Heuristics

Consider an Internet Movie Database:
```
### Movies Table

| **MovieID** | **Title**            | **ReleaseYear** | **Genre**   |
|-------------|----------------------|-----------------|-------------|
| 1           | Inception           | 2010            | Sci-Fi      |
| 2           | The Dark Knight     | 2008            | Action      |
| 3           | Pulp Fiction        | 1994            | Crime       |
| 4           | The Matrix          | 1999            | Sci-Fi      |
| 5           | Forrest Gump        | 1994            | Drama       |


### Actors Table

| **ActorID** | **Name**             | **MovieID** | **Role**      |
|-------------|----------------------|-------------|---------------|
| 1           | Leonardo DiCaprio   | 1           | Protagonist   |
| 2           | Christian Bale      | 2           | Protagonist   |
| 3           | Samuel L. Jackson   | 3           | Supporting    |
| 4           | Keanu Reeves        | 4           | Protagonist   |
| 5           | Tom Hanks           | 5           | Protagonist   |


### Reviews Table

| **ReviewID** | **MovieID** | **Reviewer** | **Rating** |
|--------------|-------------|--------------|------------|
| 1            | 1           | Alice        | 9.0        |
| 2            | 2           | Bob          | 8.5        |
| 3            | 3           | Charlie      | 9.3        |
| 4            | 4           | Alice        | 8.7        |
| 5            | 5           | Bob          | 8.9        |

```


Cardinalities:
Movies: 1K rows
ActorsInMovies: 10K rows
Reviews: 50K rows


Consider the following SQL Query:
 Return the ratings of the movies that Christian Bale played in and that happened after 2010.

```
SELECT m.Title, a.role AS Actor, r.Rating
FROM Movies m, ActorsInMovies a, Reviews r
WHERE m.MovieID = a.MovieID
AND m.MovieID = r.MovieID;
AND m.ReleaseYear >= 2010
AND a.Name = 'Christian Bale'
```

---
1 
```
from graphviz import Digraph

query_plan = Digraph(format="svg", graph_attr={"rankdir": "BT"})

query_plan.node("Movies", "Movies (Base Table)")
query_plan.node("ActorsInMovies", "ActorsInMovies (Base Table)")
query_plan.node("Reviews", "Reviews (Base Table)")

with query_plan.subgraph() as s:
    s.attr(rank="same")
    s.node("Movies")
    s.node("ActorsInMovies")
    s.node("Reviews")

query_plan.node("Cross1", "Cross Product: Movies × ActorsInMovies")
query_plan.node("Cross2", "Cross Product: (Cross1) × Reviews")

query_plan.node("Filter1", "Filter: m.MovieID = a.MovieID")
query_plan.node("Filter2", "Filter: m.MovieID = r.MovieID")
query_plan.node("Filter3", "Filter: m.ReleaseYear > 2010")
query_plan.node("Filter4", "Filter: a.name = 'Christian Bale'")

query_plan.node("Projection", "Projection: m.Title, a.role, r.Rating")

query_plan.edge("Movies", "Cross1")
query_plan.edge("ActorsInMovies", "Cross1")
query_plan.edge("Cross1", "Cross2")
query_plan.edge("Reviews", "Cross2")

query_plan.edge("Cross2", "Filter1")
query_plan.edge("Filter1", "Filter2")
query_plan.edge("Filter2", "Filter3")
query_plan.edge("Filter3", "Filter4")
query_plan.edge("Filter4", "Projection")

query_plan
```

![](image/Pasted%20image%2020260121193323.png)



2
```
# solution
from graphviz import Digraph

query_plan = Digraph(format="svg", graph_attr={"rankdir": "BT"})

query_plan.node("Movies", "Movies (Base Table)")
query_plan.node("ActorsInMovies", "ActorsInMovies (Base Table)")
query_plan.node("Reviews", "Reviews (Base Table)")

with query_plan.subgraph() as s:
    s.attr(rank="same")
    s.node("Movies")
    s.node("ActorsInMovies")
    s.node("Reviews")

query_plan.node("Join1", "Join: Movies ⋈ ActorsInMovies ON m.MovieID = a.MovieID")
query_plan.node("Join2", "Join: (Join1) ⋈ Reviews ON m.MovieID = r.MovieID")

query_plan.node("Filter1", "Filter: m.ReleaseYear > 2010")
query_plan.node("Filter2", "Filter: a.Name = 'Christian Bale'")

query_plan.node("Projection", "Projection: m.Title, a.role, r.Rating")

query_plan.edge("Movies", "Join1")
query_plan.edge("ActorsInMovies", "Join1")
query_plan.edge("Join1", "Join2")
query_plan.edge("Reviews", "Join2")

query_plan.edge("Join2", "Filter1")
query_plan.edge("Filter1", "Filter2")
query_plan.edge("Filter2", "Projection")

query_plan
```

![](image/Pasted%20image%2020260121193359.png)


3 
```
# solution
from graphviz import Digraph

query_plan = Digraph(format="svg", graph_attr={"rankdir": "BT"})

query_plan.node("Movies", "Movies (Base Table)")
query_plan.node("ActorsInMovies", "ActorsInMovies (Base Table)")
query_plan.node("Reviews", "Reviews (Base Table)")

with query_plan.subgraph() as s:
    s.attr(rank="same")
    s.node("Movies")
    s.node("ActorsInMovies")
    s.node("Reviews")

query_plan.node("Join1", "Join: Movies ⋈ ActorsInMovies ON m.MovieID = a.MovieID")
query_plan.node("Join2", "Join: (Join1) ⋈ Reviews ON m.MovieID = r.MovieID")

query_plan.node("Filter1", "Filter: m.ReleaseYear > 2010")
query_plan.node("Filter2", "Filter: a.Name = 'Christian Bale'")

query_plan.node("Projection", "Projection: m.Title, a.role, r.Rating")

query_plan.edge("Movies", "Filter1")
query_plan.edge("Filter1", "Join1")
query_plan.edge("Filter2", "Join1")

query_plan.edge("ActorsInMovies", "Filter2")
query_plan.edge("Join1", "Join2")
query_plan.edge("Reviews", "Join2")

query_plan.edge("Join2", "Projection")

query_plan
```

![](image/Pasted%20image%2020260121193418.png)

![](image/Pasted%20image%2020260121203230.png)

# 3 Cardinality Estimation

SELECT COUNT(*)
FROM Movies
WHERE Genre = 'Sci-Fi';

## 3.1 What is the output cardinality of the selection?

![](image/Pasted%20image%2020260121193816.png)


## 3.2 How will the estimate change given statistics for Movies.Genre?

```
import matplotlib.pyplot as plt

genres = [
    "Action",
    "Drama",
    "Comedy",
    "Sci-Fi",
    "Horror",
    "Thriller",
    "Romance",
    "Fantasy",
    "Documentary",
    "Crime",
]
counts = [150, 180, 120, 200, 50, 100, 80, 70, 30, 20]

plt.figure(figsize=(8, 5))  # Smaller figure size for better visibility
plt.bar(genres, counts, color="skyblue", edgecolor="black")

plt.xlabel("Genre", fontsize=10)
plt.ylabel("Number of Movies", fontsize=10)
plt.title("Movie Genre Counts", fontsize=12)
plt.xticks(rotation=45, ha="right")

plt.text(2.5, 201, f"Sci-Fi: {counts[3]}", fontsize=10, color="red")
plt.tight_layout()
plt.show()
```

![](image/Pasted%20image%2020260121194021.png)

![](image/Pasted%20image%2020260121194119.png)


## 3.3 Selection with Conjunction

```
SELECT Title, Genre, ReleaseYear
FROM Movies
WHERE Genre = 'Sci-Fi' AND ReleaseYear > 2000;
```

![](image/Pasted%20image%2020260121194147.png)


![](image/Pasted%20image%2020260121194221.png)


## 3.4 Selection with Disjunction

```
SELECT Title, Genre, ReleaseYear
FROM Movies
WHERE Genre = 'Drama' OR ReleaseYear < 1995;
```

![](image/Pasted%20image%2020260121194339.png)

# 4 Join Ordering / Cost-based Optimization

Santa's workshop is bustling with activity as Christmas Eve approaches. Santa gathers his most trusted elves and says:

"Elves, we have a critical task ahead! We need to make sure every child's wish comes true. I need you to query our magical database and find out exactly where the presents need to be picked up from and which child they need to be delivered to. Start with the children's wishlists, match them with the available presents, and then figure out which factory each present needs to be collected from. Finally, link everything back to the children so we know where to deliver. And remember, optimize your joins—time is of the essence, and inefficiencies could delay Christmas!"

"精灵们，我们面前有一项至关重要的任务！我们必须确保每一个孩子的愿望都能实现。
我需要你们去查询我们的魔法数据库，弄清楚礼物应该从哪里取走，又该送给哪个孩子。
从孩子们的愿望清单开始，把它们与现有的礼物进行匹配，然后找出每件礼物需要从哪一家工厂取货。
最后，把所有信息重新关联回孩子们，这样我们就知道该把礼物送到哪里。
还有，记得优化你们的连接（joins）——时间非常紧迫，任何低效的操作都可能耽误圣诞节！"



```
### Children

| cid            | ChildName | City     |
| -------------- | --------- | -------- |
| 1              | Anna      | Munich   |
| 2              | Ben       | New York |
| 3              | Catherine | London   |
| 4              | Daniel    | Sydney   |
| 5              | Ella      | Tokyo    |

### Wishlists

| cid → Children            | pid → Presents            | WishlistItem |
| ------------------------- | ------------------------- | ------------ |
| 1                         | 201                       | Teddy Bear   |
| 1                         | 202                       | Remote Car   |
| 2                         | 203                       | LEGO Set     |
| 3                         | 201                       | Teddy Bear   |
| 4                         | 204                       | Board Game   |

### Presents

| id             | PresentName | fid → Factories |
| -------------- | ----------- | --------------- |
| 201            | Teddy Bear  | 301             |
| 202            | Remote Car  | 302             |
| 203            | LEGO Set    | 303             |
| 204            | Board Game  | 304             |
| 205            | Soccer Ball | 305             |

### Factories

| fid            | FactoryName         | Location   |
| -------------- | ------------------- | ---------- |
| 301            | North Pole Workshop | Arctic     |
| 302            | Santa\'s Tech Hub   | California |
| 303            | LEGO Factory        | Denmark    |
| 304            | Board Game Co.      | Germany    |
| 305            | Sports Center       | Brazil     |

```

## 4.1 Query

```
SELECT 
    c.ChildName, 
    w.WishlistItem, 
    f.FactoryName, 
    p.PresentName
FROM 
    Children c
    JOIN Wishlists w ON c.ChildID = w.ChildID
    JOIN Presents p ON w.PresentID = p.PresentID
    JOIN Factories f ON p.FactoryID = f.FactoryID;
```

![](image/Pasted%20image%2020260121194720.png)

## 4.2 Approach

![](image/Pasted%20image%2020260121194744.png)

![](image/Pasted%20image%2020260121194751.png)



# 5 # 4 Join Ordering / Cost-based Optimization 2

![](image/Pasted%20image%2020260121203412.png)



![](image/Pasted%20image%2020260121203311.png)


![](image/Pasted%20image%2020260121203322.png)

![](image/Pasted%20image%2020260121203333.png)


![](image/Pasted%20image%2020260121203356.png)

