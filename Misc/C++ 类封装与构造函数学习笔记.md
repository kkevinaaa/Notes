# 1 基于 `TemperatureController` 类的代码分析

```cpp
#include <stdexcept>

class TemperatureController {
private:
    double currentTemp;

public:
    TemperatureController(double initialTemp) {
        if (initialTemp < -50.0 || initialTemp > 100.0) {
            throw std::invalid_argument("温度超出合理范围");
        }
        currentTemp = initialTemp;
    }

    double getTemperature() const {
        return currentTemp;
    }

    void setTemperature(double newTemp) {
        if (newTemp < -50.0 || newTemp > 100.0) {
            throw std::invalid_argument("温度只能在 -50 到 100 之间");
        }
        currentTemp = newTemp;
    }

    void heat(double amount) {
        setTemperature(currentTemp + amount);
    }

    void cool(double amount) {
        setTemperature(currentTemp - amount);
    }
};
```

---

## 1.1 成员函数后面的 `const` 限定符

成员函数声明末尾的 `const` 关键字表示该函数为**只读函数**。

- 该函数承诺不会修改类的任何非静态成员变量。
- 如果在该函数体内试图修改 `currentTemp`，编译器会报错。
- 该限定符允许该函数被 `const` 类型的对象调用。非 `const` 成员函数无法被 `const` 对象调用。

```cpp
const TemperatureController ctrl(25.0);
double t = ctrl.getTemperature();  // 合法，因为 getTemperature 是 const 成员函数
// ctrl.setTemperature(30.0);      // 非法，setTemperature 不是 const 成员函数
```

---

## 1.2 访问控制与 `class` / `struct` 的区别

`class` 中的成员默认访问权限为 `private`。如果省略 `public:` 标签，所有成员（包括构造函数、成员函数）均对外部不可见，无法进行任何有意义的交互。

| 关键字 | 默认访问权限 |
| :--- | :--- |
| `class` | `private`（外部不可访问） |
| `struct` | `public`（外部可直接访问） |

```cpp
// 使用 struct，默认公开，外部可直接修改数据成员，破坏封装性
struct TemperatureControllerStruct {
    double currentTemp;  // 默认 public，外部可以直接读写
};

// 使用 class，默认私有，必须通过 public 接口访问
class TemperatureControllerClass {
    double currentTemp;  // 默认 private，外部无法直接访问
};
```

建议：当需要封装数据并提供接口时，使用 `class`；当仅作为简单数据容器时，可以使用 `struct`。

---

## 1.3 构造函数（Constructor）

构造函数是一种特殊的成员函数，用于初始化类的对象。它的特性如下：

- 函数名与类名完全相同。
- **没有返回值**，也不能写 `void`。这是语法硬性规定，不是编码风格。
- 在声明该类的对象时，构造函数被自动调用。

```cpp
TemperatureController a(25.0);  // 自动调用构造函数，传入 25.0
```

如果代码中定义了一个或多个带参数的构造函数，编译器将不再生成默认的无参构造函数。

```cpp
class Example {
public:
    Example(int x) { }  // 定义了带参构造函数
};

// Example e;   // 错误：没有合适的默认构造函数（无参构造不存在）
Example e(10);  // 正确，必须传入参数
```

---

## 1.4 异常抛出（`throw`）与 `std::invalid_argument`

`std::invalid_argument` 是 C++ 标准库（头文件 `<stdexcept>`）中预定义的一个异常类，专门用于表示“传入的参数无效”这种错误情形。

语法拆解：

```cpp
throw std::invalid_argument("温度超出合理范围");
```

- `throw` 是 C++ 关键字，用于抛出异常，中断当前程序流程。
- `std::invalid_argument` 是标准库定义的类。
- `("温度超出合理范围")` 是传递给该异常类构造函数的字符串，外部捕获后可通过 `e.what()` 方法获取该字符串。

构造函数无法通过返回值报告错误（因为它没有返回值），因此 `throw` 是其报告初始化失败、防止生成非法对象的主要手段。

```cpp
try {
    TemperatureController tc(-1000);
} catch (const std::invalid_argument& e) {
    // 输出："温度超出合理范围"
    std::cout << e.what() << std::endl;
}
```

---

## 1.5 默认初始值的设置与构造函数的省略

如果类不需要外界传入初始值，而是使用内部固定的默认值，可以避免显式编写构造函数。推荐做法是在声明成员变量时直接赋予默认值（C++11 及以后标准支持）。

```cpp
class TemperatureController {
private:
    double currentTemp = 25.0;  // 默认值为 25 度

public:
    // 无需显式编写构造函数，编译器生成默认构造，
    // 该默认构造会将 currentTemp 初始化为 25.0

    double getTemperature() const { return currentTemp; }
    void setTemperature(double newTemp) {
        if (newTemp < -50.0 || newTemp > 100.0) {
            throw std::invalid_argument("温度超范围");
        }
        currentTemp = newTemp;
    }
    // heat、cool 函数略...
};
```

此时在外部声明对象：

```cpp
TemperatureController a;  // a.currentTemp 被初始化为 25.0
```

**两种方案的选择原则：**

- 需要外部传入初始参数：编写带参数的构造函数。
- 使用内部固定默认值：在成员变量声明处直接赋值，省略构造函数（或显式编写 `= default` 以强调意图）。
- 需要同时支持两种方式：使用函数重载，同时提供带参和无参构造函数。

---

# 2 基于 `Lifecycle` 类的完整代码：类设计、内存与函数参数传递

```cpp
#pragma once

#include <stdint.h>

enum class AppState : uint8_t {
    Clock,
    Countdown,
};

enum class FaceId : uint8_t {
    None = 0,
    Face1 = 1,
    Face2 = 2,
    Face3 = 3,
    Face4 = 4,
};

struct DeviceConfig {
    uint32_t faceDurationMs[4] = {
        5UL * 60UL * 1000UL,
        10UL * 60UL * 1000UL,
        25UL * 60UL * 1000UL,
        60UL * 60UL * 1000UL,
    };
};

struct Effects {
    bool countdownStarted = false;
    uint32_t durationMs = 0;
};

class Lifecycle {
public:
    explicit Lifecycle(DeviceConfig config);

    Effects onStableFace(FaceId face);

    AppState state() const;
    uint32_t remainingMs() const;

private:
    uint32_t durationFor(FaceId face) const;

    DeviceConfig config_;
    AppState state_ = AppState::Clock;
    FaceId activeFace_ = FaceId::None;
    uint32_t remainingMs_ = 0;
};
```

---

## 2.1 类的设计意图与封装结构

### 2.1.1 公有接口（外部可见）
- `explicit Lifecycle(DeviceConfig config);`：构造函数，传入配置。
- `Effects onStableFace(FaceId face);`：核心业务函数。当检测到朝上的侧面稳定时调用，返回需要执行的效果指令。
- `AppState state() const;`：查询当前状态（时钟/倒计时）。
- `uint32_t remainingMs() const;`：查询剩余毫秒数。

### 2.1.2 私有成员（外部不可见）
- `uint32_t durationFor(FaceId face) const;`：辅助查询函数，根据侧面编号查配置时长。
- `DeviceConfig config_;`：存储配置。
- `AppState state_ = AppState::Clock;`：当前状态，默认时钟。
- `FaceId activeFace_ = FaceId::None;`：当前活跃的侧面。
- `uint32_t remainingMs_ = 0;`：剩余时间。

### 2.1.3 命名规范：成员变量后缀下划线 `_`
所有私有成员变量末尾带有下划线（如 `config_`）。这是一种广泛使用的编码约定，用于在代码中快速区分成员变量和局部变量（如函数参数 `FaceId face` 与成员 `activeFace_`）。

### 2.1.4 成员变量的就地初始化

```cpp
AppState state_ = AppState::Clock;
FaceId activeFace_ = FaceId::None;
uint32_t remainingMs_ = 0;
```

这些赋值在**编译阶段**就被记录，当对象被创建时，这些成员会首先被赋予这些默认值。如果构造函数（或初始化列表）没有显式覆盖它们，它们就使用这里指定的值。

---

## 2.2 `explicit` 关键字的作用

```cpp
explicit Lifecycle(DeviceConfig config);
```

- 构造函数默认可以被用于**隐式类型转换**（比如编译器可能会把 `DeviceConfig` 偷偷变成 `Lifecycle`）。
- 加上 `explicit` 后，**禁止**这种自动转换。你必须显式地写 `Lifecycle lc(config);` 来创建对象。主要目的是防止编译器自作聪明地转换，避免产生难以追踪的 bug。

---

## 2.3 核心难点：函数参数传递方式（值传递、引用传递、指针传递）

**假设场景**：外部有一个定义好的 `DeviceConfig myConfig;`，现在要把它传给构造函数。

### 2.3.1 值传递（代码中的写法）：`DeviceConfig config`

```cpp
explicit Lifecycle(DeviceConfig config) : config_(config) {}
```

- **内存行为**：在函数被调用时，编译器会在内存的**新位置**完整复制一份 `myConfig` 的所有数据（16 字节）。这个副本就是函数形参 `config`。
- **操作权限**：函数内部修改 `config` 不会影响外部的 `myConfig`，因为是两份独立的数据。
- **存储去向**：通过初始化列表 `: config_(config)`，把这份副本再次复制到成员变量 `config_` 中。总共发生了两次复制（外部 -> 形参，形参 -> 成员）。
- **适用场景**：数据体积很小（如小于 16 字节），且不需要修改外部原件时。

### 2.3.2 引用传递（如果改成这样）：`const DeviceConfig& config`

```cpp
explicit Lifecycle(const DeviceConfig& config) : config_(config) {}
```

- **内存行为**：**不发生复制**。`config` 只是一个“别名”，它直接指向外部 `myConfig` 所在的内存地址。传递的实质是一个内存地址（通常占用 4 或 8 字节）。
- **操作权限**：因为加了 `const` 修饰，函数内部只能读取 `myConfig` 的数据，不能修改。
- **存储去向**：`: config_(config)` 将外部 `myConfig` 的数据复制一份存到成员变量 `config_` 中。整个过程中**只发生了一次复制**（外部 -> 成员）。
- **适用场景**：数据体积较大，或者希望避免复制开销，且不需要修改外部数据。**这是 C++ 中最推荐使用的参数传递方式之一（只读大对象）。**

### 2.3.3 指针传递（如果改成这样）：`DeviceConfig* config`

```cpp
explicit Lifecycle(DeviceConfig* config) : config_(*config) {}
```

- **内存行为**：传递的是外部变量 `myConfig` 的内存地址（门牌号）。同样**不发生复制**。
- **操作权限**：因为没有 `const` 限制，函数内部可以通过 `config->faceDurationMs[0] = 0;` 直接修改外部的 `myConfig`。
- **指针的特殊性**：
  - 指针可以为空（`nullptr`），引用不能为空。
  - 指针可以在函数内部被重新赋值，让它指向别的地址，引用则不能。
- **存储去向**：初始化列表中必须用 `*config`（解引用）取出真正的数据，才能赋值给 `config_`。

---

## 2.4 为什么代码中没写 `config_ = config;` 也能运行？（初始化列表的关键作用）

假设构造函数体为空，只有函数签名：

```cpp
explicit Lifecycle(DeviceConfig config) {
    // 空函数体
}
```

此时，如果不使用 `: config_(config)` 初始化列表，成员变量 `config_` 会使用在 `struct DeviceConfig` 内部定义的默认值（数组填充了 5分钟、10分钟等）。**外部传入的 `config` 参数虽然被复制了一份，但完全没有被使用，直接丢失了。** 这是一个非常隐蔽的逻辑 bug。

**正确的做法**，也是业界标准，必须使用 **初始化列表**：

```cpp
explicit Lifecycle(DeviceConfig config) : config_(config) {
    // 此处在对象成员分配内存的瞬间，直接将参数 config 的值搬进 config_
}
```

**进一步优化建议**：既然 `DeviceConfig` 有 16 字节，复制两次（外部 -> 形参，形参 -> 成员）虽然可以接受，但如果追求极致效率，**更推荐直接使用 `const` 引用传递**，只复制一次：

```cpp
explicit Lifecycle(const DeviceConfig& config) : config_(config) {}
```

---

## 2.5 总结速查表

| 传递方式       | 写法                       | 复制数据？     | 可否修改外部原件？     | 可否为空？          | 推荐使用场景                         |
| :--------- | :----------------------- | :-------- | :------------ | :------------- | :----------------------------- |
| **值传递**    | `T func(T param)`        | 是（完整复制）   | 否（改的是副本）      | 否（必须合法对象）      | 基础类型（int, char）或很小的结构体（< 16字节） |
| **常量引用传递** | `T func(const T& param)` | 否（直接访问原件） | 否（const 保护）   | 否（必须绑定合法对象）    | **大型对象、字符串、容器等只读场景（首选）**       |
| **引用传递**   | `T func(T& param)`       | 否（直接访问原件） | 是             | 否              | 需要修改外部变量的场景                    |
| **指针传递**   | `T func(T* param)`       | 否（传地址）    | 是（通过 `->` 操作） | 是（可以传 nullptr） | 需要表达“可选”或“可为空”的入参，或 C 风格接口兼容   |
