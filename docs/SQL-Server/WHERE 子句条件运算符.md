# SQL Server WHERE 子句条件运算符大全

## 一、比较运算符

| 运算符       | 说明                | 示例                                |
| ------------ | ------------------- | ----------------------------------- |
| `=`          | 等于                | `WHERE age = 25`                    |
| `<>` 或 `!=` | 不等于              | `WHERE status <> 'inactive'`        |
| `>`          | 大于                | `WHERE price > 100`                 |
| `<`          | 小于                | `WHERE quantity < 50`               |
| `>=`         | 大于等于            | `WHERE score >= 60`                 |
| `<=`         | 小于等于            | `WHERE create_date <= '2024-01-01'` |
| `!>`         | 不大于（相当于 <=） | `WHERE age !> 30`                   |
| `!<`         | 不小于（相当于 >=） | `WHERE age !< 18`                   |

## 二、逻辑运算符

| 运算符 | 说明           | 示例                                         |
| ------ | -------------- | -------------------------------------------- |
| `AND`  | 所有条件都为真 | `WHERE age > 18 AND status = 'active'`       |
| `OR`   | 任一条件为真   | `WHERE dept = 'Sales' OR dept = 'Marketing'` |
| `NOT`  | 取反           | `WHERE NOT status = 'deleted'`               |

## 三、范围运算符

| 运算符                    | 说明                 | 示例                              |
| ------------------------- | -------------------- | --------------------------------- |
| `BETWEEN ... AND ...`     | 在范围内（包含边界） | `WHERE price BETWEEN 100 AND 200` |
| `NOT BETWEEN ... AND ...` | 不在范围内           | `WHERE age NOT BETWEEN 18 AND 60` |

## 四、集合运算符

| 运算符       | 说明                   | 示例                                                               |
| ------------ | ---------------------- | ------------------------------------------------------------------ |
| `IN`         | 在集合中               | `WHERE city IN ('北京', '上海', '广州')`                           |
| `NOT IN`     | 不在集合中             | `WHERE city NOT IN ('北京', '上海')`                               |
| `EXISTS`     | 子查询有结果           | `WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = c.id)`     |
| `NOT EXISTS` | 子查询无结果           | `WHERE NOT EXISTS (SELECT 1 FROM orders WHERE customer_id = c.id)` |
| `ANY`        | 与子查询任意值比较为真 | `WHERE price > ANY (SELECT price FROM products)`                   |
| `ALL`        | 与子查询所有值比较为真 | `WHERE price > ALL (SELECT price FROM products)`                   |
| `SOME`       | 同 ANY                 | `WHERE price = SOME (SELECT price FROM products)`                  |

## 五、NULL 判断运算符

| 运算符        | 说明        | 示例                      |
| ------------- | ----------- | ------------------------- |
| `IS NULL`     | 值为 NULL   | `WHERE email IS NULL`     |
| `IS NOT NULL` | 值不为 NULL | `WHERE phone IS NOT NULL` |

## 六、字符串模式匹配

| 运算符     | 说明       | 示例                           |
| ---------- | ---------- | ------------------------------ |
| `LIKE`     | 模式匹配   | `WHERE name LIKE '张%'`        |
| `NOT LIKE` | 不匹配模式 | `WHERE name NOT LIKE '%测试%'` |

### LIKE 通配符：

| 通配符 | 说明           | 示例                           |
| ------ | -------------- | ------------------------------ |
| `%`    | 任意长度字符串 | `LIKE '张%'`（以张开头）       |
| `_`    | 单个字符       | `LIKE '张_'`（张+1个字符）     |
| `[ ]`  | 范围内单个字符 | `LIKE '[A-C]%'`（A-C开头）     |
| `[^]`  | 范围外单个字符 | `LIKE '[^0-9]%'`（非数字开头） |

## 七、全文搜索运算符

| 运算符     | 说明             | 示例                                     |
| ---------- | ---------------- | ---------------------------------------- |
| `CONTAINS` | 全文索引精确搜索 | `WHERE CONTAINS(content, '数据库')`      |
| `FREETEXT` | 全文索引模糊搜索 | `WHERE FREETEXT(description, 'SQL教程')` |

## 八、取反运算符组合

| 组合          | 说明         | 示例                                      |
| ------------- | ------------ | ----------------------------------------- |
| `NOT LIKE`    | 不匹配模式   | `WHERE name NOT LIKE 'test%'`             |
| `NOT BETWEEN` | 不在范围     | `WHERE age NOT BETWEEN 18 AND 60`         |
| `NOT IN`      | 不在集合     | `WHERE city NOT IN ('北京', '上海')`      |
| `NOT EXISTS`  | 子查询无结果 | `WHERE NOT EXISTS (SELECT 1 FROM orders)` |
| `IS NOT NULL` | 不为空       | `WHERE email IS NOT NULL`                 |

## 九、运算符优先级（从高到低）

1. 括号 `( )`
2. 取反 `NOT`
3. 比较运算符：`=`, `>`, `<`, `>=`, `<=`, `<>`, `!=`, `!>`, `!<`
4. 特殊运算符：`BETWEEN`, `IN`, `LIKE`, `EXISTS`
5. 逻辑与 `AND`
6. 逻辑或 `OR`

## 十、快速参考示例

```sql
-- 综合示例
SELECT * FROM orders
WHERE
    -- 比较运算符
    amount >= 1000

    -- 逻辑运算符
    AND (status = 'completed' OR status = 'shipped')

    -- 范围运算符
    AND order_date BETWEEN '2024-01-01' AND '2024-12-31'

    -- 集合运算符
    AND customer_id IN (SELECT id FROM vip_customers)

    -- NULL判断
    AND delivery_address IS NOT NULL

    -- 模式匹配
    AND customer_name LIKE '张%'

    -- 存在性检查
    AND EXISTS (
        SELECT 1 FROM payments
        WHERE order_id = orders.id
    )
```

这个列表包含了SQL Server中WHERE子句可用的所有条件运算符，按功能分类便于查阅！
