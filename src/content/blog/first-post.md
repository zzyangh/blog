---
title: 'TypeScript枚举的告别与反思'
description: '探讨TypeScript枚举（Enum）的争议与未来发展方向'
pubDate: 'Mar 01 2025'
heroImage: '/blog-placeholder-3.jpg'
---

##### TypeScript枚举的告别与反思

> 原文链接：[https://blog.disintegrator.dev/posts/ode-to-typescript-enums/](https://blog.disintegrator.dev/posts/ode-to-typescript-enums/)  
> 主题：不要再使用ts Enum

##### 枚举（Enum）的争议

**社区趋势**  
主流观点推荐优先使用字面量联合类型（如 `type X = "A" | "B"`），因其更轻量、符合人体工程学。

**Enum独有优势**  
1. **文档化支持**: Enum 成员可附加详细注释（包括 `@deprecated`），且 IDE 提示更直观
2. **代码可维护性**: 在大型代码库中，Enum 的强类型和集中化管理更不易出错

##### 字面量联合的局限性

- 联合类型的成员无法单独注释，文档需写在类型定义处，导致上下文信息缺失
- 示例对比：

```typescript
// Enum 支持成员级注释
enum PaymentMethod {
  /** @deprecated */
  Check = "check"
}

// 联合类型注释集中在类型定义
type PaymentMethod = "check" /** @deprecated */ | "credit" | "debit";
```

##### TypeScript 5.8 的变革

- 引入 `--erasableSyntaxOnly` 编译标志，禁止使用 Enum 和 Namespace 等特性
- 这些特性会被转译为 JS 对象，无法直接运行 TS 文件
- Node.js v23、Deno、Bun 已支持无需构建步骤直接运行 TS，但需遵守 "可擦除语法" 规则

##### 可擦除语法

在 TypeScript 开发中，"可擦除语法"（Eraseable Syntax）是指那些在编译为 JavaScript 时可以被完全移除而不会影响运行时行为的语法特性。这是 Node.js v23、Deno、Bun 等运行时支持直接执行 TypeScript 代码的核心前提条件。以下是关键要点：

**核心特征**

无运行时影响：这些语法元素在编译为 JavaScript 后不会留下任何运行时痕迹，例如：
- 类型注解：`let x: number = 1;` → `let x = 1;`
- 接口/类型别名：`interface User { ... }` → 完全移除
- 泛型参数：`function id<T>(x: T) { ... }` → `function id(x) { ... }`

静态检查专用：这些语法仅用于 TypeScript 的类型检查阶段，不涉及 JavaScript 的运行时逻辑。

**与非可擦除语法的对比**

| 可擦除语法 | 非可擦除语法 |
|----------|------------|
| 类型注解 | 装饰器 |
| 接口 | 枚举 |
| 类型别名 | 命名空间 |
| 泛型参数 | 实验性语法 |

非可擦除语法需要生成额外的 JavaScript 代码或依赖运行时支持，因此无法直接执行。

**运行时支持原理**

即时转换（JIT Transpile）：运行时（如 Deno/Bun）内部通过快速转换工具（如 swc 或 esbuild）将 TypeScript 转换为 JavaScript，但仅处理可擦除语法。

限制条件：若代码包含非可擦除语法（如装饰器），运行时可能：
- 直接报错（默认行为）
- 要求显式配置（如 Deno 需启用 `--unstable` 标志）

**代码示例**

可擦除语法（允许）：
```typescript
// 类型注解会被擦除
function sum(a: number, b: number): number {
  return a + b;
}

// 接口和泛型会被完全移除
interface Result<T> {
  data: T;
}
const res: Result<string> = { data: "ok" };
```

不可擦除语法（禁止或需额外配置）：
```typescript
// 装饰器需要生成运行时代码
@Controller("/user")  // ❌ 报错（除非配置支持）
class UserController {}

// 枚举会生成 IIFE 代码
enum Status {         // ❌ 可能报错
  Success = 200
}
```

##### 结语

尽管 Enum 终将淡出，但其在工程实践中的优势值得被铭记，期待语言特性进一步演进。
