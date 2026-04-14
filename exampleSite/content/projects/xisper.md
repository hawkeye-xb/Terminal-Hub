---
title: "Xisper"
description: "AI 驱动的语音输入工具，支持 20+ 语言，延迟 < 200ms"
status: "stable"
version: "1.2.0"
tech: ["Vue", "TypeScript", "OpenAI", "Cloudflare Workers"]
weight: 1
links:
  - name: "Demo"
    url: "https://xisper.app"
  - name: "GitHub"
    url: "https://github.com/user/xisper"
---

## 特点

- 高精度语音识别
- 极速响应（< 200ms）
- 支持 20+ 种语言
- 完全在边缘节点运行

## 技术架构

前端使用 Vue 3 + TypeScript，后端跑在 Cloudflare Workers 上，调用 OpenAI Whisper API 进行语音转文字。

整个系统没有传统服务器，延迟极低。
