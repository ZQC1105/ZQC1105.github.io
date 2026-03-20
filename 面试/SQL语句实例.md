# SQL 查询示例集合

本文档整理了多个SQL查询示例，涵盖了从基础查询到高级分析的各类场景。

## 目录

- [完整员工信息查询](#完整员工信息查询)
- [简单条件查询](#简单条件查询)
- [NULL值处理示例](#null值处理示例)
- [连接查询](#连接查询)
- [分组统计](#分组统计)
- [窗口函数高级分析](#窗口函数高级分析)
- [子查询示例](#子查询示例)
- [查询优化示例](#查询优化示例)
- [公用表表达式(CTE)](#公用表表达式cte)
- [表变量使用](#表变量使用)
- [综合查询示例](#综合查询示例)

---

## 完整员工信息查询

```sql
-- 查看完整的员工信息
SELECT
    E.EmployeeID,
    E.Name AS 员工姓名,
    D.DepartmentName AS 部门,
    E.Salary AS 薪资,
    E.HireDate AS 入职日期,
    M.Name AS 经理姓名,
    E.Status AS 状态
FROM Employees E
LEFT JOIN Departments D ON E.DepartmentID = D.DepartmentID
LEFT JOIN Employees M ON E.ManagerID = M.EmployeeID
ORDER BY D.DepartmentID, E.Salary DESC;
```

---

## 简单条件查询

```sql
-- 查询2024年后入职的员工
SELECT Name, HireDate
FROM Employees
WHERE HireDate > '2024-01-01'
ORDER BY HireDate DESC;
```

---

## NULL值处理示例

```sql
-- 使用ISNULL处理NULL值
SELECT Name, HireDate, ISNULL(ManagerID, 0) AS 测试
FROM Employees
WHERE HireDate > '2001-01-01'
ORDER BY HireDate DESC;

-- 使用COALESCE处理NULL值
SELECT Name, HireDate, COALESCE(ManagerID, 0) AS 测试
FROM Employees
WHERE HireDate > '2001-01-01'
ORDER BY HireDate DESC;
```

---

## 连接查询

```sql
-- 左连接查询员工及其部门信息
SELECT E.Name, E.Salary, D.DepartmentName
FROM Employees E
LEFT JOIN Departments D ON E.DepartmentID = D.DepartmentID;
```

---

## 分组统计

```sql
-- 按部门统计员工人数和平均薪资
SELECT DepartmentID,
       COUNT(*) AS 员工人数,
       AVG(Salary) AS 平均薪资
FROM Employees
GROUP BY DepartmentID
ORDER BY 平均薪资 DESC;
```

---

## 窗口函数高级分析

```sql
-- 部门薪资综合分析（包含多种窗口函数）
SELECT
    DepartmentID,
    COUNT(*) AS 员工人数,
    AVG(Salary) AS 平均薪资,
    MAX(Salary) AS 最高薪资,
    MIN(Salary) AS 最低薪资,
    RANK() OVER (ORDER BY AVG(Salary) DESC) AS 薪资排名,
    PERCENT_RANK() OVER (ORDER BY AVG(Salary) DESC) AS 薪资百分位,
    ROW_NUMBER() OVER (PARTITION BY COUNT(*) ORDER BY AVG(Salary) DESC) AS 同规模部门内排名,
    AVG(AVG(Salary)) OVER () AS 公司总平均薪资,
    AVG(Salary) - AVG(AVG(Salary)) OVER () AS 与公司平均差额,
    SUM(SUM(Salary)) OVER (ORDER BY AVG(Salary) DESC) /
        SUM(SUM(Salary)) OVER () * 100 AS 薪资累计百分比
FROM Employees
GROUP BY DepartmentID
ORDER BY 平均薪资 DESC;
```

---

## 子查询示例

```sql
-- 查询薪资低于平均值的员工
SELECT Name, Salary
FROM Employees
WHERE Salary < (SELECT AVG(Salary) FROM Employees);
```

---

## 查询优化示例

```sql
-- 使用窗口函数优化子查询（查询薪资高于平均值的员工）
SELECT Name, Salary
FROM (
    SELECT Name, Salary,
           AVG(Salary) OVER () AS AvgSalary
    FROM Employees
) t
WHERE Salary > AvgSalary;
```

---

## 公用表表达式(CTE)

```sql
-- 使用CTE查询薪资高于平均值的员工
WITH AvgCTE AS (
    SELECT AVG(Salary) AS AvgSalary FROM Employees
)
SELECT e.Name, e.Salary
FROM Employees e
CROSS JOIN AvgCTE a
WHERE e.Salary > a.AvgSalary;
```

---

## 表变量使用

```sql
-- 使用表变量存储平均值并查询
BEGIN
    DECLARE @AvgSalary DECIMAL(10,2);

    SELECT @AvgSalary = AVG(Salary) FROM Employees;

    SELECT Name, Salary
    FROM Employees
    WHERE Salary > @AvgSalary;
END;
```

---

## 综合查询示例

```sql
-- 查询2024年入职的员工完整信息
SELECT
    E.EmployeeID,
    E.Name AS 员工姓名,
    D.DepartmentName AS 部门,
    E.Salary AS 薪资,
    E.HireDate AS 入职日期,
    M.Name AS 经理姓名,
    E.Status AS 状态
FROM Employees E
LEFT JOIN Departments D ON E.DepartmentID = D.DepartmentID
LEFT JOIN Employees M ON E.ManagerID = M.EmployeeID
WHERE YEAR(E.HireDate) = 2024
ORDER BY D.DepartmentID, E.Salary DESC;
```

---

## 使用说明

1. **执行环境**：这些查询适用于支持标准SQL的数据库系统（SQL Server、MySQL、PostgreSQL等）
2. **注意事项**：
   - 部分函数（如 `ISNULL`）是SQL Server特有的，在其他数据库中可能需要调整
   - 确保表名和字段名与实际的数据库结构匹配
   - 窗口函数需要数据库版本支持（SQL Server 2005+，MySQL 8.0+，PostgreSQL 8.4+）

3. **查询目的**：
   - 基础查询：适合日常数据检索
   - 分组统计：用于报表和数据分析
   - 窗口函数：进行复杂的数据分析计算
   - 优化示例：展示不同写法的性能对比
