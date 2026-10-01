---
type: review-queue
generated: 2026-10-01T16:24
---
# Review queue

228 drafts waiting for your review.

Tick a note once you have read it and agree with it, then ask Claude to file your approved notes:
`verify-ticked` sets `status: verified` on exactly the ticked notes and `promote` files them.
A tick counts only while the note is unchanged since this queue was written; a note edited after
that comes back unticked for another read. Claude never ticks a box here.

## concurrency → 10-notes/concurrency/ (11)
- [ ] [[SELECT FOR UPDATE on a parent row blocks inserts of child rows that reference it]] · 0.931 · ready · check the numbers %%h:298eb6efcc89%%
- [ ] [[A synchronized block cannot prevent a lost update between two application instances]] · 0.925 · ready · check the numbers %%h:8ef26302b289%%
- [ ] [[A conditional UPDATE with the stock check in its WHERE clause cannot oversell under READ COMMITTED]] · 0.91 · ready · check the numbers %%h:aeb346257839%%
- [ ] [[An optimistic version check still takes a row lock, so an overlapping writer waits and then updates zero rows]] · 0.906 · ready · check the numbers · read with [[A version column detects a lost update at write time instead of blocking the other writer]] %%h:cde4aa9d7ab6%%
- [ ] [[SKIP LOCKED lets several workers claim different rows of a job table without waiting]] · 0.883 · ready · check the numbers %%h:6ede4f9a15f1%%
- [ ] [[A version column detects a lost update at write time instead of blocking the other writer]] · 0.862 · ready · check the numbers %%h:4e8f799ccfa8%%
- [ ] [[A deadlock aborts the whole transaction in PostgreSQL but only one statement in Oracle]] · 0.85 · ready · check the numbers %%h:002db4394990%%
- [ ] [[A PostgreSQL row-lock wait never times out by default, while NOWAIT and lock_timeout make it fail with 55P03]] · 0.845 · ready · check the numbers %%h:c8accf9a7f03%%
- [ ] [[volatile makes count++ visible to other threads but does not make it atomic]] · 0.903 · borderline · check the numbers %%h:8b42c37a6f72%%
- [ ] [[SELECT FOR UPDATE makes competing writers queue on the row until the lock holder commits]] · 0.88 · borderline %%h:613f640e02c4%%
- [ ] [[SELECT FOR UPDATE cannot prevent a double booking because there is no row to lock yet]] · 0.842 · borderline · check the numbers %%h:1031876f48b9%%

## database → 10-notes/database/ (31)
- [ ] [[PgBouncer transaction pooling breaks session state such as SET, LISTEN and session advisory locks]] · 0.933 · ready · check the numbers %%h:614920b48762%%
- [ ] [[A plain CREATE INDEX blocks writes to the table until the build finishes]] · 0.924 · ready %%h:36203e320902%%
- [ ] [[A trigger-maintained counter on a parent row makes concurrent writers of that parent queue on its row lock]] · 0.913 · ready · check the numbers %%h:466e92916127%%
- [ ] [[A partial index cannot serve a generic plan whose bind parameter decides the predicate]] · 0.912 · ready · check the numbers %%h:c84ea5c0db4a%%
- [ ] [[A PostgreSQL sequence never reuses a value taken by a rolled-back transaction, so identity keys have gaps]] · 0.907 · ready · check the numbers %%h:9785e7807c61%%
- [ ] [[Wrapping an indexed column in a function hides it from a plain PostgreSQL index]] · 0.907 · ready · check the numbers %%h:f20df0924d62%%
- [ ] [[MySQL InnoDB REPEATABLE READ applies UPDATE and DELETE to the latest committed rows, not to the snapshot]] · 0.901 · ready · check the numbers %%h:d4fb3d8dcac4%%
- [ ] [[Under a non-C collation a prefix LIKE cannot use a default PostgreSQL B-tree index]] · 0.893 · ready · check the numbers %%h:8829e13dcf2e%%
- [ ] [[PostgreSQL SERIALIZABLE can abort transactions that changed different rows when their reads scanned the whole table]] · 0.891 · ready · check the numbers %%h:d8792b618ab9%%
- [ ] [[A B-tree index speeds lookups and range scans at the cost of slower writes and more storage]] · 0.89 · ready · check the numbers %%h:30d94cfd3528%%
- [ ] [[A serialization failure must be retried as a new transaction that re-runs its reads]] · 0.885 · ready · check the numbers %%h:6103263a25fc%%
- [ ] [[N+1 queries make round trips grow with the data, while one batch query per level keeps them constant]] · 0.885 · ready · check the numbers %%h:7b274e179b16%%
- [ ] [[PostgreSQL REPEATABLE READ rejects a lost update with SQLSTATE 40001 instead of overwriting the newer row]] · 0.883 · ready · check the numbers %%h:46031eedc29b%%
- [ ] [[A PostgreSQL UNIQUE constraint accepts several NULLs unless it is declared NULLS NOT DISTINCT]] · 0.88 · ready · check the numbers %%h:48dd31409271%%
- [ ] [[Splitting a table is lossless when the columns the two parts share are a key of one part]] · 0.879 · ready · check the numbers %%h:3893cd5fca76%%
- [ ] [[A PostgreSQL materialized view shows the data of its last refresh, not the current data]] · 0.877 · ready · check the numbers %%h:d1d7c1f1d5dd%%
- [ ] [[OFFSET pagination skips or repeats rows when rows before the page change between requests]] · 0.875 · ready · check the numbers %%h:8ec988b73678%%
- [ ] [[Random UUID primary keys make a B-tree index larger than keys inserted in order]] · 0.874 · ready · check the numbers %%h:2f3126c7512a%%
- [ ] [[Correlated columns make the PostgreSQL planner underestimate rows until extended statistics record the dependency]] · 0.872 · ready · check the numbers %%h:9ba59c7e77ca%%
- [ ] [[A covering index lets PostgreSQL answer a query with an index-only scan]] · 0.863 · ready · check the numbers %%h:9010f8825da9%%
- [ ] [[Keyset pagination stays fast on deep pages because it does not read the skipped rows]] · 0.861 · ready · check the numbers %%h:c6e29f1cc784%%
- [ ] [[In PostgreSQL an UPDATE of one indexed column adds a new entry to every index on the table]] · 0.849 · ready · check the numbers %%h:da1215ef332d%%
- [ ] [[In PostgreSQL 15 a keyset condition written as a row-value comparison is an index bound, while the equivalent OR is only a filter]] · 0.783 · ready · check the numbers %%h:82132b086a88%%
- [ ] [[READ COMMITTED gives each statement its own snapshot, so two reads in one transaction can disagree]] · 0.883 · borderline · check the numbers %%h:d4dd19fad4f8%%
- [ ] [[Denormalise a read path only with a mechanism that keeps the copies consistent]] · 0.873 · borderline · check the numbers %%h:d31108329678%%
- [ ] [[A composite B-tree index is most efficient when the query constrains its leading columns]] · 0.872 · borderline · check the numbers %%h:81da7e1cda2b%%
- [ ] [[Write skew survives snapshot isolation because the two transactions write different rows]] · 0.871 · borderline · check the numbers %%h:d31d7626b315%%
- [ ] [[EXPLAIN ANALYZE shows actual rows next to the planner's estimates for every plan node]] · 0.863 · borderline · check the numbers %%h:ff21299bb158%%
- [ ] [[Third normal form removes update anomalies by making every non-key column depend only on the key]] · 0.858 · borderline · check the numbers %%h:1609e8ead4c3%%
- [ ] [[Cardinality decides where a relationship's foreign key goes]] · 0.839 · borderline · check the numbers %%h:a00ae9bfa62a%%
- [ ] [[PostgreSQL forbids more anomalies than the SQL standard requires at READ UNCOMMITTED and REPEATABLE READ]] · 0.824 · borderline · check the numbers %%h:b3c274701915%%

## frontend-interview → 10-notes/frontend-interview/ (4)
- [ ] [[The microtask queue is drained completely after every task, before the next task or a render]] · 0.907 · ready %%h:e357a1511bd1%%
- [ ] [[Awaiting an already-resolved promise does not let the browser render or handle input]] · 0.875 · ready %%h:6d692bfde101%%
- [ ] [[setTimeout(fn, 0) queues a later task instead of running fn immediately]] · 0.861 · ready · check the numbers %%h:986f68b8e1ff%%
- [ ] [[An async function runs synchronously until its first await]] · 0.852 · ready %%h:db438d8d5d21%%

## java-core → 10-notes/java-core/ (27)
- [ ] [[A bound method reference evaluates its receiver once, when the reference is created]] · 0.915 · ready · check the numbers %%h:7ff98420a805%%
- [ ] [[In JDK 21 ZGC is single-generation unless the ZGenerational flag is set]] · 0.914 · ready · check the numbers %%h:24ac463ad6dd%%
- [ ] [[LinkedHashMap in access order with removeEldestEntry is an LRU cache]] · 0.912 · ready %%h:4d0313c0b799%%
- [ ] [[Arrays.asList returns a fixed-size view that writes through to the array]] · 0.911 · ready · check the numbers %%h:eeca1697da28%%
- [ ] [[TreeMap trades HashMap's constant time for sorted keys and range queries]] · 0.9 · ready %%h:770e1b6856b8%%
- [ ] [[List.remove(1) on a List-Integer- removes the element at index 1, not the value 1]] · 0.898 · ready · check the numbers %%h:17071dda27d5%%
- [ ] [[forEach on a parallel stream does not respect the encounter order]] · 0.895 · ready · check the numbers %%h:f097f9caf979%%
- [ ] [[Inside a lambda, this refers to the enclosing instance, not to the lambda]] · 0.895 · ready · check the numbers %%h:315e0fe7bf81%%
- [ ] [[Object.finalize is deprecated for removal, so cleanup belongs in close() with a Cleaner as the safety net]] · 0.893 · ready · check the numbers %%h:df8682436341%%
- [ ] [[A WeakHashMap entry whose value refers to its key is never removed]] · 0.888 · ready · check the numbers %%h:296b704e1f12%%
- [ ] [[A young-generation collection costs roughly what survives it, because most objects die young]] · 0.888 · ready · check the numbers %%h:16ed822241fc%%
- [ ] [[Overriding equals without hashCode makes HashMap lookups miss]] · 0.888 · ready %%h:650081af2316%%
- [ ] [[Since JDK 18, javac stores the enclosing instance in an inner class only when the class uses it]] · 0.887 · ready · check the numbers %%h:8ffe4dbffbbe%%
- [ ] [[A HashMap key whose hashCode changes after put cannot be found until its hash changes back]] · 0.882 · ready · check the numbers %%h:94c3df2299e5%%
- [ ] [[ArrayDeque replaces both Stack and LinkedList for stacks and queues]] · 0.88 · ready %%h:101e7fc40895%%
- [ ] [[A reduce seed that is not a true identity is added once per chunk in a parallel stream]] · 0.877 · ready · check the numbers %%h:bfe1e36e661a%%
- [ ] [[On G1, System.gc() triggers a stop-the-world full collection by default]] · 0.869 · ready · check the numbers %%h:8a6e0e22ba4e%%
- [ ] [[ArrayList is the default List because LinkedList walks its nodes on every indexed access]] · 0.864 · ready %%h:bd905b1bc8fa%%
- [ ] [[Stream intermediate operations are lazy and run only when a terminal operation starts]] · 0.863 · ready %%h:54f6c5ab1ac9%%
- [ ] [[A method reference bound to this is a new, unequal object each time it is evaluated]] · 0.857 · ready · check the numbers %%h:e9599dbb59cb%%
- [ ] [[A HashMap bin turns into a red-black tree only once the table has at least 64 buckets]] · 0.856 · ready · check the numbers %%h:fb671d69e18e%%
- [ ] [[Removing the second-to-last element inside a for-each loop skips the last element without a ConcurrentModificationException]] · 0.85 · ready · check the numbers %%h:d74cb567022a%%
- [ ] [[HashMap iteration order follows the buckets and changes when the table resizes]] · 0.84 · ready · check the numbers %%h:124aef8547a1%%
- [ ] [[Parallel streams share the JVM-wide common ForkJoinPool]] · 0.839 · ready %%h:2cbeaba1c233%%
- [ ] [[A lambda captures the value of a local variable, which is why the variable must be effectively final]] · 0.881 · borderline · check the numbers %%h:158ef0d4a9f1%%
- [ ] [[Stream.toList() returns an unmodifiable list that still accepts null elements]] · 0.878 · borderline · check the numbers %%h:b91a47a5546e%%
- [ ] [[Collectors.toMap throws on a duplicate key unless you pass a merge function]] · 0.871 · borderline · check the numbers %%h:3ebbd28fb1b9%%

## microservices-and-messaging → 10-notes/microservices-and-messaging/ (19)
- [ ] [[Kafka exactly-once covers Kafka-to-Kafka processing, not external side effects]] · 0.94 · ready · check the numbers %%h:7255fc6f8e4a%%
- [ ] [[A saga has no isolation, so concurrent sagas can read and overwrite each other's partly applied updates]] · 0.924 · ready %%h:f1c3b50feffd%%
- [ ] [[An idempotent consumer records a stable message key in the same transaction as its effect, under a unique constraint]] · 0.919 · ready %%h:5724da0051da%%
- [ ] [[Clients that retry a struggling service immediately and without limit keep it from recovering]] · 0.918 · ready %%h:a05997c8ee48%%
- [ ] [[A Kafka consumer reads the records of aborted transactions unless it sets isolation.level to read_committed]] · 0.917 · ready · check the numbers %%h:a26150ff7e5b%%
- [ ] [[A new Kafka consumer group starts at the end of the log by default, so it skips every record already there]] · 0.917 · ready · check the numbers %%h:3565000c3694%%
- [ ] [[Kafka's idempotent producer drops only its own retries within one session, not a second send() of the same record]] · 0.91 · ready · check the numbers %%h:841c6ba35225%%
- [ ] [[A new Kafka producer with the same transactional.id fences the old instance and aborts its open transaction]] · 0.906 · ready · check the numbers %%h:2a634bd4985d%%
- [ ] [[Choreography spreads a saga's flow across event subscriptions, while orchestration keeps it in one coordinator]] · 0.905 · ready %%h:257052a5e73d%%
- [ ] [[A Kafka consumer slower than max.poll.interval.ms loses its partitions to another member, and its offset commit fails]] · 0.902 · ready · check the numbers %%h:4e910c73e2b3%%
- [ ] [[Kafka auto-commit commits the offsets that poll() returned, not the records the application finished]] · 0.898 · ready · check the numbers %%h:8d9090412312%%
- [ ] [[Committing Kafka offsets after processing gives at-least-once delivery, so consumers must be idempotent]] · 0.897 · ready · check the numbers %%h:ce3d78b5a042%%
- [ ] [[Kafka orders records only within a partition, and the record key chooses the partition]] · 0.887 · ready · check the numbers %%h:c9fd4482d9fe%%
- [ ] [[The default Kafka rebalance revokes every partition, while cooperative rebalancing revokes only the ones that move]] · 0.884 · ready · check the numbers %%h:6aa32e3298da%%
- [ ] [[The partition count caps how many consumers in a Kafka consumer group can do work]] · 0.877 · ready %%h:da90c1f21ede%%
- [ ] [[A message that fails on every delivery needs a delivery limit and a dead-letter queue, or it is retried forever]] · 0.856 · ready · check the numbers %%h:9c1d45e253b6%%
- [ ] [[A transactional outbox sends a message if and only if the database transaction commits, but its relay can send it twice]] · 0.821 · ready %%h:e9929070c977%%
- [ ] [[RabbitMQ prefetch caps unacknowledged deliveries per channel, which is back-pressure on the consumer side]] · 0.887 · borderline · check the numbers %%h:42c99847b0c8%%
- [ ] [[A synchronous service call couples the caller to the callee's availability and latency]] · 0.88 · borderline %%h:626e40192250%%

## ops-and-cloud → 10-notes/ops-and-cloud/ (28)
- [ ] [[A StatefulSet partition updates only the Pods whose ordinal is at or above it]] · 0.931 · ready · check the numbers %%h:889b02946733%%
- [ ] [[A StatefulSet gives each pod a stable identity and its own PersistentVolumeClaim]] · 0.922 · ready · check the numbers %%h:d1f41145ae6d%%
- [ ] [[A Kubernetes canary built from two Deployments splits traffic by replica count]] · 0.921 · ready · check the numbers %%h:8e5a5688223f%%
- [ ] [[A canary's errors are diluted in whole-service metrics, so it must be compared with a control group]] · 0.913 · ready · check the numbers %%h:b9f713c04d5b%%
- [ ] [[A StatefulSet Pod keeps its DNS name across restarts but not its IP address]] · 0.91 · ready · check the numbers %%h:1101438f715d%%
- [ ] [[A Deployment rolling update is bounded by maxSurge and maxUnavailable]] · 0.908 · ready · check the numbers %%h:0ad6f7b8baa2%%
- [ ] [[An RDS Multi-AZ standby gives failover, not read capacity]] · 0.904 · ready · check the numbers %%h:2f52bc3d8042%%
- [ ] [[kubectl rollout undo rolls back only the Deployment's Pod template]] · 0.904 · ready · check the numbers %%h:d0f0c678a546%%
- [ ] [[Reverting the template does not unstick a StatefulSet rollout until the broken Pods are deleted]] · 0.903 · ready · check the numbers %%h:28d99070b29e%%
- [ ] [[Every Helm install, upgrade or rollback creates a new release revision]] · 0.899 · ready · check the numbers %%h:f332246d0dfb%%
- [ ] [[Resources created by a Helm hook are not part of the release, so helm uninstall leaves them behind]] · 0.899 · ready · check the numbers %%h:5869dae14a50%%
- [ ] [[The shared responsibility model leaves data and IAM with the customer even on managed services]] · 0.891 · ready · check the numbers %%h:de1a325aac78%%
- [ ] [[A Deployment's Recreate strategy keeps old and new Pods apart only during upgrades]] · 0.886 · ready · check the numbers %%h:825e619352d3%%
- [ ] [[Kubernetes reports a stalled Deployment rollout but never rolls it back by itself]] · 0.884 · ready · check the numbers %%h:a009e7fafca0%%
- [ ] [[A Helm upgrade that changes only a ConfigMap does not restart the Pods that read it]] · 0.883 · ready · check the numbers %%h:923e89414e9d%%
- [ ] [[A PodDisruptionBudget does not limit a Deployment or StatefulSet rolling update]] · 0.88 · ready · check the numbers %%h:2e6b63dc8895%%
- [ ] [[RDS point-in-time recovery reaches back only as far as the backup retention period you set]] · 0.877 · ready · check the numbers %%h:337ae1c669e5%%
- [ ] [[Helm installs the CRDs in a chart's crds directory but never upgrades or deletes them]] · 0.876 · ready · check the numbers %%h:722579e2d5e0%%
- [ ] [[A Helm 4 upgrade without --wait can be recorded as deployed before its Pods are ready]] · 0.867 · ready · check the numbers %%h:225631fd8953%%
- [ ] [[A terminating Pod can still receive requests after TERM because its endpoint removal runs concurrently]] · 0.852 · ready · check the numbers %%h:fc2dd5122258%%
- [ ] [[A Deployment treats its pods as interchangeable replicas of a stateless workload]] · 0.847 · ready · check the numbers %%h:8dc4616816f2%%
- [ ] [[Rolling back a Kafka producer leaves its new-format events in the topic for the old consumers]] · 0.845 · ready · check the numbers %%h:01b0e2855128%%
- [ ] [[Expand and contract schema changes keep the previous version runnable after a rollback]] · 0.834 · ready %%h:fec6ca296e94%%
- [ ] [[A failed liveness probe restarts the container while a failed readiness probe only stops its traffic]] · 0.884 · borderline · check the numbers %%h:b504626f76d6%%
- [ ] [[On Amazon EKS AWS runs the control plane, while who patches the nodes depends on the node type]] · 0.867 · borderline %%h:40944ae2e7fb%%
- [ ] [[A feature flag rolls back a feature without a deploy but cannot undo what the new code path wrote]] · 0.816 · borderline %%h:a3391b2d07f8%%
- [ ] [[A Flyway undo migration can reverse a schema change but not lost data, so production databases roll forward]] · 0.743 · borderline · check the numbers %%h:f5dcc28bd2a7%%
- [ ] [[Blue-green switches all traffic at once while a canary shifts a subset of users first]] · 0.787 · parked %%h:d5dfe69492d0%%

## python → 10-notes/python/ (4)
- [ ] [[asyncio suits I-O-bound work, not CPU-bound work]] · unjudged %%h:984e8f30554f%%
- [ ] [[multiprocessing sidesteps the GIL at the cost of process overhead]] · unjudged %%h:a8f1fa622ef1%%
- [ ] [[Mutable default arguments are evaluated once at definition time]] · unjudged %%h:06ab2883bf56%%
- [ ] [[Python memory is managed by reference counting plus a cycle collector]] · unjudged %%h:7002eaf262dc%%

## security → 10-notes/security/ (24)
- [ ] [[Docker build arguments and ENV values persist in the image, so a build needs secret mounts for credentials]] · 0.917 · ready %%h:1e4562d51db2%%
- [ ] [[Turning on Kubernetes encryption at rest leaves existing Secrets unencrypted until they are rewritten]] · 0.916 · ready · check the numbers %%h:c8f1fd8f6d09%%
- [ ] [[TLS 1.3 early data can be replayed, so HTTP sends only safe methods in 0-RTT]] · 0.913 · ready · check the numbers %%h:e2eb257f51dd%%
- [ ] [[With HS256 every service that can verify a JWT can also forge one]] · 0.913 · ready · check the numbers %%h:2e05a80d103c%%
- [ ] [[A secret committed to Git must be revoked or rotated first, because removing it from history does not undo the leak]] · 0.907 · ready %%h:476ef94ea2ff%%
- [ ] [[A self-contained JWT stays valid until it expires unless verifiers check revocation state]] · 0.907 · ready · check the numbers %%h:7217c33378b7%%
- [ ] [[A Kubernetes Secret consumed as an environment variable keeps its old value until the Pod restarts, while a mounted volume updates in place]] · 0.906 · ready · check the numbers %%h:2c4e8531514b%%
- [ ] [[HashiCorp Vault dynamic secrets are generated per client with a lease and can be revoked]] · 0.906 · ready · check the numbers %%h:4bda3d1afffa%%
- [ ] [[A signed JWT is readable by anyone who holds it]] · 0.902 · ready · check the numbers %%h:4c4faa55687e%%
- [ ] [[Refresh token rotation turns a stolen refresh token into a detectable reuse]] · 0.899 · ready · check the numbers %%h:74846ef4c86b%%
- [ ] [[Edge TLS termination leaves the hop to the Pods in plaintext unless the proxy re-encrypts or passes TLS through]] · 0.896 · ready · check the numbers %%h:30f502a20fbe%%
- [ ] [[A JWT signing key is rotated by publishing the new public key in the JWK Set before signing with it]] · 0.882 · ready · check the numbers %%h:ba19b72a4d85%%
- [ ] [[TLS 1.3 key exchange is ephemeral, so a stolen server private key cannot decrypt recorded sessions]] · 0.878 · ready · check the numbers %%h:55749ffb4bb9%%
- [ ] [[Behind a TLS-terminating proxy, forwarded headers are client input unless the proxy wrote them]] · 0.875 · ready · check the numbers %%h:e807564aa30b%%
- [ ] [[A Pod can log in to Vault with its service-account token, so it needs no static secret to reach Vault]] · 0.869 · ready %%h:448c46b8ebb0%%
- [ ] [[Without a typ check, an ID token from the same issuer passes as a JWT access token]] · 0.868 · ready · check the numbers %%h:61903270c72c%%
- [ ] [[A JWT validator must pin the accepted algorithms instead of trusting the alg header]] · 0.86 · ready · check the numbers %%h:92a06d566781%%
- [ ] [[A JWT verifier must take the key from the trusted issuer's JWK Set, never from the token's jku, x5u or jwk header]] · 0.847 · ready · check the numbers %%h:1ef8d9e926da%%
- [ ] [[A server certificate proves identity only together with a SAN host-name match and a CertificateVerify signature]] · 0.826 · ready · check the numbers %%h:5533e5febfcb%%
- [ ] [[A JWT with a valid signature must still be rejected when exp, iss or aud fail validation]] · 0.892 · borderline · check the numbers %%h:a9969224f6fd%%
- [ ] [[A backend for frontend keeps OAuth tokens out of reach of injected JavaScript]] · 0.89 · borderline · check the numbers %%h:0cf471feafd0%%
- [ ] [[HTTPS hides the request path and headers but not the IP addresses or the SNI hostname]] · 0.87 · borderline · check the numbers %%h:dc14d9cecda5%%
- [ ] [[A Kubernetes Secret is only base64-encoded and is stored unencrypted in etcd by default]] · 0.856 · borderline · check the numbers %%h:8cf00e129378%%
- [ ] [[A TLS 1.3 full handshake authenticates the server and agrees keys in one round trip]] · 0.838 · borderline · check the numbers %%h:da1f47d3c029%%

## spring → 10-notes/spring/ (35)
- [ ] [[Spring does not call @PreDestroy on prototype-scoped beans]] · 0.92 · ready %%h:161aada9fac9%%
- [ ] [[A checked exception commits a Spring @Transactional method by default]] · 0.919 · ready %%h:4dac614cebc2%%
- [ ] [[An injected Map of beans keeps registration order and ignores @Order]] · 0.917 · ready · check the numbers %%h:eb7dd3dcdce5%%
- [ ] [[Without an isolation attribute, a Spring transaction runs at the database's default isolation level]] · 0.9 · ready · check the numbers %%h:868cacde4b47%%
- [ ] [[An application BeanPostProcessor sees a bean before its @PostConstruct method has run]] · 0.899 · ready · check the numbers %%h:b5bbc0240e69%%
- [ ] [[@Autowired(required = false) leaves a collection field null, not empty, when no bean matches]] · 0.897 · ready · check the numbers %%h:481e1bbabd80%%
- [ ] [[new RestTemplate() and RestClient.create() send requests through different HTTP engines]] · 0.897 · ready · check the numbers · read with [[RestClient replaces RestTemplate as Spring's synchronous HTTP client]] %%h:05a087931237%%
- [ ] [[REQUIRES_NEW keeps the outer connection while it borrows a second one from the pool]] · 0.894 · ready %%h:e8e8316ee458%%
- [ ] [[Only ObjectProvider.orderedStream() sorts beans by @Order, while stream() does not]] · 0.892 · ready · check the numbers %%h:85d3739fabc0%%
- [ ] [[Calling block() on a Reactor Netty event-loop thread throws IllegalStateException]] · 0.891 · ready · check the numbers %%h:7efeb3b60a6c%%
- [ ] [[A joined REQUIRED transaction ignores its own isolation, timeout and readOnly attributes]] · 0.889 · ready %%h:0d66003ddf7d%%
- [ ] [[SmartLifecycle beans start after every singleton is initialised and stop before the destroy callbacks]] · 0.889 · ready %%h:ca7211b0bef4%%
- [ ] [[Spring Boot proxies a bean with CGLIB even when the bean implements an interface]] · 0.889 · ready %%h:92907131afa5%%
- [ ] [[A Spring ResourceAccessException means no response arrived, so the request's outcome is unknown]] · 0.883 · ready · check the numbers %%h:ed0111ab7d62%%
- [ ] [[Catching an exception from a joined REQUIRED method ends in UnexpectedRollbackException]] · 0.882 · ready %%h:765bce94067c%%
- [ ] [[Spring calls a public close() or shutdown() on a @Bean object at shutdown unless destroyMethod is empty]] · 0.882 · ready %%h:e6ea736757da%%
- [ ] [[Work handed to another thread runs outside the caller's Spring transaction]] · 0.882 · ready · check the numbers %%h:53aa246f88a1%%
- [ ] [[An HTTP client's read timeout is measured between reads, so it does not bound the whole call]] · 0.878 · ready %%h:ceb08fe4ed56%%
- [ ] [[Injecting a List of an interface gives every bean of that type, sorted by @Order]] · 0.877 · ready %%h:6c4838cd19e1%%
- [ ] [[Injecting a String-keyed Map gives a strategy registry keyed by bean name]] · 0.877 · ready %%h:c0add26b6631%%
- [ ] [[A @Transactional timeout fails the next database access after the deadline instead of interrupting the method]] · 0.875 · ready · check the numbers %%h:4a49762f4b27%%
- [ ] [[An advised final class fails Spring Boot startup because CGLIB cannot subclass it]] · 0.873 · ready · check the numbers %%h:ee1fa4b8091b%%
- [ ] [[@Transactional does not apply inside @PostConstruct because the proxy is created after initialisation]] · 0.871 · ready %%h:430b66e709c8%%
- [ ] [[Spring destroys a singleton before the beans it depends on]] · 0.871 · ready · check the numbers %%h:29fa103303e8%%
- [ ] [[Self-invocation bypasses the Spring @Transactional proxy]] · 0.868 · ready %%h:21aa8c75abbe%%
- [ ] [[readOnly = true on a JPA transaction silently drops changes to managed entities]] · 0.85 · ready · check the numbers %%h:09898a412887%%
- [ ] [[Proposed addition to Injecting a String-keyed Map gives a strategy registry keyed by bean name]] · 0.808 · ready %%h:4a404690df49%%
- [ ] [[RestClient and WebClient take their timeouts from the underlying HTTP library]] · 0.808 · ready · check the numbers %%h:5ce5e7461fd9%%
- [ ] [[A final method on a CGLIB-proxied Spring bean runs on the proxy instance, where injected fields are null]] · 0.798 · ready %%h:08f2b7af9bd7%%
- [ ] [[Spring AOP applies @Aspect annotations through proxies, without the AspectJ weaver]] · 0.794 · ready · check the numbers %%h:42d38b8db848%%
- [ ] [[A BeanFactoryPostProcessor edits bean definitions before the container instantiates any other bean]] · 0.881 · borderline · check the numbers %%h:b23314fe906e%%
- [ ] [[WebClient saves threads only when the caller does not block on it]] · 0.877 · borderline · check the numbers %%h:eb83ff35cea4%%
- [ ] [[RestClient replaces RestTemplate as Spring's synchronous HTTP client]] · 0.861 · borderline · check the numbers %%h:522262c4916e%%
- [ ] [[Since Spring 6.0, class-based proxies make protected and package-private @Transactional methods transactional]] · 0.843 · borderline · check the numbers %%h:31e497073877%%
- [ ] [[@TransactionalEventListener moves a side effect after the commit but loses it if the process dies]] · 0.841 · parked %%h:7b4bd6a3dece%%

## system-design → 10-notes/system-design/ (19)
- [ ] [[A queue in front of a service levels load spikes at the cost of an immediate response]] · 0.923 · ready %%h:dfa986fe2611%%
- [ ] [[A shared-cache outage sends the whole read load to the database, so the fallback needs its own limit]] · 0.909 · ready %%h:4d5c0e16b4c3%%
- [ ] [[Routing rows by hash(key) mod N moves most keys when a shard is added]] · 0.906 · ready %%h:99f3a5210a7c%%
- [ ] [[In Redis a plain SET on an existing key clears its TTL]] · 0.903 · ready · check the numbers %%h:e78f78ebc278%%
- [ ] [[Write-behind caching makes the cache the system of record until its queue is flushed]] · 0.899 · ready · check the numbers %%h:5104779f6977%%
- [ ] [[Read replicas scale reads but serve stale data while replication lags]] · 0.896 · ready · check the numbers %%h:a2e78d6def4a%%
- [ ] [[Cache-aside loads data on a miss and leaves cache consistency to the application]] · 0.892 · ready %%h:acd804f2cc9c%%
- [ ] [[Scale queue workers on the age of the oldest message, not on queue depth]] · 0.89 · ready · check the numbers %%h:62e059c56c45%%
- [ ] [[A write-heavy database scales by sharding, because replicas add only read capacity]] · 0.885 · ready %%h:74554b50f7cd%%
- [ ] [[Adding nodes to a Redis Cluster cannot relieve a single hot key]] · 0.885 · ready · check the numbers %%h:3720eb750bc5%%
- [ ] [[A correctly ordered cache-aside delete still lets a slow reader write a stale value back]] · 0.875 · ready %%h:3153cb5fb962%%
- [ ] [[Autoscaling flaps unless the scale-in threshold sits well below the scale-out threshold]] · 0.874 · ready · check the numbers %%h:e50835b86e4a%%
- [ ] [[Sharding makes cross-shard operations expensive, so the shard key must keep most work on one shard]] · 0.861 · ready %%h:937ef2e6bbd9%%
- [ ] [[A small database connection pool often beats a large one, because extra connections only time-slice the same cores]] · 0.853 · ready · check the numbers %%h:d0ef50d67d77%%
- [ ] [[Batching writes raises throughput by paying the per-request overhead once per batch]] · 0.844 · ready · check the numbers %%h:24db1cef7d06%%
- [ ] [[CQRS with separate read and write stores costs eventual consistency, an outbox and an idempotent projection]] · 0.892 · borderline %%h:edef4bbf500e%%
- [ ] [[Stateless services scale out, while scaling up one machine stops at a hardware limit]] · 0.869 · borderline %%h:d36223958ae1%%
- [ ] [[Redis evicts keys only at maxmemory, and maxmemory 0, the 64-bit default, sets no limit at all]] · 0.862 · borderline · check the numbers %%h:0565dc6729c7%%
- [ ] [[A cache stampede happens when a hot key expires and many requests regenerate it at once]] · 0.777 · borderline · check the numbers %%h:d395e082132b%%

## Maps → 20-moc/ (18)
- [ ] [[@Transactional MOC]] · map %%h:2743dfa8c1da%%
- [ ] [[Caching MOC]] · map %%h:ce885a2d6887%%
- [ ] [[Concurrency MOC]] · map %%h:8f372a4f9e83%%
- [ ] [[Database MOC]] · map %%h:6b4b9360973e%%
- [ ] [[Frontend interview MOC]] · map %%h:f7908070c04c%%
- [ ] [[Indexes and query performance MOC]] · map %%h:5884db9ecef0%%
- [ ] [[Java backend interview MOC]] · map %%h:be1820896bcc%%
- [ ] [[Java collections MOC]] · map %%h:e170c7e2867c%%
- [ ] [[Java core MOC]] · map %%h:c9781f34d036%%
- [ ] [[JWT MOC]] · map %%h:c0cece073af8%%
- [ ] [[Kafka MOC]] · map %%h:e214571d74cc%%
- [ ] [[Kubernetes MOC]] · map %%h:bc0b8cf770bc%%
- [ ] [[Microservices and messaging MOC]] · map %%h:d8d82a1c13f6%%
- [ ] [[Ops and cloud MOC]] · map %%h:7b2384928270%%
- [ ] [[Security MOC]] · map %%h:d5fcd64e53db%%
- [ ] [[Spring MOC]] · map %%h:452582d26d24%%
- [ ] [[System design MOC]] · map %%h:490b209b6431%%
- [ ] [[TLS MOC]] · map %%h:761a1bba95b0%%

## In several areas (database, system-design): set up: to one before it can be filed (2)
- [ ] [[Read-your-writes on an asynchronous PostgreSQL replica needs recency routing, WAL-position routing or remote_apply]] · 0.888 · ready · check the numbers %%h:07f250eaac7e%%
- [ ] [[Scaling out app instances multiplies database connections, so instances times pool size must stay under max_connections]] · 0.881 · ready · check the numbers %%h:c7979bc1d218%%

## In several areas (java-core, jvm): set up: to one before it can be filed (3)
- [ ] [[A lambda is linked through invokedynamic at run time, not compiled to its own class file]] · 0.899 · ready %%h:a9e92b7df79b%%
- [ ] [[ZGC trades some throughput for sub-millisecond pauses, so G1 stays the default collector]] · 0.876 · borderline · check the numbers %%h:300b63f9c220%%
- [ ] [[A Java memory leak is an object that stays reachable after the program stops needing it]] · 0.867 · borderline %%h:2eedb36fe2b7%%

## In several areas (jvm, python): set up: to one before it can be filed (1)
- [ ] [[The GIL lets only one thread run Python bytecode at a time]] · unjudged %%h:5f9b7d10195d%%

## Under no MOC: add up: before it can be filed (2)
- [ ] [[A swarm of coding agents drifts from intent without any agent going rogue]] · 0.739 · ready %%h:a1efdf3efff7%%
- [ ] [[Trace every agent decision back to the original intent to stop fleet drift]] · 0.644 · parked %%h:65df25f4234e%%
