# SQL 中 `WHERE` 和 `HAVING` 的区别详解

## 1. **核心区别总结**

| 特性                 | WHERE                       | HAVING                           |
| -------------------- | --------------------------- | -------------------------------- |
| **执行时机**         | **分组前**，对原始数据过滤  | **分组后**，对分组结果过滤       |
| **作用对象**         | 单个数据行                  | 分组后的聚合结果                 |
| **能否使用聚合函数** | ❌ 不能直接使用             | ✅ 可以直接使用                  |
| **能否使用别名**     | ❌ 不能使用 SELECT 中的别名 | ✅ 可以使用 SELECT 中的别名      |
| **性能影响**         | 先过滤，减少分组数据量      | 后过滤，可能分组大量数据后再过滤 |

## 2. **执行顺序图示**

```sql
SELECT department, COUNT(*) as emp_count, AVG(salary) as avg_salary  -- 5. 选择列
FROM employees                                                       -- 1. 数据来源
WHERE salary > 3000                                                  -- 2. 行过滤（单个员工）
GROUP BY department                                                  -- 3. 分组
HAVING COUNT(*) > 3 AND AVG(salary) > 5000                           -- 4. 分组过滤
ORDER BY avg_salary DESC                                             -- 6. 排序
LIMIT 10;                                                            -- 7. 限制结果
```

**执行流程：**

1. `FROM` - 从 employees 表获取数据
2. `WHERE` - 过滤掉 salary ≤ 3000 的员工
3. `GROUP BY` - 按部门分组剩余员工
4. `HAVING` - 过滤掉员工数 ≤ 3 或平均工资 ≤ 5000 的部门
5. `SELECT` - 选择要显示的列
6. `ORDER BY` - 按平均工资降序排序
7. `LIMIT` - 只返回前10条

## 3. **详细对比示例**

### 示例数据 `employees`：

| id  | name | department | salary | city |
| --- | ---- | ---------- | ------ | ---- |
| 1   | 张三 | 销售部     | 5000   | 北京 |
| 2   | 李四 | 技术部     | 8000   | 上海 |
| 3   | 王五 | 销售部     | 4000   | 北京 |
| 4   | 赵六 | 技术部     | 9000   | 上海 |
| 5   | 孙七 | 销售部     | 3500   | 广州 |
| 6   | 周八 | 销售部     | 4500   | 北京 |
| 7   | 吴九 | 人事部     | 6000   | 北京 |
| 8   | 郑十 | 人事部     | 5500   | 上海 |

### 示例 1：基本使用

```sql
-- WHERE：筛选 salary > 4000 的员工
SELECT department, COUNT(*) as emp_count, AVG(salary) as avg_salary
FROM employees
WHERE salary > 4000          -- 先过滤行
GROUP BY department;

-- HAVING：筛选平均工资 > 5500 的部门
SELECT department, COUNT(*) as emp_count, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 5500;   -- 后过滤分组
```

### 示例 2：WHERE 和 HAVING 同时使用

```sql
-- 查询：统计有3人以上且平均工资超过5000的部门
-- 但只统计工资超过3500的员工
SELECT
    department,
    COUNT(*) as emp_count,
    AVG(salary) as avg_salary
FROM employees
WHERE salary > 3500           -- 1. 先过滤掉工资≤3500的员工
GROUP BY department
HAVING COUNT(*) >= 2 AND AVG(salary) > 5000  -- 2. 再过滤分组结果
ORDER BY avg_salary DESC;
```

**执行过程分析：**

1. `WHERE salary > 3500` 过滤后数据：
   ```
   张三 销售部 5000
   李四 技术部 8000
   王五 销售部 4000  ❌ 被过滤（4000≤3500? 不，4000>3500，保留）
   赵六 技术部 9000
   孙七 销售部 3500  ❌ 被过滤（3500≤3500）
   周八 销售部 4500
   吴九 人事部 6000
   郑十 人事部 5500
   ```
2. `GROUP BY department` 分组后：
   - 销售部：张三(5000)、王五(4000)、周八(4500) → 3人，平均4500
   - 技术部：李四(8000)、赵六(9000) → 2人，平均8500
   - 人事部：吴九(6000)、郑十(5500) → 2人，平均5750
3. `HAVING COUNT(*) >= 2 AND AVG(salary) > 5000`：
   - 销售部：3人≥2 ✅，但4500>5000? ❌ 被过滤
   - 技术部：2人≥2 ✅，8500>5000 ✅ 保留
   - 人事部：2人≥2 ✅，5750>5000 ✅ 保留

**结果：**
| department | emp_count | avg_salary |
|------------|-----------|------------|
| 技术部 | 2 | 8500 |
| 人事部 | 2 | 5750 |

## 4. **关键区别详解**

### 4.1 **聚合函数的使用**

```sql
-- ❌ 错误：WHERE 不能使用聚合函数
SELECT department, AVG(salary)
FROM employees
WHERE AVG(salary) > 5000    -- 错误！
GROUP BY department;

-- ✅ 正确：HAVING 可以使用聚合函数
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 5000;  -- 正确！

-- ✅ 也可以在 WHERE 中使用子查询达到类似效果
SELECT department, AVG(salary) as avg_sal
FROM employees
GROUP BY department
HAVING avg_sal > (
    SELECT AVG(salary) FROM employees  -- 全公司平均工资
);
```

### 4.2 **别名的使用**

```sql
-- ❌ 错误：WHERE 不能使用 SELECT 中定义的别名
SELECT department, COUNT(*) as emp_count
FROM employees
WHERE emp_count > 3          -- 错误！别名在 WHERE 时还未定义
GROUP BY department;

-- ✅ 正确：使用原始表达式
SELECT department, COUNT(*) as emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 3;         -- 正确！

-- ✅ HAVING 可以使用别名（大多数数据库支持）
SELECT department, COUNT(*) as emp_count
FROM employees
GROUP BY department
HAVING emp_count > 3;        -- 正确！HAVING 在 SELECT 之后执行
```

### 4.3 **性能影响**

```sql
-- 情况1：先过滤大量数据（推荐）
SELECT department, AVG(salary)
FROM employees
WHERE salary > 3000          -- 先过滤，减少分组的数据量
GROUP BY department;

-- 情况2：后过滤大量数据（不推荐）
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 3000;   -- 先分组所有数据，再过滤

-- 实际应该结合使用
SELECT department, AVG(salary)
FROM employees
WHERE department IN ('销售部', '技术部')  -- 先过滤部门
GROUP BY department
HAVING AVG(salary) > 5000;               -- 再过滤分组
```

## 5. **实际应用场景**

### 场景 1：统计热门商品

```sql
-- 查询月销量超过1000且平均评分>4.5的商品
SELECT
    product_id,
    COUNT(*) as order_count,
    AVG(quantity) as avg_quantity,
    AVG(rating) as avg_rating
FROM orders
WHERE order_date >= '2024-01-01'    -- 只统计今年订单
  AND status = 'completed'          -- 只统计已完成订单
GROUP BY product_id
HAVING COUNT(*) > 1000 AND AVG(rating) > 4.5
ORDER BY order_count DESC;
```

### 场景 2：学生成绩分析

```sql
-- 查询平均分>80且至少有5名学生的班级
SELECT
    class_id,
    COUNT(DISTINCT student_id) as student_count,
    AVG(score) as avg_score,
    MAX(score) as max_score,
    MIN(score) as min_score
FROM exam_scores
WHERE exam_type = '期末'          -- 只统计期末考试成绩
GROUP BY class_id
HAVING COUNT(DISTINCT student_id) >= 5
   AND AVG(score) > 80
ORDER BY avg_score DESC;
```

### 场景 3：员工绩效分析

```sql
-- 查询绩效优秀（A级）员工超过3人的部门
SELECT
    department,
    COUNT(*) as high_performer_count,
    AVG(salary) as avg_salary,
    STRING_AGG(name, ', ') as high_performers
FROM employees
WHERE performance_rating = 'A'      -- 先筛选A级员工
GROUP BY department
HAVING COUNT(*) >= 3                -- 再筛选有3人以上的部门
ORDER BY high_performer_count DESC;
```

## 6. **常见错误和陷阱**

### 错误 1：混淆过滤条件

```sql
-- ❌ 错误：试图在 WHERE 中过滤分组结果
SELECT department, AVG(salary)
FROM employees
WHERE AVG(salary) > 5000    -- 错误！WHERE 不能过滤分组结果
GROUP BY department;

-- ✅ 正确：使用 HAVING
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 5000;  -- 正确！
```

### 错误 2：误用别名

```sql
-- ❌ MySQL 中 HAVING 可以使用别名，但要注意兼容性
SELECT department, COUNT(*) as cnt
FROM employees
GROUP BY department
HAVING cnt > 5;  -- MySQL 支持，但某些数据库不支持

-- ✅ 更通用的写法
SELECT department, COUNT(*) as cnt
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;  -- 所有数据库都支持
```

### 错误 3：不必要的 HAVING

```sql
-- ❌ 不必要：可以用 WHERE 更高效
SELECT department
FROM employees
GROUP BY department
HAVING department LIKE '销售%';  -- 应该在 WHERE 中过滤

-- ✅ 更高效：在 WHERE 中过滤
SELECT department
FROM employees
WHERE department LIKE '销售%'
GROUP BY department;
```

## 7. **最佳实践建议**

### 7.1 **过滤优先级**

```sql
-- 优化原则：尽可能早地过滤数据
SELECT ...
FROM table
WHERE condition1          -- 1. 先用 WHERE 过滤行级条件
  AND condition2          -- 尽可能多地过滤
GROUP BY ...
HAVING aggregate_condition  -- 2. 再用 HAVING 过滤分组级条件
```

### 7.2 **结合索引使用**

```sql
-- 创建合适的索引
CREATE INDEX idx_dept_salary ON employees(department, salary);

-- 查询可以利用索引
SELECT department, AVG(salary)
FROM employees
WHERE department = '销售部'    -- 可以利用索引
  AND salary > 4000           -- 也可以利用索引
GROUP BY department
HAVING COUNT(*) > 2;
```

### 7.3 **复杂条件的处理**

```sql
-- 复杂的过滤逻辑可以使用 CASE 语句
SELECT
    department,
    COUNT(*) as total,
    SUM(CASE WHEN salary > 8000 THEN 1 ELSE 0 END) as high_salary_count,
    AVG(CASE WHEN city = '北京' THEN salary END) as beijing_avg_salary
FROM employees
WHERE status = 'active'  -- 先过滤活跃员工
GROUP BY department
HAVING COUNT(*) > 10
   AND SUM(CASE WHEN salary > 8000 THEN 1 ELSE 0 END) >= 3;
```

## 8. **特殊场景**

### 场景：没有 GROUP BY 时能否用 HAVING？

```sql
-- ✅ 可以：对整个结果集作为一组进行过滤
SELECT COUNT(*) as total_employees, AVG(salary) as company_avg_salary
FROM employees
HAVING AVG(salary) > 5000;  -- 相当于 WHERE，但可以使用聚合函数

-- 等价于：
SELECT COUNT(*) as total_employees, AVG(salary) as company_avg_salary
FROM employees
WHERE 1 = 1  -- 所有行
HAVING AVG(salary) > 5000;
```

### 场景：窗口函数中的过滤

```sql
-- 使用子查询或 CTE 过滤窗口函数结果
WITH ranked_employees AS (
    SELECT
        department,
        name,
        salary,
        RANK() OVER(PARTITION BY department ORDER BY salary DESC) as rank_in_dept
    FROM employees
)
SELECT department, name, salary
FROM ranked_employees
WHERE rank_in_dept <= 3;  -- 需要在子查询外使用 WHERE 过滤
```

## **记忆口诀**

> **WHERE 先行 HAVING 后，**
> **行过滤完再分组。**
> **聚合函数 HAVING 用，**
> **别名排序它可用。**
> **先 WHERE 来减数据，**
> **后 HAVING 提效率。**

## **总结表格**

| 方面            | WHERE                    | HAVING                   |
| --------------- | ------------------------ | ------------------------ |
| **执行顺序**    | 分组前                   | 分组后                   |
| **过滤对象**    | 原始数据行               | 分组后的聚合结果         |
| **聚合函数**    | 不可用                   | 可用                     |
| **SELECT 别名** | 不可用                   | 可用（大多数情况）       |
| **性能影响**    | 减少分组数据量，提高性能 | 分组后过滤，可能降低性能 |
| **适用场景**    | 行级条件过滤             | 分组级条件过滤           |
| **与 GROUP BY** | 可有可无                 | 通常与 GROUP BY 一起使用 |
| **索引利用**    | 可以利用索引优化         | 通常无法直接利用索引     |

**黄金法则：**

- 如果过滤条件**不涉及聚合计算** → 用 **WHERE**
- 如果过滤条件**涉及聚合计算** → 用 **HAVING**
- 两者可结合使用：先用 WHERE 减少数据量，再用 HAVING 过滤分组结果
