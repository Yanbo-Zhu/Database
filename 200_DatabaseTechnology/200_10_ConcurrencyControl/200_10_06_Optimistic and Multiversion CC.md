
# 1 Disadvantages of Locking

• Pessimistic approach to concurrency control
    • Imposes a lot of overhead, including
    • Locks for read-only transactions
    • deadlock detection
• Optimistic approaches are based on the hope that conflicts will be rare
    • Fix things if conflicts happen, do not preempt conflicts
    • If something goes wrong, abort and restart transaction

Possible solutions:
    • Timestamp ordering protocol
    • Multi-Version Concurrency Control



# 2 Time Stamp Ordering Protocol

‣ Idea: order transactions based on their timestamp
‣ Protocol has to make sure that for each item accessed by Conflicting Operations in the schedule, the order in which the item is accessed does not violate the ordering
‣ Tool: add two timestamp values to each database item:
    ‣ W_TS(X) is the largest timestamp of any transaction that executed write(X) successfully on this item
    ‣ R_TS(X) is the largest timestamp of any transaction that executed read(X) successfully on this item
    ‣ TS(Ti) => every Tx gets a timestamp based on when it enters the system


Simple TSO Algorithm
‣ Case 1: Whenever a Transaction T issues a write(X) operation, check the following conditions:
    ‣ If R_TS(X) > TS(T) then abort (a more recent thread is already relying on the old value)
    ‣ If W_TS(X) > TS(T), then skip (Thomas rule state a later write will anyway overwrite this so ignore it)
    ‣ Else: Execute W_item(X) operation of T and set W_TS(X) to TS(T).
‣Case 2: Whenever a Transaction T issues a read(X) operation, check the following conditions:
    ‣ If W_TS(X) > TS(T) then abort (a more recent thread has overwritten the value)
    ‣ Else: execute the R_item(X) operation of T and set R_TS(X) to the larger of TS(T) and current R_TS(X).
‣ Whenever two conflicting operations occur in incorrect order, it reject the latter of the two by aborting the transaction


# 3 Multiversion Concurrency Control

• Keep multiple versions of data items
• Each transaction operates on private copy of data, copies are merged later
• It can in addition support transaction-time temporal databases and time-travel queries

![](image/Pasted%20image%2020250118202446.png)




Snapshot Isolation
• A transaction T executing in SI reads data from a snapshot of the committed data as of the time the transaction started
• Start-Timestamp(T): any time before the transaction’s first read
• T’s reads are never blocked as long as the snapshot can be maintained
• T’s writes are reflected in the snapshot, and are read again
• Updates by transactions that have not committed before Start- Timestamp(T) are invisible



Multiversion Concurrency Control for SI
• When T1 is ready to commit, it obtains a Commit-Timestamp
    • Larger than any other ST or CT
• T1 commits if there is no T2 with CT(T2) in [ST(T1),CT(T1)]
• “First committer wins” rule
    • It is not possible to have two concurrently active transactions that both commit and modify the same data item
• SI allows more concurrency than S2PL, especially for read-mostly transactions
• Implemented in many systems
• Algorithm can be extended to allow serializability with modest overhead
(Paper: M. J. Cahill et al. Serializable isolation for snapshot databases. SIGMOD 2008)