## GORM 主要功能详解

GORM 是一个功能强大的 ORM 库，提供了丰富的特性。让我为你详细介绍其主要功能：

## 1. **模型定义与约定**

### 基础模型

```go
// 默认约定：表名是结构体名的复数形式，主键是 ID，创建时间字段是 CreatedAt 等
type User struct {
    ID        uint           `gorm:"primarykey"`                 // 主键
    Name      string         `gorm:"size:100;not null"`          // 字符串大小，非空
    Email     string         `gorm:"size:100;uniqueIndex"`       // 唯一索引
    Age       int            `gorm:"default:18"`                 // 默认值
    Active    bool           `gorm:"default:true"`               // 布尔默认值
    Salary    float64        `gorm:"type:decimal(10,2)"`         // 自定义类型
    CreatedAt time.Time      `gorm:"autoCreateTime"`             // 自动创建时间
    UpdatedAt time.Time      `gorm:"autoUpdateTime"`             // 自动更新时间
    DeletedAt gorm.DeletedAt `gorm:"index"`                      // 软删除
}

// 自定义表名
func (User) TableName() string {
    return "my_users"
}

// 使用嵌入结构体
type BaseModel struct {
    ID        uint      `gorm:"primarykey"`
    CreatedAt time.Time
    UpdatedAt time.Time
}

type Product struct {
    BaseModel
    Name  string
    Price float64
}
```

## 2. **CRUD 操作**

### 创建

```go
// 单条创建
user := User{Name: "张三", Email: "zhangsan@example.com", Age: 25}
result := db.Create(&user) // 返回结果对象

// 批量创建
users := []User{{Name: "李四"}, {Name: "王五"}}
db.Create(&users)

// 有选择的创建
db.Select("Name", "Email").Create(&user)  // 只创建指定字段
db.Omit("Age", "Active").Create(&user)    // 忽略指定字段

// 批量插入（性能优化）
db.CreateInBatches(users, 100) // 每批100条

// 检查错误
if result.Error != nil {
    // 处理错误
}
fmt.Println(result.RowsAffected) // 影响的行数
```

### 查询

```go
// 查询单条
var user User
db.First(&user, 1)                 // 根据主键查询
db.First(&user, "name = ?", "张三") // 条件查询
db.Take(&user)                      // 获取一条记录，不排序
db.Last(&user)                      // 获取最后一条

// 查询所有
var users []User
db.Find(&users)                     // 查询所有
db.Find(&users, "age > ?", 18)      // 条件查询

// 条件查询
db.Where("name = ?", "张三").First(&user)
db.Where("name LIKE ?", "%张%").Find(&users)
db.Where("age > ? AND active = ?", 18, true).Find(&users)
db.Where(map[string]interface{}{"name": "张三", "age": 25}).Find(&users)
db.Where([]int{1, 2, 3}).Find(&users) // 主键 IN (1,2,3)

// 使用 Struct 条件
db.Where(&User{Name: "张三", Age: 25}).First(&user)

// Not 条件
db.Not("name = ?", "张三").Find(&users)
db.Not([]int{1, 2, 3}).Find(&users)

// Or 条件
db.Where("name = ?", "张三").Or("name = ?", "李四").Find(&users)

// 选择特定字段
db.Select("name", "email").Find(&users)
db.Select("name, email").Find(&users)

// 排序
db.Order("age desc, name asc").Find(&users)

// 限制和偏移
db.Limit(10).Offset(5).Find(&users)

// 分组和 Having
db.Model(&User{}).Select("age, count(*) as total").Group("age").Having("count(*) > ?", 1).Find(&results)

// 去重
db.Distinct("name", "age").Find(&users)

// 扫描到结构体
type Result struct {
    Name  string
    Email string
}
var results []Result
db.Table("users").Select("name, email").Scan(&results)
```

## 3. **高级查询**

### 链式操作

```go
// 查询构建
query := db.Where("active = ?", true)
if name != "" {
    query = query.Where("name LIKE ?", "%"+name+"%")
}
if minAge > 0 {
    query = query.Where("age >= ?", minAge)
}
query.Order("created_at desc").Limit(10).Find(&users)
```

### 子查询

```go
// 子查询
subQuery := db.Select("AVG(age)").Where("active = ?", true).Table("users")
db.Where("age > (?)", subQuery).Find(&users)

// 在 Where 中使用子查询
db.Where("amount > (?)", db.Table("orders").Select("AVG(amount)")).Find(&orders)
```

### 原生 SQL

```go
// 执行原生 SQL
db.Raw("SELECT id, name, age FROM users WHERE age > ?", 18).Scan(&users)

// Exec 执行 SQL
db.Exec("UPDATE users SET active = ? WHERE age < ?", false, 18)
```

## 4. **关联关系**

### 一对一

```go
type User struct {
    gorm.Model
    Name    string
    Profile Profile // 拥有一个 Profile
}

type Profile struct {
    gorm.Model
    UserID uint
    Bio    string
    User   User // 属于 User
}

// 使用
db.Preload("Profile").Find(&users)
```

### 一对多

```go
type User struct {
    gorm.Model
    Name    string
    Orders  []Order // 拥有多个订单
}

type Order struct {
    gorm.Model
    UserID uint
    Amount float64
    User   User // 属于 User
}

// 查询包含订单的用户
db.Preload("Orders").Find(&users)

// 带条件的预加载
db.Preload("Orders", "amount > ?", 100).Find(&users)
```

### 多对多

```go
type User struct {
    gorm.Model
    Name    string
    Roles   []Role `gorm:"many2many:user_roles"`
}

type Role struct {
    gorm.Model
    Name  string
    Users []User `gorm:"many2many:user_roles"`
}

// 使用
db.Preload("Roles").Find(&users)
```

## 5. **事务处理**

```go
// 手动事务
tx := db.Begin()
defer func() {
    if r := recover(); r != nil {
        tx.Rollback()
    }
}()

if err := tx.Create(&user).Error; err != nil {
    tx.Rollback()
    return err
}

if err := tx.Create(&profile).Error; err != nil {
    tx.Rollback()
    return err
}

tx.Commit()

// 自动事务
db.Transaction(func(tx *gorm.DB) error {
    if err := tx.Create(&user).Error; err != nil {
        return err
    }
    if err := tx.Create(&profile).Error; err != nil {
        return err
    }
    return nil
})

// 嵌套事务
db.Transaction(func(tx *gorm.DB) error {
    tx.Create(&user)

    tx.Transaction(func(tx2 *gorm.DB) error {
        tx2.Create(&profile)
        return nil
    })

    return nil
})
```

## 6. **钩子函数**

```go
// 创建钩子
func (u *User) BeforeCreate(tx *gorm.DB) error {
    u.CreatedAt = time.Now()
    u.UpdatedAt = time.Now()
    return nil
}

func (u *User) AfterCreate(tx *gorm.DB) error {
    // 创建后的操作，如发送邮件
    log.Printf("用户 %s 已创建", u.Name)
    return nil
}

// 查询钩子
func (u *User) AfterFind(tx *gorm.DB) error {
    // 查询后的操作
    return nil
}

// 更新钩子
func (u *User) BeforeUpdate(tx *gorm.DB) error {
    u.UpdatedAt = time.Now()
    return nil
}

// 删除钩子
func (u *User) BeforeDelete(tx *gorm.DB) error {
    // 删除前的操作
    return nil
}
```

## 7. **索引和约束**

```go
type User struct {
    gorm.Model
    Name  string `gorm:"index:idx_name,priority:1"`          // 普通索引
    Email string `gorm:"uniqueIndex;size:255"`                // 唯一索引
    Age   int    `gorm:"index:,class:FULLTEXT,comment:年龄索引"` // 全文索引
}

// 复合索引
type Product struct {
    gorm.Model
    Code  string `gorm:"index:idx_code_price"`
    Price int    `gorm:"index:idx_code_price"`
}

// 自定义外键
type User struct {
    gorm.Model
    Name        string
    CreditCards []CreditCard `gorm:"foreignKey:UserID"`
}

type CreditCard struct {
    gorm.Model
    Number string
    UserID uint
}
```

## 8. **日志和调试**

```go
// 配置日志级别
db, err := gorm.Open(sqlserver.Open(dsn), &gorm.Config{
    Logger: logger.Default.LogMode(logger.Info), // 显示所有 SQL
})

// 自定义日志
newLogger := logger.New(
    log.New(os.Stdout, "\r\n", log.LstdFlags), // io writer
    logger.Config{
        SlowThreshold:             time.Second,  // 慢 SQL 阈值
        LogLevel:                  logger.Info,  // 日志级别
        IgnoreRecordNotFoundError: true,         // 忽略 ErrRecordNotFound 错误
        Colorful:                  true,         // 启用颜色
    },
)

db, err := gorm.Open(sqlserver.Open(dsn), &gorm.Config{
    Logger: newLogger,
})

// 调试单个操作
db.Debug().Where("name = ?", "张三").Find(&users)
```

## 9. **迁移功能**

```go
// 自动迁移（仅创建表、添加字段和索引，不会删除列）
db.AutoMigrate(&User{}, &Product{}, &Order{})

// 检查表是否存在
db.Migrator().HasTable(&User{})

// 创建表
db.Migrator().CreateTable(&User{})

// 删除表
db.Migrator().DropTable(&User{})

// 重命名表
db.Migrator().RenameTable(&User{}, &Customer{})

// 添加索引
db.Migrator().CreateIndex(&User{}, "Name")

// 添加列
db.Migrator().AddColumn(&User{}, "Age")

// 删除列
db.Migrator().DropColumn(&User{}, "Age")
```

## 10. **作用域**

```go
// 定义作用域
func ActiveUser(db *gorm.DB) *gorm.DB {
    return db.Where("active = ?", true)
}

func AgeGreaterThan(age int) func(db *gorm.DB) *gorm.DB {
    return func(db *gorm.DB) *gorm.DB {
        return db.Where("age > ?", age)
    }
}

// 使用作用域
db.Scopes(ActiveUser, AgeGreaterThan(18)).Find(&users)
```

## 11. **性能优化**

```go
// 预编译 SQL
db, err := gorm.Open(sqlserver.Open(dsn), &gorm.Config{
    PrepareStmt: true, // 预编译 SQL
})

// 禁用默认事务
db, err := gorm.Open(sqlserver.Open(dsn), &gorm.Config{
    SkipDefaultTransaction: true,
})

// 批量插入优化
db.CreateInBatches(users, 100)

// 只查询需要的字段
db.Select("id", "name").Find(&users)

// 使用 Pluck 获取单列
var names []string
db.Model(&User{}).Pluck("name", &names)
```

## 12. **插件系统**

```go
// 使用插件
import "gorm.io/plugin/soft_delete"

type User struct {
    ID        uint
    Name      string
    DeletedAt soft_delete.DeletedAt `gorm:"uniqueIndex:idx_deleted_at"`
}

// 使用乐观锁
import "gorm.io/plugin/optimisticlock"

type User struct {
    ID    int
    Name  string
    Lock  optimisticlock.Version
}
```

## 完整的配置示例

```go
package main

import (
    "gorm.io/driver/sqlserver"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
    "time"
)

func SetupGORM() (*gorm.DB, error) {
    dsn := "sqlserver://sa:YourPassword123@localhost:1433?database=testdb"

    // GORM 配置
    config := &gorm.Config{
        // 跳过默认事务（提高性能）
        SkipDefaultTransaction: true,
        // 预编译 SQL
        PrepareStmt: true,
        // 日志配置
        Logger: logger.Default.LogMode(logger.Info),
        // 命名策略
        NamingStrategy: schema.NamingStrategy{
            TablePrefix:   "t_",   // 表前缀
            SingularTable: false,  // 复数表名
        },
        // 禁用外键约束
        DisableForeignKeyConstraintWhenMigrating: true,
    }

    db, err := gorm.Open(sqlserver.Open(dsn), config)
    if err != nil {
        return nil, err
    }

    // 连接池配置
    sqlDB, err := db.DB()
    if err != nil {
        return nil, err
    }

    sqlDB.SetMaxIdleConns(10)
    sqlDB.SetMaxOpenConns(100)
    sqlDB.SetConnMaxLifetime(time.Hour)

    return db, nil
}
```

GORM 的这些功能足以应对绝大多数应用场景。你准备开始使用 GORM 了吗？我可以帮你写具体的业务代码。
