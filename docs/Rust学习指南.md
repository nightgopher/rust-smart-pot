# 🦀 Rust 学习指南（Java / Go 背景版）

> 本文档专为**有 Java 和 Go 经验**的你量身定制，只讲本项目（智能花盆后端）真正用得到的部分，不讲废话。

---

## 一、先认清你的优势：你能迁移什么

你有 Java 和 Go 背景，这是学 Rust 的**极好起点**。很多概念你能直接平移：

| 概念 | Java | Go | Rust（本项目） |
|------|------|----|--------------|
| 接口/抽象 | `interface` | `interface`（隐式实现） | `trait`（显式实现）✅ 你 Go 的 interface 思维能无缝迁移 |
| 错误处理 | `try/catch` | `if err != nil` | `Result<T, E>` ✅ Go 的显式错误思维几乎一样 |
| 并发单元 | `Thread` | `goroutine` | `tokio::spawn`（异步 task）✅ |
| 通道 | `BlockingQueue` | `channel` | `tokio::sync::mpsc` ✅ Go channel 思维直接迁移 |
| 互斥锁 | `synchronized` / `ReentrantLock` | `sync.Mutex` | `std::sync::Mutex` ✅ |
| 泛型 | `<T>` | `[]T`（有限） | `<T>`（更强大，类似 Java）✅ |
| 集合 | `ArrayList` / `HashMap` | `slice` / `map` | `Vec<T>` / `HashMap<K,V>` ✅ |
| 没有 null | （Optional） | （需检查 nil） | `Option<T>` ✅ |

**结论：你的 Go 经验尤其值钱**——Go 的 `if err != nil`、channel、interface 思维，和 Rust 高度同构。你可以把 Rust 当成「带编译期内存安全检查的、更强的 Go」。

---

## 二、必须重点突破的唯一难关：所有权系统

这是 Java/Go 开发者学 Rust **唯一的硬骨头**，也是 Rust 的灵魂。因为 Java 和 Go 都有 GC（垃圾回收），你从没操心过"这块内存归谁管"。Rust 没有 GC，靠**所有权（Ownership）**在编译期管理内存。

### 核心三件套：所有权 / 借用 / 生命周期

#### 1. 所有权（Ownership）—— 每个值只有一个主人

```rust
let s1 = String::from("薄荷");  // s1 拥有这个 String
let s2 = s1;                     // 所有权转移给 s2，s1 失效！

// println!("{}", s1);  // ❌ 编译错误：s1 已经被 move 了
println!("{}", s2);     // ✅ 只有 s2 能用
```

> 🤯 **Java/Go 开发者的震撼**：在 Java/Go 里 `s2 = s1` 只是复制引用，两个都能用。Rust 里这是**移动（move）**，原变量直接失效。

**什么时候会 move，什么时候是复制？**
- 基本类型（`i32`, `f64`, `bool`）→ **复制**（实现了 `Copy` trait，像 Java 的值类型）
- `String`, `Vec`, `HashMap`, `struct`（默认）→ **移动**

#### 2. 借用（Borrowing）—— 不转移所有权，只是"借来用"

不想交出所有权怎么办？**借引用**：

```rust
let s1 = String::from("薄荷");
let len = calculate_length(&s1);  // & 表示借用（只读引用）
println!("'{}' 长度 {}", s1, len); // ✅ s1 还能用

fn calculate_length(s: &String) -> usize {  // 参数是 &String（引用）
    s.len()
}
```

**借用规则（编译器强制）：**
- 任意时刻，要么有**多个不可变引用** `&T`，要么有**唯一一个可变引用** `&mut T`，不能同时存在。
- 这条规则在编译期消灭了数据竞争——**你再也不用调试并发 bug 了**。

```rust
let mut s = String::from("hi");
let r1 = &s;        // 不可变借用
let r2 = &s;        // 不可变借用（多个 OK）
let r3 = &mut s;    // ❌ 编译错误：已有不可变借用，不能再可变借用
```

#### 3. 生命周期（Lifetime）—— 引用不能活得比数据久

大多数时候编译器能自动推断。只有**函数返回引用**时才需要手动标注 `<'a>`：

```rust
// 'a 表示返回的引用和传入的引用活得一样久
fn longest<'a>(s1: &'a str, s2: &'a str) -> &'a str {
    if s1.len() > s2.len() { s1 } else { s2 }
}
```

> 💡 **本项目里你几乎不用手写生命周期标注**，99% 的情况编译器自动推断。先理解概念即可，遇到编译器报错再查。

### 学这一章的预期

- **第一遍一定懵**——这很正常，因为它是全新范式。
- 建议：**动手写，让编译器教你**。Rust 编译器的错误提示是所有语言里最好的，它会精确告诉你哪里违反了所有权规则、怎么改。

---

## 三、思维转换对照表（Java / Go → Rust）

这是你最需要建立的新直觉：

| 概念 | Java/Go 的做法 | Rust 的做法 | 转换提示 |
|------|---------------|------------|---------|
| **变量默认值** | 默认可变 | **默认不可变**，加 `mut` 才能改 | 养成写 `let mut` 的习惯 |
| **空值** | `null` / `nil` | **没有 null**，用 `Option<T>` | `Some(v)` / `None`，强制你处理空的情况 |
| **错误** | 异常 / `err` | `Result<T, E>`，**没有 try-catch** | `Ok(v)` / `Err(e)`，用 `?` 简化传播 |
| **继承** | class 继承 class | **没有继承**，用 `trait` + 组合 | 用 trait 定义行为，struct 组合数据 |
| **类** | `class` | `struct`（数据）+ `impl`（方法） | 数据和行为是分开写的 |
| **内存管理** | GC 自动回收 | 所有权在编译期决定 | 移动 vs 借用要想清楚 |
| **共享可变状态** | 锁 / channel | 锁 / channel，但**编译器强制安全** | 思维一样，只是编译器帮你查错 |

---

## 四、本项目会用到的 Rust 特性（带着场景学）

下面每个特性都标注了**在本项目哪里用到**，让你学的时候有具体目标。

### 🔴 必学（不学写不动）

#### 1. 所有权 / 借用 / 引用
- **项目场景**：传感器数据从采集函数传到判断函数、存进数据库，全程要考虑 move 还是 borrow。
- **学多久**：3~5 天，反复练。

#### 2. `Result<T, E>` 和 `?` 操作符
- **项目场景**：读传感器可能失败、数据库查询可能失败、HTTP 请求可能失败——全返回 `Result`，用 `?` 向上传播错误。
- **你的优势**：Go 的 `if err != nil` 思维直接迁移。Rust 的 `?` 就是 `if err != nil { return err }` 的语法糖。

```rust
// Rust 的 ? 等价于 Go 的：
// data, err := read_sensor()
// if err != nil { return err }

fn read_and_parse() -> Result<f64, SensorError> {
    let raw = read_sensor()?;           // 失败则自动 return Err
    let value = parse(raw)?;            // 失败则自动 return Err
    Ok(value)
}
```

#### 3. `Option<T>`
- **项目场景**：查询数据库可能查不到记录，返回 `Option<Plant>`。没有 null，你被迫处理"没有"的情况。
- **你的优势**：Java 8 的 `Optional` 一模一样。

#### 4. `enum` + `match`（模式匹配）
- **项目场景**：浇水状态机 `enum State { Idle, Watering, Cooldown }`，用 `match` 分发。

```rust
enum WaterState { Idle, Watering, Cooldown(Instant) }

match state {
    WaterState::Idle => check_moisture(),
    WaterState::Watering => monitor_pump(),
    WaterState::Cooldown(since) => check_cooldown_done(since),
}
```

#### 5. `struct` + `impl`
- **项目场景**：定义 `struct SensorReading { moisture: f64, temp: f64, ... }`，给它写方法 `impl SensorReading { fn is_dry(&self) -> bool {...} }`。
- **你的优势**：就是把 Java 的 class 拆成 struct（字段）+ impl（方法）。

#### 6. `trait`（接口）
- **项目场景**：定义多种浇水策略（薄荷/多肉不同阈值）。
- **你的优势**：Go 的 interface 思维完美迁移，区别是 Rust 要**显式写 `impl Trait for Type`**。

```rust
// 就像 Go 的 interface，定义行为契约
trait WaterStrategy {
    fn should_water(&self, moisture: f64) -> bool;
    fn max_duration(&self) -> Duration;
}

struct MintStrategy;      // 薄荷策略（喜湿）
struct SucculentStrategy; // 多肉策略（耐旱）

impl WaterStrategy for MintStrategy {
    fn should_water(&self, moisture: f64) -> bool { moisture < 30.0 }
    fn max_duration(&self) -> Duration { Duration::from_secs(60) }
}
```

### 🟡 项目必需（边用边学）

#### 7. `Vec<T>` 和 `HashMap<K,V>`
- **项目场景**：`Vec<WateringRecord>` 存浇水历史，`HashMap<DeviceId, Device>` 管理多花盆。
- **你的优势**：就是 Java 的 `ArrayList` 和 `HashMap`。

#### 8. 闭包（Closure / Lambda）
- **项目场景**：传感器读数去极值、过滤、排序。
- **你的优势**：Java 的 `() -> {}`、Go 的 `func() {}`，语法是 `|x| { ... }`。

```rust
let readings = vec![30.0, 31.0, 999.0, 29.0, 30.0]; // 999 是异常值
let filtered: Vec<f64> = readings.into_iter()
    .filter(|&x| x < 100.0)   // 闭包：过滤异常
    .collect();
```

#### 9. 错误定义（自定义 Error 类型）
- **项目场景**：定义 `SensorError`, `DbError` 等。
- **推荐**：用 `thiserror` 库（派生宏自动实现），几行搞定。

### 🟢 异步生态（搭后端时跟着学）

#### 10. `tokio`（异步运行时）
- **类比 Go**：`tokio` 之于 Rust，就像 runtime 之于 Go。`#[tokio::main]` 相当于启动了 Go 的调度器。
- `tokio::spawn` ≈ `go func()`。
- `tokio::sync::mpsc` ≈ Go 的 channel。

```rust
#[tokio::main]
async fn main() {
    // 像 go spawn 一个任务
    tokio::spawn(async {
        loop {
            check_moisture().await;
            tokio::time::sleep(Duration::from_secs(60)).await;
        }
    });
}
```

#### 11. `serde`（JSON 序列化）
- **类比 Java**：就是 Jackson / Gson。加 `#[derive(Serialize, Deserialize)]` 就能自动转 JSON。
- **项目场景**：设备上报的传感器数据 ↔ JSON ↔ Rust struct。

#### 12. `axum`（Web 框架）
- **类比 Go**：类似 Gin / Echo。用 trait 做中间件，类型安全。
- **类比 Java**：类似 Spring MVC 但更轻量。
- **项目场景**：`POST /api/report`（设备上报）、`POST /api/water`（远程浇水）。

---

## 五、不用学的（本项目暂时用不到）

为了让你聚焦，这些**现阶段可以跳过**，需要时再查：

- ❌ 高级生命周期标注（`'a`, `'static` 的复杂场景）——99% 自动推断
- ❌ 不安全代码（`unsafe`）——写应用层用不到
- ❌ 过程宏（proc-macro）开发——只用现成的，不用自己写
- ❌ 宏规则（`macro_rules!`）——看懂 `println!` / `vec!` 即可
- ❌ 复杂的 trait bound（`where T: A + B + C`）——项目规模用不到
- ❌ 零拷贝 / 内联汇编 / FFI 高级用法

---

## 六、推荐学习资源

### 📖 主教程（选一个跟到底）
| 资源 | 适合 | 链接 |
|------|------|------|
| **Rust 程序设计语言**（官方书中文版） | 系统学，推荐通读前 10 章 | https://kaisery.github.io/trpl-zh-cn/ |
| **Rustlings**（互动练习） | 喜欢边敲边学，强烈推荐 | https://github.com/rust-lang/rustlings |
| **Rust by Example**（通过例子学） | 喜欢看代码不喜欢看大段文字 | https://rustwiki.org/zh-CN/rust-by-example/ |

### 🎬 视频辅助
- B 站搜「Rust 入门」，挑一个播放量高的跟着过一遍语法

### 🛠️ 查文档
- **Rust 标准库文档**：https://doc.rust-lang.org/std/（查 API 必备）
- **crate 搜索**：https://crates.io/（找第三方库）

### 🤝 遇到问题
- Rust 编译器的报错信息是最好的老师，**读懂报错 = 学会一半**
- https://rustwiki.org/zh-CN/rust-by-example/ 查用法

---

## 七、学习节奏建议（针对本项目）

```
第 1 周：基础语法
├─ 所有权 / 借用 / 引用（重点！多花时间）
├─ enum / match / Option / Result
├─ struct / impl
└─ trait（你的 Go 经验让你很快上手）

第 2 周：标准库 + 错误处理
├─ Vec / HashMap / 字符串
├─ 自定义 Error + ? 操作符
└─ 闭包 / 迭代器

第 3 周：搭后端骨架时边查边学
├─ tokio 异步（类比 goroutine）
├─ serde（类比 Jackson）
└─ axum（类比 Gin）

之后：在项目实战中加深，遇到不懂的回来查
```

> 💡 **核心心态**：不要试图先把 Rust 学"完"再动手。学到 trait 那章就可以开始参与写后端了，剩下的在实战中补。Rust 编译器会手把手教你纠错。

---

## 八、一张速查图：本项目概念映射

```
Java/Go 概念          →    Rust 对应              →    项目里的应用
─────────────────────────────────────────────────────────────────
class                 →    struct + impl          →    SensorReading, Device
interface             →    trait                  →    WaterStrategy
try-catch / if err    →    Result + ?             →    所有可能失败的操作
Optional              →    Option                 →    数据库查询结果
switch                →    match                  →    浇水状态机
ArrayList             →    Vec<T>                 →    浇水历史记录
HashMap               →    HashMap<K,V>           →    多花盆管理
goroutine             →    tokio::spawn           →    定时采集任务
channel               →    tokio::sync::mpsc      →    任务间通信
GC                    →    所有权系统（编译期）     →    （需要重新建立直觉）
null                  →    不存在，用 Option      →    消除空指针
```

---

学完基础语法（前两周内容）后，回来找我搭后端骨架。到时候我会边写边给你讲每段代码用到的 Rust 概念，保证你不是在看天书。🦀
