# AI 执行上下文授权模型 (AI Execution Context Authorization Model) [English](./README.md)

AI 不应该通过 **身份（Identity）** 被授权。
AI 应该通过 **执行上下文（Execution Context）** 被授权。

传统授权模型：

**User → Permission → Action**

AI 的执行过程：

**Input → Reasoning → Action**

由于 AI
的推理过程及其结果本质上具有不确定性，因此最终产生的行为同样具有不确定性。\
这导致传统授权模型与 AI 执行行为之间产生根本性的结构不匹配。

------------------------------------------------------------------------

## 概述

本项目定义了一种用于 AI 系统的 **执行上下文授权模型（Execution Context
Authorization Model）**。

该模型用于描述以下关系：

-   AI 推理过程中产生的能力请求（Capability Requests）
-   对这些请求的验证关系
-   执行过程中产生的外部可观察行为

本项目 **不是产品、不是框架、也不是实现指南**。

它是一个用于描述 **AI 执行授权关系的形式化协议模型（Formal Protocol
Model）**。

------------------------------------------------------------------------

## 一句话模型

AI 的所有推理步骤都必须在 **Execution Context（执行上下文）** 中执行。

    Execution Context
    ├─ 创建时确定输入
    ├─ 创建时确定能力上限
    ├─ 推理步骤在 Context 内执行
    └─ 外部事件必须满足 capability request 与 validation relation

------------------------------------------------------------------------

## 为什么需要这个模型

在执行过程中，AI 系统可能会产生如下行为：

-   在推理过程中生成新的工具调用
-   从推理步骤产生外部操作
-   通过推理循环不断扩展行为范围

传统的基于身份（Identity）的授权模型无法很好地描述这种过程，因为真正的安全边界不再是静态身份。

本模型通过以下约束解决这一问题：

-   能力上限在 Context 创建时确定
-   推理步骤只能 **请求（request）能力**
-   请求必须在授权建立之前被验证
-   外部事件必须满足授权关系

------------------------------------------------------------------------

## 核心性质

该模型定义了以下核心约束。

### 1. Capability Ceiling 是固定的

能力上限在 Execution Context 创建时确定：

    B = G(I)

在 Context 生命周期内：

    Req(s) ⊆ Ceiling(s)

能力上限在推理过程中 **不能被扩展**。

------------------------------------------------------------------------

### 2. 外部事件必须满足授权条件

一个外部事件只有在满足以下条件时才是合法的：

    Map(e) ⊆ B ∧ ∃ s ∈ C : Map(e) ⊆ Req(s) ∧ Valid_C(R_s,s)=⊤

这意味着一个外部行为只有在以下条件成立时才有效：

-   行为仍然位于 capability ceiling 之内
-   它对应 Context 内某个推理步骤提出的 capability request
-   该请求满足 Context 的 validation relation

------------------------------------------------------------------------

### 3. Context 隔离

不同的 Execution Context 必须相互独立：

    C₁ ≠ C₂ → CID(C₁) ≠ CID(C₂)

这意味着授权不能在不同 Context 之间隐式共享。

------------------------------------------------------------------------

## 本规范定义的内容

完整规范：

-   **AI Execution Context Authorization Model v1.0**\
    `./ai-execution-context-authorization-model-v1.0.md`

-   **AI Execution Context Authorization Model v1.0（Plain Language
    Version）**\
    `./ai-execution-context-authorization-model-v1.0-Plain-Language-Version.md`

该规范定义了：

-   Execution Context
-   Capability Ceiling
-   Capability Request
-   Validation Relation
-   External Event Authorization
-   Scope Constraint

------------------------------------------------------------------------

## 项目目标

该模型为 **AI 推理执行过程中的授权关系** 提供一种形式化描述方法。

其目标包括：

-   在 Context 创建时确定授权范围
-   在推理过程中约束 capability request
-   为外部事件定义授权条件

------------------------------------------------------------------------

## 定位

本项目专注于 **授权模型本身**。

它不提供：

-   工程实现解释
-   具体实现建议
-   runtime 设计要求

作为协议模型，每一个权限请求都需要获得批准。

但在实际实现中，系统设计者可能会在安全性与可用性之间进行量化权衡，例如：

-   由另一个 AI 系统自动批准
-   通过基于 RAG 的权限特征库进行自动批准

协议模型本身 **独立于具体实现**。

不要把它做成 Vista :)
