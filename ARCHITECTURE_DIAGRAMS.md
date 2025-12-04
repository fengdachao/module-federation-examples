# Module Federation 架构图表集

> 以下图表使用 Mermaid 格式，可在 GitHub、GitLab 或任何支持 Mermaid 的 Markdown 渲染器中查看。

---

## 1. 核心概念关系图

```mermaid
graph TB
    subgraph "Module Federation 核心概念"
        Host[🏠 Host Application<br/>消费者/宿主应用]
        Remote[📦 Remote Application<br/>提供者/远程应用]
        Shared[🔗 Shared Dependencies<br/>共享依赖]
        
        Host -->|"remotes 配置"| Remote
        Remote -->|"exposes 配置"| Host
        Host <-->|"singleton/版本协商"| Shared
        Remote <-->|"singleton/版本协商"| Shared
    end
    
    subgraph "运行时文件"
        RE[remoteEntry.js<br/>远程入口文件]
        Remote --> RE
        Host -->|"动态加载"| RE
    end
```

---

## 2. 应用角色与关系

```mermaid
flowchart LR
    subgraph Shell["🏠 Shell/Host (主应用)"]
        direction TB
        H1[本地组件]
        H2[路由管理]
        H3[状态管理]
    end
    
    subgraph Remote1["📦 Remote 1"]
        R1[Button 组件]
        R2[Card 组件]
    end
    
    subgraph Remote2["📦 Remote 2"]
        R3[Chart 组件]
        R4[Table 组件]
    end
    
    subgraph Remote3["📦 Remote 3"]
        R5[Form 组件]
        R6[Modal 组件]
    end
    
    Shell -->|"消费"| Remote1
    Shell -->|"消费"| Remote2
    Shell -->|"消费"| Remote3
    
    Remote1 -.->|"共享 React"| Shell
    Remote2 -.->|"共享 React"| Shell
    Remote3 -.->|"共享 React"| Shell
```

---

## 3. 双向联邦架构

```mermaid
flowchart LR
    subgraph App1["App 1 (Port: 3001)"]
        A1E["exposes:<br/>./RedButton"]
        A1R["remotes:<br/>app2/BlueButton"]
    end
    
    subgraph App2["App 2 (Port: 3002)"]
        A2E["exposes:<br/>./BlueButton"]
        A2R["remotes:<br/>app1/RedButton"]
    end
    
    A1E <-->|"互相消费"| A2R
    A2E <-->|"互相消费"| A1R
    
    SharedDeps[("🔗 共享依赖<br/>React, React-DOM")]
    App1 <--> SharedDeps
    App2 <--> SharedDeps
```

---

## 4. 嵌套远程加载

```mermaid
flowchart LR
    App1["🏠 App1<br/>(Host)<br/>Port: 3001"]
    App2["📦 App2<br/>(Remote/Host)<br/>Port: 3002"]
    App3["📦 App3<br/>(Remote)<br/>Port: 3003"]
    
    App1 -->|"请求 ButtonContainer"| App2
    App2 -->|"请求 Button"| App3
    App3 -->|"返回 Button"| App2
    App2 -->|"返回 ButtonContainer<br/>(包含 Button)"| App1
    
    style App1 fill:#e1f5fe
    style App2 fill:#fff3e0
    style App3 fill:#e8f5e9
```

---

## 5. 运行时加载流程

```mermaid
sequenceDiagram
    participant User as 👤 用户
    participant Host as 🏠 Host App
    participant Runtime as ⚙️ MF Runtime
    participant Remote as 📦 Remote App
    
    User->>Host: 1. 访问 Host 应用
    Host->>Host: 2. 加载本地代码
    Host->>Runtime: 3. 初始化 Module Federation
    
    User->>Host: 4. 点击加载远程组件
    Host->>Runtime: 5. loadRemote('remote/Button')
    Runtime->>Remote: 6. 请求 remoteEntry.js
    Remote-->>Runtime: 7. 返回模块映射
    
    Runtime->>Runtime: 8. 协商共享依赖
    Note over Runtime: 检查 singleton 版本<br/>决定使用哪个版本
    
    Runtime->>Remote: 9. 请求组件 chunk
    Remote-->>Runtime: 10. 返回组件代码
    Runtime-->>Host: 11. 返回组件实例
    Host->>User: 12. 渲染远程组件
```

---

## 6. 共享依赖协商流程

```mermaid
flowchart TB
    Start([开始加载远程模块])
    CheckShared{检查共享依赖}
    HostHas{Host 有此依赖?}
    VersionMatch{版本兼容?}
    Singleton{是 Singleton?}
    UseHost[使用 Host 版本]
    UseRemote[使用 Remote 版本]
    LoadBoth[加载两个版本]
    End([完成依赖协商])
    
    Start --> CheckShared
    CheckShared --> HostHas
    
    HostHas -->|是| VersionMatch
    HostHas -->|否| UseRemote
    
    VersionMatch -->|兼容| UseHost
    VersionMatch -->|不兼容| Singleton
    
    Singleton -->|是| UseHost
    Singleton -->|否| LoadBoth
    
    UseHost --> End
    UseRemote --> End
    LoadBoth --> End
    
    style UseHost fill:#c8e6c9
    style UseRemote fill:#fff9c4
    style LoadBoth fill:#ffcdd2
```

---

## 7. SSR 架构

```mermaid
flowchart TB
    subgraph Client["🌐 客户端"]
        Browser[浏览器]
        ClientBundle[Client Bundle]
    end
    
    subgraph Server["🖥️ 服务端"]
        NodeServer[Node.js Server]
        ServerBundle[Server Bundle]
        SSR[SSR Middleware]
    end
    
    subgraph Remote1["📦 Remote 1"]
        R1Client[/client/remoteEntry.js]
        R1Server[/server/remoteEntry.js]
    end
    
    subgraph Remote2["📦 Remote 2"]
        R2Client[/client/remoteEntry.js]
        R2Server[/server/remoteEntry.js]
    end
    
    Browser -->|"Hydration"| ClientBundle
    ClientBundle -->|"加载"| R1Client
    ClientBundle -->|"加载"| R2Client
    
    NodeServer --> SSR
    SSR -->|"UniversalFederation"| ServerBundle
    ServerBundle -->|"加载"| R1Server
    ServerBundle -->|"加载"| R2Server
    
    SSR -->|"返回 HTML"| Browser
```

---

## 8. 跨框架状态共享

```mermaid
flowchart TB
    subgraph Shell["🏠 Shell (React)"]
        ReactApp[React 应用]
        Display[状态显示]
    end
    
    subgraph Store["📦 Shared Store"]
        Effector[Effector Store]
        Counter[Counter State]
    end
    
    subgraph ReactRemote["📦 React Counter"]
        ReactCounter[React 计数器]
        ReactButtons[增加/减少按钮]
    end
    
    subgraph VueRemote["📦 Vue Counter"]
        VueCounter[Vue 计数器]
        VueButtons[增加/减少按钮]
    end
    
    ReactApp --> Display
    Display -->|"订阅"| Counter
    
    ReactButtons -->|"dispatch"| Counter
    VueButtons -->|"dispatch"| Counter
    
    Counter -->|"effector-react"| ReactRemote
    Counter -->|"effector-vue"| VueRemote
    
    style Effector fill:#fff3e0
    style Counter fill:#fff3e0
```

---

## 9. 动态远程加载

```mermaid
flowchart LR
    subgraph Host["🏠 Host App"]
        Init["init({ remotes: [...] })"]
        Load["loadRemote('app2/Widget')"]
        Render[渲染组件]
    end
    
    subgraph Runtime["⚙️ Runtime"]
        Registry[远程注册表]
        Loader[模块加载器]
        Cache[模块缓存]
    end
    
    subgraph Remotes["📦 远程应用"]
        App2["App2:3002"]
        App3["App3:3003"]
        App4["App4:3004"]
    end
    
    Init -->|"注册"| Registry
    Load -->|"查找"| Registry
    Registry -->|"获取入口"| Loader
    Loader -->|"动态加载"| Remotes
    Loader -->|"缓存"| Cache
    Cache -->|"返回组件"| Render
    
    style Runtime fill:#e3f2fd
```

---

## 10. 项目结构总览

```mermaid
mindmap
  root((Module Federation<br/>Examples))
    基础示例
      basic-host-remote
      bi-directional
      nested-remote
      self-healing
    高级 API
      dynamic-remotes
      runtime-plugins
      automatic-vendor-sharing
    SSR
      react-18-ssr
      nextjs-ssr
      angular-universal-ssr
    多框架
      react-in-vue
      vue2-in-vue3
      shared-store-cross-framework
    构建工具
      Webpack
      Rspack
      Vite
      Modern.js
    TypeScript
      typescript-monorepo
      typescript-react-fallback
```

---

## 11. 实现步骤流程

```mermaid
flowchart TB
    Step1["📋 Step 1: 项目规划<br/>• 确定 Host/Remote 边界<br/>• 识别共享模块<br/>• 规划端口"]
    Step2["📦 Step 2: 创建 Remote<br/>• 配置 ModuleFederationPlugin<br/>• 设置 exposes<br/>• 配置 shared"]
    Step3["🏠 Step 3: 创建 Host<br/>• 配置 remotes<br/>• 配置相同的 shared"]
    Step4["🔄 Step 4: 异步边界<br/>• index.js 动态导入 bootstrap<br/>• 让 webpack 协商依赖"]
    Step5["🔗 Step 5: 使用远程组件<br/>• import 远程模块<br/>• React.lazy + Suspense"]
    Step6["🚀 Step 6: 测试部署<br/>• 本地测试<br/>• 配置 CORS<br/>• 生产环境 URL"]
    
    Step1 --> Step2 --> Step3 --> Step4 --> Step5 --> Step6
    
    style Step1 fill:#e8eaf6
    style Step2 fill:#e3f2fd
    style Step3 fill:#e8f5e9
    style Step4 fill:#fff8e1
    style Step5 fill:#fce4ec
    style Step6 fill:#f3e5f5
```

---

## 12. 错误处理流程

```mermaid
flowchart TB
    Load[加载远程模块]
    Error{发生错误?}
    Retry{重试次数<3?}
    DoRetry[执行重试]
    ErrorBoundary[Error Boundary 捕获]
    Fallback[显示 Fallback 组件]
    Success[成功渲染]
    Report[上报错误日志]
    
    Load --> Error
    Error -->|否| Success
    Error -->|是| Retry
    Retry -->|是| DoRetry
    DoRetry --> Load
    Retry -->|否| ErrorBoundary
    ErrorBoundary --> Fallback
    ErrorBoundary --> Report
    
    style Success fill:#c8e6c9
    style Fallback fill:#fff9c4
    style Report fill:#ffcdd2
```

---

## 使用说明

这些图表可以在以下平台正确渲染：
- GitHub (README.md, Issues, PR)
- GitLab
- VS Code (Mermaid 插件)
- Notion
- Typora
- 各种支持 Mermaid 的在线编辑器

如需导出为图片，可以使用：
- [Mermaid Live Editor](https://mermaid.live/)
- VS Code Mermaid 插件导出功能
- `mmdc` CLI 工具
