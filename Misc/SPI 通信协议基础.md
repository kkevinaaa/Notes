## 1 代码整体结构

```C++
#include <Arduino.h>
#include <SPI.h>

constexpr uint8_t PIN_AD5941_SCLK  = 4;
constexpr uint8_t PIN_AD5941_MISO  = 5;
constexpr uint8_t PIN_AD5941_MOSI  = 6;
constexpr uint8_t PIN_AD5941_CS    = 7;
constexpr uint8_t PIN_AD5941_RESET = 3;

constexpr uint32_t AD5941_SPI_HZ = 500000;
constexpr uint8_t SPICMD_SETADDR = 0x20;
constexpr uint8_t SPICMD_READREG = 0x6D;
constexpr uint16_t REG_ADIID = 0x0400;
constexpr uint16_t REG_CHIPID = 0x0404;

SPISettings ad5941SpiSettings(AD5941_SPI_HZ, MSBFIRST, SPI_MODE0);

void ad5941HardwareReset() {
  digitalWrite(PIN_AD5941_RESET, HIGH);
  delay(1);
  digitalWrite(PIN_AD5941_RESET, LOW);
  delay(2);  // /RESET 低电平保持至少 1 ms
  digitalWrite(PIN_AD5941_RESET, HIGH);
  delay(10); // 等待芯片复位完成后再访问 SPI
}

uint16_t ad5941ReadReg16(uint16_t address) {
  // 第一次 transaction：告诉 AD5941 接下来要访问哪个寄存器。
  SPI.beginTransaction(ad5941SpiSettings);
  digitalWrite(PIN_AD5941_CS, LOW);
  SPI.transfer(SPICMD_SETADDR);
  SPI.transfer(static_cast<uint8_t>(address >> 8));
  SPI.transfer(static_cast<uint8_t>(address));
  digitalWrite(PIN_AD5941_CS, HIGH);
  SPI.endTransaction();

  delayMicroseconds(1); // 满足两次 CS 选通之间的最小间隔

  // 第二次 transaction：发送读命令和 dummy byte，再提供时钟读回 16 位数据。
  SPI.beginTransaction(ad5941SpiSettings);
  digitalWrite(PIN_AD5941_CS, LOW);
  SPI.transfer(SPICMD_READREG);
  SPI.transfer(0x00); // dummy byte：启动 AD5941 的寄存器回读
  const uint8_t highByte = SPI.transfer(0x00); // 先返回高字节
  const uint8_t lowByte  = SPI.transfer(0x00); // 再返回低字节
  digitalWrite(PIN_AD5941_CS, HIGH);
  SPI.endTransaction();

  return (static_cast<uint16_t>(highByte) << 8) | lowByte;
}

void printRegisterReads(const char *name, uint16_t address) {
  for (uint8_t i = 1; i <= 5; ++i) {
    const uint16_t value = ad5941ReadReg16(address);
    Serial.printf("%s [%u] = 0x%04X\n", name, i, value);
    delay(20);
  }
}

void runIdentityTest() {
  Serial.println("\n--- AD5941 identity read ---");
  printRegisterReads("ADIID ", REG_ADIID);
  printRegisterReads("CHIPID", REG_CHIPID);
  Serial.println("Expected ADIID: 0x4144");
}

void setup() {
  // 尽早让片选和复位脚进入非激活状态；板载上拉负责程序接管前的 /RESET 电平。
  pinMode(PIN_AD5941_CS, OUTPUT);
  digitalWrite(PIN_AD5941_CS, HIGH);
  pinMode(PIN_AD5941_RESET, OUTPUT);
  digitalWrite(PIN_AD5941_RESET, HIGH);

  Serial.begin(115200);
  delay(1500); // 给 Upload and Monitor 留出连接串口的时间

  Serial.println("\nAD5941 minimal SPI identity test");
  Serial.printf("SCLK=%u, MISO=%u, MOSI=%u, CS=%u, RESET=%u\n",
                PIN_AD5941_SCLK, PIN_AD5941_MISO, PIN_AD5941_MOSI,
                PIN_AD5941_CS, PIN_AD5941_RESET);
  Serial.printf("SPI: Mode 0, MSB first, %lu Hz\n",
                static_cast<unsigned long>(AD5941_SPI_HZ));

  SPI.begin(PIN_AD5941_SCLK, PIN_AD5941_MISO,
            PIN_AD5941_MOSI, PIN_AD5941_CS);
  ad5941HardwareReset();

  runIdentityTest();
}

void loop() {
  delay(3000);
  Serial.println("ESP32-C3 alive; repeating read...");
  runIdentityTest();
}
```

程序为一个针对 AD5941 模拟前端芯片的最小 SPI 通信测试。代码初始化 ESP32-C3 的 SPI 外设，复位 AD5941，并连续读取其身份寄存器（ADIID、CHIPID）以验证通信是否正常。整个程序分为 `setup()` 与 `loop()`：前者完成一次性配置并执行首次读取，后者每 3 秒重复读取，用于观察通信稳定性。

## 2 常量与整数类型

```cpp
constexpr uint8_t PIN_AD5941_SCLK  = 4;
constexpr uint32_t AD5941_SPI_HZ = 500000;
constexpr uint8_t SPICMD_SETADDR = 0x20;
constexpr uint16_t REG_ADIID = 0x0400;
```

`constexpr` 表示编译期常量，值在编译时确定，不能被修改。使用它可提高可读性并让编译器进行优化。`uint8_t`、`uint16_t`、`uint32_t` 是固定宽度的无符号整数类型，分别占 8、16、32 位，来自 `<stdint.h>`。在嵌入式编程中明确位宽很重要，因为寄存器地址、命令码、引脚编号等往往要求精确位数，避免不同平台 `int` 长度差异造成问题。

## 3 SPI 配置对象与事务

```cpp
SPISettings ad5941SpiSettings(AD5941_SPI_HZ, MSBFIRST, SPI_MODE0);
```

`SPISettings` 是 Arduino SPI 库定义的类，用于封装一次 SPI 通信所需的参数：时钟频率、位顺序、SPI 模式。此处传入 500 kHz、高位先发（MSBFIRST）、模式 0（CPOL=0，CPHA=0），与 AD5941 数据手册要求一致。该行在创建对象的同时调用构造函数完成初始化，之后将对象传递给 `SPI.beginTransaction()`，SPI 库据此配置硬件。此设计将相关参数打包，避免每次调用时重复书写，也方便管理多个不同配置的 SPI 设备。

实际通信使用事务模式：

```cpp
SPI.beginTransaction(ad5941SpiSettings);
digitalWrite(PIN_AD5941_CS, LOW);
// 传输数据
digitalWrite(PIN_AD5941_CS, HIGH);
SPI.endTransaction();
```

`beginTransaction()` 配置 SPI 硬件并屏蔽中断，防止多设备冲突；`endTransaction()` 恢复设置。CS 引脚需手动控制，低电平选中设备。

## 4 AD5941 寄存器读取时序

AD5941 的寄存器读操作需两次 CS 选通。第一次发送地址设置命令：

```cpp
SPI.transfer(SPICMD_SETADDR);            // 0x20
SPI.transfer(static_cast<uint8_t>(address >> 8));
SPI.transfer(static_cast<uint8_t>(address));
```

发送命令 `0x20` 后跟 16 位寄存器地址（高字节在前），随后拉高 CS 结束本次选通。`address >> 8` 取得高字节，低字节为 `address` 本身，显式转换为 `uint8_t` 避免符号扩展。两次选通之间加入 `delayMicroseconds(1)` 满足芯片要求的间隔。

第二次选通发送读命令并接收数据：

```cpp
SPI.transfer(SPICMD_READREG);   // 0x6D
SPI.transfer(0x00);             // dummy byte
uint8_t highByte = SPI.transfer(0x00);
uint8_t lowByte  = SPI.transfer(0x00);
```

SPI 为全双工，主机每发送一个字节同时收到一个字节。读命令后发送一个 dummy byte 提供时钟并丢弃无效数据；随后两次传输分别收到寄存器高字节与低字节。最后通过 `(highByte << 8) | lowByte` 合成 16 位值。

## 5 GPIO 初始化与硬件复位

```cpp
pinMode(PIN_AD5941_CS, OUTPUT);
digitalWrite(PIN_AD5941_CS, HIGH);
pinMode(PIN_AD5941_RESET, OUTPUT);
digitalWrite(PIN_AD5941_RESET, HIGH);
```

使用数字引脚作为输出前必须调用 `pinMode` 设置为 `OUTPUT`，否则引脚处于输入状态，无法提供可靠的推挽输出。CS 初始为高（片选无效），复位引脚初始为高（非复位状态）。复位函数：

```cpp
void ad5941HardwareReset() {
  digitalWrite(PIN_AD5941_RESET, HIGH);
  delay(1);
  digitalWrite(PIN_AD5941_RESET, LOW);
  delay(2);   // 低电平至少 1 ms
  digitalWrite(PIN_AD5941_RESET, HIGH);
  delay(10);  // 等待复位完成
}
```

通过拉低复位引脚 2 ms 再拉高，并等待 10 ms 确保芯片就绪。该时序依据数据手册，封装为函数便于复用。

## 6 格式化输出与程序流程

使用 ESP32 特有的 `Serial.printf` 进行格式化打印：

```cpp
Serial.printf("%s [%u] = 0x%04X\n", name, i, value);
```

格式说明：`%s` 字符串，`%u` 无符号十进制，`%04X` 宽度 4 的十六进制大写补零。相比 `Serial.print` 更简洁，适合调试输出。

`runIdentityTest()` 对两个寄存器各连续读取 5 次，每次间隔 20 ms，用于观察读取稳定性。`loop()` 每 3 秒重复调用，可判断通信是否随运行时间变化。

## 7 关键概念澄清

### 7.1 SPISettings 与配置容器

`SPISettings` 可视为强类型的配置对象，类似于将时钟频率、位顺序、模式打包在一起的结构体。创建时传入参数即完成内部保存，无需额外赋值。其设计目的是避免在每次 `beginTransaction` 时重复书写多个参数，提高可读性与安全性。

### 7.2 pinMode 的必要性

不调用 `pinMode` 而直接 `digitalWrite` 无法使引脚输出有效电平。未配置为输出的引脚默认为输入，`digitalWrite` 只会改变输出寄存器，但引脚不连接输出驱动，外部电压不受控制。写 `HIGH` 可能启用内部上拉，但驱动能力弱，不能可靠控制片选或复位信号。

### 7.3 SPI 频率与 UART 波特率的区别

SPI 是同步全双工通信，由主机提供时钟（SCLK），从机根据时钟收发数据，因此参数为**时钟频率**（Hz），而非波特率。一次 `SPI.transfer()` 发送与接收同时进行，不存在“一来一回”的间隔。相邻两次传输的间隔由代码执行速度和是否主动添加延迟决定，与 SPI 时钟频率无直接关系。UART 是异步通信，无时钟线，双方需约定相同波特率，如 `Serial.begin(115200)` 指定的是 ESP32 与电脑串口通信的波特率，与 SPI 频率无关。

>[!tips] SPI 时序的基本单位
> SPI 是同步通信，每个时钟周期传输 **1 个 bit**。一个字节（8 bit）需要 8 个时钟周期。如果 SPI 时钟频率为 500 kHz，则一个时钟周期为 2 µs，一个字节的传输时间为 16 µs。连续传输两个字节理论上是 16 个时钟周期，共 32 µs（在无任何额外间隔的情况下）。
> 时钟频率只决定每个 bit 的宽度（即每个时钟周期的时间），从而决定单个字节传输所需的总时间。但它不决定两个字节之间是否存在额外空闲时间。字节之间的额外空闲时间完全取决于：
> 软件何时发起下一次传输：CPU 执行到下一个 `SPI.transfer()` 需要时间。
> 是否主动插入延迟：如 `delayMicroseconds()`。

在 SPI 通信中，不会出现“代码跑得比数据传递速度快导致数据丢失”的问题。原因在于 `SPI.transfer()` 是**阻塞式**函数：当程序调用它时，CPU 会等待 SPI 硬件完成一个完整字节的发送和接收后才返回。也就是说，代码执行速度再快，也必须等这个字节按设定的时钟频率传输完毕，才能继续执行下一条指令。
