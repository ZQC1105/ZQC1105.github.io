## [Transact-SQL 参考](https://learn.microsoft.com/zh-cn/sql/t-sql/language-reference?view=sql-server-ver17)

## [Transact-SQL 语法约定](https://learn.microsoft.com/zh-cn/sql/t-sql/language-elements/transact-sql-syntax-conventions-transact-sql?view=sql-server-ver17&tabs=code)

## [变量（Transact-SQL）](https://learn.microsoft.com/zh-cn/sql/t-sql/language-elements/variables-transact-sql?view=sql-server-ver17)

- T-SQL 语言元素（如控制流语句、运算符、函数等）
- 数据定义语言（DDL）、数据操作语言（DML）、数据控制语言（DCL）语法
- 系统存储过程、内置函数、数据类型等详细说明

Transact-SQL（通常缩写为 T-SQL）是微软和 Sybase 公司共同开发的一种关系数据库通用语言 SQL 的扩展。它在标准 SQL 的基础上增加了编程功能，如变量、流程控制、函数等，主要用于 Microsoft SQL Server 和 Sybase Adaptive Server 等数据库管理系统中。

### T-SQL 的主要特点：

1. **数据操作语言 (DML) 扩展**：
   - 支持标准的 `SELECT`, `INSERT`, `UPDATE`, `DELETE` 语句。
   - 提供更强大的 `SELECT` 功能，如 `TOP`, `OFFSET-FETCH`, 窗口函数（`ROW_NUMBER()`, `RANK()`, `OVER()` 等）。

2. **数据定义语言 (DDL) 支持**：
   - 用于创建、修改和删除数据库对象，如表、视图、索引等。
   - 示例：`CREATE TABLE`, `ALTER TABLE`, `DROP INDEX`。

3. **数据控制语言 (DCL)**：
   - 用于管理数据库权限和安全性。
   - 示例：`GRANT`, `REVOKE`, `DENY`。

4. **流程控制语句**：
   - `IF...ELSE`
   - `WHILE` 循环
   - `BEGIN...END` 块
   - `TRY...CATCH` 异常处理
   - `GOTO`（不推荐使用）

5. **变量与批处理**：
   - 局部变量：以 `@` 开头，如 `@name NVARCHAR(50)`
   - 全局变量（系统函数）：以 `@@` 开头，如 `@@VERSION`, `@@ROWCOUNT`, `@@ERROR`

6. **内置函数**：
   - 字符串函数：`LEN()`, `SUBSTRING()`, `REPLACE()`
   - 数值函数：`ROUND()`, `CEILING()`, `FLOOR()`
   - 日期函数：`GETDATE()`, `DATEADD()`, `DATEDIFF()`
   - 聚合函数：`SUM()`, `COUNT()`, `AVG()`, `MAX()`, `MIN()`

7. **存储过程与函数**：
   - 存储过程（Stored Procedure）：预编译的 SQL 代码块，可带参数，提高性能和复用性。
   - 用户定义函数（UDF）：可返回标量值或表。

8. **事务控制**：
   - `BEGIN TRANSACTION`
   - `COMMIT`
   - `ROLLBACK`
   - 支持事务的 ACID 特性。

---

### 示例代码：

```sql
-- 声明变量
DECLARE @EmployeeID INT = 101;
DECLARE @Salary DECIMAL(10,2);

-- 使用 SELECT 查询赋值
SELECT @Salary = Salary
FROM Employees
WHERE ID = @EmployeeID;

-- 流程控制
IF @Salary > 5000
    PRINT 'High Salary';
ELSE
    PRINT 'Normal Salary';

-- 异常处理示例
BEGIN TRY
    BEGIN TRANSACTION;
    UPDATE Accounts SET Balance -= 100 WHERE AccountID = 1;
    UPDATE Accounts SET Balance += 100 WHERE AccountID = 2;
    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT 'Transaction failed: ' + ERROR_MESSAGE();
END CATCH;
```

---

下面给你 3 条“能直接下载、用 SSDT 打开就能跑”的开源 Transact-SQL 演练项目，按“先易后难”排好，5 分钟就能在 Visual Studio 里按 F5 调试，最适合快速把 T-SQL + SSDT 的完整工作流跑通。

---

1. **Northwind-SSDT**（微软经典库，零门槛）  
   仓库：https://github.com/microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs

- 已拆成 .sql 脚本，直接“文件 → 新建 →SQL Server 数据库项目 → 导入脚本”即可生成完整项目。
- 表、视图、存储过程、触发器一应俱全，适合练“单步调试存储过程”“重构索引”等基础操作。

2. **WideWorldImporters-SSDT**（微软官方“现代”示例，带 CI/CD）  
   **1 分钟拿到源码**：打开官方仓库 https://github.com/microsoft/sql-server-samples → 右侧 Releases → 找到“WideWorldImporters SSDT sample database”（标签名 wwi-ssdt）→ 下载 Source code (zip)。  
   解压后路径：`samples\databases\wide-world-importers\wide-world-importers-ssdt\WideWorldImporters.sqlproj`  
   双击 .sqlproj 即可用 Visual Studio + SSDT 打开，F5 直接跑。

- 官方 SSDT 项目（.sqlproj），下载即用，README 里自带 Azure DevOps yaml，可顺带把“Git → Build → Publish”整条 DevOps 流水线跑一遍。
- 包含大量 2016+ 新语法（JSON、列存、CCI、安全策略），想练“现代 T-SQL”就它。

3. **tSQLt-SSDT-Template**（单元测试框架，进阶必备）  
   仓库：https://github.com/tSQLt-org/tSQLt-SSDT-Template

- 把主流单元测试框架 tSQLt 打包成 SSDT 项目，F5 后自动部署 tSQLt 与示例测试。
- 可练“测试先行”模式：写测试类 → 写实现 → 调试 → 提交 Git，完美模拟企业级数据库开发流程。

---

## 5 分钟上手步骤（以 WideWorldImporters 为例）

1. **装 SSDT**：打开 Visual Studio Installer → 勾选“SQL Server Data Tools”工作负载。
2. **拿代码**：下载解压后，双击 `.sqlproj` 打开。
3. **配调试库**：右键项目 → 属性 → 调试 → 新建 LocalDB 或指向自己的 SQL Server 实例。
4. **F5 一键部署+调试**：选“Start (调试)”，VS 会自动生成 .dacpac、publish 到调试库、附加调试器，随后即可在存储过程里打断点、单步、看变量。

---

## 两库对比

| 对比维度          | Northwind-SSDT（经典版）                  | WideWorldImporters-SSDT（现代版）                                                                            |
| :---------------- | :---------------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| **设计年代**      | 1996 年随 Office 发布，90 年代小商户场景  | 2016 年随 SQL Server 2016 设计，面向云端与现代 ERP                                                           |
| **目标 SQL 版本** | SQL Server 2000-2012 语法为主             | SQL Server 2016+ / Azure SQL DB，需 2016 SP1+ 跑全功能                                                       |
| **项目体积**      | ≈ 1-2 MB，几十张表，数据 1-10 MB 级       | 空库 30 MB，生成 3.5 年数据后 ≈ 160 MB（可扩到 GB 级）                                                       |
| **新特性覆盖**    | 无（无列存、内存、JSON、时态表、分区）    | 全部都有：列存、内存 OLTP、时态表、JSON、分区、数据掩码、行级安全等                                          |
| **SSDT 项目结构** | 手工拆分脚本，简单直观，适合“第一次按 F5” | 官方原生 `.sqlproj`，预置 300+ 对象，分 Schema 文件夹，企业级模板                                            |
| **数据生成方式**  | 静态 INSERT 脚本，数据量固定              | 内置 `DataLoadSimulation.PopulateDataToCurrentDate` 存储过程，可参数化按日生成订单，支持周末系数、静默模式等 |
| **场景定位**      | ① 零基础学 T-SQL ② 教课 ③ 做演示用        | ① 练“现代”T-SQL/新特性 ② 练 CI/CD 发布 ③ 做性能测试（列存、内存）                                            |
| **调试体验**      | 表少、逻辑简单，存储过程断点一目了然      | 对象多，但自带示例测试存储过程，可练“大库”下调试 & 单元测试                                                  |
| **与 CI/CD 结合** | 无官方 yaml，需自己写                     | 同仓库直接给 Azure DevOps/GitHub Actions 样例，push 即发布                                                   |

---

## 学习路线图（按工作日每天 1.5h 估算）

### 起点 A：只会简单 SELECT / 没用过 SSDT

| 阶段         | 目标                  | 任务清单（可量化）                                                                           | 耗时                    |
| :----------- | :-------------------- | :------------------------------------------------------------------------------------------- | :---------------------- |
| ① 入门       | 把 SSDT 玩顺          | 克隆 Northwind-SSDT → F5 成功 → 单步调试 5 个存储过程                                        | 3 天                    |
| ② 基础 SQL   | 写 CRUD + 视图 + 事务 | 在 Northwind 里自己写 10 个查询、3 个视图、2 个带事务的存储过程并通过 tSQLt 测试             | 7 天                    |
| ③ 迁移现代库 | 体验“大数据量”        | 把 Northwind 订单表导入 WideWorldImporters，跑通 `PopulateDataToCurrentDate` 生成 100 万订单 | 3 天                    |
| ④ 新特性     | 掌握 2016+ 功能       | 在 WideWorldImporters 里各做一次：列存索引、内存 OLTP、时态表、JSON 查询、行级安全           | 10 天                   |
| ⑤ 实战       | 端到端 DevOps         | 用 GitHub Actions 把 WideWorldImporters 自动发布到 Docker 版 SQL 2025，加单元测试闸门        | 5 天                    |
| **合计**     |                       |                                                                                              | **≈ 4 周（20 工作日）** |

### 起点 B：已会 T-SQL，第一次用 SSDT

跳过阶段 ①②，直接从阶段 ③ 开始， **≈ 2 周** 可完成。

### 起点 C：熟手，只想快速补新特性 / CI/CD

只跑阶段 ④⑤， **≈ 1 周** 就够。

---

## 加速技巧（通用）

1. 每天任务拆成 ≤30 min 的小块，用 tSQLt 测试保证一次做对。
2. 把 WideWorldImporters 数据生成脚本参数调小（`@YearNumberToLoad=1`），笔记本也能 3 min 跑完。
3. 用 VS Code 的 mssql 扩展 + SSDT 同时开，查询窗口和项目文件切换更快。
4. 周末集中 2 h 做“综合演练”——写个迷你业务需求（如“给 Northwind 加会员积分表”）从需求 → 建表 → 存储过程 → 测试 →Git 提交一条龙，一次性把知识缝起来。

---

## 验收标准（你能独立做出即算“掌握”）

- 不用向导，手工新建 SSDT 项目，把任意业务库（≥15 张表）反向工程进来并推送到 Git。
- 能用列存 + 分区把 1 亿行销售表查询压到 1 s 内。
- 一条 `git push` 自动跑 tSQLt 测试 → 生成发布脚本 → 部署到 Docker SQL 实例 → 回写结果。

达到这三条，两个示例库就可以毕业，接下来直接拿生产项目练手即可。祝你 1 个月后就能上台讲“SQL Server 现代开发工作流”！

## T-SQL 语句的主要作用

T-SQL (Transact-SQL) 是 Microsoft SQL Server 的扩展 SQL 语言，它在标准 SQL 的基础上增加了多种功能。以下是 T-SQL 语句的主要作用分类：

## 一、数据查询与操作

### 1. **数据查询 (DQL - Data Query Language)**

```sql
-- 基本查询
SELECT Name, Salary FROM Employees;

-- 条件过滤
SELECT * FROM Employees WHERE DepartmentID = 1;

-- 聚合查询
SELECT DepartmentID, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY DepartmentID;

-- 连接查询
SELECT e.Name, d.DepartmentName
FROM Employees e
JOIN Departments d ON e.DepartmentID = d.DepartmentID;
```

### 2. **数据操作 (DML - Data Manipulation Language)**

```sql
-- 插入数据
INSERT INTO Employees (Name, Salary, DepartmentID)
VALUES ('张三', 8000, 1);

-- 更新数据
UPDATE Employees
SET Salary = Salary * 1.1
WHERE DepartmentID = 1;

-- 删除数据
DELETE FROM Employees WHERE EmployeeID = 100;

-- 合并数据（存在则更新，不存在则插入）
MERGE INTO Employees AS target
USING NewEmployees AS source
ON target.EmployeeID = source.EmployeeID
WHEN MATCHED THEN
    UPDATE SET Salary = source.Salary
WHEN NOT MATCHED THEN
    INSERT (Name, Salary) VALUES (source.Name, source.Salary);
```

### 3. **数据定义 (DDL - Data Definition Language)**

```sql
-- 创建表
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    Name VARCHAR(100),
    Salary DECIMAL(10,2)
);

-- 修改表结构
ALTER TABLE Employees ADD HireDate DATE;

-- 创建索引
CREATE INDEX IX_Employees_Salary ON Employees(Salary);

-- 创建视图
CREATE VIEW HighSalaryEmployees AS
SELECT Name, Salary FROM Employees WHERE Salary > 10000;
```

## 二、流程控制与业务逻辑

### 1. **条件控制**

```sql
-- IF-ELSE 语句
IF (SELECT AVG(Salary) FROM Employees) > 8000
BEGIN
    PRINT '平均工资超过8000'
    UPDATE Employees SET Bonus = 1000
END
ELSE
BEGIN
    PRINT '平均工资低于8000'
    UPDATE Employees SET Bonus = 500
END

-- CASE 表达式
SELECT Name, Salary,
    CASE
        WHEN Salary > 10000 THEN '高薪'
        WHEN Salary > 6000 THEN '中薪'
        ELSE '低薪'
    END AS SalaryLevel
FROM Employees;
```

### 2. **循环控制**

```sql
-- WHILE 循环
DECLARE @Counter INT = 1;
WHILE @Counter <= 10
BEGIN
    UPDATE Employees
    SET Salary = Salary * 1.05
    WHERE EmployeeID = @Counter;

    SET @Counter = @Counter + 1;
END

-- 循环中的跳转控制
WHILE @Counter > 0
BEGIN
    IF @Counter = 5
        BREAK;  -- 跳出循环

    IF @Counter = 8
    BEGIN
        SET @Counter = @Counter - 1;
        CONTINUE;  -- 跳过本次循环
    END

    -- 其他操作
    SET @Counter = @Counter - 1;
END
```

## 三、变量与临时存储

### 1. **变量管理**

```sql
-- 标量变量
DECLARE @EmployeeName VARCHAR(100);
DECLARE @Salary DECIMAL(10,2);
DECLARE @Counter INT = 0;

-- 表变量
DECLARE @TempTable TABLE (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Amount DECIMAL(10,2)
);

-- 变量赋值
SELECT @Salary = Salary FROM Employees WHERE EmployeeID = 1;
SET @Counter = @Counter + 1;
```

### 2. **临时表**

```sql
-- 本地临时表
CREATE TABLE #TempEmployees (
    EmployeeID INT,
    Name VARCHAR(100)
);

-- 全局临时表
CREATE TABLE ##GlobalTemp (
    Data VARCHAR(100)
);

-- 表变量 vs 临时表
-- 表变量：内存中，适用于小数据量
-- 临时表：磁盘上，适用于大数据量
```

## 四、错误处理与事务

### 1. **错误处理**

```sql
BEGIN TRY
    -- 可能出错的代码
    INSERT INTO Employees (EmployeeID, Name) VALUES (1, '张三');
    INSERT INTO Employees (EmployeeID, Name) VALUES (1, '李四'); -- 主键冲突
END TRY
BEGIN CATCH
    -- 错误处理代码
    SELECT
        ERROR_NUMBER() AS ErrorNumber,
        ERROR_MESSAGE() AS ErrorMessage,
        ERROR_LINE() AS ErrorLine;

    -- 记录错误日志
    INSERT INTO ErrorLog (ErrorNumber, ErrorMessage, ErrorDate)
    VALUES (ERROR_NUMBER(), ERROR_MESSAGE(), GETDATE());
END CATCH
```

### 2. **事务管理**

```sql
BEGIN TRANSACTION
BEGIN TRY
    UPDATE Accounts SET Balance = Balance - 1000 WHERE AccountID = 1;
    UPDATE Accounts SET Balance = Balance + 1000 WHERE AccountID = 2;

    -- 检查余额
    IF (SELECT Balance FROM Accounts WHERE AccountID = 1) < 0
        THROW 50000, '余额不足', 1;

    COMMIT TRANSACTION;
    PRINT '转账成功';
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT '转账失败：' + ERROR_MESSAGE();
END CATCH
```

## 五、高级功能

### 1. **窗口函数**

```sql
-- 排名
SELECT Name, Salary,
    ROW_NUMBER() OVER (ORDER BY Salary DESC) AS RowNum,
    RANK() OVER (ORDER BY Salary DESC) AS Rank,
    DENSE_RANK() OVER (ORDER BY Salary DESC) AS DenseRank
FROM Employees;

-- 移动计算
SELECT Date, Sales,
    AVG(Sales) OVER (ORDER BY Date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS MovingAvg,
    SUM(Sales) OVER (ORDER BY Date ROWS UNBOUNDED PRECEDING) AS RunningTotal
FROM DailySales;

-- 分区计算
SELECT DepartmentID, Name, Salary,
    AVG(Salary) OVER (PARTITION BY DepartmentID) AS DeptAvg,
    MAX(Salary) OVER (PARTITION BY DepartmentID) AS DeptMax
FROM Employees;
```

### 2. **公用表表达式 (CTE)**

```sql
-- 基本CTE
WITH DeptAvg AS (
    SELECT DepartmentID, AVG(Salary) AS AvgSalary
    FROM Employees
    GROUP BY DepartmentID
)
SELECT e.Name, e.Salary, d.AvgSalary
FROM Employees e
JOIN DeptAvg d ON e.DepartmentID = d.DepartmentID
WHERE e.Salary > d.AvgSalary;

-- 递归CTE
WITH EmployeeHierarchy AS (
    -- 锚点成员
    SELECT EmployeeID, Name, ManagerID, 1 AS Level
    FROM Employees
    WHERE ManagerID IS NULL

    UNION ALL

    -- 递归成员
    SELECT e.EmployeeID, e.Name, e.ManagerID, h.Level + 1
    FROM Employees e
    JOIN EmployeeHierarchy h ON e.ManagerID = h.EmployeeID
)
SELECT * FROM EmployeeHierarchy;
```

### 3. **动态SQL**

```sql
-- 构建动态查询
DECLARE @TableName VARCHAR(100) = 'Employees';
DECLARE @SQL NVARCHAR(MAX);

SET @SQL = N'SELECT * FROM ' + QUOTENAME(@TableName) +
           N' WHERE Salary > @MinSalary';

EXEC sp_executesql @SQL,
    N'@MinSalary DECIMAL(10,2)',
    @MinSalary = 5000;
```

## 六、性能优化

### 1. **索引优化**

```sql
-- 创建索引
CREATE NONCLUSTERED INDEX IX_Employees_Salary
ON Employees(Salary) INCLUDE (Name, DepartmentID);

-- 索引提示
SELECT Name, Salary
FROM Employees WITH (INDEX(IX_Employees_Salary))
WHERE Salary > 5000;
```

### 2. **查询优化**

```sql
-- 使用 EXISTS 代替 IN
SELECT Name FROM Employees e
WHERE EXISTS (SELECT 1 FROM Departments d
              WHERE d.DepartmentID = e.DepartmentID
              AND d.Budget > 100000);

-- 使用表连接代替子查询
SELECT e.Name, d.DepartmentName
FROM Employees e
JOIN Departments d ON e.DepartmentID = d.DepartmentID;
```

## 七、系统管理

### 1. **权限管理**

```sql
-- 创建用户
CREATE USER John FOR LOGIN John;

-- 授予权限
GRANT SELECT, INSERT ON Employees TO John;

-- 拒绝权限
DENY DELETE ON Employees TO John;

-- 角色管理
EXEC sp_addrolemember 'db_datareader', 'John';
```

### 2. **元数据查询**

```sql
-- 查询表结构
SELECT * FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Employees';

-- 查询索引信息
SELECT * FROM sys.indexes
WHERE object_id = OBJECT_ID('Employees');
```

## 总结

T-SQL 的主要作用可以概括为：

| 类别         | 主要作用       | 典型语句                       |
| ------------ | -------------- | ------------------------------ |
| **数据操作** | 增删改查数据   | SELECT, INSERT, UPDATE, DELETE |
| **数据定义** | 管理数据库对象 | CREATE, ALTER, DROP            |
| **流程控制** | 实现业务逻辑   | IF, WHILE, CASE                |
| **变量存储** | 临时数据存储   | DECLARE, SET, SELECT INTO      |
| **错误处理** | 异常捕获与处理 | TRY-CATCH, THROW               |
| **事务管理** | 保证数据一致性 | BEGIN TRAN, COMMIT, ROLLBACK   |
| **高级分析** | 复杂数据分析   | 窗口函数, CTE, 分析函数        |
| **性能优化** | 提升查询效率   | 索引, 执行计划, 查询提示       |
| **安全管理** | 控制访问权限   | GRANT, DENY, REVOKE            |

T-SQL 让 SQL Server 成为一个完整的开发平台，不仅能进行简单的数据操作，还能实现复杂的业务逻辑和高级数据分析。
