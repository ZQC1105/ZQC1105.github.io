把“冗余”拆开来看，其实就是 **同一份主表数据被重复传输了多少次**。

下面用具体数字说话。

---

### 实验数据
```sql
Blogs
Id  Name
--  ----
1   'EF Core'

Posts
Id  BlogId  Title
--  ------  -----
1   1       'Post A'
2   1       'Post B'
3   1       'Post C'
```

- 主表 1 行  
- 子表 3 行  
- 真正有用的“信息量” = 1 条 Blog + 3 条 Posts = 4 行数据

---

### 1. 默认（非拆分）—— 第二条 SQL 用 LEFT JOIN
```sql
SELECT [b].[Id], [b].[Name],          -- Blog 字段
       [p].[Id], [p].[Title], [p].[BlogId]
FROM [Blogs] AS [b]
LEFT JOIN [Posts] AS [p] ON [b].[Id] = [p].[BlogId]
WHERE [b].[Id] IN (1);
```

结果集
| b.Id | b.Name  | p.Id | p.Title | p.BlogId |
| ---- | ------- | ---- | ------- | -------- |
| 1    | EF Core | 1    | Post A  | 1        |
| 1    | EF Core | 2    | Post B  | 1        |
| 1    | EF Core | 3    | Post C  | 1        |

- 主表字段 `(1, 'EF Core')` 被重复了 **3 次**——这就是“**数据冗余**”。  
- 网络包大小 ≈ 3 × (Blog 字段) + 3 × (Post 字段)  
- 如果 Blog 还有 10 个字段，Posts 有 100 条，这份冗余会被放大 100 倍。

---

### 2. AsSplitQuery()——两条纯单表查询
第一条  
```sql
SELECT [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
WHERE [b].[Id] IN (1);
```
结果  
| Id  | Name    |
| --- | ------- |
| 1   | EF Core | ← 只出现 1 次 |

第二条  
```sql
SELECT [p].[Id], [p].[Title], [p].[BlogId]
FROM [Posts] AS [p]
WHERE [p].[BlogId] IN (1);
```
结果  
| Id  | Title  | BlogId |
| --- | ------ | ------ |
| 1   | Post A | 1      |
| 2   | Post B | 1      |
| 3   | Post C | 1      |

- 主表数据 **只在第一条出现一次**，第二条完全不再带 Blog 字段。  
- 网络包大小 = 1 × (Blog 字段) + 3 × (Post 字段)，**没有乘性膨胀**。  
- Posts 再多，Blog 字段也永远不会被重复。

---

### 一句话总结
“数据冗余”指的就是 **LEFT JOIN 时主表行因子表多条而被重复拉回**；  
`AsSplitQuery()` 把主表和子表拆成两条独立的单表查询，从根本上 **消灭了这一份重复**，子表数量再大，主表字段也只会传输一次。