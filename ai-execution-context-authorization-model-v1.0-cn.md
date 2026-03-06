# AI 执行上下文授权模型  
## AI Execution Context Authorization Model  
### Version v1.0

---

# 0. 约定与符号

## 0.1 逻辑符号

| 符号 | 含义         |
| ---- | ------------ |
| ∀    | 全称量词     |
| ∃    | 存在量词     |
| ∃!   | 唯一存在量词 |
| ∧    | 逻辑与       |
| ∨    | 逻辑或       |
| ¬    | 逻辑否       |
| →    | 蕴含         |
| ↔    | 等价         |
| ∈    | 属于         |
| ⊆    | 子集         |
| ⊂    | 真子集       |
| =    | 等于         |
| ≠    | 不等于       |
| ∅    | 空集         |

---

## 0.2 集合与类型

| 符号     | 含义                              |
| -------- | --------------------------------- |
| S        | 推理步骤集合（全域）              |
| C        | Execution Context（推理步骤子集） |
| E        | 外部事件集合                      |
| O        | 输出集合                          |
| Cap      | 能力原子集合                      |
| I        | Context 创建输入变量集合          |
| V        | 变量集合                          |
| Σ        | 系统状态空间                      |
| Σ_C      | Context 内部状态空间              |
| CIDSet   | Context 标识集合                  |
| ObsSpace | 外部可观测空间                    |

---

## 0.3 唯一性约定

若关系未显式声明为多值关系，则默认满足唯一性约束。

---

## 0.4 线性化执行约定（消除并发漂移）

本模型中的步骤集合 **S** 被约定为某一次系统执行的 **线性化序列域**，满足：

1. 时间函数为全域线性化  

$$
T : S \rightarrow \mathbb{R}
$$

2. Context 生命周期区间不允许被其他 Context 的步骤穿插  

3. 时间戳单射  

$$
\forall s,s' \in S : (s \neq s') \rightarrow (T(s) \neq T(s'))
$$

4. 线性化严格序一致性  

$$
\forall s,s' \in S :
(s' <_S s) \leftrightarrow (s' \leq_S s \land s' \neq s)
$$

> 可接受边界：本协议的 v1.0 仅覆盖 **非嵌套、非交叠** 的 Context 集合。  
> 若某实现存在嵌套/分层 Context，需要通过“上层-下层分解”为多个互不交叠的 Context 后再适配本协议。

---

# 一、模型目标

本模型定义一种 **Execution Context 级别的授权机制**，用于确保 AI 推理系统在执行过程中：

1. 外部副作用必须经过授权  
2. 授权范围在 Context 创建时确定  
3. 推理过程中不可扩展能力上限  
4. 外部副作用必须可追溯到能力请求与验证  

---

# 二、基本结构

## 2.1 推理步骤与时间

推理过程被建模为步骤集合

$$
S
$$

步骤时间函数

$$
T : S \rightarrow \mathbb{R}
$$

定义时间弱序关系

$$
\leq_S \subseteq S \times S
$$

$$
s' \leq_S s \iff T(s') \leq T(s)
$$

严格序

$$
s' <_S s \iff T(s') < T(s)
$$

---

## 2.2 Context 与步骤划分

### 2.2.1 时间凸性

$$
\forall s \in S :
(\exists s_1,s_2 \in C :
T(s_1) \le T(s) \le T(s_2))
\rightarrow (s \in C)
$$

生命周期端点

$$
T(s_{create}) =
\min \{T(s) \mid s \in C\}
$$

$$
T(s_{term}) =
\max \{T(s) \mid s \in C\}
$$

---

### 2.2.2 Context 区间不交叠

定义

$$
MinT(C)=T(s_{create}(C))
$$

$$
MaxT(C)=T(s_{term}(C))
$$

约束

$$
C_1 \ne C_2
\rightarrow
(MaxT(C_1)<MinT(C_2)) \lor
(MaxT(C_2)<MinT(C_1))
$$

---

### 2.2.3 Context 步骤互斥

$$
\forall s\in S,\forall C_1,C_2\subseteq S :
(s\in C_1 \land s\in C_2)
\rightarrow
(C_1=C_2)
$$

---

### 2.2.4 步骤到 Context 的归属函数

$$
ContextOf : S \rightharpoonup P(S)
$$

$$
ContextOf(s)=C \iff (s\in C)
$$

Context 标识

$$
CIDOf : S \rightharpoonup CIDSet
$$

$$
CIDOf(s)=CID(ContextOf(s))
$$

---

## 2.3 Context 创建

### 2.3.1 Context 创建函数

$$
F : P(I) \rightarrow P(S)
$$

$$
C = F(I)
$$

约束

$$
\forall I : F(I)\ne \varnothing
$$

---

### 2.3.2 Context 输入约束

I MUST NOT 包含任何在同一 Execution Context 内推理过程中产生的变量。

输入必须来自：

- 外部系统  
- 上下文创建前确定的数据  

---

### 2.3.3 生命周期端点

创建步骤

$$
CreateStep :
\{X\subseteq S \mid X\ne\varnothing\}
\rightarrow S
$$

$$
s_{create}=CreateStep(C)
$$

约束

$$
s_{create}\in C
$$

$$
\forall s\in C :
T(s_{create})\le T(s)
$$

终止步骤

$$
TerminateStep :
\{X\subseteq S \mid X\ne\varnothing\}
\rightarrow S
$$

$$
s_{term}=TerminateStep(C)
$$

约束

$$
s_{term}\in C
$$

$$
\forall s\in C :
T(s)\le T(s_{term})
$$

---

### 2.3.4 Context 唯一标识

$$
CID : P(S) \rightarrow CIDSet
$$

$$
C_1 \ne C_2 \rightarrow
CID(C_1) \ne CID(C_2)
$$

---

## 2.4 Capability Ceiling

$$
G : P(I) \rightarrow P(Cap)
$$

$$
B = G(I)
$$

确定时刻

$$
T_B(C)=T(s_{create}(C))
$$

谓词

$$
DeterminedBefore(B,t)
\iff
T_B(C)<t
$$

---

## 2.5 Ceiling 不可变性

$$
Ceiling : S \rightarrow P(Cap)
$$

$$
\forall I :
let\ C=F(I),B=G(I)\ in
(\forall s\in C :
Ceiling(s)=B)
$$

---

## 2.6 请求集合

$$
Req : S \rightarrow P(Cap)
$$

$$
R_s = Req(s)
$$

约束

$$
\forall s\in S :
R_s\subseteq Cap
$$

---

## 2.7 事件、输出与能力映射

### 2.7.1 事件发生

$$
Occur :
(E\cup O)\times S
\rightarrow
\{\top,\bot\}
$$

$$
T_E : (E\cup O) \rightarrow \mathbb{R}
$$

约束

$$
Occur(x,s)=\top
\rightarrow
T_E(x)=T(s)
$$

唯一落地

$$
\forall x\in(E\cup O)
:\exists! s\in S :
Occur(x,s)=\top
$$

---

### 2.7.2 能力映射

$$
Map :
(E\cup O)
\rightarrow
P(Cap)
$$

约束

$$
Map(x)\ne\varnothing
$$

$$
Map(x)\subseteq Cap
$$

确定性

$$
Map(x)=f(x,obs(x))
$$

---

# 三、验证语义

## 3.1 验证关系

$$
Validation :
CIDSet \times P(Cap) \times S
\rightarrow
\{\top,\bot\}
$$

$$
Valid_C(R,s)=
Validation(CIDOf(s),R,s)
$$

$$
ValidateAt : S \rightarrow \{\top,\bot\}
$$

$$
ValidateAt(s)=\top
\leftrightarrow
Valid_C(R_s,s)=\top
$$

---

# 四、执行约束

## 4.1 请求时序

$$
DeterminedAt : S \rightarrow \{\top,\bot\}
$$

$$
DeterminedAt(s)
\leftrightarrow
(ContextOf(s)\text{ 有定义})
\land
(T(s_{create}(ContextOf(s)))<T(s))
$$

约束

$$
Req(s)\subseteq Ceiling(s)
$$

创建步骤约束

$$
Req(s_{create}(C))=\varnothing
$$

---

# 五、信息流授权

## 5.1 信息流模型

$$
V
$$

$$
AST
$$

$$
Expr : V \rightarrow AST
$$

---

## 5.2 信息依赖

$$
Depends : V \rightarrow P(V)
$$

传递闭包

$$
Depends^{+}(v)
$$

有限性

$$
\forall v\in V :
Depends^{+}(v)
\text{ 为有限集}
$$

---

# 六、外部事件授权

$$
Authorized(e)=
(Map(e)\subseteq B)
\land
(\exists s\in C :
Map(e)\subseteq R_s
\land
Valid_C(R_s,s)=\top)
$$

---

# 七、Scope 与请求约束

$$
Scope \in L
$$

$$
\forall x :
Decide(Scope,x)
\in
\{\top,\bot\}
$$

$$
Scope \subseteq B
$$

$$
\forall s\in C :
Req(s)\subseteq Scope
$$

---

# 八、不可扩展原则

$$
\forall s\in S :
Req(s)\subseteq Ceiling(s)
$$

---

# 九、安全闭环

$$
AuthorizedEvent(e)=
(Map(e)\subseteq B_e)
\land
(\exists s\in C_e :
Map(e)\subseteq Req(s)
\land
Valid_C(Req(s),s)=\top)
$$

---

# 十、模型性质

### 10.1 Ceiling 不可扩展

$$
Req(s)\subseteq Ceiling(s)
$$

### 10.2 副作用可追溯

$$
Occur(e,s)=\top
\rightarrow
\exists s'\in ContextOf(s):
s'\le_S s
\land
Map(e)\subseteq Req(s')
$$

### 10.3 验证先行

$$
Occur(e,s)=\top
\rightarrow
\exists s'\in ContextOf(s):
s'\le_S s
\land
Valid_C(Req(s'),s')=\top
$$

### 10.4 Context 隔离

$$
C_1\ne C_2
\rightarrow
CID(C_1)\ne CID(C_2)
$$

---

# 十一、内部步骤与无外部副作用

$$
InternalStep(s,C)
\leftrightarrow
(s\in C)
\land
\neg\exists x\in(E\cup O):
Occur(x,s)=\top
$$

状态变化约束

$$
InternalStep(s,C)
\land
\Delta\Sigma(s)\ne 0
\rightarrow
(Req(s)\ne\varnothing
\land
Valid_C(Req(s),s)=\top)
$$

---

# 十二、适用边界（v1.0 冻结）

1. 仅覆盖 **非嵌套、非交叠** Context  
2. Map 与 Cause 只要求确定性  
3. 不比较跨执行 Context 等同性  
4. Scope 语言必须可判定  
