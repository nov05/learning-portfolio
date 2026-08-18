# 🟢 **Read Skew**  

Page 317 / 673    
```text
Say Aaliyah has $1,000 of savings at a bank, split across two accounts with $500 each.
A transaction transfers $100 from one of her accounts to the other. If she is unlucky
enough to look at her list of account balances in the same moment as that transaction
is being processed, she may see one account balance before the incoming payment
has arrived (still at $500) and the other after the outgoing transfer has been made (the
new balance being $400). To Aaliyah, it now appears as though she has only a total of
$900 in her accounts—it seems that $100 has vanished into thin air.

This anomaly is called read skew, and it is an example of a nonrepeatable read: if
Aaliyah were to read the balance of account 1 again at the end of the transaction,
she would see a different value ($600) than she saw in her previous query. Read skew
is considered acceptable under read-committed isolation: the account balances that
Aaliyah saw were indeed committed at the time when she read them.
```

Figure 8-6. Read skew: Aaliyah observes the database in an inconsistent state  

```text
Snapshot isolation [38] is the most common solution to this problem.
```

<br>  

