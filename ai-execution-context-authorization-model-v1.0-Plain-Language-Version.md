# **AI Execution Context Authorization Model (Plain Language Version)**

## **1. What Problem This Protocol Solves**

This model addresses a core problem:

**When is AI allowed to perform “external actions” during execution?**

For example:

- Calling APIs
- Accessing databases
- Writing files
- Sending messages
- Controlling devices

All of these behaviors produce **external side effects**.

Without constraints, AI could:

- Invoke capabilities arbitrarily
- Continuously expand permissions during reasoning
- Produce behaviors that cannot be traced

The goal of this protocol is to ensure:

1. **The capability upper bound of AI is determined at task start**
2. **Capabilities cannot expand during reasoning**
3. **Any external behavior must first request capability**
4. **Capabilities must be validated**
5. **Every external action must be traceable to an authorization**

In simple terms:

> AI may perform extensive reasoning, but only authorized capabilities may affect external state.

## **2. Core Concepts**

There are several key concepts in this model.

### **1 Execution Context**

**Execution Context can be understood as:**

> One complete AI task execution.

For example:

- A user query
- An agent task
- An automated workflow

The task from **start to finish** constitutes a Context.

Characteristics:

- Has a beginning
- Has an end
- Contains a continuous reasoning process internally

Example:

```
User request -> Create Context
AI reasoning
AI invokes capability
AI continues reasoning
Task finished -> Context ends
```

### **2 Reasoning Step (Step)**

AI execution is divided into **multiple steps**.

For example:

```
Step1 reasoning
Step2 reasoning
Step3 request capability
Step4 reasoning
Step5 output
```

The protocol treats the entire execution as:

```
A time sequence
```

Each step has a chronological order.

### **3 Context Isolation**

The protocol specifies:

**Contexts must not interleave.**

Meaning:

```
Context A
---------
step1
step2
step3

Context B
---------
step4
step5
```

The following must not occur:

```
A step1
B step1
A step2
B step2
```

In other words:

**A Context must occupy a continuous time interval.**

This prevents:

- Reasoning contamination
- Authorization confusion
- Unclear responsibility

### **4 Context ID**

Each Context has a unique ID.

Therefore:

- All authorizations
- All capabilities
- All side effects

can be bound to a **specific Context**.

## **3. Capability System**

AI may use many capabilities, such as:

- HTTP requests
- File reading
- File writing
- Databases
- Shell

These capabilities are decomposed into **minimal capability units**:

```
Capability
```

For example:

```
ReadFile
WriteFile
HTTP
SendEmail
```

## **4. Capability Ceiling**

This is one of the core mechanisms of the protocol.

When a **Context is created**:

the system computes a:

```
Capability Ceiling
```

Meaning:

> The maximum capabilities allowed for this task.

Example:

User request:

```
Read logs and summarize
```

The system may produce:

```
Ceiling = {
ReadFile
LLM
}
```

If AI attempts:

```
HTTP request
Delete file
```

it will be rejected.

## **5. Capability Ceiling Is Immutable**

The protocol specifies:

**Once the Ceiling is determined, it cannot change during the entire Context lifecycle.**

That is:

```
At task start:
Ceiling = {A,B,C}

At any later step:
Ceiling must remain {A,B,C}
```

The following is prohibited:

```
Adding permissions later
```

For example:

```
Suddenly allowing shell
Suddenly allowing root
```

All are forbidden.

## **6. Capability Request (Request)**

At each step, AI may **request capabilities**.

For example:

```
Req(step3) = {ReadFile}
```

Meaning:

```
This step intends to use ReadFile
```

The protocol specifies:

```
Requests must belong to the Ceiling
```

That is:

```
Req ⊆ Ceiling
```

Otherwise the request is rejected.

## **7. Validation Mechanism**

After AI requests a capability, **validation** is required.

Meaning:

the system determines:

```
Whether the request is allowed
```

For example:

```
Req = {ReadFile}

Validation system:
Allow / Reject
```

If validation succeeds:

```
Valid = true
```

Otherwise:

```
Valid = false
```

## **8. External Events**

Any behavior that **affects the external world** is called:

```
Event
```

For example:

- HTTP calls
- File writes
- Sending messages
- API calls

The protocol requires:

> Every external event must correspond to an execution step.

Meaning:

```
A specific step triggered the event
```

The following must not occur:

```
Event with unknown origin
```

## **9. Capability Mapping**

Each external event must map to a **capability set**.

For example:

```
HTTP request -> {HTTP}
Write file -> {WriteFile}
```

This allows the system to check:

```
What capabilities are required for this event
```

## **10. External Event Authorization Rules**

An external event **may occur only if the following conditions are satisfied**.

### **Condition 1**

The capabilities required by the event must be within the Ceiling.

That is:

```
Map(event) ⊆ Ceiling
```

### **Condition 2**

There must exist a capability request.

```
Map(event) ⊆ Req(step)
```

### **Condition 3**

The request must have passed validation.

```
Valid = true
```

In simple terms:

**No request → execution not allowed**

**No validation → execution not allowed**

**Exceeding the capability ceiling → execution not allowed**

## **11. Scope**

Scope is:

> The **sub-range of capabilities** allowed in the current Context.

The relationship is:

```
Scope ⊆ Ceiling
Req ⊆ Scope
```

Meaning:

```
Ceiling  = maximum permissions
Scope    = current task range
Req      = current step request
```

The structure is:

```
Req ⊆ Scope ⊆ Ceiling
```

## **12. Information Flow Control**

The protocol also constrains:

**Dependency relationships between variables.**

Meaning:

If variable A depends on variable B:

```
A = f(B)
```

Then the system can trace:

```
The origin of A
```

This prevents:

- Data privilege escalation
- Information contamination

## **13. Security Closure**

The protocol ultimately forms a **security closed loop**.

Any external behavior must satisfy:

```
External behavior
   ↓
Capability mapping
   ↓
Capability request
   ↓
Capability validation
   ↓
Context capability ceiling
```

If any step fails:

```
The action is rejected
```

## **14. Four Properties Guaranteed by the Protocol**

This model guarantees four key security properties.

### **1 Capability Non-Expansion**

AI cannot increase permissions during execution.

### **2 Side-Effect Traceability**

Any external behavior can be traced to:

```
Which Context
Which Step
Which Request
```

### **3 Validation Before Execution**

All capabilities must follow:

```
Validate first
Then execute
```

### **4 Context Isolation**

Between different tasks:

```
Complete isolation
```

Permissions are not shared.

## **15. Difference Between Internal Reasoning and External Behavior**

AI internal reasoning:

```
Allowed freely
```

For example:

- Thinking
- Planning
- Reasoning

These produce **no external side effects**.

However, if reasoning would:

```
Change system state
```

authorization is still required.

## **16. Protocol Scope**

Current v1.0 has some limitations.

It supports only:

```
Non-nested Context
Non-concurrent Context
```

Meaning:

only one task is processed at a time.

If a complex system requires nesting:

it must be decomposed into multiple Contexts.

## **One-Sentence Summary**

The core idea of this protocol is:

**Shift AI permission control from “identity authorization” to “execution context authorization”.**

That is:

```
Permissions bind to tasks
rather than binding to the AI itself
```

This ensures:

- Permissions cannot expand during reasoning
- External behaviors are fully traceable
- Each task is strictly controlled
