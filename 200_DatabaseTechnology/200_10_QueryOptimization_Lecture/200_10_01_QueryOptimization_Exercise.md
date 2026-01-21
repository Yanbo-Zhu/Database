
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
