# AI Execution Context Authorization Model

[中文](./README-cn.md)

AI should not be authorized by **identity**. 
AI should be authorized by **Execution Context**.

Traditional authorization model:
User → Permission → Action

AI execution process:
Input → Reasoning → Action

Because the AI reasoning process and its results are:
uncontrollable, this leads to uncertainty in Action, and the traditional authorization model cannot match it.

This project defines an **Execution Context authorization model**,  
used to describe the authorization relationship between capability requests in the AI reasoning process and external behaviors.

## **One-Sentence Model**

All reasoning steps of AI must be executed within an **Execution Context**.

```
Execution Context
 ├─ Input determined at creation
 ├─ Capability ceiling determined at creation
 ├─ Reasoning steps execute within the Context
 └─ External events must satisfy the capability request and validation relation
```

## **Problems Solved by the Model**

During execution, AI systems may produce the following situations:

- Generating new tool calls during the reasoning process
- Reasoning steps producing external operations
- Reasoning loops continuously expanding the scope of behavior

This model constrains such situations through the following mechanisms:

- Capability ceiling is determined when the Context is created
- Reasoning steps can only request capabilities
- Requests must be validated
- External events must satisfy authorization conditions

## **Core Properties**

The model defines the following constraints.

#### **1 Capability Ceiling Is Fixed**

The capability ceiling is determined when the Execution Context is created:

```
B = G(I)
```

During the Context lifecycle:

```
Req(s) ⊆ Ceiling(s)
```

The capability ceiling cannot be expanded during the reasoning process.

#### **2 External Events Must Satisfy Authorization Conditions**

An external event is valid only when the following conditions are satisfied:

```
Map(e) ⊆ B
∧
∃ s ∈ C :
Map(e) ⊆ Req(s)
∧
Valid_C(R_s,s)=⊤
```

#### **3 Context Isolation**

Different Execution Contexts are mutually independent:

```
C₁ ≠ C₂ → CID(C₁) ≠ CID(C₂)
```

## **Protocol Specification**

Full specification:

[**AI Execution Context Authorization Model v1.0**](./ai-execution-context-authorization-model-v1.0.md)

[**AI Execution Context Authorization Model v1.0 (Plain Language Version)**](./ai-execution-context-authorization-model-v1.0-Plain-Language-Version.md)

This specification defines:

- Execution Context
- Capability Ceiling
- Capability Request
- Validation Relation
- External Event Authorization
- Scope Constraint

## **Project Objective**

This model provides a formalized approach to describe **authorization relationships during the AI reasoning execution process**.

Objectives include:

- Determining the authorization scope when the Context is created
- Constraining capability requests during the reasoning process
- Defining authorization conditions for external events

Finally:

I do not provide engineering interpretation and do not give implementation recommendations. I will only mention one point: as a protocol, every permission request requires approval. However, in implementation, model capabilities, engineering methods, and quantitative trade-offs between security and usability can be used. According to requirements, an appropriate balance can be configured to perform “approval suppression”, such as automatic approval via another AI or a permission feature library using RAG. Do not make it like Vista :)
