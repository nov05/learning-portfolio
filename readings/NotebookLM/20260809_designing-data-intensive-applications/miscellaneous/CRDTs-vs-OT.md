Page 251 / 673

* CRDTs = confict-free replicated datatypes  
* OT = operational transformation


```text
Confict-free replicated datatypes and operational transformation

Two families of algorithms are commonly used to implement automatic conflict reso!
lution: con!ict-free replicated datatypes (CRDTs) [46] and operational transformation
(OT) [47]. They have different design philosophies and performance characteristics,
but both are able to perform automatic merges for all the aforementioned types of
data.

Figure 6-11 shows an example of how OT and a CRDT merge concurrent updates to
a text. Assume you have two replicas that both start off with the text ice. One replica
prepends the letter n to make nice, while concurrently the other replica appends an
exclamation mark to make ice!.
```

