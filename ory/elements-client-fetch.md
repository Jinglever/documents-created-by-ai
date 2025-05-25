# 使用 Ory Elements 和 React + Vite 构建 Ory Kratos 自定义 UI 分步指南

## 第一部分：Ory Kratos、"Bring Your Own UI" 与 Ory Elements 简介

### 1.1 Ory Kratos 及其无头 (Headless) 特性概述

Ory Kratos 是一个以 API 为优先的身份管理、认证和授权系统 [1]。其核心设计理念之一是“无头”（Headless），这意味着 Ory Kratos 本身不提供预构建的用户界面 (UI) 或 HTML 模板引擎 [2]。它专注于提供强大的后端身份验证逻辑和 API 端点，而将用户界面的设计和实现完全交给开发者。

这种无头特性为开发者带来了极大的灵活性，允许他们根据自身业务需求和品牌风格完全控制用户体验 (UX) 和界面设计。然而，这也意味着开发者需要自行构建所有面向用户的界面，包括登录、注册、账户设置、密码恢复等。Ory Kratos 的这种设计使其能够轻松集成到任何前端技术栈或现有应用中，无论是 Web 应用、移动应用还是其他类型的客户端。

### 1.2 "Bring Your Own UI" (BYOUI) 概念解析

"Bring Your Own UI" (BYOUI) 是 Ory Kratos 设计哲学的直接体现 [1]。它强调开发者可以携带并集成自己定制的用户界面，而不是被限制在由身份提供商预设的、可能不符合其应用风格的界面中。Ory Identities (Kratos 的一部分) 的设计目标之一就是简化 UI 自定义过程，它会为表单（如登录、注册表单）准备所有必要的数据和结构信息 [1]。

BYOUI 的核心优势在于其无与伦比的灵活性。开发者可以：

*   **完全掌控品牌和用户体验**：确保认证流程与应用程序的整体设计语言和用户交互模式无缝融合。
*   **选择任何前端技术**：无论是 React, Vue, Angular, Svelte，还是原生移动开发，都可以与 Kratos 的 API 对接。
*   **逐步集成或替换**：可以在现有应用中逐步引入 Kratos 的认证功能，而无需一次性替换整个认证系统。

Kratos 通过其 API 提供流程状态和 UI 描述（通常是 JSON 格式），前端应用负责解析这些信息并渲染成用户可见的表单和元素。

### 1.3 Ory Elements：简化 React UI 开发的利器

为了应对构建自定义 UI 的挑战，特别是对于使用 React 的开发者，Ory 社区推出了 Ory Elements。Ory Elements 是一套专为 Ory 生态系统（包括 Kratos 和 Hydra）设计的 UI 组件库，旨在显著简化在 React 应用中构建登录、注册、账户管理等页面的过程 [4]。

目前，强烈推荐使用其针对 React 的特定包 `@ory/elements-react` [6]。该库的主要特性包括：

*   **模块化**：开发者可以只选择和使用他们需要的组件。
*   **可定制化**：组件的外观和行为可以在一定程度上进行调整。
*   **主题化**：支持通过主题提供者 (ThemeProvider) 和覆盖 (themeOverrides) 来匹配应用的设计风格 [5]。
*   **动态适应**：Ory Elements 组件能够根据 Ory Kratos 的配置（例如 Identity Schema、启用的登录方式如社交登录或无密码登录）动态地渲染和调整表单字段和选项 [5]。

Ory Kratos 的 API 驱动设计是其灵活性的根源，但也带来了实现用户界面的工作量。Ory Elements 的出现，正是为了填补这一空白，它将 Kratos 返回的抽象 UI 描述（节点）转化为具体的 React 组件，从而大幅降低了前端的集成复杂度和开发时间。如果 Kratos 自带了固定的 UI，那么像 Ory Elements 这样的库的需求就会大大降低。使用 Ory Elements 意味着开发者在享受便利性的同时，也在一定程度上接受了其提供的设计模式和抽象层。虽然它提供了定制能力，但对于追求极致自由度的 UI 设计，开发者可能仍需考虑不使用 Elements，直接与 Kratos API 交互并手动渲染所有 UI 节点。本指南将聚焦于如何有效利用 Ory Elements 来构建 UI。

## 第二部分：准备工作与项目初始化

在开始构建自定义 UI 之前，需要完成一系列准备工作，包括设置 Ory Kratos 实例、初始化 React 项目、安装必要的依赖以及配置开发环境。

### 2.1 Ory Kratos 实例准备

首先，需要一个正在运行的 Ory Kratos 实例。开发者可以根据需求选择以下两种方式之一：

*   **Ory Network (云服务)**：这是 Ory 官方提供的托管服务。使用 Ory Network，开发者无需自行部署和维护 Kratos 实例。只需在 Ory Console (<https://console.ory.sh>) 创建一个账户和一个新项目即可 [3]。这种方式对于快速启动和原型开发非常方便。
*   **自托管 Ory Kratos**：开发者也可以选择在自己的服务器或本地环境中使用 Docker 部署 Ory Kratos [2]。这提供了对 Kratos 配置和数据的完全控制，但需要更多的运维知识。自托管通常涉及配置数据库连接 (DSN) 和身份模型的默认 Schema ID 等。

无论选择哪种方式，确保 Kratos 实例可以正常访问是后续开发的前提。

### 2.2 React + Vite 项目初始化

本指南针对 React 和 Vite 技术栈。使用 Vite 可以快速创建一个现代化的 React 项目，并享受其带来的优秀开发体验，如极速的冷启动和模块热替换 (HMR)。

可以使用以下命令初始化一个新的 React + TypeScript 项目 [8]：

```bash
npm create vite@latest your-ory-ui-app -- --template react-ts
cd your-ory-ui-app
```

将 `your-ory-ui-app` 替换为你的项目名称。

### 2.3 安装 Ory 相关依赖

接下来，需要安装与 Ory Kratos 集成和使用 Ory Elements 构建 UI 的核心库：

*   `@ory/elements-react`: 这是 Ory Elements 专为 React 提供的组件库 [4]。

```bash
npm install @ory/elements-react react react-dom
```

*   `@ory/client-fetch`: 这是 Ory Kratos 的官方 JavaScript/TypeScript SDK，用于在前端与 Kratos API 进行通信 [4]。

```bash
npm install @ory/client-fetch
```

*   （可选）`@ory/elements-test`: 如果需要编写针对 Ory Elements UI 的测试，可以安装此包 [5]。

这些依赖是构建交互式、动态认证界面的基础。

### 2.4 配置 Ory Kratos UI URL

Ory Kratos 在处理各种自服务流程（如登录、注册）时，会在特定阶段将用户重定向到预定义的 UI 页面。因此，必须在 Kratos 的配置中指定这些自定义 UI 页面的 URL [2]。

配置方式取决于 Kratos 实例的部署方式：

*   **Ory Network**：可以在 Ory Console 的项目设置中，找到 "User Interface" 或类似部分，为每个流程（登录、注册、设置、恢复、验证等）配置对应的 UI URL [3]。例如，登录 UI 可能配置为 `http://localhost:5173/login` (假设 Vite 开发服务器运行在 5173 端口)。
*   **自托管 Kratos**：可以直接修改 Kratos 的配置文件 (通常是 `kratos.yml`)，在 `selfservice.flows` 下为每个流程类型（如 `login`, `registration`）设置 `ui_url` [2]。也可以通过 Ory CLI 使用 `patch` 命令更新项目配置 [3]。

正确的 UI URL 配置是确保 Kratos 流程能够正确引导用户到自定义界面的关键。如果这些 URL 配置不正确，用户在尝试登录或注册等操作时可能会遇到重定向错误或看到 Ory 的默认错误页面。

### 2.5 本地开发环境：Ory CLI 与 Ory Tunnel

Ory Kratos 的安全模型严重依赖 HTTP Cookie（例如会话 Cookie 和 CSRF Cookie）[2]。然而，在本地开发 SPA (Single Page Application) 时，前端应用（例如运行在 `http://localhost:5173`）和 Kratos API（如果使用 Ory Network，则为其云端地址；如果自托管在 Docker 中，可能为 `http://127.0.0.1:4433`）通常不在同一个域。这会导致浏览器的同源策略阻止 Cookie 的正确发送和接收，从而破坏认证流程。

为了解决这个问题，Ory 提供了 Ory CLI 和 Ory Tunnel 工具 [5]：

1.  **安装 Ory CLI**：根据官方文档安装 Ory CLI [8]。
2.  **运行 Ory Tunnel**：在项目根目录下，打开一个新的终端窗口，运行以下命令 [5]：

    ```bash
    # 将 <your-project-slug> 替换为 Ory Network 项目的 slug
    # 将 http://localhost:5173 替换为 Vite 开发服务器的实际地址和端口
    ory tunnel http://localhost:5173 --project <your-project-slug> --dev
    ```

    如果自托管 Kratos，并且 Kratos API 监听在例如 `http://127.0.0.1:4433`，则 Tunnel 命令可能需要调整为指向该 Kratos 实例。然而，更常见的本地开发模式是 UI 通过 Tunnel 连接到 Ory Network 或一个配置为接受来自 Tunnel 的请求的本地 Kratos。
    Ory Tunnel 会启动一个本地代理，将 Ory Kratos API 镜像到本地的一个端口（通常是 `http://localhost:4000`）。同时，它会智能地重写 Cookie 的 Domain 和 Path 属性，使其与你的本地前端应用（如 `http://localhost:5173`）兼容 [8]。
3.  **配置 SDK basePath**：在 React 应用中，需要将 `@ory/client-fetch` SDK 的 `basePath` 配置为 Ory Tunnel 提供的本地镜像地址。这通常通过 Vite 的环境变量来实现。在项目根目录创建 `.env.local` 文件，并添加：

    ```
    VITE_ORY_SDK_URL=http://localhost:4000
    ```

    然后在 SDK 初始化代码中使用此环境变量 [5]：

    ```typescript
    // src/ory.ts 或类似文件
    import { FrontendApi, Configuration } from '@ory/client-fetch';

    const orySdk = new FrontendApi(
      new Configuration({
        basePath: import.meta.env.VITE_ORY_SDK_URL as string,
        credentials: 'include', // 确保 Cookie 被发送
      })
    );

    export default orySdk;
    ```

重要提示：在开发过程中，务必始终一致地使用 `127.0.0.1` 或 `localhost` 作为主机名，不要混用。浏览器会将它们视为不同的域，从而导致 Cookie 问题 [2]。

正确的初始设置，特别是 UI URL 的配置和本地开发环境中 Ory Tunnel 的使用，对于顺利集成 Ory Kratos 至关重要。Kratos 对 Cookie 的依赖性，决定了在 SPA 开发中必须仔细处理 CORS 和凭证（`credentials: "include"`）。Ory Tunnel 正是针对本地开发环境解决这一系列问题的有效方案。如果忽视 Tunnel 或未正确配置 `VITE_ORY_SDK_URL`，开发者在本地将很可能遇到难以追踪的 Cookie 和重定向问题。

## 第三部分：Ory Kratos UI 集成核心概念

在深入到具体的 UI 组件开发之前，理解 Ory Kratos UI 集成背后的一些核心概念至关重要。这些概念构成了与 Kratos API 交互的基础，并直接影响如何使用 Ory Elements。

### 3.1 理解 Ory Kratos 自服务流程 (Flows)

Ory Kratos 的功能围绕一系列“自服务流程” (Self-Service Flows) 构建。这些流程代表了用户可以独立完成的身份验证操作，例如：

*   登录 (Login) [10]
*   注册 (Registration) [10]
*   账户设置 (Settings) [10]
*   账户恢复 (Recovery) [10]
*   邮件/账户验证 (Verification) [10]

每个流程都有一个明确的生命周期和一系列 API 端点与之对应。一个典型的流程会经历以下五个主要操作阶段 [10]：

1.  **创建/初始化流程 (Creating the flow)**：前端应用向 Kratos 发起请求，以启动特定类型的流程（例如，用户访问登录页面时初始化登录流程）。
2.  **获取流程数据并渲染 UI (Rendering the UI)**：Kratos 返回一个包含当前流程状态和 UI 定义的 JSON 对象。前端应用（或 Ory Elements 组件）使用这些数据来动态展示用户界面（例如，表单字段、按钮、消息）。
3.  **提交流程数据 (Submitting the flow)**：用户在 UI 中输入信息（例如，用户名和密码）并提交表单。前端应用将这些数据发送回 Kratos。
4.  **处理错误 (Handling errors)**：如果用户输入无效或发生其他错误，Kratos 会返回更新后的流程数据，其中包含错误信息。UI 需要重新渲染以显示这些错误，并允许用户修正。
5.  **处理成功提交 (Handling successful submission)**：如果流程成功完成（例如，用户成功登录），Kratos 会执行相应的操作（例如，设置会话 Cookie）并通常会指示前端进行重定向或返回成功状态。

前端 UI 的开发工作本质上就是围绕这些流程的各个阶段展开的。理解每个流程的目的、其可能的中间状态以及与之交互的 API 端点，是构建健壮 UI 的基础。

### 3.2 Kratos UI 有效载荷 (Payload) 结构

当 Kratos 初始化或更新一个流程时，其 API 响应的核心部分是一个 JSON 对象，通常位于响应体内的 `ui` 字段 (例如 `flow.ui`) [1]。这个对象包含了前端渲染用户界面所需的所有信息。其主要结构包括：

*   `action` (字符串)：表单应该提交到的目标 URL [1]。
*   `method` (字符串)：表单提交应该使用的 HTTP 方法，通常是 `POST` [1]。
*   `nodes` (数组)：这是一个 UI 节点的数组，是 UI Payload 中最关键的部分。每个节点对象代表一个具体的 UI 元素或信息展示 [1]。常见的节点属性包括：
    *   `type` (字符串)：节点的类型，例如 `input` (输入框), `script` (脚本), `img` (图片), `text` (文本信息)。
    *   `group` (字符串)：节点所属的组，例如 `default` (通常包含 CSRF 令牌), `password` (密码登录相关字段), `oidc` (社交登录按钮), `profile` (用户资料字段) [1]。这允许对不同类型的 UI 元素进行分组、过滤或特殊渲染。
    *   `attributes` (对象)：包含该节点渲染为 HTML 元素时所需的各种属性，例如：
        *   `name` (字符串)：表单字段的名称，提交时使用。
        *   `type` (字符串)：对于 `input` 节点，表示输入类型，如 `text`, `password`, `email`, `hidden`, `checkbox`, `submit`, `button`。
        *   `value` (任意类型)：字段的当前值或默认值。
        *   `required` (布尔值)：字段是否必填。
        *   `disabled` (布尔值)：字段是否禁用。
        *   `onclick` (字符串)：对于 `button` 类型节点，可能包含 JavaScript 代码片段（例如用于 WebAuthn）[1]。
        *   `src`, `async`, `integrity` 等：对于 `script` 或 `img` 节点。
    *   `messages` (数组)：针对该特定节点的消息（例如，字段验证错误）。每个消息对象通常包含 `id`, `text`, `type` (如 `error`, `info`) 和 `context` [1]。
    *   `meta` (对象)：包含元信息，例如 `label` 对象，其中包含节点的标签文本和类型 [1]。
*   `messages` (数组，位于 `ui` 对象顶层)：流程级别的全局消息，例如整个表单提交失败的通用错误提示 [1]。

Ory Elements 组件的核心功能之一就是解析这个 `flow.ui` 对象，特别是 `nodes` 数组，并将其动态地、准确地渲染成用户可交互的 React 组件。理解这个 Payload 结构，对于调试问题以及在需要时进行更高级的 UI 定制非常有帮助。例如，`flow.ui.nodes` 的设计直接影响了 Ory Elements 组件的内部实现方式；Elements 组件在很大程度上扮演了这些 `nodes` 的智能渲染器和交互处理器的角色。

### 3.3 @ory/client-fetch SDK 的角色与配置

`@ory/client-fetch` SDK 是 React 应用与 Ory Kratos 后端 API 进行通信的官方桥梁 [16]。它封装了底层的 HTTP 请求、错误处理和类型定义，使得前端开发者可以更便捷地与 Kratos 交互。

**核心配置与使用**：

1.  **初始化 `FrontendApi`**： 首先需要创建 `FrontendApi` 的一个实例。这通常在一个单独的文件中完成，以便在整个应用中复用。

    ```typescript
    // src/ory.ts (或类似命名的文件)
    import { FrontendApi, Configuration } from '@ory/client-fetch';

    const basePath = import.meta.env.VITE_ORY_SDK_URL as string || 'http://localhost:4000'; // Added fallback for clarity based on context

    const ory = new FrontendApi(
      new Configuration({
        basePath: basePath,
        credentials: 'include', // **至关重要**
      })
    );

    export default ory;
    ```
    [8, 17]

2.  **`basePath` (基础路径)**：
    `basePath` 参数告诉 SDK Kratos API 的根 URL 是什么。在 Vite 项目中，推荐使用环境变量 (例如 `.env.local` 文件中定义的 `VITE_ORY_SDK_URL`) 来设置此值 [5]。在本地开发时，如果使用了 Ory Tunnel，`basePath` 通常指向 Tunnel 监听的地址 (如 `http://localhost:4000`) [8]。对于生产环境，它会指向你部署的 Kratos 实例的公共 URL 或 Ory Network 的项目端点。

3.  **`credentials: 'include'` (凭证包含)**：
    这个配置项**极其重要** [2]。它指示浏览器在发送 AJAX 请求到 Kratos API 时，即使是跨域请求（在本地开发中常见），也要自动包含任何相关的 Cookie（如会话 Cookie, CSRF Cookie）。如果遗漏此设置，Kratos 的许多基于 Cookie 的安全机制将无法工作，导致认证流程静默失败，并且这类问题通常难以调试。Kratos 对 Cookie 的依赖性是其安全模型的基础，因此确保凭证被正确发送是 SPA 集成成功的关键。

4.  **SDK 方法**：
    SDK 提供了与 Kratos REST API 端点对应的方法，用于执行各种流程操作，例如：
    *   创建新的流程 (如 `createBrowserLoginFlow`, `initializeSelfServiceRegistrationFlowForBrowsers`)。
    *   获取已存在流程的详细信息 (如 `getLoginFlow`, `getSelfServiceSettingsFlow`)。
    *   更新或提交流程数据 (如 `updateLoginFlow`, `submitSelfServiceVerificationFlow`)。
    *   检查当前用户会话状态 (如 `toSession`)。
    *   初始化登出流程 (如 `createBrowserLogoutFlow`)。 [3]

以下表格总结了一些关键的 `@ory/client-fetch` API 方法及其在 UI 中的典型使用场景：

**关键表格 1: 主要的 `@ory/client-fetch` API 方法**

| SDK 方法名 (示例)                            | 目的                                          | UI 中的典型使用场景                                                                                                                                                                                                                                                                                                                                                                                     |
| :------------------------------------------- | :-------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `createBrowserLoginFlow()`                   | 初始化新的用户登录流程。                      | 用户首次访问登录页面 (`/login`) 时调用，以获取登录表单的定义。                                                                                                                                                                                                                                                                                                                          |
| `getLoginFlow({ id: flowId })`               | 获取已存在的登录流程的当前状态和数据。        | 当 Kratos 将用户重定向回登录页面并附带 `?flow=<flowId>` 参数时调用，用于恢复流程或显示错误信息。                                                                                                                                                                                                                                                                                        |
| `updateLoginFlow({ flow, updateLoginFlowBody })` | 提交登录表单数据以完成登录尝试。              | 用户在登录表单中填写凭据并点击“登录”按钮后，收集表单数据并调用此方法。                                                                                                                                                                                                                                                                                                                    |
| `createBrowserRegistrationFlow()`            | 初始化新的用户注册流程。                      | 用户首次访问注册页面 (`/register`) 时调用。                                                                                                                                                                                                                                                                                                                                             |
| `getRegistrationFlow({ id: flowId })`        | 获取已存在的注册流程的当前状态和数据。        | Kratos 重定向回注册页面并附带 `?flow=<flowId>` 参数时调用。                                                                                                                                                                                                                                                                                                                             |
| `updateRegistrationFlow({ flow, updateRegistrationFlowBody })` | 提交注册表单数据。                            | 用户填写注册信息并点击“注册”按钮后调用。                                                                                                                                                                                                                                                                                                                                                |
| `createBrowserSettingsFlow()`                | 初始化账户设置流程（需用户已登录）。          | 已登录用户访问账户设置页面 (`/settings`) 时调用。                                                                                                                                                                                                                                                                                                                                       |
| `getSettingsFlow({ id: flowId })`            | 获取已存在的账户设置流程的当前状态和数据。    | Kratos 重定向回设置页面并附带 `?flow=<flowId>` 参数时调用。                                                                                                                                                                                                                                                                                                                             |
| `updateSettingsFlow({ flow, updateSettingsFlowBody })` | 提交账户设置表单数据（如修改密码、资料）。    | 用户在设置表单中做出更改并点击“保存”按钮后调用。                                                                                                                                                                                                                                                                                                                                        |
| `createBrowserRecoveryFlow()`                | 初始化账户恢复流程（如忘记密码）。            | 用户访问账户恢复页面 (`/recovery`) 时调用。                                                                                                                                                                                                                                                                                                                                             |
| `updateRecoveryFlow({ flow, updateRecoveryFlowBody })` | 提交账户恢复请求（如邮箱）或恢复代码。        | 用户输入邮箱以请求恢复链接/代码，或输入收到的恢复代码后调用。                                                                                                                                                                                                                                                                                                                           |
| `createBrowserVerificationFlow()`            | 初始化邮件/账户验证流程。                     | 用户注册后需要验证邮箱，或访问验证页面 (`/verification`) 时调用。                                                                                                                                                                                                                                                                                                                       |
| `updateVerificationFlow({ flow, updateVerificationFlowBody })` | 提交验证请求（如邮箱）或验证代码。            | 用户输入邮箱以请求验证链接/代码，或输入收到的验证代码后调用。                                                                                                                                                                                                                                                                                                                           |
| `toSession()`                                | 检查当前是否存在有效的用户会话。              | 在应用加载时、路由守卫中、或需要显示用户特定信息（如用户名、头像）前来确定用户登录状态。                                                                                                                                                                                                                                                                                                |
| `createBrowserLogoutFlow()`                  | 初始化用户登出流程。                          | 用户点击“登出”按钮时调用，以获取一个安全的登出 URL。Kratos 会使当前会话失效。                                                                                                                                                                                                                                                                                                           |
| `performNativeLogout(body: { session_token })` | （原生应用）执行登出。                        | 主要用于非浏览器客户端，通过会话令牌登出。（与本 React SPA 指南相关性较低，但可作对比）                                                                                                                                                                                                                                                                                                 |

理解这些 SDK 方法及其在流程中的作用，对于将 Kratos 的认证逻辑正确地集成到 React UI 中至关重要。这个表格为开发者提供了一个快速参考，帮助他们将 UI 操作与相应的后端 API 调用联系起来。

## 第四部分：使用 Ory Elements 进行 UI 分步开发

在完成了准备工作并理解了核心概念之后，现在可以开始使用 `@ory/elements-react` 组件逐步构建用户界面了。本部分将详细介绍全局设置以及各个核心认证流程页面的实现。

### 4.1 全局设置

一些设置需要在应用的顶层进行配置，以确保 Ory Elements 组件能够正确工作并与应用的整体风格保持一致。

#### 4.1.1 使用 `ThemeProvider` 包装应用

`@ory/elements-react` 提供了一个名为 `<ThemeProvider />` 的组件，它应该用来包裹应用的根组件。这通常在 `src/main.tsx` (对于 Vite + React 项目) 或 `src/App.tsx` 中完成 [5]。

```typescript
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { ThemeProvider } from '@ory/elements-react/theme'; // 确保路径正确
import "@ory/elements-react/theme/styles.css"; // 导入默认样式
import { BrowserRouter } from 'react-router-dom'; // 假设使用 React Router

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <BrowserRouter>
      <ThemeProvider> {/* 使用 Ory Elements 的 ThemeProvider */}
        <App />
      </ThemeProvider>
    </BrowserRouter>
  </React.StrictMode>
);
```

`ThemeProvider` 为其所有子 Ory Elements 组件提供了基础的样式上下文和主题化能力。

#### 4.1.2 基础样式与主题定制 (themeOverrides)

Ory Elements 带有一套默认的样式。可以通过导入其 CSS 文件来应用这些基础样式 [4]：

```typescript
import "@ory/elements-react/theme/styles.css";
```

如果需要进一步定制组件的外观以匹配应用的品牌风格，可以使用 `ThemeProvider` 的 `themeOverrides` prop。这个 prop 接受一个 `Theme` 对象，允许你覆盖默认的主题颜色、字体、边框等 [5]。具体的 `Theme` 对象结构需要参考 Ory Elements 的文档或类型定义，但其核心思想是通过一个集中的配置对象来调整视觉表现。

例如 (伪代码，具体结构需查阅文档)：

```typescript
// src/main.tsx
//... 其他导入
import { ThemeProvider, type Theme } from '@ory/elements-react/theme';

const customTheme: Partial<Theme> = { // Theme 类型可能需要从包中导入
  // 示例：自定义颜色
  // colors: {
  //   primary: '#007bff',
  //   accent: '#6c757d',
  //   error: '#dc3545',
  // },
  // 更多自定义选项...
};

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <BrowserRouter>
      <ThemeProvider themeOverrides={customTheme}>
        <App />
      </ThemeProvider>
    </BrowserRouter>
  </React.StrictMode>
);
```

对于更细致的样式调整，虽然可以直接覆盖 Ory Elements 生成的 CSS 类名，但这通常不被推荐，因为这些类名在库版本更新时可能会发生变化，导致样式失效 [19]。更稳妥的方式是利用 `themeOverrides` 或通过更高层级的 CSS 选择器（如果必须）来影响组件样式。

#### 4.1.3 （可选）国际化 (i18n) 设置

对于需要支持多种语言的应用，国际化 (i18n) 是一个重要方面。旧版本的 `@ory/elements` (非 `-react` 后缀的包) 曾通过 `<IntlProvider>` 组件提供 i18n 支持 [7]。对于 `@ory/elements-react`，开发者应查阅其最新文档以确定推荐的 i18n 方案。如果 Elements 本身不直接提供全面的 i18n 功能，可以考虑集成像 `react-intl` 这样的标准 React i18n 库，并根据需要为 Elements 组件中的文本提供翻译。

**关键表格 2: 主要的 Ory Elements 流程组件**

Ory Elements 为 Kratos 的各个自服务流程提供了高级别的 React 组件。这些组件封装了从 `flow` 对象渲染 UI 节点的逻辑。

| 流程类型       | `@ory/elements-react` 组件 (示例)                      | 核心 Props (推测或已知)                                                               | 简要描述                                                                                 |
| :------------- | :----------------------------------------------------- | :------------------------------------------------------------------------------------ | :--------------------------------------------------------------------------------------- |
| 登录           | `<Login />` (来自 `@ory/elements-react/theme` [4])    | `flow` (必需, 类型：`LoginFlow`), `onSubmit` (必需, 类型：`(values: UpdateLoginFlowBody) => Promise<void>`), `config` (可选), `components` (可选 [20]) | 渲染登录表单，处理密码登录和社交登录（如果已配置）。                                     |
| 注册           | `<Registration />` (来自 `@ory/elements-react/theme` [4]) | `flow` (必需, 类型：`RegistrationFlow`), `onSubmit` (必需, 类型：`(values: UpdateRegistrationFlowBody) => Promise<void>`), `config` (可选 [4]) | 渲染注册表单，字段根据 Kratos Identity Schema 动态生成。                                 |
| 账户设置       | `<Settings />` (或通用组件如 `<UserSettingsCard />` [6]) | `flow` (必需, 类型：`SettingsFlow`), `onSubmit` (必需, 类型：`(values: UpdateSettingsFlowBody) => Promise<void>`) | 允许用户修改个人资料、密码、管理 2FA、关联社交账户等。                                   |
| 账户恢复       | `<Recovery />` [21]                                    | `flow` (必需, 类型：`RecoveryFlow`), `onSubmit` (必需, 类型：`(values: UpdateRecoveryFlowBody) => Promise<void>`) | 处理忘记密码流程，例如请求恢复邮件或输入恢复代码。                                       |
| 邮件/账户验证  | `<Verification />` [9]                                 | `flow` (必需, 类型：`VerificationFlow`), `onSubmit` (必需, 类型：`(values: UpdateVerificationFlowBody) => Promise<void>`) | 用于验证用户邮箱或在注册后激活账户。                                                     |
| 通用卡片       | `<UserAuthCard />` [6]                                 | `flow`, `flowType` ("login", "registration", etc.), `additionalProps` (如 `signupURL`), `title`, `onSubmit` | 一个更通用的卡片组件，可能被上述特定流程组件内部使用或作为其替代。                       |
| 通用卡片       | `<UserSettingsCard />` [6]                             | `flow`                                                                                | 类似于 `UserAuthCard`，但针对设置场景。                                                  |

**注意**:
*   `flow` prop 对于所有流程组件都是核心，它携带了从 Kratos API 获取的流程定义和状态。
*   `onSubmit` prop 通常是必需的。Ory Elements 组件负责收集表单数据并通过此回调将其传递给父页面组件。父页面组件随后负责调用相应的 Ory SDK 方法来提交数据到 Kratos。这种模式（有时称为受控组件或回调模式）提供了良好的关注点分离：Elements 处理 UI 渲染和数据收集，而页面组件处理业务逻辑和 API 调用 [3]。
*   `config` prop 可能用于传递 Ory SDK 的配置信息，例如 `basePath`，尽管 SDK 实例通常在更高层级或单独模块中初始化。
*   `components` prop (如在 `<Login />` 中 [20]) 可能允许更细粒度的组件替换或自定义，例如替换内部的卡片组件。

这个表格为开发者提供了一个起点，帮助他们快速识别和使用适合特定 Kratos 流程的 Ory Elements 组件。

### 4.3 实现登录流程 (Login Flow)

登录是任何认证系统的核心功能。以下步骤将指导如何使用 Ory Elements 和 `@ory/client-fetch` SDK 实现登录页面。

#### 4.3.1 创建登录页面组件 (src/pages/Login.tsx)

首先，为登录流程创建一个新的 React 组件。假设应用使用 React Router 进行路由管理，可以创建一个路径为 `/login` 的页面组件。

```typescript
// src/pages/Login.tsx
import React, { useEffect, useState } from 'react';
import { useNavigate, useSearchParams } from 'react-router-dom';
import ory from '../ory'; // 假设 ory SDK 实例在 src/ory.ts 中导出
import { LoginFlow, UpdateLoginFlowBody } from '@ory/client-fetch';
import { Login } from '@ory/elements-react/theme'; // Ory Elements 登录组件
import { handleFlowError } from '../oryHelpers'; // 假设的错误处理辅助函数

const LoginPage: React.FC = () => {
  const [flow, setFlow] = useState<LoginFlow | null>(null);
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();

  const flowId = searchParams.get('flow');
  // aal (Authentication Assurance Level) 和 refresh 参数用于高级流程，如此处不处理，可忽略
  // const refresh = searchParams.get('refresh');
  // const aal = searchParams.get('aal');
  const returnTo = searchParams.get('return_to');

  useEffect(() => {
    // 如果用户已登录，并且不是在刷新会话或提升 AAL，则重定向到首页
    ory.toSession()
      .then(() => {
        // if (refresh || aal) return; // 如果是 refresh 或 aal 流程，则不重定向
        navigate('/'); // 已登录，跳转到首页
      })
      .catch(() => {
        // 用户未登录，继续加载登录流程
        // 如果 URL 中有 flow ID (通常是 Kratos 重定向过来的)，则获取该流程
        if (flowId) {
          ory.getLoginFlow({ id: String(flowId) })
            .then(({ data }) => setFlow(data))
            .catch(handleFlowError(navigate, 'login', setFlow, '/login')); // 错误处理
        } else {
          // 否则，初始化一个新的登录流程
          ory.createBrowserLoginFlow({
            returnTo: returnTo || undefined,
            // refresh: Boolean(refresh),
            // aal: aal? String(aal) : undefined,
          })
            .then(({ data }) => setFlow(data))
            .catch(handleFlowError(navigate, 'login', setFlow, '/login')); // 错误处理
        }
      });
  }, [flowId, navigate, returnTo]); // 依赖项

  const handleSubmit = async (values: UpdateLoginFlowBody) => {
    if (!flow) return;
    try {
      const { data: sessionResponse } = await ory.updateLoginFlow({
        flow: flow.id,
        updateLoginFlowBody: values,
      });

      // 登录成功，Kratos 通常会设置会话 cookie
      // 检查是否有 return_to URL，否则导航到仪表盘或首页
      if (sessionResponse.session && flow.return_to) {
        window.location.href = flow.return_to; // 使用 window.location.href 以确保完整的页面加载和 cookie 处理
      } else {
        navigate('/'); // 导航到首页
      }
    } catch (error: any) {
      if (error.response && error.response.data) {
        setFlow(error.response.data); // Kratos 返回更新后的 flow 对象，其中包含错误信息
      } else {
        // 处理其他网络错误等
        console.error('Login submission error:', error);
        // 可以在此设置一个通用的错误消息状态
      }
    }
  };

  if (!flow) {
    return <div>Loading...</div>; // 或显示骨架屏
  }

  return (
    <div>
      <h1>Login</h1>
      <Login flow={flow} onSubmit={handleSubmit} />
      {/* 可以添加指向注册页面的链接等 */}
      <p>
        Don't have an account? <a href="/registration">Register here</a>
      </p>
      <p>
        Forgot your password? <a href="/recovery">Recover account</a>
      </p>
    </div>
  );
};

export default LoginPage;
```

在 `src/oryHelpers.ts` (或类似文件) 中，可以定义 `handleFlowError`：

```typescript
// src/oryHelpers.ts
import { NavigateFunction } from 'react-router-dom';
import { AxiosError } from 'axios'; // @ory/client-fetch 使用 axios

export const handleFlowError = <TFlow,>(
  navigate: NavigateFunction,
  flowType: 'login' | 'registration' | 'settings' | 'recovery' | 'verification',
  setFlow: React.Dispatch<React.SetStateAction<TFlow | null>>,
  defaultRedirectPath?: string
) => {
  return async (error: AxiosError): Promise<void | AxiosError> => {
    if (!error.response) {
      console.error('Network error or no response from server:', error);
      // 可以设置一个全局错误状态或重定向到错误页面
      return Promise.reject(error);
    }

    const { data, status } = error.response;

    switch (status) {
      case 400: // Bad Request - 通常是表单验证错误或流程状态问题
        if (typeof data === 'object' && data && 'id' in data) { // 假设 Kratos 返回了更新的 flow 对象
          setFlow(data as TFlow);
        } else {
          // 未知的 400 错误结构
          navigate(defaultRedirectPath || `/error?code=${status}`);
        }
        return;
      case 401: // Unauthorized - 通常是会话无效或需要重新认证
        navigate('/login'); // 强制用户重新登录
        return;
      case 403: // Forbidden - 用户无权访问或执行操作
        // 例如，尝试访问设置页面但未登录
        navigate('/login');
        return;
      case 404: // Not Found - 流程 ID 无效或端点不存在
        navigate(defaultRedirectPath || `/error?message=flow-not-found`);
        return;
      case 410: // Gone - 流程已过期
        // 重新初始化流程
        navigate(`/${flowType}`);
        return;
      case 422: // Unprocessable Entity - 通常用于需要浏览器重定向的特殊流程 (如 OIDC, WebAuthn)
        if (typeof data === 'object' && data && 'redirect_browser_to' in data) {
          window.location.href = (data as { redirect_browser_to: string }).redirect_browser_to;
        } else {
          // 无法处理的 422 错误
          navigate(defaultRedirectPath || `/error?code=${status}`);
        }
        return;
      default:
        // 其他未处理的错误
        console.error('Unhandled Kratos error:', error);
        navigate(defaultRedirectPath || `/error?code=${status}`);
        return Promise.reject(error);
    }
  };
};
```

#### 4.3.2 初始化登录流程 (SDK 调用)

在 `LoginPage` 组件的 `useEffect` Hook 中：

1.  **检查现有会话**：首先调用 `ory.toSession()`。如果用户已经登录（并且不是在执行需要强制重新认证的特殊流程，如 refresh 或 aal 提升），则直接导航到应用的主页或仪表盘 [8]。
2.  **获取或创建流程**：
    *   如果用户未登录，检查 URL 的查询参数中是否包含 `flow ID`。这种情况通常发生在 Kratos 因某种原因（如表单提交错误后、或从邮件链接返回）将用户重定向回此登录页面时。如果存在 `flow ID`，则调用 `ory.getLoginFlow({ id: flowIdFromUrlParams })` 来获取该流程的当前状态 [3]。
    *   如果 URL 中没有 `flow ID`（即用户直接访问 `/login` 页面），则调用 `ory.createBrowserLoginFlow()` 来初始化一个新的登录流程 [3]。此时，可以传递 `return_to` 参数，以便在登录成功后将用户重定向到他们之前尝试访问的页面 [23]。
3.  **状态管理**：将从 Kratos API 获取到的 `flow` 对象存储在组件的 state 中 (使用 `useState`) [3]。
4.  **错误处理**：在调用 SDK 方法时，使用 `.catch()` 块或集中的错误处理函数 (如上面示例的 `handleFlowError`) 来处理可能发生的错误。例如，如果 Kratos 服务不可用，或者用户尝试访问一个已过期的流程，都需要给用户适当的反馈或引导。`@ory/client-fetch` 提供的 `handleFlowError` 辅助函数对于简化此逻辑非常有用 [4]。

#### 4.3.3 渲染 `<Login />` 组件

一旦从 Kratos 获取到 `flow` 对象并存储在 state 中，就可以渲染 `@ory/elements-react/theme` 提供的 `<Login />` 组件了 [4]。

*   将获取到的 `flow` 对象作为 `flow` prop 传递给 `<Login />` 组件。
*   提供一个 `onSubmit` 回调函数作为 prop，该函数将在用户提交表单时被调用。
*   （可选）可以传递 `config` prop，如果 Elements 组件需要额外的 Ory 配置信息 [4]。
*   （可选）如果使用更通用的 `<UserAuthCard />` 组件，还可以传递 `additionalProps` 来定义例如“忘记密码”或“注册”链接的 URL [6]。

`<Login />` 组件会智能地解析 `flow.ui.nodes` 数组，并动态渲染出相应的表单元素，包括标准的用户名/密码输入框、提交按钮，以及根据 Kratos 配置自动出现的社交登录按钮 (OIDC) 或其他登录方法 (如 WebAuthn/Passkey) 的触发器。

#### 4.3.4 处理表单提交 (onSubmit 回调)

`onSubmit` 回调函数是连接 Ory Elements UI 和 Kratos 后端逻辑的关键。

1.  **接收表单数据**：当用户填写完表单并点击提交按钮时，`<Login />` 组件会调用你提供的 `onSubmit` 函数，并将收集到的表单数据（一个包含所有字段名和值的对象）作为参数传递进来 [3]。
2.  **调用 SDK 提交流程**：在 `onSubmit` 函数内部，使用 `ory.updateLoginFlow()` (或旧版 SDK 中的 `submitSelfServiceLoginFlow()`) 方法将这些数据提交回 Kratos [3]。你需要传递当前流程的 `id` (可从 `flow.id` 获取) 以及包含用户输入和 `csrf_token` 的 `updateLoginFlowBody` 对象。`csrf_token` 通常由 Ory Elements 作为隐藏字段包含在表单中，并自动包含在传递给 `onSubmit` 的 `values` 对象中。
3.  **处理成功响应**：如果凭据有效且登录成功，Kratos 会：
    *   设置一个 HTTP-only 的会话 Cookie。
    *   通常会返回一个包含会话信息（`session`）和可能的会话令牌（`session_token`，主要用于原生应用）的响应。对于浏览器流程，主要关注会话 Cookie 的设置。
    *   响应中可能包含一个 `redirect_browser_to` 字段，指示浏览器应重定向到哪里（例如，配置的默认重定向 URL 或之前在流程初始化时提供的 `return_to` URL）[2]。
    *   在 React 应用中，收到成功响应后，通常需要使用 `navigate` (来自 `react-router-dom`) 将用户导航到仪表盘页面或其他受保护的路由。如果 Kratos 返回了 `redirect_browser_to`，直接使用 `window.location.href = redirect_browser_to` 可能更可靠，以确保所有 Cookie 和重定向逻辑都由浏览器正确处理。
4.  **处理错误响应**：如果登录失败（例如，密码错误、用户不存在），Kratos API 会返回一个 HTTP 错误状态码（通常是 400 或 401），其响应体中会包含一个更新后的 `flow` 对象。这个新的 `flow` 对象在其 `ui.messages` 或特定节点的 `messages` 字段中包含了详细的错误信息 [1]。你需要用这个新的 `flow` 对象更新组件的 state (`setFlow(error.response.data)`)。Ory Elements 组件在接收到新的 `flow` prop 后，会自动重新渲染并显示这些错误消息给用户。

这种由页面组件提供 `onSubmit` 回调给 Ory Elements 组件的模式，体现了良好的职责分离。Elements 组件专注于 UI 的渲染和用户输入数据的收集，而页面组件则负责具体的业务逻辑，例如何时以及如何调用 SDK、成功后的导航逻辑、以及更复杂的错误处理策略。这使得在表单提交前后注入自定义逻辑（如数据预处理、分析事件跟踪）成为可能。

#### 4.3.5 集成社交登录 (OIDC)

如果已在 Ory Kratos 项目中配置了一个或多个社交登录提供商（如 Google, GitHub, Facebook 等，通过 OIDC 标准），Ory Elements 的 `<Login />` 组件（或其内部渲染逻辑）会自动处理这些。

*   **动态渲染按钮**：组件会检查 `flow.ui.nodes` 中 `group: 'oidc'` 的节点，并为每个配置的提供商渲染一个相应的社交登录按钮 [1]。开发者无需手动为每个提供商编写按钮代码。
*   **处理提交流程**：当用户点击一个社交登录按钮时，Ory Elements 通常会触发一次表单提交。这次提交会将选定的提供商信息（例如，通过一个名为 `provider` 的字段，其值为提供商 ID，如 `google`）发送到 `flow.ui.action` URL。
*   **Kratos 处理重定向**：Kratos 接收到这个提交后，会启动 OIDC 流程，将用户的浏览器重定向到相应的社交登录提供商的认证页面。用户在该提供商处完成认证授权后，提供商会将用户重定向回 Kratos 配置的回调 URL。Kratos 完成令牌交换和用户身份验证/链接后，最终会将用户重定向回你在 Kratos 中为登录流程配置的 `ui_url`（通常会附带更新后的 `flow ID` 或直接完成登录并重定向到目标页面）。
*   **SPA 中的注意事项**：对于 SPA，OIDC 流程中的某些重定向可能需要通过原生的浏览器 `POST` 请求来发起，而不是 AJAX [23]。Ory Elements 旨在简化或封装这些复杂性，使得开发者体验更平滑。如果遇到 422 Unprocessable Entity 响应，这通常表明 Kratos 需要浏览器执行一个特定的重定向来继续流程（例如，跳转到 OIDC 提供商）。响应体中通常会包含一个 `redirect_browser_to` 字段，前端应使用 `window.location.href` 来导航到此 URL [23]。

Ory Elements 的一个核心优势就是动态适应 Kratos 的配置。这意味着社交登录的集成工作主要集中在 Ory Kratos 的后端配置上（例如，在 Ory Console 中添加 OIDC 提供商并配置其客户端 ID 和密钥），前端 UI 会自动反映这些更改。

#### 4.3.6 显示错误和消息

Kratos 通过 `flow` 对象在其 `ui.messages` 字段（用于流程级别的消息）或每个节点的 `messages` 字段（用于特定字段的验证错误）提供反馈信息 [1]。Ory Elements 组件被设计为能够自动检测并显示这些消息，无需开发者进行额外的处理。当 `flow` prop 更新时，组件会重新渲染，任何新的错误或提示信息都会展示在 UI 中相应的位置。这为用户提供了清晰、及时的反馈，例如“密码无效”或“此字段为必填项”。

### 4.4 实现注册流程 (Registration Flow)

注册流程的实现与登录流程非常相似，主要区别在于使用的 Ory Elements 组件和相应的 SDK 方法。

1.  **创建注册页面组件** (`src/pages/Registration.tsx`)：类似于登录页面。
2.  **初始化注册流程 (SDK 调用)**：
    *   检查 URL 中是否有 `flow ID`。
    *   若有，调用 `ory.getRegistrationFlow({ id: flowIdFromUrlParams })`。
    *   若无，调用 `ory.createBrowserRegistrationFlow({ returnTo: returnTo || undefined })` [10]。
    *   同样使用 `useState` 存储 `flow` 对象，并进行错误处理。
3.  **渲染 `<Registration />` 组件**：
    *   从 `@ory/elements-react/theme` 导入 `<Registration />` 组件 [4]。
    *   传递 `flow` prop 和 `onSubmit` 回调。
    *   一个关键特性是，`<Registration />` 组件会根据 Ory Kratos 中配置的 Identity Schema 动态生成注册表单的字段 [5]。例如，如果 Schema 定义了 `traits.firstName` 和 `traits.lastName`，这些字段会自动出现在注册表单中。
4.  **处理表单提交 (onSubmit 回调)**：
    *   回调接收表单数据 `values`。
    *   调用 `ory.updateRegistrationFlow({ flow: flow.id, updateRegistrationFlowBody: values })` [10]。
    *   成功注册后，Kratos 的行为取决于其配置（例如，是否自动登录、是否需要邮件验证）。应用可能需要导航到登录页、等待验证页面或直接进入仪表盘。
    *   错误处理（如密码太弱、邮箱已被使用）与登录流程类似：用 Kratos 返回的新 `flow` 对象更新 state，Elements 组件会自动显示错误。

注册流程的动态字段生成是 Ory Kratos 和 Elements 结合的一大优势，它使得后端对身份属性的调整能够无缝反映到前端 UI，而无需修改前端代码。

### 4.5 实现账户设置流程 (Settings Flow)

账户设置流程允许已登录用户修改他们的个人资料、密码、管理双因素认证 (2FA) 方法、关联或取消社交登录账户等。

1.  **创建设置页面组件** (`src/pages/Settings.tsx`)：此页面通常是受保护的，只允许已登录用户访问。
2.  **初始化设置流程 (SDK 调用)**：
    *   此流程必须在用户已登录（即存在有效 Kratos 会话）的情况下初始化 [13]。在页面加载时，应首先调用 `ory.toSession()` 确认用户已登录。如果未登录，应重定向到登录页。
    *   如果已登录，调用 `ory.createBrowserSettingsFlow()` 来初始化设置流程。
    *   URL 中也可能存在 `flow ID` (例如，从邮件链接完成某个设置子流程后返回)，此时调用 `ory.getSettingsFlow({ id: flowIdFromUrlParams })`。
    *   存储 `flow` 对象并处理错误。
3.  **渲染 `<Settings />` 组件** (或通用的 `<UserSettingsCard />` [6])：
    *   传递 `flow` prop 和 `onSubmit` 回调。
    *   `<Settings />` 组件会根据 Kratos 的配置和当前用户的状态动态显示可用的设置选项 [5]。例如，如果用户已启用密码登录，会显示修改密码的选项；如果配置了 WebAuthn，会显示添加安全密钥的选项。
4.  **处理表单提交 (onSubmit 回调)**：
    *   回调接收特定设置表单（如修改密码表单、更新资料表单）的数据 `values`。
    *   调用 `ory.updateSettingsFlow({ flow: flow.id, updateSettingsFlowBody: values })` [10]。
    *   成功更新后，Kratos 可能会返回更新后的 `flow` 对象（例如，显示成功消息）或指示重定向。通常，页面会重新加载设置流程以反映更改，或显示一个成功通知。
    *   错误处理与之前流程类似。

设置流程的 UI 会根据启用的特性（如密码策略、OIDC 提供商、WebAuthn、TOTP）动态变化，Ory Elements 会负责渲染这些动态部分。

### 4.6 实现账户恢复流程 (Recovery Flow)

账户恢复流程用于帮助用户在忘记密码或无法访问其账户时重新获得访问权限。

1.  **创建恢复页面组件** (`src/pages/Recovery.tsx`)。
2.  **初始化恢复流程 (SDK 调用)**：
    *   检查 URL 中是否有 `flow ID` (例如，用户通过邮件中的恢复链接访问此页面，链接中可能包含 `flow` 和 `token`)。
    *   若有，调用 `ory.getRecoveryFlow({ id: flowIdFromUrlParams })`。
    *   若无（用户直接访问 `/recovery` 页面），调用 `ory.createBrowserRecoveryFlow()` [10]。
    *   存储 `flow` 对象并处理错误。
3.  **渲染 `<Recovery />` 组件** [21]：
    *   传递 `flow` prop 和 `onSubmit` 回调。
    *   `<Recovery />` 组件通常会首先要求用户输入其注册时使用的身份标识符（如邮箱）。
4.  **处理表单提交 (onSubmit 回调)**：
    *   **第一阶段 (请求恢复)**：用户输入邮箱并提交。`onSubmit` 回调调用 `ory.updateRecoveryFlow()`，将邮箱地址发送给 Kratos。Kratos 会向该邮箱发送一封包含恢复链接或恢复代码的邮件。此时，UI 应显示消息提示用户检查邮箱。
    *   **第二阶段 (完成恢复，如果使用代码)**：如果 Kratos 配置为使用恢复代码，用户在收到代码后，会回到 UI（可能是同一个页面，也可能是通过邮件链接跳转到的特定状态的恢复页面）输入代码。再次提交表单，`onSubmit` 调用 `ory.updateRecoveryFlow()` 并包含恢复代码。
    *   **通过链接恢复**：如果用户点击邮件中的恢复链接，该链接通常会直接将用户带到一个已预填了必要令牌的流程状态，或者直接完成恢复并允许用户设置新密码（这可能过渡到一个特殊的设置流程）。
    *   成功完成恢复后，用户通常会被允许设置新密码，并重新获得账户访问权限。应用应相应地导航用户。
    *   错误处理（如邮箱不存在、恢复代码无效/过期）同样通过更新 `flow` 对象并在 UI 中显示消息来完成。

### 4.7 实现邮件/账户验证流程 (Verification Flow)

验证流程用于确认用户提供的身份凭证（通常是邮箱地址）的有效性，或在某些场景下用于激活新注册的账户。

1.  **创建验证页面组件** (`src/pages/Verification.tsx`)。
2.  **初始化验证流程 (SDK 调用)**：
    *   与恢复流程类似，检查 URL 中是否有 `flow ID` 和 `token` (用户可能通过验证邮件中的链接访问)。
    *   若有，调用 `ory.getVerificationFlow({ id: flowIdFromUrlParams })`。
    *   若无，调用 `ory.createBrowserVerificationFlow()` [10]。
    *   存储 `flow` 对象并处理错误。
3.  **渲染 `<Verification />` 组件** [9]：
    *   传递 `flow` prop 和 `onSubmit` 回调。
    *   UI 通常会提示用户输入其需要验证的邮箱地址（如果流程刚开始），或者如果通过链接进入，可能会直接显示一个确认信息或要求输入验证码（如果配置为使用代码）。
4.  **处理表单提交 (onSubmit 回调)**：
    *   **请求验证**：用户输入邮箱并提交。`onSubmit` 调用 `ory.updateVerificationFlow()` 将邮箱发送给 Kratos。Kratos 发送验证邮件。
    *   **完成验证 (如果使用代码)**：用户输入收到的验证码并提交。`onSubmit` 再次调用 `ory.updateVerificationFlow()` 并包含验证码。
    *   **通过链接验证**：点击邮件中的验证链接通常会直接完成验证，并将用户重定向回应用。
    *   成功验证后，用户的相应身份凭证（如邮箱）会被标记为已验证。UI 应显示成功消息，并可能导航用户到登录页或仪表盘。
    *   错误处理（如验证码无效/过期、链接无效）通过更新 `flow` 对象来处理。

所有这些自服务流程在前端的实现遵循一个高度相似的模式：通过 SDK 获取或初始化流程，将 `flow` 对象传递给相应的 Ory Elements 组件进行渲染，并通过页面组件提供的 `onSubmit` 回调来处理用户提交和后续的 SDK 调用。Ory Elements 的核心价值在于它极大地抽象了直接操作和渲染 Kratos `flow.ui.nodes` 的复杂性，使得开发者能够更专注于业务逻辑而非底层的 UI 节点解析。同时，`onSubmit` 回调模式确保了业务逻辑的灵活性和可扩展性。对于社交登录等复杂场景，Ory Elements 也旨在通过动态渲染和封装交互来简化前端的集成工作。

## 第五部分：高级主题与最佳实践

在掌握了基本流程的 UI 实现之后，可以关注一些高级主题和最佳实践，以提升用户体验、增强安全性并使应用更加健壮。

### 5.1 管理重定向 (return_to)

在许多认证场景中，用户在完成一个流程（如登录或注册）后，需要被重定向回他们最初尝试访问的页面，或者一个特定的目标页面。Ory Kratos 支持通过 `?return_to=<url>` 查询参数来实现这一点 [23]。

*   **初始化流程时传递 `return_to`**：当你的应用检测到用户需要认证才能访问某个受保护资源时，可以在初始化 Kratos 流程（如登录流程）时，将该资源的 URL 作为 `return_to` 参数附加到 Kratos 的初始化端点上。例如，如果用户尝试访问 `/protected-page` 但未登录，应用可以将用户重定向到 `/login?return_to=/protected-page`，然后登录页面在调用 `ory.createBrowserLoginFlow()` 时，可以将这个 `return_to` 值传递给 Kratos。
*   **Kratos 处理 `return_to`**：Kratos 会在其内部存储这个 `return_to` 值。当流程成功完成后（例如，用户成功登录），如果存在 `return_to` 值，Kratos 通常会在最终的重定向中将用户导向该 URL。
*   **跨流程传递 `return_to`**：需要注意的是，`return_to` 参数通常不会在不同的 Kratos 流程之间自动持久化 [23]。例如，如果用户从一个带有 `return_to` 的登录流程跳转到了注册流程，那么注册流程完成后，Kratos 可能不会使用最初登录流程的 `return_to` 值。在这种情况下，前端应用需要在流程转换时（例如，从登录页导航到注册页）手动从当前流程对象中提取 `flow.return_to` 值，并将其作为参数传递给下一个流程的初始化请求。
*   **例外情况**：一个已知的 `return_to` 会持久化的场景是从账户恢复流程过渡到账户设置流程（用于重置密码）[23]。

正确管理 `return_to` 对于提供流畅的用户体验至关重要，尤其是在会话超时后或访问受保护内容前的认证场景。

### 5.2 会话管理

用户成功登录后，Kratos 会为其创建一个会话。前端应用需要能够检查会话状态、处理登出以及理解会话的生命周期。

#### 5.2.1 检查会话状态 (toSession)

`@ory/client-fetch` SDK 提供了 `ory.toSession()` 方法，用于检查当前浏览器是否存在有效的 Kratos 会话 [8]。

*   **调用时机**：通常在应用加载时（例如在根组件的 `useEffect` 中）、路由守卫中（用于保护需要认证的路由）、或者在任何需要显示用户特定信息（如用户名、头像）或根据登录状态决定 UI行为的地方调用。
*   **成功响应**：如果存在有效会话，`toSession()` 会返回一个包含会话详细信息的对象，其中最重要的部分是 `session.identity`，它包含了用户的身份特征 (`traits`) [8]。
*   **失败响应**：
    *   如果不存在有效会话（例如，用户未登录或会话已过期），API 通常会返回 HTTP 401 Unauthorized 错误。此时，应用应将用户重定向到登录页面。
    *   如果会话存在但需要额外操作，例如需要完成双因素认证 (2FA)，API 可能会返回 HTTP 403 Forbidden 错误，并在响应体中包含特定的错误 ID，如 `session_aal2_required` [23]。应用需要捕获此类错误，并引导用户完成相应的流程（例如，重定向到登录流程并附带 `aal=aal2` 参数）。

#### 5.2.2 实现登出

安全地终止用户会话是认证系统的重要组成部分。

*   **浏览器应用登出流程** [8]：
    1.  前端应用首先调用 `ory.createBrowserLogoutFlow()`。此方法会联系 Kratos，Kratos 会准备一个与当前会话关联的安全登出令牌。
    2.  SDK 调用成功后，会返回一个包含 `logout_url` 的对象。
    3.  前端应用必须将用户的浏览器重定向到这个 `logout_url` (例如，通过 `window.location.href = logout_url;`)。
    4.  当浏览器访问 `logout_url` 时，Kratos 会使当前会话失效（例如，删除会话 Cookie），然后通常会将用户重定向到配置的登出后跳转页面（例如，登录页或应用首页）。
*   **原生应用登出**：对于非浏览器的原生应用，登出流程不同，通常是直接向 `/self-service/logout/api` 端点发送一个包含会话令牌的 `DELETE` 请求 [10]。这与本 React SPA 指南的关联性较低，但了解其差异有助于全面理解 Kratos 的会话机制。

#### 5.2.3 会话刷新与生命周期

*   **会话刷新 (强制重新认证)**：在某些情况下，即使当前会话仍然有效，应用也可能希望用户重新确认其身份（例如，在执行非常敏感的操作之前）。这可以通过在初始化登录流程时附加 `?refresh=true` 查询参数来实现 [23]。Kratos 在收到此参数时，即使用户已有有效会话，也会强制用户重新进行登录验证。
*   **会话生命周期**：Kratos 会话具有定义的生命周期，包括创建时间、过期时间 (`expires_at`) 和最后认证时间 (`authenticated_at`) [24]。`authenticated_at` 会在用户完成登录或刷新会话（包括通过 2FA）时更新。前端应用虽然通常不直接操作这些时间戳，但理解它们的存在有助于理解会话行为，例如 Kratos 何时可能会因为会话过期而要求重新登录。

### 5.3 全面的错误处理 (`handleFlowError`)

与 Kratos API 的交互可能会产生多种类型的响应，包括成功、验证错误、流程状态变更、需要重定向等。`@ory/client-fetch` SDK 提供了一个名为 `handleFlowError` 的辅助函数，旨在简化和统一这些流程 API 调用中的错误处理逻辑 [4]。

`handleFlowError` 通常接受一个包含多个回调函数的对象作为参数，例如：

*   `onValidationError(flow)`: 当 Kratos 返回指示输入数据验证失败的错误时调用。通常，这意味着需要用 Kratos 返回的更新后的 `flow` 对象（其中包含错误消息）来更新 UI 状态。
*   `onRestartFlow()`: 当发生不可恢复的错误（例如流程已过期或损坏）需要重新启动流程时调用。此回调通常会导航用户到相应流程的初始页面。
*   `onRedirect(url, external)`: 当 Kratos 的响应指示需要浏览器重定向时调用（例如，在 OIDC 流程中跳转到身份提供商，或在 WebAuthn 流程中）。此回调应负责执行实际的浏览器重定向。

使用 `handleFlowError` 可以使错误处理代码更加清晰、集中和健壮，避免在每个 SDK 调用点编写大量重复的 `if/else` 或 `switch` 逻辑。这对于构建可靠的用户界面至关重要，因为前端必须能够稳健地处理来自后端的各种可能的响应状态。

### 5.4 CSRF 令牌 (`csrf_token`) 处理

为了防止跨站请求伪造 (Cross-Site Request Forgery, CSRF) 攻击，Ory Kratos 在其所有自服务流程中都内置了 CSRF 保护机制 [10]。

*   **令牌生成与包含**：当初始化或更新一个流程时，Kratos 会生成一个 CSRF 令牌。这个令牌通常会作为 `flow.ui.nodes` 数组中的一个隐藏输入字段（例如，`<input type="hidden" name="csrf_token" value="..." />`）包含在返回的 UI Payload 中 [10]。
*   **令牌提交**：当用户提交表单时，前端应用必须将这个 `csrf_token` 的值包含在发送给 Kratos 的请求体中。同时，浏览器会自动发送一个与当前会话关联的 CSRF Cookie（通常名为 `csrf_cookie_*`）。Kratos 后端会验证请求体中的令牌是否与 Cookie 中的期望值匹配。
*   **Ory Elements 的角色**：Ory Elements 组件（如 `<Login />`, `<Registration />` 等）在渲染表单时，应该会自动处理从 `flow.ui.nodes` 中找到并渲染这个隐藏的 `csrf_token` 字段。这意味着在使用 Elements 组件时，开发者通常不需要手动提取和添加 CSRF 令牌到提交的数据中，组件内部会处理。

确保 CSRF 令牌被正确处理是维护应用安全的关键一环。

### 5.5 处理双因素认证 (2FA) 和无密码 (WebAuthn/Passkey) 流程

Ory Kratos 支持包括 TOTP (Time-based One-Time Password)、WebAuthn (Passkey) 在内的多种高级认证方法。Ory Elements 旨在简化这些复杂流程的 UI 实现。

*   **双因素认证 (2FA)** [23]：
    *   **触发**：可以在初始化登录流程时通过 `?aal=<level>` 查询参数（例如 `aal=aal2`）来请求更高的认证保障级别。如果用户的账户配置了 2FA，并且当前会话级别低于所请求的级别，Kratos 会引导用户完成第二因素认证。
    *   **`toSession` 响应**：如果一个已登录用户的会话需要进行 2FA 才能满足某些操作的要求（例如，访问某个需要 `aal2` 的资源，或在设置中更改密码前），`ory.toSession()` 可能会返回一个 403 Forbidden 错误，并在错误响应中包含 `error.id === "session_aal2_required"`。
    *   **UI 引导**：当检测到需要 2FA 时，应用应将用户重定向到登录流程，并可能附带特定参数（如 `aal=aal2`），以提示 Kratos 进入 2FA 质询阶段。
    *   **Ory Elements 渲染**：Ory Elements 组件应能从 `flow.ui.nodes` 中识别并渲染与 2FA 相关的 UI 元素，例如 TOTP 代码输入框、备用码使用选项等。
*   **无密码认证 (WebAuthn/Passkey)** [1]：
    *   **引入脚本**：要在应用中使用 WebAuthn，通常需要在相关的 UI 页面（如登录、注册、设置页面）的 `<head>` 部分包含 Ory 提供的 WebAuthn JavaScript 辅助脚本。这个脚本的 URL 通常是 `/.well-known/ory/webauthn.js`（相对于 Kratos 的基础 URL）[1]。
    *   **Ory Elements 渲染与交互**：Ory Elements 会从 `flow.ui.nodes` 中找到 `group: 'webauthn'` 的节点（例如，一个“使用安全密钥登录/注册”的按钮），并渲染它们。这些按钮通常会带有一个 `onclick` 属性，其值是调用 WebAuthn 浏览器 API 所需的 JavaScript 代码片段。Elements 组件需要正确处理这些 `onclick` 事件，或者确保 Kratos 提供的脚本能够正确地为这些按钮绑定事件处理器。
    *   **SPA 中的 422 Unprocessable Entity 错误**：在 SPA 中执行 WebAuthn 流程（特别是注册新密钥或使用密钥登录时），Kratos API 有时会返回 HTTP 422 Unprocessable Entity 状态码 [23]。这通常表明 Kratos 需要浏览器执行一个重定向（例如，与浏览器的 WebAuthn API 交互后，需要将结果 `POST` 回 Kratos 的某个端点，但这不能通过 AJAX 完成）。响应体中会包含一个 `redirect_browser_to` 字段，指明浏览器应该导航到的 URL，以及一个新的 `flow` 对象或 `flow ID`。前端应用需要捕获这个 422 错误，从响应中提取新的 `flow ID`，然后使用这个 ID 调用 `getLoginFlow` (或相应流程的 `get` 方法) 来获取更新后的流程数据，并重新渲染 UI。或者，如果 `redirect_browser_to` 指向一个外部或需要原生表单提交的地址，则可能需要 `window.location.href`。

这些高级认证流程本身在技术实现上较为复杂。Ory Kratos 和 Ory Elements 通过提供专门的 UI 节点类型、辅助脚本以及流程状态管理来极大地抽象和简化了前端的集成工作。然而，开发者仍需理解这些流程的基本原理和在 SPA 环境下可能遇到的特殊情况（如 422 响应的处理）。

## 第六部分：Vite 特定注意事项

将 Ory Kratos 与 React SPA 集成时，如果使用 Vite 作为构建工具和开发服务器，有一些特定的配置和注意事项可以帮助确保流程顺畅。

### 6.1 环境变量管理 (`VITE_ORY_SDK_URL`)

Vite 使用一种特定的方式来处理环境变量。在客户端代码中，只有以 `VITE_` 为前缀的环境变量才会被暴露出来 [8]。

*   **定义变量**：为了配置 Ory SDK (`@ory/client-fetch`) 的 `basePath`，通常会定义一个名为 `VITE_ORY_SDK_URL` 的环境变量。这个变量可以在项目根目录下的 `.env` 文件（例如 `.env.local` 用于本地开发，`.env.production` 用于生产构建）中设置，或者在启动开发服务器或构建过程时通过命令行导出 [5]。 例如，在 `.env.local` 中：

    ```
    VITE_ORY_SDK_URL=http://localhost:4000
    ```

    (假设 `http://localhost:4000` 是 Ory Tunnel 监听的地址)。
*   **访问变量**：在 JavaScript/TypeScript 代码中，可以通过 `import.meta.env.VITE_ORY_SDK_URL` 来访问这个环境变量的值 [8]。

    ```typescript
    // src/ory.ts
    import { FrontendApi, Configuration } from '@ory/client-fetch';

    const basePath = import.meta.env.VITE_ORY_SDK_URL as string;

    const ory = new FrontendApi(
      new Configuration({
        basePath: basePath,
        credentials: 'include',
      })
    );
    export default ory;
    ```
*   **重要性**：这种机制使得在不同环境（开发、预演、生产）中使用不同的 Kratos API 端点（例如，本地 Tunnel 地址 vs 生产 Kratos 地址）变得简单和安全，而无需硬编码 URL。

### 6.2 构建过程与部署 (`vite build`)

当准备将应用部署到生产环境时，使用 Vite 的构建命令：

```bash
npm run build
```

或者直接调用：

```bash
vite build
```
[26]

*   **入口点**：默认情况下，Vite 会将项目根目录下的 `index.html` 作为构建的入口点。
*   **输出**：该命令会生成一个经过优化和打包的静态资源集合（HTML, CSS, JavaScript 文件等），通常位于 `dist` 目录下。这些静态资源可以直接部署到任何静态文件托管服务（如 Vercel, Netlify, AWS S3 + CloudFront 等）。
*   **浏览器兼容性**：Vite 的默认构建目标是支持现代浏览器。如果需要支持旧版浏览器，可能需要调整 `build.target` 配置或添加额外的 polyfill。

### 6.3 公共基础路径配置 (`base` 选项)

如果你的 React SPA 不是部署在域名的根路径下，而是部署在一个子目录下（例如，`https://example.com/my-cool-app/`），那么你需要配置 Vite 的 `base` 选项，以确保所有静态资源的引用路径都是正确的 [26]。

*   **配置方法**：在 `vite.config.js` (或 `vite.config.ts`) 文件中设置 `base` 选项：

    ```typescript
    // vite.config.ts
    import { defineConfig } from 'vite';
    import react from '@vitejs/plugin-react';

    export default defineConfig({
      plugins: [react()],
      base: '/my-cool-app/', // 设置为你的应用部署的子路径
    });
    ```
*   **影响**：Vite 在构建时会自动将所有生成的资源 URL（例如 CSS 中的 `url()`，HTML 中的 `<script src="...">`）前面加上这个基础路径。
*   **动态访问基础路径**：如果在代码中需要动态地构造 URL（例如，图片路径），可以使用 Vite 注入的全局变量 `import.meta.env.BASE_URL`，它会自动反映 `base` 配置的值 [26]。

正确配置 `base` 选项对于在非根路径部署的应用至关重要，否则可能会导致 CSS、JavaScript 文件或图片等资源加载失败 (404错误)。

总的来说，Vite 的特性与 Ory Kratos 的集成是兼容的，并没有已知的直接冲突。关键在于通过环境变量正确配置 Ory SDK 的 `basePath`，并根据部署需求调整 Vite 的 `base` 构建选项。Vite 的 `base` 配置使得将集成了 Ory Kratos 的 React SPA 灵活地部署到各种 URL 路径成为可能，这对于具有特定路径要求的企业部署或复杂托管场景非常有用。

## 第七部分：总结与进一步资源

本指南详细介绍了如何使用 Ory Elements 和 React + Vite 技术栈为 Ory Kratos 构建自定义用户界面。通过遵循这些步骤，开发者可以创建一个既符合自身品牌风格又能充分利用 Kratos强大身份验证功能的 UI。

### 7.1 关键要点回顾

*   **Ory Kratos 的无头特性** 为 UI 设计提供了极大的灵活性，允许开发者完全控制用户体验，但也要求自行构建所有面向用户的界面。
*   **Ory Elements (`@ory/elements-react`)** 显著简化了在 React 应用中集成 Kratos 的过程。它提供了一系列预构建的、可定制的、主题化的组件，这些组件能够动态地根据 Kratos 返回的 `flow` 对象渲染 UI，包括处理各种输入类型、错误消息和社交登录等。
*   **`@ory/client-fetch` SDK** 是前端与 Kratos API 通信的桥梁。正确配置其 `basePath` (通常通过 Vite 的环境变量 `VITE_ORY_SDK_URL` 指向 Ory Tunnel 或 Kratos 实例) 和 `credentials: 'include'` (确保 Cookie 传递) 是成功集成的基础。
*   **Kratos 自服务流程 (Flows)** 是 UI 开发的核心。每个流程（登录、注册、设置、恢复、验证）都有其特定的目的和 API 交互模式。前端 UI 的实现模式在这些流程中高度相似：获取或初始化流程 -> 将 `flow` 对象传递给 Elements 组件 -> 通过 `onSubmit` 回调处理表单提交并调用 SDK -> 根据 SDK 返回结果更新 `flow` 或导航。
*   **错误处理** 对于健壮的 UI至关重要。利用 SDK 提供的 `handleFlowError` 辅助函数或自定义的错误处理逻辑，可以有效地向用户反馈问题并引导他们进行后续操作。
*   **高级功能** 如 `return_to` 重定向管理、会话检查与登出、CSRF 保护、以及对 2FA 和 WebAuthn/Passkey 的支持，都是构建完整且安全认证体验的重要组成部分。Ory Kratos 和 Ory Elements 为这些功能提供了相应的机制。
*   **Vite 环境** 与 Ory 集成良好，通过环境变量和构建选项可以方便地进行配置和部署。

核心在于理解 Kratos 的流程驱动机制，并有效利用 Ory Elements 来抽象 UI 渲染的复杂性，同时通过 SDK 与 Kratos 后端进行稳健的通信。

### 7.2 进一步 Ory 文档与社区资源链接

Ory 生态系统拥有丰富的文档和活跃的社区，可以为开发者提供进一步的支持和学习资源：

*   **Ory Kratos 官方文档**: <https://www.ory.sh/docs/kratos/>
    *   这里包含了关于 Kratos 概念、API 参考、配置指南、各种流程的详细说明等所有核心信息。
*   **Ory Elements GitHub 仓库**: <https://github.com/ory/elements> [9]
    *   可以在这里找到 `@ory/elements-react` 包的源代码、示例应用 (`examples/react-spa` 等) [2]、Issue 跟踪以及相关的讨论。
    *   `@ory/elements-react` 的 README 和包内文档通常会提供最新的组件 API 和用法示例 [4]。
*   **Ory SDK (包括 `@ory/client-fetch`) GitHub 仓库**: <https://github.com/ory/sdk> [16]
    *   了解 SDK 的生成方式、版本更新和更详细的 API 信息。
*   **Ory 社区**:
    *   Ory Community Forums: <https://community.ory.sh/> (通常 Ory 的 GitHub Discussions 也扮演了社区论坛的角色 [25])
    *   Ory Chat (通常是 Discord 或 Slack): 查找 Ory 官网获取最新的社区聊天链接，这里可以与其他开发者和 Ory 团队成员实时交流。
    *   Ory Blog: <https://www.ory.sh/blog/>
        *   经常发布关于 Ory 产品更新、最佳实践、集成教程（如 Next.js 集成 [3]）等有价值的文章。

通过深入研究这些资源，开发者可以解决在集成过程中遇到的特定问题，学习更高级的用例，并跟上 Ory 生态系统的最新发展。构建自定义 UI 是一个涉及前端和后端紧密协作的过程，充分利用官方文档和社区支持将是成功的关键。

---

### Works cited

1.  Understanding UI nodes and error messages - Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/concepts/ui-user-interface>
2.  Quickstart | Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/quickstart>
3.  Add custom login, registration, user settings to your Next.js & React single page application (SPA) - Ory, accessed May 25, 2025, <https://www.ory.sh/blog/nextjs-authentication-spa-custom-flows-open-source>
4.  @ory/elements-react - npm, accessed May 25, 2025, <https://www.npmjs.com/package/%40ory%2Felements-react>
5.  Custom user interface with Ory Elements, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/bring-your-own-ui/custom-ui-ory-elements>
6.  ory/elements - NPM, accessed May 25, 2025, <https://www.npmjs.com/package/@ory/elements>
7.  ory/elements-legacy - GitHub, accessed May 25, 2025, <https://github.com/ory/elements-legacy>
8.  Integrate authentication into React + API | Ory, accessed May 25, 2025, <https://www.ory.sh/docs/getting-started/integrate-auth/react>
9.  Ory Elements is a component library that makes building login, registration and account pages for Ory a breeze. - GitHub, accessed May 25, 2025, <https://github.com/ory/elements>
10. Basic information you need to integrate your UI with Ory | Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/bring-your-own-ui/custom-ui-basic-integration>
11. Login - Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/self-service/flows/user-login>
12. Registration | Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/self-service/flows/user-registration/>
13. Settings and profile updates - Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/self-service/flows/user-settings>
14. Account recovery and password reset - Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/self-service/flows/account-recovery-password-reset>
15. Verify addresses associated with users accounts - Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/self-service/flows/verify-email-account-activation>
16. Software Development Kit (SDK) - Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/sdk/overview>
17. I ve been having a hard time getting Ory integrated into a m Ory Community #ory-network, accessed May 25, 2025, <https://archive.ory.sh/t/27527017/i-ve-been-having-a-hard-time-getting-ory-integrated-into-a-m>
18. How we built a student project platform using Graphql, React ..., accessed May 25, 2025, <https://dev.to/peteole/how-we-built-a-student-project-platform-using-graphql-react-golang-ory-kratos-and-kubernetes-part-3-authentication-2603>
19. More direction on how to create custom styling · ory elements · Discussion #162 - GitHub, accessed May 25, 2025, <https://github.com/ory/elements/discussions/162>
20. Secure authentication in Next.js: Best practices & implementation - Ory, accessed May 25, 2025, <https://www.ory.sh/blog/add-auth-to-nextjs-security-best-practices>
21. < chilly king 10285> when trying to verify through the ory t Ory Community #ory-copilot, accessed May 25, 2025, <https://archive.ory.sh/t/26927895/u04uq68083h-when-trying-to-verify-through-the-ory-tunnel-the>
22. README.md - ory/elements - GitHub, accessed May 25, 2025, <https://github.com/ory/elements/blob/main/README.md>
23. Advanced integration | Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/bring-your-own-ui/custom-ui-advanced-integration>
24. Overview of sessions, Ory Session Cookies, and Ory Session Tokens | Ory, accessed May 25, 2025, <https://www.ory.sh/docs/kratos/session-management/overview>
25. elements/packages/elements-react/README.md at main · ory ..., accessed May 25, 2025, <https://github.com/ory/elements/blob/main/packages/elements-react/README.md>
26. Building for Production - Vite, accessed May 25, 2025, <https://vite.dev/guide/build>
27. ory/awesome-ory: A curated collection of examples and solutions created by the Ory Community. - GitHub, accessed May 25, 2025, <https://github.com/ory/awesome-ory>
28. @ory/client-fetch - npm, accessed May 25, 2025, <https://www.npmjs.com/package/@ory/client-fetch>
