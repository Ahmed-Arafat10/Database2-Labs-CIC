## Written By Ahmed Arafat

### Let's recap on important Terminologies

- `Transaction` : A transaction in a database is a logical unit of work that comprises one or more database operations,
  such as reads or
  writes, grouped together to perform a specific task. It represents a sequence of actions that must be executed as a
  single indivisible unit, ensuring either all operations within the transaction are completed successfully and
  committed
  to the database, or none of them are applied (rolled back), maintaining the database's consistency and integrity.
  Transactions in databases adhere to the ACID (Atomicity, Consistency, Isolation, Durability)

- `Concurrency Vs Serial`
    - `Concurrency` in databases allows multiple transactions to execute simultaneously, enabling parallelism and
      efficient utilization of resources. It enables concurrent access to shared data but necessitates mechanisms like
      locks and isolation levels to maintain consistency and prevent conflicts, ensuring that transactions don't
      interfere with each other's operations.
    - `Serial` execution refers to the sequential processing of transactions, where each transaction executes one after
      the other without any overlap. It guarantees simplicity and avoids concurrency-related issues but can result in
      reduced system throughput and resource underutilization, especially in multi-user environments with high demand
      for access to data.

- `Concurrency control problems` in databases primarily include issues like conflicts between transactions accessing the
  same data concurrently, leading to anomalies such as lost updates, dirty reads, non-repeatable reads, and phantom
  reads. These problems arise due to the simultaneous execution of transactions without proper coordination, potentially
  compromising data consistency and integrity unless managed by mechanisms like locking, isolation levels, or
  timestamp-based protocols to ensure proper synchronization and prevent conflicting interactions between transactions.

- `Schedule` : A schedule in a database context is a chronological sequence of operations (read or write) from multiple
  transactions, representing the order in which these operations are executed. It determines the timeline of actions
  performed by transactions on the database, illustrating their concurrency and interaction.

- `Serializability`:  is a concept in database transactions that ensures the transactions are executed in a manner that
  `produces the same result as if they were executed serially`, one after the other, rather than concurrently. It
  guarantees that the final state of the database is consistent and equivalent to some serial execution of the
  transactions.

Techniques like `locking`, concurrency control protocols, and isolation levels (such as serializable isolation level in
databases) are used to achieve serializability. These methods ensure that transactions are scheduled and executed in a
way that prevents interference or conflicting operations on the database, maintaining the consistency and integrity of
the data despite concurrent execution by multiple users or processes.

#### `Lock-based protocols`

`Lock-based protocols` are a fundamental approach used in database management systems to control concurrent access to
shared resources, particularly in multi-user environments where multiple transactions might attempt to access and modify
the same data simultaneously.

These protocols utilize locks as a means to control access to data items, ensuring that only one transaction can modify
a resource at a time while maintaining data consistency and integrity. Locks can come in various types:

1. `Shared Locks (S):` Allow multiple transactions to read a resource concurrently but prevent any transaction from
   modifying it.

2. `Exclusive Locks (X):` Grant exclusive access to a single transaction for writing or modifying a resource, preventing
   other transactions from reading or writing it simultaneously.

Lock-based protocols typically involve the following operations:

- `Lock:` A transaction requests a lock (either shared or exclusive) on a data item before accessing it.

- `Unlock:` Once a transaction has completed its operations on a data item, it releases the lock, allowing other
  transactions to access it.

### `Exclusive (X) mode`

`Exclusive mode` : in lock-based concurrency control refers to a locking mechanism where a transaction holds exclusive
access to a resource, typically a data item or a portion of the database, preventing other transactions from reading or
writing to that resource simultaneously.

In the context of database concurrency control:

- Exclusive locks are used when a transaction intends to modify a data item.
- While a transaction holds an exclusive lock on a resource, no other transaction can obtain a conflicting lock (like a
  shared lock or another exclusive lock) on the same resource until the lock is released.
- This mode ensures that only one transaction at a time has the privilege to modify the locked data, preventing issues
  such as dirty reads, non-repeatable reads, and phantom reads.

When a transaction requests an exclusive lock, the database management system grants it if no other transaction holds a
conflicting lock on the resource. If another transaction already holds a lock on the same resource, the requesting
transaction might need to wait until the lock is released.

This approach helps maintain data consistency and integrity by ensuring that changes made by one transaction are not
interfered with by concurrent transactions. However, it can also lead to potential issues like `deadlocks` if not
managed properly, where two or more transactions are stuck waiting for resources held by each other, causing a cycle of
inactivity. Therefore, deadlock detection and resolution mechanisms are essential in lock-based concurrency control
systems.

### Example:

- In case of not using lock-based protocol

````
    t1                          t2
   
Read(A) // A=100
A=A+20 // A=120
                            Read(A) // A=100
                            A=A+50 // A=150
                            Write(A) // A = 150
Write(A) // A = 120
````

> now `A` will be `120` instead of `170` <br>
> As we said before this will lead to data inconsistency

- To solve this using `Exclusive (X) Lock`:

````
    t1                          t2
    
lock-X(A);// Acquire exclusive lock on A
Read(A); // A=100
A=A+20 // A=120
                            lock-X(A);// Wait for exclusive lock on A
                            Read(A);// Read current balance ($100) after T1's update (120)
                            A=A+50 // A=170
                            Write(A);// Update balance to 170
                            unlock(A);// Release lock

Write(A);// Update balance to $120
unlock(A);// Release lock
````


### `Shared (S) mode`

the Shared (S) mode refers to a type of lock that allows multiple transactions to read a resource concurrently while
preventing any transaction from writing to that resource.

Specifically, in the context of database systems:

- Shared locks (also known as read locks) are used when a transaction wants to read data without intending to modify it.
- Multiple transactions can hold shared locks simultaneously on the same resource.
- Shared locks are compatible with other shared locks but are incompatible with exclusive locks. This means that
  multiple transactions can read a resource concurrently, but when a transaction wants to modify (write to) that
  resource, it needs an exclusive lock, which is not compatible with shared locks.

The purpose of shared locks is to facilitate concurrent access for reading operations while ensuring that no conflicting
operations (write operations) can occur simultaneously. This mechanism helps maintain consistency and prevents issues
like dirty reads, where a transaction reads uncommitted changes made by another transaction.

For instance, if Transaction T1 holds a shared lock on a data item, another transaction (T2) can also acquire a shared
lock on the same data item, allowing both transactions to read the data concurrently. However, if any of these
transactions want to modify the data (write operation), they would need to acquire an exclusive lock, which isn't
possible while other transactions hold shared locks. 
- example fo preventing Non-Repeatable Reads:
````
        t1                      t2
        Lock-S(A)
        read(A) // A=100
                                read(A) // A=100  
                                A=A+100 //A=200  
                                write(A);// T2 tries to update the balance but waits due to T1's lock
        read(A);// Reads the balance again: $100 (still under shared lock)
        Unlock(A)
````

### `lock compatibility matrix`

A lock compatibility matrix is a representation that outlines which types of locks are compatible with each other and
which types conflict, preventing concurrent access in lock-based concurrency control mechanisms. This matrix helps in
understanding the rules governing the compatibility of different types of locks when multiple transactions are
attempting to access shared resources concurrently.

Here's a simplified example of a lock compatibility matrix:

| Lock Type     | Shared (S)   | Exclusive (X) |
|---------------|--------------|---------------|
| Shared (S)    | Compatible   | Incompatible  |
| Exclusive (X) | Incompatible | Incompatible  |

In this basic matrix:

- Shared locks (S) are compatible with other shared locks because they allow multiple transactions to read a resource
  concurrently.
- Exclusive locks (X) are incompatible with both shared and other exclusive locks. They prevent any other locks (shared
  or exclusive) from being held on the same resource, ensuring exclusive access for writing or modifying the resource.

The matrix helps determine whether a transaction can acquire a lock on a resource based on the types of locks already
held by other transactions. For example:

- If Transaction T1 holds an exclusive lock (X) on a resource, any attempt by Transaction T2 to acquire either a
  shared (S) or exclusive (X) lock on the same resource will be incompatible and will have to wait until T1 releases its
  exclusive lock.
- If Transaction T3 holds a shared lock (S) on a resource, other transactions can also acquire shared locks (S) on the
  same resource concurrently, but any attempt to acquire an exclusive lock (X) will be incompatible until all shared
  locks are released.

These compatibility rules are crucial for maintaining data consistency, preventing conflicts between transactions, and
avoiding issues like deadlocks or data corruption in concurrent environments by regulating access to shared resources.

The sequence of locks and unlocks as outlined in Transaction T2:

```
T2:
lock-S(A);
read (A);
unlock(A);
lock-S(B);
read (B);
unlock(B);
display(A+B);
```

This sequence doesn't ensure serializability due to the lack of a consistent and coordinated approach to locking
multiple resources. The issue arises from the fact that T2 releases lock A before acquiring lock B, potentially leading
to a scenario where another transaction could modify data between the releases and acquisitions of locks A and B,
causing inconsistency.

For example, consider this scenario:

1. Transaction T1 reads and modifies data item A.
2. T2 acquires a shared lock on A, reads its value, and then releases the lock on A.
3. In between the release of lock A by T2 and its acquisition of lock B, Transaction T3 modifies data item A.
4. T2 acquires a shared lock on B and reads its value.

In this case, T2's read of B after releasing the lock on A might lead to an inconsistency because A was modified by T3
in the interim period, which T2 wasn't aware of due to the lock release.

To ensure serializability and consistency in such cases, a more robust approach would involve:

- Acquiring and holding all necessary locks until the `entire` transaction is complete to prevent interference from
  other
  transactions.

In the given scenario, ensuring that Transaction T2 maintains its locks on both A and B until the display operation
would help guarantee serializability and prevent potential data inconsistencies due to interleaving transactions
modifying shared resources. somthing like this:

```
T2:
lock-S(A);
read (A);
lock-S(B);
read (B);
display(A+B);
unlock(A);
unlock(B);
```

### `Deadlock`

`Deadlock` is a situation in a concurrent system where two or more competing actions are each waiting for the other to
finish, preventing all involved actions from progressing. In a database context with exclusive locks, a deadlock can
occur when two or more transactions are each holding locks that the other transactions need before they can proceed.

Here's an example illustrating a deadlock situation with exclusive locks:

Consider two transactions, T1 and T2, operating on two different data items, A and B, and using exclusive locks.

1. Transaction T1 acquires an exclusive lock on data item A and then needs data item B.

2. At the same time, Transaction T2 holds an exclusive lock on data item B and needs data item A.

````
    t3              t4
  lock-X(A)     
    ----
    ----
    ----    
                   lock-S(B)
                   Read(B)
                   Lock-S(A)
  lock-X(B)   
````

Now, both T1 and T2 are waiting for the other to release the data item they hold before they can proceed. But neither
transaction can progress because each is waiting for the other's locked resource to become available.

The situation can be depicted as follows:

- T1 holds A and requests B
- T2 holds B and requests A

As a result, both transactions are stuck in a state of waiting for the other, which creates a deadlock. In such a
scenario, without proper deadlock detection and resolution mechanisms in place, these transactions will remain in a
deadlock state indefinitely, causing a halt in the system's progress.

Database management systems implement various deadlock handling techniques, such as `timeout` mechanisms or using
algorithms to detect and break deadlocks by `aborting` one of the transactions involved to allow the others to proceed.
These mechanisms aim to prevent indefinite waiting and ensure the system can continue to function by resolving deadlocks
as they occur.

### `Starvation`

Starvation refers to a scenario in a system where a transaction, process, or entity is unable to make progress because
it's persistently bypassed or delayed by other transactions or processes. In the context of lock-based protocols in
databases, starvation can occur when a transaction is unable to acquire a required lock due to the continuous granting
of the lock to other transactions, causing it to wait indefinitely.

In lock-based protocols, different transactions contend for access to shared resources, such as data items, through
acquiring locks. Starvation might occur due to various reasons:

- `Lock Priority`: Some lock-based protocols might prioritize certain transactions over others. If higher priority
  transactions continually acquire locks, lower priority transactions might be starved, waiting indefinitely for access
  to the resources they need.
- `Resource Utilization`: If certain transactions continuously monopolize resources, repeatedly acquiring and releasing
  locks in a way that doesn’t allow others to proceed, it can lead to starvation for transactions that require those
  resources.
