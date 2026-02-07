一句话记住  
LEFT → “左边全要，右边可选”  
INNER → “两边都要，少一条就全丢”  
RIGHT → “右边全要，左边可选” （其实就是把表顺序换过来的 LEFT JOIN）

下面用同一份极简数据把三种JOIN画给你看。

```
Table A (主)          Table B (从)
id  name             id  a_id  score
--  ----             --  ----  -----
1   Alice            1   1     90
2   Bob              2   1     85
3   Carol            3   3     77
```

---

### 1. LEFT OUTER JOIN（LEFT JOIN）
```sql
SELECT A.name, B.score
FROM A
LEFT JOIN B ON A.id = B.a_id;
```
结果  
| name  | score |
| ----- | ----- |
| Alice | 90    |
| Alice | 85    | ← Alice 有两条子记录，所以出现两次     |
| Bob   | NULL  | ← Bob 没有子记录，仍保留，用 NULL 补空 |
| Carol | 77    |

**特点**  
- 主表(A)的行**一条不少**；子表(B)没匹配时填 NULL。  
- EF Core 的 `.Include()` 默认就是这种，保证主实体不会被“过滤掉”。

---

### 2. INNER JOIN
```sql
SELECT A.name, B.score
FROM A
JOIN B ON A.id = B.a_id;
```
结果  
| name  | score |
| ----- | ----- |
| Alice | 90    |
| Alice | 85    |
| Carol | 77    |

**特点**  
- 只保留**两边都能匹配上**的行；Bob 因为没成绩，直接被丢弃。  
- 在 EF 里，当你用 `Where(b => b.Posts.Any())` 这类过滤时，SQL 会退成 INNER JOIN 或 EXISTS，确保“只拿有子数据的主实体”。

---

### 3. RIGHT OUTER JOIN（RIGHT JOIN）
```sql
SELECT A.name, B.score
FROM A
RIGHT JOIN B ON A.id = B.a_id;
```
结果  
| name  | score |
| ----- | ----- |
| Alice | 90    |
| Alice | 85    |
| Carol | 77    |

看起来和 INNER 一样？因为示例数据里 B 的每一行都能在 A 找到对应 id。  
如果往 B 再插一条“孤儿”记录：`4  99  60`（a_id=99 在 A 表不存在），结果会多一条：

| name | score |
| ---- | ----- |
| NULL | 60    | ← 右边(B)必须全保留，左边没匹配就填 NULL |

**特点**  
- 右边(B)的行**一条不少**；左边没匹配时填 NULL。  
- 实际开发很少写 RIGHT JOIN，把表顺序调换一下再用 LEFT JOIN 即可，逻辑更直观。

---

### 一张图总结
```
A 左 ←→ 右 B
LEFT:     A 全保留，B 可选               (EF .Include 默认)
INNER:    只保留 A 与 B 能配上的行       (EF 过滤后常见)
RIGHT:    B 全保留，A 可选               （极少手写，可换表顺序变 LEFT）
```

记住“以谁为主”就行：  
**LEFT 以左表为主，RIGHT 以右表为主，INNER 以交集为主**。