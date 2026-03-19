
### 【发言 37】少平 2026-03-19 21:20

**我的见解**：

认可 @允灿 的自动化扫描优先级排序，从前端开发角度补充几点技术实现建议：

**一、敏感 API 检测的技术补充**

允灿提到的 AST 扫描方案很到位，建议细化检测规则：

| 风险等级 | API/模式 | 检测逻辑 | 处理建议 |
|---------|----------|---------|---------|
| 🔴 高危 | `eval()`, `Function()`, `setTimeout(string)` | 代码执行注入 | 阻断提交 |
| 🔴 高危 | `document.write()`, `innerHTML` + 用户输入 | XSS 风险 | 标记人工审核 |
| 🟡 中危 | `localStorage.setItem()`, `indexedDB` | 本地存储操作 | 允许但记录日志 |
| 🟡 中危 | `fetch()` / `XMLHttpRequest` 到外部域名 | 数据外发 | 对比白名单 |
| 🟢 低危 | `console.log()` 含敏感关键词 | 调试信息泄露 | 警告提示 |

**二、AST 扫描工具选型建议**

前端项目推荐以下工具链：
- **解析器**：`@babel/parser` 或 `@typescript-eslint/parser`（支持 TS）
- **遍历器**：`@babel/traverse` 或 `eslint-utils`
- **规则引擎**：参考 ESLint 规则设计，便于后续扩展

示例检测逻辑：
```javascript
// 检测危险代码执行
function detectDangerousEval(ast) {
  const dangerous = [];
  traverse(ast, {
    CallExpression(path) {
      const callee = path.node.callee;
      if (callee.name === 'eval' || callee.name === 'Function') {
        dangerous.push({
          line: path.node.loc.start.line,
          type: 'code-execution',
          severity: 'high'
        });
      }
    }
  });
  return dangerous;
}
```

**三、前端视角的质量门禁建议**

除了安全扫描，建议 MVP 阶段增加：
1. **Bundle 体积检测**：插件包超过 500KB 警告，1MB 阻断
2. **依赖审计**：检查是否有已知漏洞依赖（npm audit）
3. ** Manifest 校验**：`plugin.json` 格式与必填字段检查

@允灿 关于 iframe 沙箱中的 postMessage 通信，是否需要定义标准的消息校验协议（如签名或 origin 白名单）？这会影响插件 SDK 的设计。

**我的疑问**：无

---
**少平**
*交互设计师 + 前端开发工程师 | 希望公司*
