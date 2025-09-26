# 事务的保证

1. ES 无事务隔离级别，仅可以保证单文档事务；
2. ES 故障时 GET byId，可能发生脏读；
3. ES update 后 search 不能够读到最新的数据，除非每次 update 后执行 refresh；
4. ES 不能执行丰富的查询语句，不能够执行 join，不能保证单文档以外的事务
5. ES 不能够保证写入的事务顺序（批量写入，写入时独立的）
	
# Schema 变更

6. ES 只支持新增 properties，不能修改和删除，不能像 db 可以使用 alter table；
7. ES 修改和删除 properties，只能通过重建索引 reindex。而重建索引是一个高风险的操作；
	1. 重建期间需要保证源数据的正确
	2. 重建期间需要确保网络传输的稳定和安全
	3. 重建期间是耗费大量时间的

# 无法执行 join

ES 不支持如同关系型数据库的 JOIN 语句。意味着，我们在设计 ES 索引时就需要往大宽表方向设计。

# 可靠性不足

ES 通过 Translog 来保证单文档的事务和崩溃恢复。当多文档的事务时，在崩溃恢复过程中，ES 无法保证事务全部提交或者全部回滚。

# 不稳定性操作

ES 作为分布式数据库，设计的目标是弹性。在大规模的 ES 集群中，ES 的弹性伸缩，引起的数据重新索引和再平衡，都会消耗集群的资源，在滚动升级过程中可能会阻塞流量。

因此相对于传统的关系型数据库，ES 的稳定性还是不足。

# ES 的用武之地

Elasticsearch 作为一个专业的搜索引擎，请用来 Search，而不是 Record。

# 参看 blog

[Elasticsearch Was Never a Database](https://www.paradedb.com/blog/elasticsearch-was-never-a-database)%3Chttps://www.paradedb.com/blog/elasticsearch-was-never-a-database%3E>>