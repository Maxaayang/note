---
annotation-target: note/23-distributedoltp_note.pdf
annotation-target-type: pdf
---

# OLTP VS OLAP

事务处理是 OLTP，分析处理是 OLAP
![[Pasted image 20220528142733.png]]

OLTP：试着更新或者读取数据库中少量数据的操作



>%%
>```annotation-json
>{"created":"2022-05-28T06:39:59.805Z","updated":"2022-05-28T06:39:59.805Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 23 Distributed OLTP Databases","link":[{"href":"urn:x-pdf:61ae494400d7ccb024ab962292f2d846"},{"href":"vault:/database/15445/note/23-distributedoltp_note.pdf"}],"documentFingerprint":"61ae494400d7ccb024ab962292f2d846"},"uri":"vault:/database/15445/note/23-distributedoltp_note.pdf","target":[{"source":"vault:/database/15445/note/23-distributedoltp_note.pdf","selector":[{"type":"TextPositionSelector","start":486,"end":822},{"type":"TextQuoteSelector","exact":"Executing these transactions is morechallenging than single-node transactions because now when the transaction commits, the DBMS has tomake sure that all the nodes agree to commit the transaction. The DBMS ensure that the database providesthe same ACID guarantees as a single-node DBMS even in the case of node failures or message loss.","prefix":"ccesses data on multiple nodes. ","suffix":"One can assume that all nodes in"}]}]}
>```
>%%
>*%%HIGHLIGHT%%Executing these transactions is more challenging than single-node transactions because now when the transaction commits, the DBMS has to make sure that all the nodes agree to commit the transaction. The DBMS ensure that the database provides the same ACID guarantees as a single-node DBMS even in the case of node failures or message loss.*
>%%LINK%%[[#^3zag8nb3ma|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^3zag8nb3ma

# Atomic Commit Protocols


>%%
>```annotation-json
>{"created":"2022-05-28T06:43:34.676Z","updated":"2022-05-28T06:43:34.676Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 23 Distributed OLTP Databases","link":[{"href":"urn:x-pdf:61ae494400d7ccb024ab962292f2d846"},{"href":"vault:/database/15445/note/23-distributedoltp_note.pdf"}],"documentFingerprint":"61ae494400d7ccb024ab962292f2d846"},"uri":"vault:/database/15445/note/23-distributedoltp_note.pdf","target":[{"source":"vault:/database/15445/note/23-distributedoltp_note.pdf","selector":[{"type":"TextPositionSelector","start":1244,"end":1360},{"type":"TextQuoteSelector","exact":"When a multi-node transaction finishes, the DBMS needs to ask all of the nodes involved whether it is safeto commit.","prefix":"ctions.3 Atomic Commit Protocols","suffix":" Depending on the protocol, a ma"}]}]}
>```
>%%
>*%%HIGHLIGHT%%When a multi-node transaction finishes, the DBMS needs to ask all of the nodes involved whether it is safeto commit.*
>%%LINK%%[[#^03a0vepfom7x|show annotation]]
>%%COMMENT%%
>如果超过一定数量的节点都同意去提交这个事务，然后，我们就会告诉所有节点，我们会提交这个事务。
>%%TAGS%%
>
^03a0vepfom7x



>%%
>```annotation-json
>{"created":"2022-05-28T06:49:09.068Z","updated":"2022-05-28T06:49:09.068Z","document":{"title":"15-445/645 Database Systems (Fall 2021) - Lecture Notes - 23 Distributed OLTP Databases","link":[{"href":"urn:x-pdf:61ae494400d7ccb024ab962292f2d846"},{"href":"vault:/database/15445/note/23-distributedoltp_note.pdf"}],"documentFingerprint":"61ae494400d7ccb024ab962292f2d846"},"uri":"vault:/database/15445/note/23-distributedoltp_note.pdf","target":[{"source":"vault:/database/15445/note/23-distributedoltp_note.pdf","selector":[{"type":"TextPositionSelector","start":1645,"end":1861},{"type":"TextQuoteSelector","exact":"Two-Phase Commit (2PC) blocks if coordinator fails after the prepare message is sent, until the coordinatorrecovers. Paxos, on the other hand, is non-blocking if a majority participants are alive, provided there is a","prefix":"first probably correct protocol)","suffix":"Fall 2021 – Lecture #23 Distribu"}]}]}
>```
>%%
>*%%HIGHLIGHT%%Two-Phase Commit (2PC) blocks if coordinator fails after the prepare message is sent, until the coordinator recovers. Paxos, on the other hand, is non-blocking if a majority participants are alive, provided there is a sufficiently long period without further failures. 2PC is used often if the nodes are in the same data center because of the number of round trips could be less than for Paxos, assuming that nodes do not fail often and are not malicious.*
>%%LINK%%[[#^flo8wiodx4i|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^flo8wiodx4i

## Two-Phase Commit
对于两阶段提交协议中所有参与该事务的节点来说，它们全员必须表示：我们需要提交这个事务，要么全员同意，要么一个都不同意。

## Paxos
只需要参与该事务的大多数节点同意提交该事务即可，在 Two-Phase Commit则需要全部都同意。
z
对于在本地彼此相邻的分布式DBMS来说，最常使用的是两阶段提交，因为这种来回发送信息的次数可能会更少。