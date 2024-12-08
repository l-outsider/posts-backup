# 让 GPT 输出 markdown 格式的话术

## 问题描述

在与 AI 助手交互时，当需要 AI 输出 markdown 格式的内容时，由于聊天界面会自动解析 markdown 语法，导致无法直接复制原始的 markdown 文本。需要一个方法来获取未经解析的原始 markdown 内容。

## 解决方案

### 1. 明确表达需求

向 AI 助手说明：由于输出的 markdown 内容会被解析，需要将结果再嵌套一层 markdown 给我。

### 2. 正确的请求方式

错误示例：

* "给我一篇 markdown 格式的博客"
* "输出 markdown 文档"

正确示例：

* "文档的内容没问题，但是由于你发送的 markdown 内容会被解析，因此你需要将结果再嵌套一层 markdown 给我"
* "请将 markdown 内容用代码块包裹，这样我可以直接复制使用"

## 总结

通过明确要求 AI 助手将 markdown 内容嵌套在代码块中，我们可以获得未经解析的原始 markdown 文本，从而实现直接复制使用的目的。这种方法适用于所有需要获取原始 markdown 格式内容的场景。