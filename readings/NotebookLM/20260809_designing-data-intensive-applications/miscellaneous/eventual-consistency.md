Page 233 / 673

```text
In this read-scaling architecture, you can increase the capacity for serving read-only
requests simply by adding more followers. However, this approach realistically works
only with asynchronous replication. If you tried to synchronously replicate to all
followers, a single node failure or network outage would make the entire system
unavailable for writing. And the more nodes you have, the likelier it is that one will be
down, so a fully synchronous configuration would be very unreliable.

Unfortunately, an application reading from an asynchronous follower may see outdated
information if the follower has fallen behind. This leads to apparent inconsistencies in
the database; if you run the same query on the leader and a follower at the same time,
you may get different results, because not all writes have been reflected in the follower.
This inconsistency is a temporary state—if you stop writing to the database and wait a
while, the followers will eventually catch up and become consistent with the leader. For
that reason, this effect is known as eventual consistency [22].

    The term eventual consistency was coined by Douglas Terry et al.
    [23] and popularized by Werner Vogels [24], and it became the
    battle cry of many NoSQL projects. However, it’s not only NoSQL
    databases that are eventually consistent; followers in an asynchro!
    nously replicated relational database have the same characteristics.

The term “eventually” is deliberately vague; in general, there is no limit to how far
a replica can fall behind. In normal operation, the delay between a write happening
on the leader and it being reflected on a follower—the replication lag—may be only
a fraction of a second and not noticeable in practice. However, if the system is
operating near capacity or if a problem occurs in the network, the lag can easily
increase to several seconds or even minutes.
```