---
title: "Vue 3 Composition API 实战"
date: 2026-04-08
tags: ["Vue", "JavaScript", "教程"]
---

从 Options API 迁移到 Composition API 的实战指南。

## 为什么迁移

- 更好的类型推断
- 更灵活的代码组织
- 更容易复用逻辑

## 基本用法

```javascript
import { ref, computed } from 'vue'

const count = ref(0)
const double = computed(() => count.value * 2)
```

## 组合函数

把可复用的逻辑抽成 `useXxx` 函数，替代 mixins。
