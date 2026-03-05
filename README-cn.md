# AI 执行上下文授权模型

[English](./README.md)

AI 不应该按 **身份** 授权。  
AI 应该按 **执行上下文（Execution Context）** 授权。

传统权限模型：
User → Permission → Action

AI 的执行过程：
Input → Reasoning → Action

因为AI Reasoning的过程和结果：
不可控，这导致Action不确定，传统的授权模型无法匹配

本项目定义一种 **Execution Context 授权模型**，  
用于描述 AI 推理过程中的能力请求与外部行为之间的授权关系。

## **一句话模型**

AI 的所有推理步骤必须在 **Execution Context** 内执行。

```
Execution Context
 ├─ 创建时确定输入
 ├─ 创建时确定能力上限
 ├─ 推理步骤在 Context 内进行
 └─ 外部事件必须满足能力请求与验证关系
```

## **模型解决的问题**

AI 系统在执行过程中可能产生以下情况：

- 推理过程中生成新的工具调用
- 推理步骤产生外部操作
- 推理循环持续扩展行为范围

该模型通过以下机制进行约束：

- Context 创建时确定能力上限
- 推理步骤只能请求能力
- 请求必须经过验证
- 外部事件必须满足授权条件

## **核心性质**

该模型定义了以下约束。

#### **1 能力上限固定**

Execution Context 创建时确定能力上限：

```
B = G(I)
```

在 Context 生命周期内：

```
Req(s) ⊆ Ceiling(s)
```

推理过程中不能扩展能力上限。

#### **2 外部事件必须满足授权条件**

外部事件只有在满足以下条件时才成立：

```
Map(e) ⊆ B
∧
∃ s ∈ C :
Map(e) ⊆ Req(s)
∧
Valid_C(R_s,s)=⊤
```

#### **3 Context 隔离**

不同 Execution Context 之间互相独立：

```
C₁ ≠ C₂ → CID(C₁) ≠ CID(C₂)
```

## **协议规范**

完整规范：

[**AI 执行上下文授权模型 v1.0**](./ai-execution-context-authorization-model-v1.0-cn.md)

[**AI 执行上下文授权模型 v1.0（白话版）**](./ai-execution-context-authorization-model-v1.0-Plain-Language-Version-cn.md)

该规范定义：

- Execution Context
- Capability Ceiling
- 能力请求
- 验证关系
- 外部事件授权
- Scope 约束

## **项目目标**

本模型提供一种形式化方式，用于描述 **AI 推理执行过程中的授权关系**。

目标包括：

- 在 Context 创建时确定授权范围
- 在推理过程中约束能力请求
- 为外部事件定义授权条件

最后：

我不做工程解读，不做实现推荐。只提一点，作为协议，会规定每一次权限请求，都需要审批。但是实现的时候可以利用模型能力，工程方法，安全性和易用度的量化，并根据需求设置合适的平衡，进行“审批抑制”。如另一个ai审批或权限特征库的RAG，等等方法来自动审批。不要做的像Vista :)
