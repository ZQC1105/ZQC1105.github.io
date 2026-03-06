# SQL Server 中 LIKE 的用法总结

## 一、基础语法

```sql
SELECT 列名 FROM 表名 WHERE 列名 LIKE '模式' [ESCAPE '转义字符']
```

## 二、通配符详解

### 1. **%**（百分号）

- 匹配任意长度的字符串（包括0个字符）

```sql
-- 以"张"开头的所有名字
SELECT * FROM users WHERE name LIKE '张%'

-- 包含"华"的所有名字
SELECT * FROM users WHERE name LIKE '%华%'

-- 以"有限公司"结尾的公司
SELECT * FROM companies WHERE name LIKE '%有限公司'
```

### 2. **\_ **（下划线）

- 匹配**单个**任意字符

```sql
-- 名字为2个字的"张"姓（张三、张四）
SELECT * FROM users WHERE name LIKE '张_'

-- 名字为3个字且中间是"小"（张小X）
SELECT * FROM users WHERE name LIKE '_小_'
```

### 3. **[ ]**（方括号）

- 匹配指定范围或集合内的**单个**字符

**范围匹配**：

```sql
-- 以A、B、C开头的产品
SELECT * FROM products WHERE name LIKE '[A-C]%'

-- 以数字开头的用户名
SELECT * FROM users WHERE username LIKE '[0-9]%'
```

**集合匹配**：

```sql
-- 以张、王、李开头的姓名
SELECT * FROM users WHERE name LIKE '[张王李]%'

-- 名字第二个字是明或红
SELECT * FROM users WHERE name LIKE '_[明红]%'
```

### 4. **[^]**（脱字符）

- 匹配**不在**指定范围或集合内的单个字符

```sql
-- 不以A、B、C开头的产品
SELECT * FROM products WHERE name LIKE '[^A-C]%'

-- 不是数字开头的用户名
SELECT * FROM users WHERE username LIKE '[^0-9]%'

-- 不是张、王、李开头的姓名
SELECT * FROM users WHERE name LIKE '[^张王李]%'
```

## 三、转义字符 ESCAPE

当需要搜索通配符本身（`%`、`_`、`[`、`]`）时使用：

```sql
-- 搜索包含"50%"的字符串
SELECT * FROM table WHERE column LIKE '%50\%%' ESCAPE '\'

-- 搜索包含"100_"的字符串
SELECT * FROM table WHERE column LIKE '%100\_%' ESCAPE '\'

-- 使用自定义转义字符
SELECT * FROM table WHERE column LIKE '%50#%%' ESCAPE '#'
```

## 四、性能优化建议

### 1. **避免前导通配符**

```sql
-- ❌ 性能差（无法使用索引）
WHERE name LIKE '%张%'

-- ✅ 性能好（可以使用索引）
WHERE name LIKE '张%'
```

### 2. **使用全文索引替代复杂LIKE**

对于大量文本搜索，考虑使用全文索引：

```sql
-- 全文搜索
WHERE CONTAINS(content, '搜索词')

-- 替代
WHERE content LIKE '%搜索词%'
```

### 3. **结合其他条件**

```sql
-- 先过滤其他条件，减少LIKE扫描范围
WHERE status = 'active'
  AND name LIKE '张%'
```

## 五、实际应用示例

### 1. **模糊匹配用户输入**

```sql
DECLARE @search NVARCHAR(100) = '张'
SELECT * FROM users
WHERE name LIKE @search + '%'  -- 搜索以"张"开头的名字
   OR name LIKE '%' + @search + '%'  -- 搜索包含"张"的名字
```

### 2. **日期模式匹配**

```sql
-- 查找2023年的记录
SELECT * FROM orders
WHERE order_date LIKE '2023%'  -- 假设日期存储为字符串
```

### 3. **复杂模式匹配**

```sql
-- 查找符合特定模式的电话号码（如 010-xxxxxxx）
WHERE phone LIKE '010-_______'  -- 7个下划线

-- 查找特定格式的邮政编码
WHERE zip_code LIKE '[1-9][0-9][0-9][0-9][0-9][0-9]'
```

## 六、注意事项

1. **大小写敏感性**：取决于数据库的排序规则（Collation）
   - 区分大小写：`LIKE 'A%'` 不匹配 `'a%'`
   - 不区分大小写：可以使用 `UPPER(column) LIKE 'A%'`

2. **NULL值处理**：`LIKE` 不会匹配 `NULL`

   ```sql
   -- 需要同时处理NULL
   WHERE name LIKE '张%' OR name IS NULL
   ```

3. **Unicode支持**：对于NVARCHAR类型，使用N前缀

   ```sql
   WHERE name LIKE N'%中%'  -- 正确处理Unicode字符
   ```

4. **性能监控**：对于频繁使用的LIKE查询，使用执行计划分析索引使用情况

## 七、与其他数据库的差异

| 特性         | SQL Server   | MySQL      | PostgreSQL | Oracle     |
| ------------ | ------------ | ---------- | ---------- | ---------- |
| 大小写敏感   | 依赖排序规则 | 默认不敏感 | 默认敏感   | 默认不敏感 |
| [ ] 范围匹配 | ✅           | ❌         | ❌         | ❌         |
| ILIKE        | ❌           | ❌         | ✅         | ❌         |
| ESCAPE       | ✅           | ✅         | ✅         | ✅         |

这些是SQL Server中LIKE的主要用法，根据实际需求选择合适的模式匹配方式！
