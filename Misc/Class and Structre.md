# C++ 类封装与构造函数学习笔记

基于 `TemperatureController` 类的代码分析，记录以下核心知识点。

---

## 原始代码

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

## 1. 成员函数后面的 `const` 限定符

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

## 2. 访问控制与 `class` / `struct` 的区别

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

## 3. 构造函数（Constructor）

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

## 4. 异常抛出（`throw`）与 `std::invalid_argument`

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

## 5. 默认初始值的设置与构造函数的省略

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
