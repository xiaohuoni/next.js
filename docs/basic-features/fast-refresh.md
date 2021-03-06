---
description: Next.js' 快速刷新是一个新的热加载的体验，在你编辑 React 组件的时候，给你即时反馈。
---

# 快速刷新

“快速刷新”是一个 Next.js 特性，它为你提供对 React 组件所做的编辑的即时反馈。默认情况下在 **9.4 或更高版本**的所有 Next.js 应用程序中都启用了快速刷新。启用 Next.js 快速刷新后，大多数编辑应该在一秒钟内可见，而且**不会丢失组件状态**。

## 它是如何工作的

- 如果编辑一个只导出React组件的文件，则快速刷新将只更新该文件的代码，然后重新呈现组件。你可以编辑该文件中的任何内容，包括样式、呈现逻辑、事件处理程序或 effects。
- 如果你编辑一个导出的文件_不是_ React 组件，那么快速刷新将重新运行这两个文件，和其他导入它的文件。因此，如果`Button.js` 和 `Modal.js` 导入 `theme.js`，编辑 `theme.js` 就会更新这两个组件。
- 最后，如果你**编辑文件**是**由 React 树之外的文件导入**，快速刷新**将回落到做一个完整的热重装**。你可能有一个文件，它渲染了一个 React 组件，但也导出了一个由**非 React 组件**导入的值。例如，也许你的组件还导出一个常量，一个非 React 实用程序文件导入了它。在这种情况下，考虑将常量迁移到一个单独的文件并分别从这两个文件中导入它。这将重新启用“快速刷新”功能。其他情况下可以通常以类似的方式解决。

## 错误恢复能力

### 语法错误

如果在开发过程中出现语法错误，可以修复它并再次保存文件。错误将自动消失，因此你不需要重启项目。**你不会丢失组件状态**。

### 运行时错误

如果你犯了一个错误，导致组件内部出现运行时错误，你将会收到一个上下文复盖。修复错误将自动取消复盖，而不重新加载应用程序。

如果在呈现期间没有发生错误，组件状态将被保留。如果在渲染过程中发生错误，React 将使用更新的代码重新安装应用程序。

如果你的应用有[错误的边界](https://reactjs.org/docs/error-boundaries.html)（这是一个很好的主意，在生产中优雅的失败），他们将在下一次编辑时重试渲染。这意味着有一个错误边界可以防止你总是得到重置到初始的应用程序状态。但是，请记住，错误边界_不_应该太细。在生产过程中，它们被 React 使用，并且应该被故意设计。

## 局限性

“快速刷新”尝试在正在编辑的组件中保留本地 React 状态，但前提是这样做是安全的。这里有几个原因可以解释为什么在每次编辑时都会重置本地状态:

- 类组件不保留本地状态（只有函数组件和钩子保留状态）。
- 你正在编辑的文件除了 React 组件外可能还有_其他_导出。
- 有时，一个文件会输出调用高阶组件的结果，如 `higherOrderComponent(WrappedComponent)`。如果返回的组件是一个类，则将重置状态。

随着更多的代码库移动到函数组件和 Hooks，你可以期望在更多的情况下保留状态。

## 提示

- 默认情况下，快速刷新会在函数组件(和钩子)中响应本地状态。
- 有时你可能有希望强制重置状态和需要要重新挂载的组件。例如，如果你正在调整仅在挂载上发生的动画，这会非常方便。为此，你可以在正在编辑的文件中的任何位置添加 `// @refresh reset` 。这个指令是该文件的本地指令，并指示“快速刷新”在每次编辑时重新装入该文件中定义的组件。
- 你可以在开发过程中将 `console.log` 或 `debugger;` 放入你编辑的组件中。

## 快速刷新和 Hooks

在可能的情况下，快速刷新尝试在编辑之间保留组件的状态。特别是，只要不更改 `useState` 和 `useRef` 的参数或调用的顺序。

带有依赖项的钩子 — 比如 `useEffect`、 `useMemo` 和 `useCallback` - 在快速刷新期间_总是_更新。当快速刷新发生时，它们的依赖项列表将被忽略。

例如，当你编辑`useMemo(() => x * 2, [x])` 成 `useMemo(() => x * 10, [x])`，即使 `x`（依赖项）没有改变，它也会重新运行。如果 React 没有这样做，你的编辑就不会显示出来！

有时，这会导致意想不到的结果。例如，即使是带有依赖项的空数组的 `useEffect` 在快速刷新期间仍然会重新运行一次。

然而，即使没有快速刷新，编写能够偶尔重新运行 `useEffect` 的代码也是一个很好的实践。这将使你更容易在以后引入新的依赖项。它是由 [React 严格模式](/docs/api-reference/next.config.js/react-strict-mode)执行的，我们强烈建议启用它。

