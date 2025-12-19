---
title: "Test Mermaid Diagram"
author: "Test"
date: "2025-12-19"
tags: ["test"]
---

# Test Mermaid Diagram

This is a simple test to verify mermaid diagrams are rendering correctly.

## Simple Flow Chart

```mermaid
graph TD
    A[Start] --> B{Is it working?}
    B -->|Yes| C[Great!]
    B -->|No| D[Debug more]
    C --> E[End]
    D --> E[End]
```

## Simple Sequence Diagram

```mermaid
sequenceDiagram
    Alice->>John: Hello John, how are you?
    John-->>Alice: Great!
```

## Regular Code Block (for comparison)

```javascript
function test() {
  console.log("This should have syntax highlighting");
}
```
