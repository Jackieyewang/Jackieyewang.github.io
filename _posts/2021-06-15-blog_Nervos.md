---
layout: post
title: "Nervos"
date: 2021-06-15
description: "Nervos"
tag: none
---   
# [Nervos](https://github.com/nervosnetwork/rfcs)

[TOC]

## 概述

Nervos 是一个采用分层架构的加密经济网络，它将加密经济的基础设施分为两层：第一层验证层基础公链是CKB（common knowledge base） ，提供托管服务；生成层(Layer2) 提供高性能的交易以及隐私保护。
第1层（CKB）用于状态验证，而第2层负责状态生成。Cell模型代替btc的UTXO。

## CKB 公共知识库

CKB 是公链，Nervos 网络的价值存储层，为在网络中创建的资产、身份和其他公共知识提供公共、安全、不受审查的托管服务。

* 主要开发语言Rust
* 基于 Cell 模型来扩展 UTXO 模型并支持存储通用状态
* 基于 Cell 模型和 CKB-VM 实现智能合约，合约计算验证分离，计算发生在链下，验证则在链上
* CKB-VM 使用 RISC-V 指令集，智能合约支持Ruby，Python等
* Flatbuffer 实现数据结构的序列化
* 内存使用基于零拷贝


###  作用

1. 价值储存，网络的基础层，维护安全、去中心化和可拓展性
2. 验证和储存状态，验证链下的状态转换
3. 第一层和第二层目前可以通过侧链链接（layer2也可以通过状态通道，commit chain连接到layer 1）
4. 调整对生态系统中所有参与者的激励措施
5. NervosDAO代币价值储存
6. 缩小监管差距（可以发udt）


## 程序模型

CKB的程序模型与btc和eth

|          | Bitcoin | Ethereum | CKB     |
| -------- | ------- | -------- | ------- |
| 指令集   | Script  | EVM      | RISC-V  |
| 密码基元 | 操作码  | 预编译   | 汇编    |
| 有无状态 | 无      | 有       | 有      |
| 状态类型 | Ledger  | General  | General |
| 状态模型 | UTXO    | 账户模型 | Cell    |
| 状态验证 | 链上    | 链上     | 链上    |
| 状态生成 | 链下    | 链上     | 链下    |

CKB的编程模型主要包括：

+ 状态生成（链下）
+ 状态验证（CKB VM）
+ 状态存储（cell模型）


### 状态生成与验证

Nervos的状态生成和验证是分开的，可以通过不同的算法进行状态的生成和验证

* **链下** 在客户端根据输入生成新状态，打包广播到网络。
* **链上** CKB节点运行相同的算法，提供与先前客户端相同的状态和用户输入，验证结果与链下输出是否匹配

![Figure1](image/layered-architecture.png)

```
client,service和侧链不对应？？？？    axon muta提供基础设施


```







<center>图 1. 分层架构的状态生成和验证分离</center>
<center>Common knowledge layer 第一层（下层）用于状态验证和储存，generation layer第二层（链下）生成状态，并将生成的状态发送给layer1，状态生成方法如下一节，对应图中从左到右</center>

状态生成和验证分离的优势在于状态生成转到链下进行，实现链下扩容，状态生成和验证并行化。
![Figure2](image/separation-of-generation-verification.png)



<center>图 2. 状态生成与验证的分离与不分离</center>
<center>上面是以太坊的流程，交易中包含的是用户请求或者发起的事件；下面是比特币/ CKB的流程，交易中包含的是链下生成状态。</center>

```
验证方式，验证是不是再进行一次计算？
```





#### 状态生成方法

* 客户端 ： 开发人员可以使用任何编程语言在本地设备上来实现生成器
* Web服务 : 应用程序web服务提供客户生成状态的服务
* 状态通道 ：两个或更多用户可以使用对等通信来生成新状态
* 生成链 ： 生成链是生成新状态并将其存储在CKB中的区块链，可以是许可链也可以是非许可链。


### cell模型

cell模型是一个更泛化的比特币的 UTXO 模型，和 UTXO 只能存数字不同，Cell 可以存任意类型的数据，是一个有状态的编程模型。
![Figure3](image/cell.png)

cell是CKB中的基本状态单位，用户可以在其中包含任意状态。单元格具有以下字段：

* capacity-cell的大小限制。cell的大小是其中包含的所有字段的总大小。
* data-状态数据存储在此单元格中。它可以为空，但是一个单元使用的总字节数（包括数据）必须始终小于或等于其容量。
* type_script：状态验证脚本。
* lock_script ：代表cell所有权的脚本。cell的所有者可以将cell转让给其他人。

```
pub struct CellOutput {
    pub capacity: Capacity, 
    pub data: Vec<u8>,
    pub lock_script: Script,
    pub type_script: Option<Script>,
}
```

capacity是CKB固有的资产，其供应受预定义规则限制。它也是状态的度量单位。容量大小确定可以在CKB上存储多少数据，即CKB的最大状态空间。这意味着存储在CKB上的状态受容量总量的限制。`capacity`是原生资产，受到预先确定的发行规则约束，其总量有限。`capacity`同时又是状态的度量，有多少`capacity`，CKB上就能放多少数据，CKB状态空间的最大值与`capacity`总量大小相等，CKB上保存的状态不会超过`capacity`总量。这两点决定了，CKB不会有状态爆炸的问题。在`capacity`发行规则适当的情况下，网络应该可以长久的保持去中心化的状态。每一个Cell都是独立状态，有明确的所有者，[原本属于公共资源的状态空间被私有化 ](https://en.wikipedia.org/wiki/Tragedy_of_the_commons#Privatization)，宝贵的共识空间可以被更有效的使用。
类型脚本（验证脚本）定义了状态转移的规则，对新状态施加约束。CKB-VM将执行类型脚本，并验证保存在新生成的单元中的状态是否符合类型脚本预定义的规则。受相同类型脚本限制的所有单元格都存储相同类型的数据。
CKB VM中执行type脚本和lock脚本，在交易输出创建cell时，VM执行type脚本，验证cell的状态在特定规则下有效，在交易输入引用cell时，执行lock脚本，确认用户是否具有更新或转让cell的权限，如果有权限则允许用户根据type脚本指定的验证规则来传输单元格或更新其数据。
type和lock脚本的设置允许：

* 用户选择自己的用于签署交易的任何加密签名方案
* 用户自主创建lock脚本
* 所有者可以将cell借给他人使用同时保持所有权

cell是不可变的，一旦创建不可修改，每个cell只能使用一次。当一个cell被销毁时，将其标记为“已销毁”，而不是从区块链中删除。每个cell只能被破坏一次。

### 交易

表示状态转换，从而导致单元传输，更新或同时发生。

#### 交易模型

![](image/640.jpeg) 

* CKB 的交易模型是 UTXO 结构，基本结构单位是cell，每一笔交易会销毁一部分 Cells，生成一部分新的 Cells。

* CKB 上的脚本是用户可以自定义的，销毁和生成的规则可以由用户定义。例如，在 CKB 上签名算法是随时可替换的，而比特币和以太坊是不可以换的。

* CKB 内的 Type Script是用户可自定义的，规定交易的规则，用户可以在此基础上进行额外的转账规则的开发。比如用户可以自定义 Type Script 中的某一个字段在转账前后的余额必须一致，这实际上就可以在 CKB 上实现新的 UDT，lock script规定交易签名

#### 交易结构内容

交易包括以下内容：

* deps：从属cell集，提供交易验证所需的只读单元格。
* inputs：单元格参考和证明。单元格引用指向在交易中传输或更新的活动单元格。证明（例如签名）证明交易创建者有权转移或更新引用的单元格。
* outputs：在此状态转换中创建的新单元格。

---

```
(?)
```



* Header Deps 可以让脚本来读取区块头。这个功能有一定的限制以确保交易被确定。

  只会在所有交易中的脚本已经有确定的结果时，才会说一笔交易已经确定了。

  Header Deps 允许脚本去读取其哈希已经列在 `header_deps` 中的区块头。还有另一个先决条件，是交易只能在所有列在 `header_deps` 的区块都已经在链上时（叔块除外），才可以被添加到链上。

具体数据结构在后面有详细介绍。

#### 交易过滤器协议

交易过滤器协议允许peers 减少其发送的交易数据量。想要检索感兴趣的交易的对等方可以选择在每个连接上设置过滤器。过滤器定义为对从交易派生的数据的[bloom过滤器](http://en.wikipedia.org/wiki/Bloom_filter)，目的是允许低容量peer（智能手机，浏览器扩展，嵌入式设备等）维护有关链中某些特定交易的最新状态的高安全性保证，或验证交易的执行。

+ SetFilter

  + peer节点将其广播的交易限制为与过滤器匹配的交易，匹配算法

    ```
    table SetFilter {
        filter: [uint8];
        num_hashes: uint8;
        hash_seed: uint32;
    }
    ```

    `filter`：任意字节对齐大小的位字段，最大大小为36,000字节。

    `num_hashes`：此过滤器中使用的哈希函数个数。此字段中允许的最大值为20。

    `hash_seed`：协议中使用[Kirsch-Mitzenmacher-Optimization](https://www.eecs.harvard.edu/~michaelm/postscripts/tr-02-05.pdf)哈希函数，hash_seed`是一个随机偏移量，`h1`哈希值的uint32低，哈希值的`h2`uint32高，第n个哈希值是`(hash_seed + h1 + n * h2) mod filter_size`。

+ AddFilter

  + 收到`AddFilter`消息后，给定的位数据将通过按位或运算符添加到现有的过滤器中

    如果一个新过滤器被添加到一个对等点，而它与网络的连接是打开的，那么这个消息是有用的，避免了重新计算和向每个对等点发送一个全新的过滤器的需要。

    ```
    table AddFilter {
        filter: [uint8];
    }
    ```

    `filter`: 任意字节对齐大小的位字段。数据大小必须小于或等于先前提供的过滤器大小。

+ ClearFilter

  + 删除过滤器，没有参数

##### FilteredBlock

After a filter has been set, peers don't merely stop announcing non-matching transactions, they can also serve filtered blocks. This message is a replacement for `Block` message of sync protocol and `CompactBlock` message of relay protocol.

过滤器还是会处理被过滤的块

```
table FilteredBlock {
    header: Header;
    transactions: [IndexTransaction];
    hashes: [H256];
}

table IndexTransaction {
    index:                      uint32;
    transaction:                Transaction;
}
```

`header`: Standard block header struct.

`transactions`: Standard transaction struct plus transaction index.

`hashes`: Partial [Merkle](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0006-merkle-tree/0006-merkle-tree.md#merkle-proof) branch proof.



##### 过滤器匹配算法

1. 测试交易本身的哈希值。
2. 对于每个CellInput，测试哈希值`previous_output`。
3. 对于每个CellOutput，测试脚本的`lock hash`和`type hash`。
4. 否则没有匹配项

### script

CKB上的脚本主要就是lock script和type script。需要区分脚本和脚本代码，脚本代码实际上是指你编写和编译并在 CKB 上运行的程序。而脚本，实际上是指 CKB 中使用的脚本数据结构，每一个CKB 的用户都可以拥有不同的 lock script ，但是却可以共用同样的 lock script code。

[ckb_syscalls.h](https://github.com/nervosnetwork/ckb-system-scripts/blob/66d7da8ec72dffaa7e9c55904833951eca2422a9/c/ckb_syscalls.h)提供了脚本保护  、脚本验证、数据访问等等功能函数。

CKB没有直接将脚本代码嵌入到脚本数据结构中，而是只包含了代码的哈希，这是实际脚本二进制代码的 Blake2b 哈希。

部署和运行一个 type 脚本的脚本步骤:

1. 将脚本编译为 RISC-V 可执行的二进制文件
2. 在 cell 的 data 部分部署二进制文件
3. 创建一个 type 脚本数据结构，使用二进制文件的 blake2b 散列作为`code hash`，补齐`args`部分中脚本代码的需要的参数
4. 用生成的 cell 中设置的 type 脚本创建一个新的交易
5. 将包含脚本代码的 cell 的 outpoint 写入到一笔交易的 deps 中去



### [CKB虚拟机](https://medium.com/nervosnetwork/an-introduction-to-ckb-vm-9d95678a7757)



CKB 虚 拟 机 是 一 个 用 来 执 行 type 和 lock 脚 本 的 ， 基 于**RISC-V** 指令集的虚拟机，64位地址空间提供核心指令，只使用标准的RISC-V指令集,好处是现有的以 C 语言或者其他语言实现的密码学库，都可以轻易地移植到 CKB 虚拟机上，并被 Cell 脚本所调用。实际上这样的虚拟机允许可以用任何语言编写任何想写的逻辑。

1. **灵活性**：CKB 虚拟机是一个与密码学操作无关的虚拟机,没有任何密码学指令写死在 CKB 虚拟机中,CKB-VM 和 Cell 模型结合在一起可以让底层灵活的增加新的密码学原语，而无需像比特币一样需要通过硬分叉去增加。

2. **运行时可见性**：问题：比特币的VM层仅提供一个堆栈（该VM不知道可以存储在堆栈上的数据量或堆栈深度）。在堆栈模式下实现的VM也会出现此问题。尽管共识层可以提供堆栈深度定义或间接提供堆栈深度（基于代码长度或gas限制），但是在VM上运行的程序无法获得堆栈深度。**由于在VM上运行的程序的开发人员必须猜测程序的运行状态，因此在VM上运行的程序无法使用VM的所有潜在功能。**

   考虑优先定义VM运行期间所有资源的限制，包括gas限制和堆栈空间大小，并为在VM上运行的程序提供查询资源使用情况的能力。这将允许VM上运行的程序根据资源可用性利用不同的算法。通过这种设计，程序可以利用VM的全部潜力。

   a）可以根据单元容量为存储数据的合约选择不同的策略。

   b）可以根据单元数据量和剩余内存大小为合约选择不同的处理机制。

   c） 对于某些常见合约，例如哈希算法，可以根据用户提供的CPU周期数选择不同的处理方法。

   ```
   这里是思路，未知是否已经实现
   指出BTC VM的功能的不足
   ```

   

3. **运行时开销**：用CPU周期数代替gas

---

CKB不支持浮点指令，CKB脚本开发人员可以根据需要选择将softfloat实现打包到二进制文件中。CKB虚拟机支持RISC-V压缩指令以减小contract大小，直接将Linux ELF格式用作contract格式.

```?
?   Linux ELF格式的内容
```

contract从ELF格式contract文件中的main函数开始，参数通过标准argc和argv传入。当main返回0时，contract执行被视为成功。请注意，出于空间考虑，我们可能不会在argv中存储完整的输入和输出数据。相反，我们可能只在argv中提供元数据，并利用其他库和syscall来支持输入/输出加载。这样，运行时间成本可以最小化。CKB VM是严格的单线程模型，contract可以附带自己的协程。

```
--- 
```



**特权架构支持**

* 为支持特权模式切换功能，以及指定 page fault 函数功能添加刚刚好足够的 CSR(控制与状态寄存器，control and status register) 指令以及 VM 修改
* [TLB](https://en.wikipedia.org/wiki/Translation_lookaside_buffer) 结构

注意这里实现的特权架构是一个默认关闭，可选开启的功能：虽然这个功能在 CKB VM 中一直存在，但是合约设计者可以自由决定是否使用这一功能。为最小 cycle 使用数优化的合约可以完全忽略这一功能。

```
特权，最高权限需要补充内容，意味、目的是什么
```





**VM syscalls**

格式：syscall(a7,a0,a1,a2,a3,a4,a5)

a0-a5为储存在寄存器中的函数的参数，a7为syscall number（？指的需要调用的指令的编号）

syscall的返回值：0表示成功，1表示输入错误或格式错误、超出范围，2表示项目丢失

a.load(), a就是a6

- [Exit](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#exit)

  ```C
  void exit(int8_t code){  syscall(93, code, 0, 0, 0, 0, 0);}
  ```

  一旦受到exit系统调用，CKB VM将根据指定的返回code终止执行对应程序。

- [Load Transaction Hash](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#load-transaction-hash)

  ```c
  int ckb_load_tx_hash(void* addr, uint64_t* len, size_t offset){  return syscall(2061, addr, len, offset, 0, 0, 0);}
  ```

  该系统调用将计算当前交易的哈希并将其复制到VM内存空间。

  + addr: 指向VM内存空间中的缓冲区的指针，表示要在何处加载序列化的交易数据。

    ```
    当前交易和交易的哈希没有作为参数传入    --- 需要问nervos
    ```

    

  + len: 指定addr的缓冲区的长度

  + offset: 偏移量，指定应该从哪个偏移量开始加载序列化的交易数据。

- [Load Script Hash](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#load-script-hash)

  ```c
  int ckb_load_script_hash(void* addr, uint64_t* len, size_t offset){  return syscall(2062, addr, len, offset, 0, 0, 0);}
  ```

  + addr: 同上

  ```
  也没有指定script作为参数传入
  ```

  

  + len：同上
  + offset：同上

  该系统调用将计算当前正在运行的脚本的哈希并将其复制到VM内存空间。这是脚本可以在其自己的单元上加载的唯一部分。

- [Load Cell](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#load-cell)

  ```C
  int ckb_load_cell(void* addr, uint64_t* len, size_t offset, size_t index, size_t source){  return syscall(2071, addr, len, offset, index, source, 0);}
  ```

  + addr、len、offset类似

  + index：要读取的cell的索引
  + source：表示要读取的cell 的源的标志，可能包括
    + input cells
    + output cells
    + dep cells

  该系统调用将根据`source`和`index`值在当前交易中定位单个单元格，将整个单元格序列化为Molecule Encoding格式，然后使用与“ *[Load Transaction Hash](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#load-transaction-hash)”*系统调用中所述的相同步骤将序列化的值馈送到VM中。

- [Load Cell By Field](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#load-cell-by-field)

  ```c
  int ckb_load_cell_by_field(void* addr, uint64_t* len, size_t offset,                           size_t index, size_t source, size_t field){  return syscall(2081, addr, len, offset, index, source, field);}
  ```

  

  - field: 表示要读取的单元格的字段的标志flag，可能的值包括：
    - 0: capacity.
    - 1: data.
    - 2: data hash.
    - 3: lock.
    - 4: lock hash.
    - 5: type.
    - 6: type hash.
  - 其他参数同上

- [Load Input](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#load-input)

  ```c
  int ckb_load_input(void* addr, uint64_t* len, size_t offset,                   size_t index, size_t source){  return syscall(2073, addr, len, offset, index, source, 0);}
  ```

  - index：表示要读取的inputs的索引
  - source：要读取的inputs的源的标志flag，可能的值有
    - 1.inputs
    - 2.outputs：note this is here to maintain compatibility of `source` flag, when this value is used in *Load Input By Field* syscall, the syscall would always return `2` since output doesn't have any input fields.
    - 3.deps：因为dep没有输入字段，总是返回2（为什么返回2）

- [Load Input By Field](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#load-input-by-field)

  ```
  int ckb_load_input_by_field(void* addr, uint64_t* len, size_t offset,                            size_t index, size_t source, size_t field){  return syscall(2083, addr, len, offset, index, source, field);}
  ```

  + `field`: 表示要读取的inputs的filed的标志位
    - 0: args.
    - 1: out_point.
    - 2: since.

- [Load Header](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#load-header)

  ```
  int ckb_load_header(void* addr, uint64_t* len, size_t offset, size_t index, size_t source){  return syscall(2072, addr, len, offset, index, source, 0);}
  ```

  

  参数设置和load cell是一样的

- [Debug](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md#debug)

  ```
  void ckb_debug(const char* s){  syscall(2177, s, 0, 0, 0, 0, 0);}
  ```

  此系统调用接受以null终止的字符串，并将其打印为CKB中的调试日志。它可以用作调试CKB中脚本的简便方法。该系统调用没有返回值。

### [序列化](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0008-serialization/0008-serialization.md)

CKB uses two major serialization formats, [Molecule](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0008-serialization/0008-serialization.md#molecule) and [JSON](https://www.json.org/).

[JSON](https://www.json.org/)通过[JSON-RPC](https://www.jsonrpc.org/specification)在节点RPC服务中使用。

#### Molecule

**零拷贝** 零拷贝主要的任务就是避免CPU将数据从一块存储拷贝到另外一块存储，主要就是利用各种零拷贝技术，避免让CPU做大量的数据拷贝任务，减少不必要的拷贝，或者让别的组件来做这一类简单的数据传输任务?

对于高速网络来说，零拷贝技术是非常重要的。这是因为高速网络的网络链接能力与 CPU 的处理能力接近，甚至会超过 CPU 的处理能力。如果是这样的话，那么 CPU 就有可能需要花费几乎所有的时间去拷贝要传输的数据，而没有能力再去做别的事情，这就产生了性能瓶颈，限制了通讯速率，从而降低了网络链接的能力。

零拷贝技术主要有下面这几种：

- 直接 I/O：对于这种数据传输方式来说，应用程序可以直接访问硬件存储，操作系统内核只是辅助数据传输：这类零拷贝技术针对的是操作系统内核并不需要对数据进行直接处理的情况，数据可以在应用程序地址空间的缓冲区和磁盘之间直接进行传输，完全不需要 Linux 操作系统内核提供的页缓存的支持。 

  ```
  直接IO 具体实现
  ```

  

- 在数据传输的过程中，避免数据在操作系统内核地址空间的缓冲区和用户应用程序地址空间的缓冲区之间进行拷贝。有的时候，应用程序在数据进行传输的过程中不需要对数据进行访问，那么，将数据从 Linux 的页缓存拷贝到用户进程的缓冲区中就可以完全避免，传输的数据在页缓存中就可以得到处理。在某些特殊的情况下，这种零拷贝技术可以获得较好的性能。Linux 中提供类似的系统调用主要有 mmap()，sendfile() 以及 splice()

  ```
  下次插入mmap代码有必要深挖，有些项目在研究零拷贝对区块链可拓展性的作用，网络层的比较好的性能提升
  ```

  

- 对数据在 Linux 的页缓存和用户进程的缓冲区之间的传输过程进行优化。该零拷贝技术侧重于灵活地处理数据在用户进程的缓冲区和操作系统的页缓存之间的拷贝操作。这种方法延续了传统的通信方式，但是更加灵活。在　 Linux 　中，该方法主要利用了写时复制技术。

  ---

  

## 经济模型

### 代币

Nervos CKB的token是“Common Knowledge Byte”，简称CKByte。CKBytes使token持有者有权占据区块链总状态存储的一部分。
Nervos CKBByte不仅是coin，还代表了cell的最大可用存储容量，用户可以使用这些Byte来存储资产、应用程序状态或是其他类型的数据资料。

### 代币发行机制

* 基础发行: 总供给量一定，基础发行数量大约每 4 年减半一次。

* 二级发行: 收取状态租金和交易手续费，每年的发行数量是不变的。基础发行停止后，二级发行仍会继续

  ```
  每年的发行数量是不变的?具体指？
  ```

  

矿工获得的收益来自于所有的基础发行（出块奖励）和部分二级发行（状态租金和交易手续费）

### 二级发行的状态租金

原生代币代表了占用全局状态的权利，所以代币发行政策会限制状态的增长，状态存储受限制并且成为了稀缺资源。对所有代币持有者都会收取状态租金支付给矿工。

* 对于使用CK Byte存储状态的用户收取状态租金。
* 对于那些没有使用 CK Byte 存储状态的所有者也收取了租金。允许这些用户将他们的原生代币存入并锁定到一个特殊合约NervosDAO中， NervosDAO 将接受部分二级发行的补偿，以弥补因为不公平造成的稀释

二次发行的出块奖励，按照使用了 CK Byte 存储状态的部分占比奖励给矿工，锁定在 NervosDAO 的合约中的部分占比按比例分配给锁币的用户，剩下的按社群制定的策略或者销毁。

​               矿工  

10000    60%   

举个例子，假设在「二级发行」时，所有 CK Byte 的 60％ 用于存储状态，用户可能选择了其中35%的CK Byte存放并锁定在 NervosDAO 的合约中，剩下的 CK Byte 保持流动性。那每次进行二级发行出块奖励的时候，60％ 的「二级发行」会奖励给矿工，35％ 的会进入 NervosDAO 按比例分配给锁定的代币（用户），最后剩下的 5％ 既没有占用也没有锁币的部分，将交由社群订定的治理机制处理；在社群未达到机制的共识之前，这部分的二级发行将会烧毁。

```
- 收取的状态租金存在哪，存储方式？cell池。。。。BTC的手续费存储在哪？？？手续费UTXO做标记？coinbase交易- 既没有占用也没有锁币的部分，交易手续费？-销毁？
```



### 交易手续费

**Nervos CKB 限制了计算速度（computation)和带宽的吞吐量。**（当用户要使用这些系统资源时，它可以有效地成为一个拍卖市场，让用户意识到这些资源是有限的）
当用户提交交易时，提交的总输入需超过总输出的 Cell，将其差值作为以原生代币来支付的交易费用，支付给打包交易的出块矿工。

在 Nervos CKB 中，手续费的支付可以透过原生代币、用户自定义代币，或是两者结合使用。
**cycles** he number of units of computation (called "cycles") 计算单位数，类似于gas，代表交易的复杂度，我的理解是交易计算所占的CPU的时钟周期。
Nervos的回答是 a way to measure program running time that is decoupled from computer specs。节点可以选择设置可接受的周期上限cyclemax，并且仅处理周期更少的交易。


### 用户自定义代币

用户可以自由地发行其他代币，例如稳定币，用来支付交易费用。因为原生代币的内在价值不是基于支付手续费的，每个拥有资产的人都必须拥有 Nervos 的原生代币，原生代币对应的是状态的存储空间，而且资产本身也占有了一定的存储空间，所有用户发行的代币不会对nervos的经济模型造成威胁。

#### UDT[User  Defined Token](未实现)（4月份已实现）

```
需要更新内容，已经实现了
```

```
ERC20和UDT比较，账户模型的所有权问题，UDT更安全，需要补充
```



[UDT](https://talk.nervos.org/t/approach-to-designing-a-user-defined-token-standard-on-ckb-part-1/3855)的架构

* UDT实例：用户在Cell 中持有的实际Token
* UDT定义-类型脚本Cell；元数据单元：系统级信息和执行转账规则集的元数据和脚本集合是我将统称为“ UDT定义”
* UDT标准-类型脚本Cell；锁定脚本Cell：UDT标准在物理上由两个Cell组成-一个锁脚本和一个类型脚本

一个UDT标准会有很多UDT定义，一个UDT定义会有很多UDT实例

##### UDT的查询和操作

**查询**

* 查询某地址下UDT余额
* 获取UDT元数据
* 获取UDT转账的依赖项
* 获得UDT实例的实际使用者
* 获得UDT定义的实际创建者
* 从定义中获得UDT实例的实际创建器
* 汇集持有该UDT实例下的地址

**操作（状态更改）**

* 发送UDT实例
* 批准UDT实例的使用者
* 烧毁UDT实例
* 创建新的UDT实例
* 更新UDT定义
* 创建带有初始UDT实例供应的新UDT定义

##### CKB 上实现 UDT 的设计思路

* 逻辑与状态分离，使用统一的业务代码。安全将得到极大的保障。
* 用户行为、管理员行为、授权行为分离。
* 仅提供最核心的 UDT 功能。指提供最核心的 UDT 功能，保障开发者对 UDT 灵活性的需求。
* 采用面向状态，注重行为结果的设计模式，去定义 UDT 的行为。

### 用于保存价值的经济模型

* 确保协议的安全性`
* 确保协议的长期可持续性
* 参与者的利益--调整不同参与者的目标，以增加协议网络的价值

#### 协议的安全性和可持续性

保障协议的安全性的主要设计：

* 原生代币代表相应状态存储空间的主张权
* 二级发行可以保证矿工的补偿是可预测的，NervosDAO确保代币长期持有者的代币价值不会因为二级发行而被稀释
* 为了保持网络的去中心化和抗审查能力，降低参与共识以及成为主节点所需要的资源门槛
* 通过调节计算和带宽的吞吐量来保护节点的运营成本

#### 调整网络参与者的利益

##### 利益关系

* 用户希望保护其资产安全
* 开发者的应用的普及度
* 矿工需要更高的收入
* 代币持有者希望币价升值

##### 方式

Bootstrapping Network Effect and Network Growth

* 减少token的循环供应并为代币的市场价格提供积极支持，增加原生代币的价值
* 更高的币价与二次发行（交易手续费）所占份额增加，保证矿工的收益，促使矿工保证网络的安全
* The pro-cyclical loop of the network's adoption and network's intrinsic value provides a powerful growth engine for the network. 网络的顺周期循环和内在价值使得nervos的CKB成为价值储存的理想选择



#### First Class Asset模型

[first class Asset模型](https://medium.com/nervosnetwork/first-class-asset-ff4feaf370c4)
基本上和cell模型等价，有了Cell模型，nervos能够在Nervos CKB上实现作为“一等公民”的用户定义资产（User Defined Assets），简称 First-class Assets.源于头等函数First-class Function可以被当作值赋给一个变量，可以被当作参数传递给其他函数，也可以被当作返回值从其它函数传出来。

ERC20：token 所有权属于合约，有较大作恶空间 

```
补充ERC20和UDT对比
```





#### 借贷（未实现,伪代码2020.1.15）

nervos虚拟机和cell模型允许nervos CKB可以借贷给别人使用。Nervos CKB代表代币和容量，即所有者借出cell的容量给别人储存数据或者借出代币给别人使用，cell所有者还是原所有者，更换了使用者。

Nervos是一个分层网络，第1层区块链即CKB知识库--主链，可以专注于安全，中立，分散和开放的公共基础设施，而较小的第2层网络可以经过特别设计以最适合其使用环境。Nervos网络中，第1层协议（通用知识库）是整个网络的价值保留层，CKB层所有成员共享，第2层协议利用第1层区块链的安全性来提供无限的可扩展性和最低的交易费用，并且还允许在信任模型，隐私和最终性方面进行特定于应用程序的权衡。

每一个 cell 都可以同时有 lockscript 和可选的 typescript 两种 script 作为验证脚本，而一个 cell 被消耗时，都需要跑一遍它的 lockscript 和可能存在的 typescript，每一个 cell 被创建时，都需要跑一遍它的 typescript：

```
cell consume:if run(cell.lock_script) == 0 && run(cell.type_script) == 0:    return successcell create:if run(cell.type_script) == 0:    return success
```



#### 方式

- 租赁方无需与出租方沟通即可自行完成租赁过程
- 租赁方可自行调整租赁时间
- 出租方可自行将拥有的 ckb 转换为待出租的状态
- 出租方可自定义租赁 epoch 单价、租赁 epoch 上限等
- 租赁期间内，租赁方可进行 cell 状态变更（Option）
- 租赁期间内，出租方无权强行收回 cell 空间



租赁的实现即是放弃 lockscript 的销毁验证，转为 typescript 生成验证，typescript 验证的逻辑是，一，确认当前 cell 的状态，二，确认生成 cell 的 type 和 lock 符合预期。而任何人都可以租赁的前提是，首先部署一个 cell 作为 always success lock ，这样，待租赁的 cell 大概和下面差不多：

```
cell {    capacity: 待租赁大小    lockscript : always success lock    typescript：租赁合约(的代码的哈希)    data：{        max_term_of_lease(unit: epoch): u32,        unit_price(ckb/epoch): u64        lock_args: 出租方 lock args        code_hash:     }}
```

```
- 可以只租出部分的capacity吗，（设置多一个参数？）需要去问nervos租赁合约代码的使用，需要在代码达成共识后吗？租赁者不知道代码逻辑，需要提问补充code_hash是不是指lock的hash ？？？ 需要再查找一下
```



#### 借贷的两种模式

##### 一次性

lockscript 用默认实现的带 since 约束的 lock_args，只需要写一个通用的 typescript，同时 typescript 的逻辑也相对来说比较简单，这种方案租赁过程是一次性的，租赁人无法在租赁期间内更改租赁 cell 的状态：

```c++
function (input, output )// 待租赁 cell 初始状态if input_cell.lock == always_success_lock_hash:    // cell 所有人退出租赁模式    if output_cell.lock_args == input_cell.data.lock_args && output_cell.lock.code_hash == input_cell.data.lock.code_hash && output_cell.capacity == input_cell.capacity:        return success    // 租赁人付费用并使用    elif output_cell.lock_args == input_cell.data.lock_args ++ time_since && output_cell.lock.code_hash == input_cell.data.code_hash && output_cell.capacity >= input_cell.capacity + witness.term_of_lease_epoch_num * input_cell.data.unit_price:        return successelse:    // 租赁中 或 转入租赁状态    return success
```

```
output cell 指的是租赁方，退出租赁模式的时候不是一笔交易吗？output_只有一个input output的来源
```

```
租赁现在已经实现了，需要更新，付费和使用的逻辑先后
```

---

##### 重点梳理

---



##### 多次

在第一种方案的基础上，需要实现租赁期间内的租赁人可变更状态的同时，限制出租人只有在租赁结束后才可以进行状态变更。这里复杂的地方是 type 和 lock 都需要做一定的修改。

**lockscript**

```c++
if lock_args ++ since == 租赁人 or lock_args  == 出租人:    return success
```

**typescript**

```
// 待租赁 cell 初始状态if input_cell.lock == always_success_lock_hash:    // cell 所有人退出租赁模式    if output_cell.lock_args == input_cell.data.lock_args && output_cell.lock_hash == input_cell.data.lock_hash && output_cell.capacity == input_cell.capacity:        return success    // 租赁人付费用并使用    elif output_cell.lock_args == any_lock_args_by_lessee + input_cell.data.lock_args + time_since && output_cell.lock.code_hash == or_code_hash  && output_cell.capacity >= input_cell.capacity + witness.term_of_lease_epoch_num * input_cell.data.unit_price && output_cell.data.lock_hash == input_cell.data.lock_hash:        return successelif  input_cell.lock.code_hash == or_code_hash:    // 租赁中租赁人变更 cell 状态    if output_cell.lock == input_cell.lock && output_cell.lock_hash == input_cell.lock_hash && output_cell.capacity >= input_cell.capacity && output_cell_type == input_cell_type && output_cell.data.lock_hash == input_cell.data.lock_hash:        return success    // 出租人取回到期的 cell    elif output_cell.lock_arg == input_cell.data.lock_arg && output_cell.lock.code_hash == input_cell.data.code_hash:        return successelif:    // 转入租赁状态    return success
```

```
- 多次租赁的场景？需要补充   A租给B B租给C- lock_hash 和 code_hash 概念混淆（input_cell.data.lock_hash)- any_lock_args_by_lessee 含义  output_cell.lock_args == any_lock_args_by_lessee + input_cell.data.lock_args + time_since需要提问- or_code_hash????- output_cell.capacity >= input_cell.capacity --- 进行交易？ // 租赁中租赁人变更 cell 状态为什么是>=变多了需要画个图
```



## 网络模型


### Nervos网络设计的核心原则

* CKB层经过加密经济学设计成为价值存储。
* 第2层提供扩展选项，实现交易功能、降低交易成本、改善的用户体验。
* 第一层CKB知识库使用POW共识算法(NC-MAX)
* 状态存储具有细粒度的所有权模型。为了向矿工提供一致的长期奖励（无论交易需求如何），状态占用产生持续的成本。

### 节点类型

CKB节点分为三种类型：

* 挖矿节点： 挖矿。参与共识过程，收集交易打包成块，通过pow产生新块。不必储存整条链的交易历史，只储存当前的cell集。
* 全节点： 验证新块和交易，relay（传递or中继）块和交易，选择分叉，储存链信息。
* 轻节点： 信任全节点，只订阅和存储所关注的cell的子集，在移动设备上运行。



### 链同步协议

**块可以处于以下 5 种状态**：

1.  Unknown: 在连接块头执行之前，块的状态是未知的。
2.  Invalid：任意一步失败，块的状态是无效的，且当一个块标记为 Invalid，它的所有子孙节点也都标记为 Invalid。
3.  Connected: 连接块头成功，且该块到创世块的所有祖先块都必须是 Connected, Downloaded 或 Accepted 的状态。
4.  Downloaded: 下载块成功，且该块到创世块的所有祖先块都必须是 Downloaded 或者 Accepted 的状态。
5.  Accepted: 采用块成功，且该块到创世块的所有祖先块都必须是 Accepted 的状态。

#### 基本步骤

1.  连接块头 (Connect Header): 获得块头，验证块头格式正确且 PoW 工作量有效
2.  下载块 (Download Block): 获得块内容，验证完整的块，但是不依赖祖先块中的交易信息。
3.  采用块 (Accept Block): 在链上下文中验证块，会使用到祖先块中的交易信息。

##### 1.连接块头

---先同步区块头，连接块头流程：

1. 当节点 Alice 连接到节点 Bob 之后，Alice 让 Bob 发送所有在 Bob 的 Best Chain 上但不在 Alice 的 **Best Header Chain** 上的块头，进行验证并确定这些块的状态是 Connected 还是 Invalid。

2. Bob返回块头

3. Alice循环询问块头，循环第一步

   Alice 在连接块头时，保持 Best Header Chain Tip 的更新

![image-20200212191955710](./image/image-20200212191955710.png)

第一步中Alice将自己 Best Header Chain 中的块进行采样，将选中块的哈希作为消息内容发给 Bob。采样模式基本原则是最近的块采样越密，越早的块越稀疏。类似如下深绿色块为locator定位块，采样块的哈希列表被发送给bob

```

```



![image-20200212193439724](./image/image-20200212193439724.png)

第二步，Bob 根据 Locator 和自己的 Best Chain 可以找出两条链的最后一个共同块。因为创世块相同，所以一定存在这样一个块。Bob 把共同块之后一个开始到 Best Chain Tip 为止的所有块头发给 Alice。

区块同步会碰到的三种情况：

1.  Bob 的 Best Chain Tip 在 Alice 的 Best Header Chain 中，最后共同块就是 Bob 的 Best Chain Tip，Bob 没有块头可以发送。
2.  Alice 的 Best Header Chain Tip 在 Bob 的 Best Chain 中并且不等于 Tip，最后共同块就是 Alice 的 Best Header Chain Tip。
3.  Alice 的 Best Header Chain 和 Bob 的 Best Chain 出现了分叉，最后共同块是发生发叉前的块。

如果要Bob发送的块很多，需要做分页处理。

Alice在收到块头消息时可以先做以下格式验证：

- 消息中的块是连续的

- 所有块和第一个块的父块(最后一个共同块）在本地状态树中的状态不是 Invalid.

  ```
  *** 第一个块的父块 --- 需要问nervos，alice如何知道Bob发送的块是不是有效的？spv验证块头，utxo，状态树需要知道*状态树*的内容
  ```

  

- 同步时不处理 Orphan Block孤块

这一步的验证包括检查块头是否满足共识规则，PoW 是否有效。因为不处理 Orphan Block，难度调整也可以在这里进行验证。

![connect-header-status](./image/connect-header-status.jpg)

如果认为 Unknown 状态块是不在状态树上的话，在连接块头阶段，会在状态树的末端新增一些 Connected 或者 Invalid 状态的节点。所以可以把连接块头看作是拓展状态树，是探路的阶段。



##### 2.下载块

滑动窗口模型

完成连接块头后，一些观测到的邻居节点的 Best Chain Tip 在状态树上的分支是以一个或者多个 Connected 块结尾的，即 Connected Chain，这时可以进入下载块流程，向邻居节点请求完整的块，并进行必要的验证。

```
状态树的内容和含义
```



因为有了状态树，可以对同步进行规划，避免做无用工作。一个有效的优化就是只有当观测到的邻居节点的 Best Chain 的累积工作量大于本地的 Best Chain 的累积工作量才进行下载块。而且可以按照 Connected Chain 累积工作量为优先级排序，优先下载累积工作量更高的分支，只有被验证为 Invalid 或者因为下载超时无法进行时才去下载优先级较低的分支。

下载某个分支时，因为块的依赖性，应该优先下载更早的块；同时应该从不同的节点去并发下载，充分利用带宽。这可以使用滑动窗口解决。              

假设分支第一个要下载的 Connected 状态块号是 M，滑动窗口长度是 N，那么只去下载 M 到 M + N - 1 这 N 个块。在块 M 下载并验证后，窗口往右移动到下一个 Connected 状态的块。如果块 M 验证失败，则分支剩余的块也就都是 Invalid 状态，不需要继续下载。如果窗口长时间没有向右移动，则可以判定为下载超时，可以在尝试其它分支之后再进行尝试，或者该分支上有新增的 Connected 块时再尝试。

![image-20200212200921810](./image/image-20200212200921810.png)

下载块会把状态树中工作量更高的 Connected Chain 中的 Connected 块变成 Downloaded 或者 Invalid。



##### 3.采用块

在上一阶段中会产生一些以一个或多个 Downloaded 状态的块结尾的链，以下简称为 Downloaded Chain。如果这些链的累积工作量大于 Best Chain Tip， 就可以对这条链进行该阶段完整的合法性验证。如果有多个这样的链，选取累积工作量最高的。这一阶段需要完成所有剩余的验证，包括所有依赖于历史交易内容的规则。

采用块会将工作量更高的 Downloaded Chain 中的 Downloaded 状态块变成 Accepted 或者 Invalid，而累积工作量最高的 Downloaded Chain 应该成为本地的 Best Chain。



##### 新块通知

当节点的 Best Chain Tip 发生变化时，通过推送的方式主动去通知邻居节点。为了避免通知重复的块，一次性发送邻居节点没有的块，可以记录给对方发送过的累积工作量最高的块头 (Best Sent Header)。发送过不但指发送过新块通知，也包括发送过在连接块头时给对方的块头的回复。



### P2P 评分系统

用于防止日蚀攻击

**日蚀攻击**的原理是攻击者通过操纵恶意节点占领受害者节点所有的 Peers 连接，以此控制受害者节点可见的网络。

```
需要详细的日蚀攻击的解释日蚀攻击的获利方式等等
```

Nervos网络的每个节点都有peerstore储存尽可能多的节点信息peerinfo，peerinfo中储存了节点的分数。网络层提供评分接口，允许 `sync`, `relay` 等上层子协议报告 peer 行为，并根据 peer 行为和 节点行为调整peer的评分。

```
PeerInfo {   NodeId, // Peer 的 NodeId  ConnectedIP,  // 连接时的 IP  Direction,  // Inbound or Outbound  LastConnectedAt, // 最后一次连接的时间  Score // 分数}
```

节点行为包括符合协议的行为（如成功连接peer，成功同步块，成功获取新块）这个是加分项，可能由于网络异常导致的行为（如异常断开连接、连接peer失败、ping timeout）会采取宽容性惩罚，明显违反协议的行为（如发送无法解码的内容、发送invalid block、发送invalid transaction）会进行较大的处罚，当peer评分低于一个值`BAN_SCORE`时，会被拉入黑名单禁止连接。

**节点 outbound peers 的选择策略**

[日蚀攻击论文](https://eprint.iacr.org/2015/263.pdf)中提到了比特币节点重启时的安全问题：

1. 攻击者事先利用比特币的节点发现规则填充受害节点的地址列表
2. 攻击者等待或诱发受害者节点重启
3. 重启后，受害者节点会从 addrman (类似 peer store) 中选择一些地址连接
4. 受害节点的所有对外的连接都连接到了恶意 peers 则攻击者攻击成功

Outbound peers 连接流程:

1. 如果当前连接的 outbound peers 小于 `ANCHOR_PEERS` 执行 2， 否则执行 3
2. 选择一个锚点 peer:
   1. 从 PeerStore 挑选最后连接过的 `max_bound` 个 outbound peers 作为 `recent_peers`
   2. 如果 `recent_peers` 为空则执行 3，否则从 `recent_peers` 中选择分数最高的节点作为 outbound peer 返回
3. 在 PeerStore 中随机选择一个分数大于 `TRY_SCORE` 且 `NetworkGroup` 和当前连接的 outbound peers 都不相同的 peer info，如果找不到这样的 peer info 则执行 4，否则将这个 peer info 返回
4. 从 `boot_nodes` 中随机选择一个返回

```
Outbound peers  的触发条件，整个流程不循环
```





网络组件每隔几分钟检测子协议中的主要协议如 `sync` 协议是否工作.当发现 `sync` 协议无法正常工作时，应该设置 `try_new_outbound_peer` 变量为 `true`，当发现 `sync` 协议恢复正常时设置 `try_new_outbound_peer` 为 `false`.`try_new_outbound_peer` 的作用是在一定时间内无法发现有效消息时，允许节点连接更多的 outbound peers.

**节点 inbound peers 接受机制**

1. 找出当前连接的所有 inbound peers 作为 `candidate_peers`
2. 保护 peers (`N` 代表每一步中要保护的 peers 数量):
   1. 从 `candidate_peers` 找出 `N` 个分数最高的 peers 删除
   2. 从 `candidate_peers` 找出 `N` 个 ping 最小的 peers 删除
   3. 从 `candidate_peers` 找出 `N` 个最近发送消息给我们的 peers 删除
   4. 从 `candidate_peers` 找出 `candidate_peers.size / 2` 个连接时间最久的 peers 删除
3. 按照 `network group` 对剩余的 `candidate_peers` 分组
4. 找出包含最多 peers 的组
5. 驱逐组中分数最低的 peer，找不到 peer 驱逐时则拒绝新 peer 的连接

**Feeler Connection**

用于测试peer是否可以被连接

节点会每隔一段时间(一般是几分钟)主动发起 feeler connection：

1. 从 PeerStore 中随机选出一个未连接过的 peer info
2. 连接该 peer
3. 执行握手协议
4. 断开连接


## 共识机制

CKB共识协议是中本聪共识的一种变体。

* 消除块传播时的障碍：将交易确认分成两个独立的步骤proposal和commit
* 缩短块传播延迟：
* 减轻自私挖矿攻击

**NC-Max**

1. 两步交易确认，以降低孤块率；
2. 动态的区块间隔和区块奖励，以最佳利用带宽；
3. 考虑难度调整中的所有块以防自私采矿。

**区块结构**

| Name                   | Description                          |
| :--------------------- | :----------------------------------- |
| header                 | block metadata                       |
| commitment zone        | transactions committed in this block |
| proposal zone          | `txpid`s proposed in this block      |
| uncle headers          | headers of uncle blocks              |
| uncles’ proposal zones | `txpid`s proposed in the uncles      |

相比BTC的区块结构，添加了叔块头和叔块提案空间。

### 两步交易确认

![figure3](image/2step.jpg)



NC-Max 将交易的确认与交易的同步解耦。在致密区块中既包含新交易proposal transaction的哈希值，以通知其他矿工同步新交易。又包含老交易的哈希值，叫 confirmed 交易，这些老交易是前几个块中已经通知过的新交易，应该完成同步了，只要矿工完成了这些老交易的同步，就可以立即开始挖下一个区块。NC-Max的区块传播过程大概是：矿工 A 挖出区块后，立即广播致密交易，矿工 B 收到致密交易后，确认老交易都已经同步过了，可以立即开始挖下一个区块。新交易的同步过程与新高度的挖矿并行进行。



```
***********************************************新交易的确认与h+1的区块 旧交易验证成功 拖慢其他节点攻击tx ids id的概念   新交易状态树get fresh txs  -->  get fensh txs block？？？？矿工停止打包当前区块高度的区块的时间是接受到新块还是完成确认新块的交易如果旧的交易验证成功，所有矿工进入下一个阶段挖h+1区块，新交易验证失败，高度为h的区块断了？验证旧交易为什么有用？只能证明旧交易在链上？   --- 问nervos出块 块上链的一个完整的流程，致密区块什么时候发送，一个共识的完整流程，执行交易        -          打包    -     致密区块的发送    -     出块      -    难度调整(交易流程，cell结构 )
```





![NC-max](./image/NC-max.jpeg)

在 Compact Block 中，并不会将整个交易（比特币每个交易大约 250Bytes）广播出去，而是广播 Compact Block 中的交易 ID。这些交易 ID 组成了 Compact Block，这些 Compact Block 比实际的区块小很多，因此速度会快很多。

当节点 A 广播 Compact Block 给节点 B 的时候，节点 B 检查这些交易 ID ，如果对于节点 B 这些交易都不是 Fresh Transaction（也就是节点 B 的交易池中包含这些交易），节点 B 能够立刻转发这些 Compact Block 给相邻节点，然而如果在 Compact Block 中有 Fresh Transaction，节点 B 将首先需要从节点 A 那边同步 Fresh Transaction，然后验证这些交易的签名，这些步骤都很耗费时间。只有当整个区块都经过验证无误之后，节点 B 才能继续广播这个区块。这个过程避免收到的区块非法，这样矿工才会在这个区块之后挖矿并广播，若不经过验证万一之后发现是非法区块，那么在这个非法区块之后的区块都是无效的





**[致密区块（compact block）](https://www.8btc.com/article/93204)：** 当全节点之间已共享许多相同内存池（mempool）内容时，使用简单的技术，就可能减少将新区块广播至全节点所需的带宽数量。发送方节点向接收方节点发出致密区块“概要内容”，这些概要内容包含以下这些信息：

* 新区块头；
* 缩短交易标识符（txids），其目的是为防止拒绝式服务攻击；
* 一些发送节点预测的，但接受对等节点不具备的完整交易；
* 接收方节点将尝试使用接收到的信息，以及在其内存池（memory pool）当中的交易，来重新构建整个区块。如果它仍然缺失某些交易，它将请求广播节点。


### 块传播协议

协议的块传播协议消除了新交易的额外往返行程。nervos的协议确保在交易往返传播过程中仅持续一跳。

1. 如果committed 已确认的交易对于发送节点是未知的，就将这些交易填充到prefilled 预填满的交易列表中与紧凑块一起发送。

2. 如果某些committed 已确认的交易丢失了，则接收方会在短时间内查询发送方。无法及时发送这些交易会导致接收方断开连接并将发送方列入黑名单，这个发送方将不能再传播区块。发送方发送的块是具有不完整承诺区域committed zones，因此不会被传播。

3. 只要committed zones承诺区域完整且有效，节点就可以在接收所有新提议的交易之前开始转发区块。?

   commutted zones ?

### 动态难度调整机制（根据孤块率调整难度）

区块传播时延是有上限的，追求越小的孤块率，意味着要设置越大的出块间隔，也就会得到越低的吞吐量。过于追求极致小的孤块率，会得到一个极致不可用的区块链。动态难度调整机制用于维持一个能平衡安全和吞吐量的孤块率。孤块率的计算方法：将区块链按固定时长划分成许多难度调整周期，每个难度调整周期的孤块率等于叔块总数除以总区块数。Nervos的难度调整机制就是稳定孤块率，当孤块率大于预设的安全阈值时，难度上升，区块间隔上升，从而使得孤块率回落到安全阈值。反之亦然。

1. Computing the Adjusted Hash Rate Estimation
2. 使用下一个epoch的孤儿率来计算下一个epoch的主链块数，然后结合计算结果计算奖励和难度目标target。



```
epoch的概念需要补充，epoch包括多个共识周期，难度调整的周期
```



输入

| Name                | Description                                                  |
| :------------------ | :----------------------------------------------------------- |
| *T*<sub>*i*</sub>   | Last epoch’s target                                          |
| *L*<sub>*i*</sub>   | Last epoch’s duration: the timestamp difference between epoch *i* and epoch (*i* − 1)’s last blocks |
| *C*<sub>*i*,m</sub> | Last epoch’s main chain block count                          |
| *C*<sub>*i*,o</sub> | Last epoch’s orphan block count:  the number of uncles embedded in epoch *i*’s main chain |

输出

| Name                | Description                         |
| :------------------ | :---------------------------------- |
| *T*<sub>*i*+1</sub> | Next epoch’s target                 |
| *C*<sub>i+1,m</sub> | Next epoch’s main chain block count |
| *r*<sub>*i*+1</sub> | Next epoch’s block reward           |

```
reward 也是动态变化的？和孤块率有关？离孤块越近奖励越高，类似以太坊。
```



[调整过的散列率估计、奖励与难度计算模型](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0020-ckb-consensus-protocol/0020-ckb-consensus-protocol.md)

### Eaglesong--Nervos CKB的工作量证明函数

Eaglesong是专门为Nervos CKB工作量证明而开发的新哈希函数。同样是H(nonce || block_header) <= t .不d一样的是哈希函数是Eaglesong ，t是根据孤块率动态变换。

设计Eaglesong的原因是为了防止中本聪共识的pow存在一些没有发现的漏洞，一旦被一些矿工发现就会带来不成比例的优势，容易发动攻击，为其共识算法的无懈可击做强有力的论证。（好像conflux做了这件事情，写了90多页的证明）an undisclosed vulnerability could lead to a mining optimization providing those who discover the vulnerability with an advantage disproportionate to their contributed mining power share. The best way to avoid this situation is to make a strong argument for invulnerability.

选择实例化[海绵构造Sponge function](https://en.wikipedia.org/wiki/Sponge_function)（与[Keccak / SHA3](https://nicodechal.github.io/2019/04/08/hash-funciton-sha3/)相同)，并使用ARX操作（加法，移位（左移右移）rotation和异或）构建的置换实例化海绵，并基于宽径策略对其安全性进行论证。 （与AES基本相同的参数）。

```
sha256的漏洞，举例，为什么要新建一个pow挖矿算法
```



#### 海绵函数

![sponge](./image/sponge.png)



- 海绵结构包含 3 个重要的组成部分：

1. 一个对数据进行等长映射的函数 *f*，即输出串长度与输入串长度相同，记为 *b*。
2. 一个参数称为比率 ( rate )，记作 *r*，是指每轮要吸收的 长度为b* 的串中数据的长度，剩余部分称为容量，记为 *c*，因此，有 b*=*r*+*c。
3. 一个填充 ( padding ) 函数，记作 pad(*x*,*m*)，返回将长度为 m 的串填充为 *x* 的整数倍长度的串。例如 pad(5, 2) = 010，指将长度为 2 的串填充为 5 的整数倍长度，需要添加一个长度为 3 的串，任意长为 3 的串均可，本例中返回值为 010，其长度为 3。



- 海绵函数的格式：

​							SPONGE[*f*,*p**a**d*,*r*] (*N*,*d*)

  f：等长映射的函数

pad：填充函数

r：比率，每轮要吸收的数据的长度

N：输入串 

d：输出串长



- 海绵函数执行流程

  


## 数据结构

### cell

```
{  "capacity": "0x19995d0ccf",  "lock": {    "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",    "args": "0x0a486fb8f6fe60f76f001d6372da41be91172259",    "hash_type": "type"  },  "type": null  "data": }
```

* capacity: cell的容量大小，必须满足capacity_in_bytes >= len(capacity) + len(data) + len(type) + len(lock)。也代表CKB coin余额。

* lock: 定义cell所有权的脚本，定义cell的所有者。

* Type: 定义cell类型的脚本，即状态验证脚本，定义data更新的规则。

* data: 存储合约代码和状态数据

  ```
  cell的lock实际代表nervos 的cell的地址utxo的地址的概念，私钥->公钥->地址data的具体结构，储存方式，存储script的代码和状态，状态改变时是否code_hash改变，代码逻辑和数据存储在cell模型中MPT  -- 状态树，状态存储是存在存储typescript的cell（合约账户）还是各自的cell的data？·····························****************************添加nervos网络上线状态，运行情况所有权的问题utxo+账户模型的优点，必要性虽然目前发的capacity容量是足够的，但是是有限的创新设计   数据结构  -> 架构 - 交易流程  ->   区块链运行流程  -> 
  ```

  


### script

```
{  "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",  "args": "0x0a486fb8f6fe60f76f001d6372da41be91172259",  "hash_type": "type"}
```

* code_hash: 包含CKB脚本的ELF格式的RISC-V二进制散列，实际脚本作为dep单元附加到current transaction中。

* args: 脚本输入

  ```
  ？args的格式，哈希
  ```

  

* hash_type: 【查找匹配的dep cell时的code哈希的译码】

  只有两个值，指这个script的格式，一个是type一个是data，type指代这个script是代码，data就是数据

  ```
  【查找匹配的dep cell时的code哈希的译码】hash_type的类型，具体格式，数据的情况
  ```

### 交易

```
{  "version": "0x0",  "cell_deps": [    {      "out_point": {        "tx_hash": "0xbd864a269201d7052d4eb3f753f49f7c68b8edc386afc8bb6ef3e15a05facca2",        "index": "0x0"      },      "dep_type": "dep_group"    },    {      "out_point": {        "tx_hash": "0xbd864a269201d7052d4eb3f753f49f7c68b8edc386afc8bb6ef3e15a05facca2",        "index": "0x1"      },      "dep_type": ""    }  ],  "header_deps": [    "0xaa1124da6a230435298d83a12dd6c13f7d58caf7853f39cea8aad992ef88a422"  ],  "inputs": [    {      "previous_output": {        "tx_hash": "0x8389eba3ae414fb6a3019aa47583e9be36d096c55ab2e00ec49bdb012c24844d",        "index": "0x1"         },      "since": "0x0"    }  ],  "outputs": [    {      "capacity": "0x746a528800",      "lock": {        "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",        "args": "0x56008385085341a6ed68decfabb3ba1f3eea7b68",        "hash_type": "type"      },      "type": null    },    {      "capacity": "0x1561d9307e88",      "lock": {        "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",        "args": "0x886d23a7858f12ebf924baaacd774a5e2cf81132",        "hash_type": "type"      },      "type": null    }  ],  "outputs_data": [    "0x",    "0x"  ],  "witnesses": [    "0x55000000100000005500000055000000410000004a975e08ff99fa000142ff3b86a836b43884b5b46f91b149f7cc5300e8607e633b7a29c94dc01c6616a12f62e74a1415f57fcc5a00e41ac2d7034e90edf4fdf800"  ]}
```

* version: 交易的版本。
* cell_deps	: 包括out_point和dep_type，outpoint:指向作为此交易依赖项（deps）的cell，只列出可用的cell，且是只读的。dep_type:解释引用的单元deps的方式,包括dep_group和code。
* header_deps: 指向交易依赖项的块头。
* inputs: 交易引入的cell输入。previous_output:前一个交易的输出作为这次交易的输入；[since](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0017-tx-valid-since/0017-tx-valid-since.md): 用于保护当前引用的输入。
* outputs: 交易输出的cell。
* outputs_data: 每个输出cell中的data。
* witnesses: 签名。交易创建者提供，用lock脚本认证。

```
密码学原语的合约改变时，怎么保证dep_group依然能找到对应的cellindex的值的意思
```



#### transaction

```
table RawTransaction {    version:        Uint32,    cell_deps:      CellDepVec,   // 依赖的 cell    header_deps:    Byte32Vec,  // 依赖的 header    inputs:         CellInputVec,  // 需要销毁的 cell 列表    outputs:        CellOutputVec,  // 生成的 cell 列表    outputs_data:   BytesVec,  // 生成 cell 列表中每个 cell 对应的 data 数据}table Transaction {    raw:            RawTransaction,    witnesses:      BytesVec,  // 见证信息}
```

- cell_deps 是脚本执行的上下文中需要的必要环境，诸如加密库依赖、hash 库依赖
- header_dep 获取链上时间的手段
- dep_group 依赖库过于庞大，需要拆分 cell 进行存储，并在运行时统一加载
- type_id 保留依赖库能更新但不 break 生态的手段（可查看第 0 个块的第二个 output 的 type script）
- scriptgroup 将参数和脚本相同的 script 合并为同一组执行，减少 cycle 消耗

### 区块

```
{  "uncles": [    {      "proposals": [      ],      "header": {        "compact_target": "0x1a9c7b1a",        "hash": "0x87764caf4a0e99302f1382421da1fe2f18382a49eac2d611220056b0854868e3",        "number": "0x129d3",        "parent_hash": "0x815ecf2140169b9d283332c7550ce8b6405a120d5c21a7aa99d8a75eb9e77ead",        "nonce": "0x78b105de64fc38a200000004139b0200",        "timestamp": "0x16e62df76ed",        "transactions_root": "0x66ab0046436f97aefefe0549772bf36d96502d14ad736f7f4b1be8274420ca0f",        "proposals_hash": "0x0000000000000000000000000000000000000000000000000000000000000000",        "uncles_hash": "0x0000000000000000000000000000000000000000000000000000000000000000",        "version": "0x0",        "epoch": "0x7080291000049",        "dao": "0x7088b3ee3e738900a9c257048aa129002cd43cd745100e000066ac8bd8850d00"      }    }  ],  "proposals": [    "0x5b2c8121455362cf70ff"  ],  "transactions": [    {      "version": "0x0",      "cell_deps": [      ],      "header_deps": [      ],      "inputs": [        {          "previous_output": {            "tx_hash": "0x0000000000000000000000000000000000000000000000000000000000000000",            "index": "0xffffffff"          },          "since": "0x129d5"        }      ],      "outputs": [        {          "capacity": "0x1996822511",          "lock": {            "code_hash": "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8",            "args": "0x2ec3a5fb4098b14f4887555fe58d966cab2c6a63",            "hash_type": "type"          },          "type": null        }      ],      "outputs_data": [        "0x"      ],      "witnesses": [        "0x590000000c00000055000000490000001000000030000000310000009bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce801140000002ec3a5fb4098b14f4887555fe58d966cab2c6a6300000000"      ],      "hash": "0x84395bf085f48de9f8813df8181e33d5a43ab9d92df5c0e77d711e1d47e4746d"    }  ],  "header": {    "compact_target": "0x1a9c7b1a",    "hash": "0xf355b7bbb50627aa26839b9f4d65e83648b80c0a65354d78a782744ee7b0d12d",    "number": "0x129d5",    "parent_hash": "0x4dd7ae439977f1b01a8c9af7cd4be2d7bccce19fcc65b47559fe34b8f32917bf",    "nonce": "0x91c4b4746ffb69fe000000809a170200",    "timestamp": "0x16e62dfdb19",    "transactions_root": "0x03c72b4c2138309eb46342d4ab7b882271ac4a9a12d2dcd7238095c2d131caa6",    "proposals_hash": "0x0000000000000000000000000000000000000000000000000000000000000000",    "uncles_hash": "0x90eb89b87b4af4c391f3f25d0d9f59b8ef946d9627b7e86283c68476fee7328b",    "version": "0x0",    "epoch": "0x7080293000049",    "dao": "0xae6c356c8073890051f05bd38ea12900939dbc2754100e0000a0d962db850d00"  }}
```


#### 区块头

* compact_target: POW的难度。
* number: 区块高度。
* parent_hash: 父块的哈希。
* nonce: 随机数。
* timestamp: 时间戳。
* transactions_root: 交易哈希的CBMT根（merkle树）和交易witness哈希的CBMT根的哈希
* proposals_hash: 交易提案的哈希。

```
交易提案的哈希,btc没有交易提案？  BTC 【6个共识周期去确认交易】，BTC的区块头没有proposal？需要去熟悉BTC的区块结构
```



* uncles_hash: 叔块头的串联散列的哈希.
* version: 区块的版本号。
* epoch: 当前的轮数（Assume number represents the current epoch number, index represents the index of the block in the current epoch(start at 0), length represents the length of current epoch. The value store here will then be (number & 0xFFFFFF)）
* dao:	包含DAO相关信息的数据

#### 其他的参数

* trasactions：包含已提交的交易。

* proposals：拟议的交易的短交易ID，交易提案。

* uncles：叔块

  

### Since

**Since**是一个64位的无符号整数，用于设置一个新的共识规则，设定规定的cell花费的相关时间，可以是相对时间和绝对时间。 

Since的高8位是flags，其余56位是value，flags & (1 << 7)`代表`relative_flag，第八位是关系位，区分相对时间和绝对时间，0代表绝对时间，1代表相对时间，flags & (1 << 6)` and `flags & (1 << 5)` together represent `metric_flag，第六位和第七位是标准位，00代表基于块的锁定时间，即区块高度，01代表基于epoch的锁定时间，epoch是nervos的难度调整周期，10代表基于时间戳。

```
1-5位代表什么  例子  since的作用
```



## Axon--Muta

Muta是一个高度可定制的高性能区块链框架，旨在支持权益证明，BFT共识和智能合约。它具有高吞吐量和低延迟的BFT共识“霸主”，并支持各种虚拟机，包括CKB-VM，EVM和WASM。具有跨VM互操作性的不同虚拟机可以同时在单个Muta区块链中使用。Muta大大降低了开发人员构建高性能区块链的障碍，同时仍提供最大的灵活性来定制其协议。
Axon是由Muta构建的完整解决方案，旨在为开发人员提供位于Nervos CKB之上的交钥匙侧链，并具有实用的安全性和令牌经济模型。Axon解决方案使用CKB进行安全资产托管，并使用基于令牌的管理机制来管理侧链验证器。Axon侧链和CKB之间以及Axon侧链之间的交互的跨链协议也会被内置。使用Axon，开发人员可以专注于构建应用程序，而不是构建基础结构和跨链协议。





## reference

- nervostalk
- ckb network