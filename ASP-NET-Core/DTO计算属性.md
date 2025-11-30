DTO（Data Transfer Object）里的“计算属性”**不是数据库列，也不是导航属性**，  
而是**只读、即时算出来**的“虚拟字段”，用来**让接口少发一次 SQL、少写一次循环**，  
把**纯内存计算**放在 DTO 里，一次性返回给前端。

---

### 1. 最简例子：三行代码搞定

```csharp
public class OrderDto
{
    public int Id { get; set; }
    public decimal UnitPrice { get; set; }
    public int Quantity { get; set; }

    // 计算属性：无 backing field，只 get
    public decimal TotalAmount => UnitPrice * Quantity;
}
```

- **不映射到数据库**（EF Core 会忽略）。  
- **序列化**（System.Text.Json / Newtonsoft）**默认包含**，前端直接拿到 `totalAmount`。  
- **无状态、线程安全**。

---

### 2. 再算复杂一点：只读 LINQ 组合

```csharp
public class TeamDto
{
    public string TeamName { get; set; }
    public List<PlayerDto> Players { get; set; }

    public int PlayerCount => Players?.Count ?? 0;
    public decimal AvgScore   => Players == null || Players.Count == 0
                                 ? 0
                                 : Players.Average(p => p.Score);
}
```

---

### 3. 依赖注入？别放 DTO！

计算属性**只应做“纯内存计算”**，  
如果要去数据库、调 `IRepository` → **请写 Service，不要塞 DTO**，  
否则序列化器会**意外触发数据库查询**（N+1 或空引用）。

---

### 4. 不想序列化怎么办？

```csharp
// System.Text.Json
[JsonIgnore]
public decimal TotalAmount => UnitPrice * Quantity;

// Newtonsoft
[JsonProperty(PropertyName = "totalAmount", NullValueHandling = NullValueHandling.Ignore)]
// 或去掉属性即可
```

---

### 5. 表达式树投影：EF Core 也能“提前算”

```csharp
var orders = _context.Orders
    .Select(o => new OrderDto
    {
        Id        = o.Id,
        UnitPrice = o.UnitPrice,
        Quantity  = o.Quantity,
        // 让 SQL 直接算完，返回时就是最终结果
        TotalAmount = o.UnitPrice * o.Quantity
    }).ToListAsync();
```

> 此时 `TotalAmount` 在 SQL 里就乘好了，**不属于客户端计算**，但仍不是数据库列。

---

### 6. 一句话总结

> DTO 里的计算属性 = **只读 get-only 字段**，**纯内存、无状态、即时算**，  
> 用来**减少后端循环、减少前端二次计算**，  
> **别依赖数据库或外部服务**，否则就把逻辑搬到 Service 层。