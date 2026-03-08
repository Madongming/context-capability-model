# AI Execution Context Authorization Model [中文](./README-cn.md)

AI should not be authorized by **identity**.
AI should be authorized by **Execution Context**.

Traditional authorization model:

**User → Permission → Action**

AI execution process:

**Input → Reasoning → Action**

Because the AI reasoning process and its results are inherently
uncertain, the resulting actions are also uncertain.\
This creates a fundamental mismatch between traditional authorization
models and AI execution behavior.

------------------------------------------------------------------------

## Overview

This project defines an **Execution Context authorization model** for AI
systems.

It is used to describe the authorization relationship between:

-   capability requests generated during AI reasoning
-   validation of those requests
-   externally observable behaviors produced by execution

This is not a product, framework, or implementation guide.

It is a **formal protocol model** for describing authorization during AI
execution.

------------------------------------------------------------------------

## One-Sentence Model

All reasoning steps of AI must be executed within an **Execution
Context**.

    Execution Context
    ├─ Input determined at creation
    ├─ Capability ceiling determined at creation
    ├─ Reasoning steps execute within the Context
    └─ External events must satisfy the capability request and validation relation

------------------------------------------------------------------------

## Why This Model Is Needed

During execution, AI systems may produce behaviors such as:

-   generating new tool calls during the reasoning process
-   producing external operations from reasoning steps
-   continuously expanding behavioral scope through reasoning loops

Traditional identity-based authorization does not map well to this
process, because the main security boundary is no longer a static
identity.

The model proposed here addresses this by introducing the following
constraints:

-   the capability ceiling is determined when the Context is created
-   reasoning steps may only **request** capabilities
-   requests must be validated before authorization is established
-   external events must satisfy the authorization relation

------------------------------------------------------------------------

## Core Properties

The model defines the following core constraints.

### 1. Capability Ceiling Is Fixed

The capability ceiling is determined when the Execution Context is
created:

    B = G(I)

During the Context lifecycle:

    Req(s) ⊆ Ceiling(s)

The capability ceiling cannot be expanded during the reasoning process.

------------------------------------------------------------------------

### 2. External Events Must Satisfy Authorization Conditions

An external event is valid only when the following conditions are
satisfied:

    Map(e) ⊆ B ∧ ∃ s ∈ C : Map(e) ⊆ Req(s) ∧ Valid_C(R_s,s)=⊤

This means that an external behavior is valid only if:

-   it remains within the capability ceiling
-   it corresponds to a capability request made inside the Context
-   that request satisfies the Context validation relation

------------------------------------------------------------------------

### 3. Context Isolation

Different Execution Contexts are mutually independent:

    C₁ ≠ C₂ → CID(C₁) ≠ CID(C₂)

This means authorization cannot be implicitly shared across different
Contexts.

------------------------------------------------------------------------

## What This Specification Defines

Full specification:

-   [**AI Execution Context Authorization Model v1.0**](./ai-execution-context-authorization-model-v1.0.md)

-   [**AI Execution Context Authorization Model v1.0 (Plain Language
    Version)**](./ai-execution-context-authorization-model-v1.0-Plain-Language-Version.md)

This specification defines:

-   Execution Context
-   Capability Ceiling
-   Capability Request
-   Validation Relation
-   External Event Authorization
-   Scope Constraint

------------------------------------------------------------------------

## Project Objective

This model provides a formalized approach for describing **authorization
relationships during the AI reasoning execution process**.

Its objectives include:

-   determining the authorization scope when the Context is created
-   constraining capability requests during the reasoning process
-   defining authorization conditions for external events

------------------------------------------------------------------------

## Positioning

This project is focused on the **authorization model itself**.

It does not provide:

-   engineering interpretation
-   implementation recommendations
-   runtime design requirements

As a protocol model, every permission request requires approval.

In implementation, however, system designers may introduce quantitative
trade-offs between security and usability, including forms of approval
suppression, such as:

-   automatic approval by another AI system
-   approval through a permission feature library using RAG

The protocol model itself remains independent of implementation.

Do not make it like Vista :)
