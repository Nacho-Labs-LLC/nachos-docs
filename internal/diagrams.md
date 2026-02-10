---
title: "Architecture Diagrams"
description: "Internal reference — interactive Mermaid diagrams of every Nachos subsystem."
---

# Architecture Diagrams

Internal reference for the Nachos team. All diagrams are interactive — zoom and pan with the controls that appear on hover.

**Jump to:** [System Overview](#system-overview) | [Gateway](#gateway-internals) | [Message Flow](#message-flow) | [Policy Engine](#salsa-policy-engine) | [Context Management](#context-management) | [Tool Execution](#tool-execution) | [Subagents](#subagent-system) | [State Layer](#state-layer) | [Bus](#nats-message-bus) | [LLM Proxy](#llm-proxy) | [Channels](#channels-architecture) | [Skills](#skills-system) | [Audit](#audit-system) | [Containers](#container-architecture) | [Network](#network-topology) | [Config Flow](#configuration-flow) | [Security Gates](#security-gate-sequence) | [Session Lifecycle](#session-lifecycle) | [Package Graph](#monorepo-package-graph) | [CLI Tree](#cli-command-tree)

---

## System Overview

Full stack: users, channels, core layer, LLM layer, tools, skills, and external services.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#ffd700', 'primaryTextColor': '#000', 'primaryBorderColor': '#b8860b', 'lineColor': '#666', 'secondaryColor': '#fff5e6', 'tertiaryColor': '#f0f0f0'}}}%%

flowchart TB
    subgraph Users["Users"]
        WebUser["Web Browser"]
        SlackUser["Slack"]
        DiscordUser["Discord"]
        TelegramUser["Telegram"]
        WhatsAppUser["WhatsApp"]
    end

    subgraph Nachos["Nachos Stack (Docker Compose)"]
        subgraph Channels["Channels Layer"]
            WebChat["WebChat\n:8080"]
            Slack["Slack Adapter\n(@slack/bolt)"]
            Discord["Discord Adapter\n(discord.js)"]
            Telegram["Telegram Adapter"]
            WhatsApp["WhatsApp Adapter"]
        end

        subgraph CoreLayer["Core Layer"]
            subgraph GatewayBox["Gateway (Orchestrator)"]
                Router["Router\nMessage Routing"]
                SessionMgr["SessionManager\nSession CRUD"]
                Salsa["Salsa\nPolicy Engine"]
                ToolCoord["ToolCoordinator\nTool Execution"]
                SubagentOrch["SubagentOrchestrator\nSubagent Queue"]
                CtxMgr["ContextManager\nToken Budget"]
                StateLayer["StateLayer\nPersistent State"]
                AuditLog["AuditLogger\nEvent Logging"]
                DLP["DLP\nContent Scanning"]
                RateLim["RateLimiter\nThrottling"]
            end
            Bus["NATS Message Bus\nPub/Sub + Request/Reply"]
        end

        subgraph LLMLayer["LLM Layer"]
            LLMProxy["LLM Proxy\nProvider Abstraction"]
            Adapters["Adapters\nAnthropic | OpenAI | Ollama"]
        end

        subgraph ToolsLayer["Tools Layer (Containers)"]
            BrowserTool["Browser\n(Playwright)"]
            FSTool["Filesystem\nRead/Write"]
            CodeRunner["Code Runner\nJS/Python"]
            WebFetchTool["Web Fetch\nHTTP + SSRF Guard"]
            CopilotTool["Copilot CLI"]
            ClaudeCodeMCP["Claude Code MCP"]
        end

        subgraph SkillsLayer["Skills Layer (In-Process)"]
            ShellTool["ShellTool\nCLI Executor"]
            GoPlaces["goplaces\n(lookup)"]
            GifGrep["gifgrep\n(media)"]
            Summarize["summarize\n(summarize)"]
            Gog["gog\n(workspace)"]
        end
    end

    subgraph External["External Services"]
        Anthropic["Anthropic API"]
        OpenAI["OpenAI API"]
        Ollama["Ollama (Local)"]
        SlackAPI["Slack API"]
        DiscordAPI["Discord API"]
        TelegramAPI["Telegram API"]
        GoogleAPI["Google APIs"]
    end

    WebUser <--> WebChat
    SlackUser <--> Slack
    DiscordUser <--> Discord
    TelegramUser <--> Telegram
    WhatsAppUser <--> WhatsApp

    Channels <-->|NATS| Bus
    Bus <--> GatewayBox
    Bus <-->|nachos.llm.*| LLMProxy
    LLMProxy --> Adapters
    Bus <-->|nachos.tool.*| ToolsLayer

    ToolCoord -->|Local| ShellTool
    ShellTool --> GoPlaces & GifGrep & Summarize & Gog

    Adapters <--> Anthropic & OpenAI & Ollama
    Slack <--> SlackAPI
    Discord <--> DiscordAPI
    Telegram <--> TelegramAPI
    GoPlaces & Gog <--> GoogleAPI
```

---

## Gateway Internals

The gateway is the central orchestrator. Every subsystem initializes from `GatewayOptions` at startup.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Gateway["Gateway (core/gateway)"]
        direction TB

        subgraph Init["Initialization"]
            Config["GatewayOptions\ndbPath, healthPort, bus,\nchannels[], policyConfig,\nauditConfig, dlpConfig, etc."]
        end

        subgraph Routing["Message Routing"]
            Router["Router\n- registerHandler()\n- route()\n- subscribeToChannel()\n- processInboundMessage()\n- sendLLMRequest()\n- sendToolRequest()\n- sendToChannel()\n- checkAndCompactContext()"]
            RateLimiter["RateLimiter\nToken bucket\nper user/action"]
        end

        subgraph Sessions["Session Management"]
            SessionManager["SessionManager\n- getOrCreateSession()\n- addMessage()\n- replaceMessages()\n- getSessionWithMessages()"]
            StateStorage["StateStorage\n(SQLite via better-sqlite3)\nTables: sessions, messages"]
        end

        subgraph Security["Security"]
            Salsa["Salsa Policy Engine\n- evaluate()\n- reload()\n- getStats()"]
            DLPLayer["DLP Security Layer\nContent scanning\nSensitive data detection"]
            AuditLogger["AuditLogger\nFile | SQLite | Webhook\nComposite provider"]
        end

        subgraph ToolExec["Tool Execution"]
            ToolCoordinator["ToolCoordinator\n- executeTools()\n- executeSingle()\nParallel/Sequential routing"]
            LocalToolHandler["LocalToolHandler\nexec/shell -> ShellTool"]
            ToolCache["ToolCache\nResult caching + TTL"]
            ApprovalManager["ApprovalManager\nUser confirmation for\nRESTRICTED tier tools"]
        end

        subgraph Context["Context Management"]
            CtxManager["ContextManager\n- checkBeforeTurn()\n- compact()"]
            MemoryPipeline["MemoryPipeline\nProactive history extraction"]
        end

        subgraph Agents["Subagent System"]
            SubagentMgr["SubagentManager\n- run(task)\nModes: full | host"]
            SubagentOrch["SubagentOrchestrator\n- enqueue()\n- drainQueue()\n- announce()"]
            DockerSandbox["DockerSubagentSandbox\nFull container isolation"]
            SandboxMgr["SandboxManager\nManages sandbox lifecycle"]
        end

        subgraph State["Persistent State"]
            SL["StateLayer\nComposed stores"]
            PromptAsm["PromptAssembler\nSystem prompt assembly"]
        end

        subgraph Stream["Streaming"]
            StreamMap["streamingSessions\nMap of sessionId to stream"]
        end
    end

    Config --> Router & SessionManager & Salsa & ToolCoordinator & SubagentOrch & SL
    Router --> RateLimiter
    Router --> CtxManager
    SessionManager --> StateStorage
    ToolCoordinator --> Salsa & LocalToolHandler & ToolCache & ApprovalManager
    SubagentOrch --> SubagentMgr
    SubagentMgr --> DockerSandbox
    SL --> PromptAsm
```

---

## Message Flow

Full sequence: user message through rate limiting, policy, DLP, session, context management, LLM, tool execution, and response delivery.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant Ch as Channel Container
    participant Bus as NATS Bus
    participant RL as RateLimiter
    participant S as Salsa
    participant DLP as DLP Scanner
    participant R as Router
    participant SM as SessionManager
    participant CM as ContextManager
    participant LP as LLM Proxy
    participant TC as ToolCoordinator
    participant T as Tool (Container/Local)

    U->>Ch: Send message
    Ch->>Bus: Publish nachos.channel.{ch}.inbound

    Bus->>R: Deliver inbound message
    R->>RL: Check rate limit
    RL-->>R: OK / Throttled

    R->>S: evaluate(resource=channel, action=receive)
    S-->>R: allowed / denied

    R->>DLP: scan(content)
    DLP-->>R: clean / redacted

    R->>SM: getOrCreateSession()
    SM-->>R: Session
    R->>SM: addMessage(user message)

    R->>CM: checkBeforeTurn(messages, budget)
    CM-->>R: {budget, needsCompaction, action}

    alt Context needs compaction
        R->>CM: compact(messages, action)
        CM-->>R: {compacted messages, extracted facts}
        R->>SM: replaceMessages(compacted)
        R->>Bus: Publish nachos.context.compaction
    end

    R->>Bus: Request nachos.llm.request
    Bus->>LP: Forward LLM request
    LP->>LP: Select provider + adapter
    LP->>LP: retryWithBackoff -> API call
    LP-->>Bus: LLM response
    Bus-->>R: Forward response

    alt Response contains tool_calls
        R->>TC: executeTools(toolCalls[])
        TC->>S: evaluate(resource=tool, action=execute)
        S-->>TC: allowed / denied

        alt Local tool (exec/shell)
            TC->>T: localToolHandler.execute()
            T-->>TC: ToolResult
        else Remote container tool
            TC->>Bus: Request nachos.tool.{name}.request
            Bus->>T: Forward to tool container
            T-->>Bus: Tool result
            Bus-->>TC: Forward result
        end

        TC-->>R: ToolResult[]
        R->>SM: addMessage(tool results)
        R->>Bus: Request nachos.llm.request (with tool results)
        Bus->>LP: Forward follow-up
        LP-->>Bus: Final response
        Bus-->>R: Forward
    end

    R->>SM: addMessage(assistant response)
    R->>Bus: Publish nachos.channel.{ch}.outbound
    Bus->>Ch: Deliver outbound
    Ch->>U: Display response
```

---

## Salsa Policy Engine

Policy loading, evaluation, condition matching, and decision output.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Input["Security Request"]
        Req["SecurityRequest\n----------\nrequestId\nuserId, sessionId\nsecurityMode\nresource: {type, id}\naction: string\nmetadata: {...}\ntimestamp"]
    end

    subgraph Salsa["Salsa Policy Engine"]

        subgraph Loader["PolicyLoader"]
            YAML["policies/*.yaml\nstrict | standard | permissive"]
            Parse["Parse YAML"]
            Validate["PolicyValidator\nSchema validation"]
            HotReload["File Watcher\nHot-reload on change"]
        end

        subgraph Evaluator["PolicyEvaluator"]
            PrioritySort["Rules sorted by\npriority (desc)"]

            subgraph RuleMatch["Rule Matching (per rule)"]
                Criteria["matchesCriteria()\n- resource type\n- action\n- resourceId"]
                Conditions["matchesConditions()\nAND logic across all"]
                Operators["Operators:\nequals | not_equals\nin | not_in\ncontains | matches\nstarts_with | ends_with"]
            end

            FieldRes["getFieldValue()\n- security_mode\n- user_id, session_id\n- resource_type, resource_id\n- action\n- metadata.* (dot notation)"]
        end
    end

    subgraph Output["Security Result"]
        Allow["allowed: true\neffect: 'allow'\nruleId, reason"]
        Deny["allowed: false\neffect: 'deny'\nruleId, reason"]
        Default["Default Effect\n(deny if no rule matches)"]
    end

    YAML --> Parse --> Validate --> PrioritySort
    HotReload -.->|File change| Parse

    Req --> PrioritySort
    PrioritySort --> Criteria
    Criteria -->|Match| Conditions
    Criteria -->|No match| PrioritySort
    Conditions --> Operators
    Operators --> FieldRes
    FieldRes -->|All conditions met| Allow
    FieldRes -->|Condition failed| PrioritySort
    PrioritySort -->|No rules matched| Default
    Default --> Deny
```

### Policy Rule Class Diagram

```mermaid
%%{init: {'theme': 'base'}}%%

classDiagram
    class PolicyFile {
        +string version
        +PolicyMetadata metadata
        +PolicyRule[] rules
    }

    class PolicyMetadata {
        +string name
        +string description
        +string mode
    }

    class PolicyRule {
        +string id
        +string description
        +number priority
        +RuleMatch match
        +RuleCondition[] conditions
        +string effect
        +string reason
    }

    class RuleMatch {
        +string|string[] resource
        +string|string[] action
        +string|string[] resourceId
    }

    class RuleCondition {
        +string field
        +string operator
        +unknown value
    }

    PolicyFile --> PolicyMetadata
    PolicyFile --> "1..*" PolicyRule
    PolicyRule --> RuleMatch
    PolicyRule --> "0..*" RuleCondition

    note for PolicyRule "priority: higher = checked first\neffect: 'allow' | 'deny'\nFirst matching rule wins"

    note for RuleCondition "Operators:\nequals, not_equals\nin, not_in\ncontains, matches\nstarts_with, ends_with"
```

### Security Modes Comparison

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart LR
    subgraph Strict["Strict Mode (default)"]
        S1["All tools disabled"]
        S2["Allowlisted DMs only"]
        S3["Full audit logging"]
        S4["No network egress"]
    end

    subgraph Standard["Standard Mode"]
        ST1["Common tools enabled"]
        ST2["Pairing-based DM access"]
        ST3["Selective audit"]
        ST4["Controlled egress"]
    end

    subgraph Permissive["Permissive Mode"]
        P1["All tools enabled"]
        P2["Full channel access"]
        P3["Minimal audit"]
        P4["Full network access"]
    end

    Strict ---|"Opt-in"| Standard ---|"Explicit opt-in"| Permissive
```

---

## Context Management

Budget zones, sliding window compaction, memory extraction pipeline, and bus events.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Trigger["Before Each LLM Turn"]
        RouterCall["Router.checkAndCompactContext()\nsessionId, contextWindow=200k,\nsystemPromptTokens"]
    end

    subgraph CM["ContextManager"]
        subgraph Check["checkBeforeTurn()"]
            Budget["BudgetCalculator\nCalculate token usage"]
            Sliding["SlidingWindowManager\nshould slide?"]
            Result["ContextCheckResult\n{budget, needsCompaction, action}"]
        end

        subgraph Compact["compact()"]
            Snapshot["1. Create snapshot\n(if enabled)"]
            Strategy["2. Determine slide strategy\nturn-based | hybrid"]
            Execute["3. Execute sliding window\nRemove oldest messages"]
            ValidateC["4. Validate result"]
            Summarize["5. Generate summary\n(if enabled)"]
            Extract["6. Extract history\n(facts, decisions, tasks)"]
            Recalc["7. Recalculate budget"]
        end
    end

    subgraph Zones["Budget Zones"]
        Green["Green\n< 70%\nSafe"]
        Yellow["Yellow\n70-80%\nMonitoring"]
        Orange["Orange\n80-90%\nWarning"]
        Red["Red\n90-95%\nCritical"]
        Critical["Critical\n> 95%\nForced compaction"]
    end

    subgraph Pipeline["MemoryPipeline"]
        MemExtract["handleExtraction()\nFacts, decisions, tasks"]
        MemStore["storeExtracted()\nPersist to MemoryStore"]
    end

    subgraph Events["Bus Events"]
        E1["nachos.context.budget_update"]
        E2["nachos.context.zone_change"]
        E3["nachos.context.compaction"]
        E4["nachos.context.extraction"]
        E5["nachos.context.snapshot"]
    end

    RouterCall --> Budget
    Budget --> Sliding
    Sliding --> Result

    Result -->|needsCompaction=true| Snapshot
    Snapshot --> Strategy --> Execute --> ValidateC --> Summarize --> Extract --> Recalc

    Budget --> Green & Yellow & Orange & Red & Critical

    Extract --> MemExtract --> MemStore

    Recalc --> E1 & E3
    Budget -->|Zone changed| E2
    MemExtract --> E4
    Snapshot --> E5
```

### Sliding Window Configuration

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart LR
    subgraph Config["Sliding Window Config"]
        Mode["mode:\nhybrid | aggressive | conservative"]
        Thresholds["thresholds:\nproactivePrune: 0.6\nlightCompaction: 0.75\naggressiveCompaction: 0.85\nemergency: 0.95"]
        KeepRecent["keepRecent:\nturns: 10\nmessages: 20\ntokenBudget: 10000"]
        SlideStrat["slideStrategy:\nturn | hybrid"]
    end

    subgraph Actions["Compaction Actions"]
        Proactive["Proactive Prune\nRemove 20% oldest"]
        Light["Light Compaction\nRemove 40% oldest\n+ summarize"]
        Aggressive["Aggressive\nRemove 60% oldest\n+ summarize + extract"]
        Emergency["Emergency\nKeep only recent N\n+ full extraction"]
    end

    Thresholds --> Proactive & Light & Aggressive & Emergency
```

---

## Tool Execution

Security tier resolution, policy checks, approval gates, caching, and local vs. remote routing.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph LLMResponse["LLM Response"]
        ToolCalls["tool_calls[]\n{id, name, input}"]
    end

    subgraph TC["ToolCoordinator"]
        Detect["canExecuteInParallel()\n- Duplicate tools? -> sequential\n- Write-then-read? -> sequential\n- Otherwise -> parallel"]

        subgraph Single["executeSingle(call)"]
            ResolveTier["Resolve Security Tier\n- code_runner -> RESTRICTED\n- filesystem_write -> ELEVATED\n- browser -> STANDARD\n- read/list -> SAFE"]

            PolicyCheck["Salsa Policy Check\nevaluate(resource=tool,\naction=execute)"]

            ApprovalCheck["Approval Check\n(if RESTRICTED tier)"]

            CacheCheck["Cache Check\ncache.get(call)"]

            subgraph Route["Execution Route"]
                IsLocal{"isLocalTool?\n(exec/shell)"}
                Local["LocalToolHandler\n-> ShellTool"]
                Remote["Bus Request\nnachos.tool.{name}.request"]
            end

            CacheSet["Cache Set\ncache.set(call, result, ttl)"]
        end
    end

    subgraph Results["Tool Results"]
        Success["ToolResult\nsuccess: true\ncontent, duration"]
        Error["ToolResult\nsuccess: false\nerror code + message"]
    end

    ToolCalls --> Detect
    Detect -->|Parallel| Single
    Detect -->|Sequential| Single

    Single --> ResolveTier --> PolicyCheck
    PolicyCheck -->|Denied| Error
    PolicyCheck -->|Allowed| ApprovalCheck
    ApprovalCheck -->|Denied| Error
    ApprovalCheck -->|Approved| CacheCheck
    CacheCheck -->|Hit| Success
    CacheCheck -->|Miss| IsLocal
    IsLocal -->|Yes| Local --> Success
    IsLocal -->|No| Remote --> Success
    Success --> CacheSet
```

### Security Tiers

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart LR
    subgraph Tiers["Security Tiers"]
        Safe["SAFE\nread, list, get\nNo approval needed"]
        Standard["STANDARD\nbrowser\nPolicy check only"]
        Elevated["ELEVATED\nfilesystem write/edit/patch\nPolicy + logging"]
        Restricted["RESTRICTED\ncode_runner\nPolicy + approval required"]
    end

    Safe --> Standard --> Elevated --> Restricted
```

### Shell Tool (Skills Execution)

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Input["Tool Call"]
        ExecCall["exec({command: 'goplaces search coffee'})\nor shell({command: ...})"]
    end

    subgraph LTH["LocalToolHandler"]
        IsLocal["isLocalTool(name)\nname === 'exec' | 'shell'"]
        ExtractCmd["Extract command\nfrom parameters"]
    end

    subgraph ST["ShellTool"]
        ParseBin["Parse binary name\nfrom command string"]
        CheckAllowed["Check allowedTools[]\nBinary in allowlist?"]
        CheckEnv["Validate required\nenvironment variables"]

        subgraph Spawn["spawnProcess()"]
            ChildProc["spawn(command)\nshell=true, cwd, env"]
            Timeout["Timeout handler\nSIGTERM then 5s then SIGKILL"]
            StdOut["Capture stdout\n(max 100KB, truncate)"]
            StdErr["Capture stderr"]
        end

        BuildResult["Build ExecResult\nexitCode, signal,\nstdout, stderr,\nduration, timedOut,\ntruncated"]
    end

    subgraph Skills["Allowed Skill Binaries"]
        GP["goplaces\ngroup: lookup\nenv: GOOGLE_PLACES_API_KEY\ntimeout: 30s"]
        GG["gifgrep\ngroup: media\ntimeout: 60s"]
        Sum["summarize\ngroup: summarize\ntimeout: 120s"]
        G["gog\ngroup: workspace\ntimeout: 45s"]
    end

    ExecCall --> IsLocal --> ExtractCmd --> ParseBin --> CheckAllowed
    CheckAllowed -->|Not allowed| ErrorOut["COMMAND_NOT_ALLOWED"]
    CheckAllowed -->|Allowed| CheckEnv
    CheckEnv -->|Missing env| ErrorOut2["ENV_MISSING"]
    CheckEnv -->|OK| ChildProc
    ChildProc --> Timeout & StdOut & StdErr
    StdOut & StdErr --> BuildResult

    Skills -.->|Registered in| CheckAllowed
```

---

## Subagent System

Queue-based orchestration, sandbox modes, and child session management.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Request["Subagent Request"]
        Req["SubagentRunRequest\ntask, model,\nsessionConfig,\nsandboxMode"]
    end

    subgraph Orchestrator["SubagentOrchestrator"]
        Enqueue["enqueue()\n- Generate runId\n- Create workspace dir\n- Create child session\n- Add to queue"]

        Queue["Queue (FIFO)\n[runId, runId, ...]"]

        Drain["drainQueue()\nwhile queue.length > 0\nAND running < maxConcurrent"]

        ExecRun["executeRun(entry)\n- Set status=running\n- Call subagentManager.run()\n- Store result\n- Set status=completed|failed"]

        Announce["announce(entry)\n- Build announce prompt\n- Run subagent for summary\n- Publish to requester channel"]

        State["Run State\nMap of runId to entry\nstatus: queued|running|\ncompleted|failed"]
    end

    subgraph Manager["SubagentManager"]
        ModeCheck{"task.sandboxMode\nor default mode?"}
        HostMode["Host Mode\nRun in gateway process\nsendRequest(task.request)"]
        FullMode["Full Sandbox Mode\nDocker container isolation"]
    end

    subgraph Docker["DockerSubagentSandbox"]
        Container["Spawn container\n- Mount workspace\n- Inject env vars\n- Run LLM request"]
        Cleanup["Container cleanup\non completion"]
    end

    subgraph Session["Child Session"]
        Create["createSubagentSession()\nchannel='subagent'\nconversationId=runId"]
        Meta["Session metadata:\nrunId, label, profile,\nagentId, requester,\nworkspaceDir"]
    end

    Req --> Enqueue
    Enqueue --> Queue
    Enqueue --> Create --> Meta
    Queue --> Drain
    Drain --> ExecRun
    ExecRun --> ModeCheck
    ModeCheck -->|host| HostMode
    ModeCheck -->|full| FullMode --> Container --> Cleanup
    ExecRun --> Announce
    ExecRun --> State
```

### Subagent Lifecycle

```mermaid
%%{init: {'theme': 'base'}}%%

stateDiagram-v2
    [*] --> Queued: enqueue()

    Queued --> Running: drainQueue() picks up

    Running --> Completed: Success
    Running --> Failed: Error

    Completed --> Announced: announce()
    Failed --> Announced: announce()

    Announced --> [*]

    note right of Queued
        Waiting in FIFO queue
        maxConcurrent controls parallelism
    end note

    note right of Running
        Executing in host or Docker sandbox
        LLM request being processed
    end note

    note right of Announced
        Result published to requester
        via router.sendToChannel()
    end note
```

---

## State Layer

Policy-gated, audit-hooked access to identity, memory, user profiles, and session state stores.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph SL["StateLayer (Orchestrator)"]
        PolicyGate["ensureAllowed()\n-> policyCheck()\nDeny = throw error"]
        AuditHook["auditAllowed()\n-> auditLogger()\nLog all operations"]
    end

    subgraph Stores["Composed Stores"]
        subgraph Identity["IdentityStore"]
            IdFS["FilesystemIdentityStore"]
            IdPG["PostgresIdentityStore"]
        end

        subgraph Memory["MemoryStore"]
            MemFS["FilesystemMemoryStore"]
            MemPG["PostgresMemoryStore"]
        end

        subgraph UserProfile["UserProfileStore"]
            UPFS["FilesystemUserProfileStore"]
            UPPG["PostgresUserProfileStore"]
        end

        subgraph SessionState["SessionStateStore"]
            SSIM["InMemorySessionStateStore"]
            SSRedis["RedisSessionStateStore"]
        end
    end

    subgraph Prompt["PromptAssembler"]
        Assemble["assemblePrompt(params)\nCombines identity + memory\n+ user profile + session\ninto system prompt"]
    end

    subgraph Operations["Available Operations"]
        IdOps["Identity:\ngetIdentity(agentId)\nputIdentity(profile)\ndeleteIdentity(agentId)"]
        MemOps["Memory:\nappendMemoryEntry(entry)\nappendMemoryFacts(facts[])\nqueryMemory(query)\ndeleteMemoryEntry(id)"]
        UPOps["UserProfile:\ngetUserProfile(agentId, userId)\nputUserProfile(profile)\ndeleteUserProfile(agentId, userId)"]
        SSOps["SessionState:\ngetSessionState(sessionId)\nsetSessionState(record)\ntouchSessionState(sessionId)\ndeleteSessionState(sessionId)"]
    end

    SL --> PolicyGate
    PolicyGate --> Identity & Memory & UserProfile & SessionState
    Identity & Memory & UserProfile & SessionState --> AuditHook
    SL --> Prompt

    IdOps -.-> Identity
    MemOps -.-> Memory
    UPOps -.-> UserProfile
    SSOps -.-> SessionState
```

### Session Storage (ER Diagram)

```mermaid
%%{init: {'theme': 'base'}}%%

erDiagram
    sessions {
        string id PK
        string channel
        string conversationId
        string userId
        string status "active | paused | closed"
        text systemPrompt
        text config "JSON"
        text metadata "JSON"
        datetime createdAt
        datetime updatedAt
    }

    messages {
        string id PK
        string sessionId FK
        string role "user | assistant | system | tool"
        text content
        text metadata "JSON"
        datetime createdAt
    }

    sessions ||--o{ messages : "has many"
```

---

## NATS Message Bus

Client API, full topic namespace, and bus implementations.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph BusClient["NachosBusClient"]
        Pub["publish(topic, data)"]
        Sub["subscribe(topic, handler)"]
        ReqRep["request(topic, data, timeout)\nCorrelation ID matching"]
        Health["health()\nPing + latency"]
        Events["Event Monitoring\nconnect | disconnect\nreconnect | error"]
    end

    subgraph Topics["NATS Topic Namespace"]
        subgraph ChTopics["Channel Topics"]
            ChIn["nachos.channel.{name}.inbound"]
            ChOut["nachos.channel.{name}.outbound"]
        end

        subgraph LLMTopics["LLM Topics"]
            LLMReq["nachos.llm.request"]
            LLMRes["nachos.llm.response"]
            LLMStream["nachos.llm.stream.{sessionId}"]
        end

        subgraph ToolTopics["Tool Topics"]
            ToolReq["nachos.tool.{name}.request"]
            ToolRes["nachos.tool.{name}.response"]
        end

        subgraph PolicyTopics["Policy Topics"]
            PolCheck["nachos.policy.check"]
            PolResult["nachos.policy.result"]
        end

        subgraph ContextTopics["Context Topics"]
            CtxCompact["nachos.context.compaction"]
            CtxExtract["nachos.context.extraction"]
            CtxZone["nachos.context.zone_change"]
            CtxSnap["nachos.context.snapshot"]
            CtxBudget["nachos.context.budget_update"]
        end

        subgraph GwTopics["Gateway Topics"]
            SaSpawn["nachos.gateway.subagents.spawn"]
            SaList["nachos.gateway.subagents.list"]
            SaInfo["nachos.gateway.subagents.info"]
            SaStop["nachos.gateway.subagents.stop"]
            SaLog["nachos.gateway.subagents.log"]
            SbExplain["nachos.gateway.sandbox.explain"]
            SbList["nachos.gateway.sandbox.list"]
        end

        subgraph OtherTopics["Other Topics"]
            AuditLog["nachos.audit.log"]
            HealthPing["nachos.health.ping"]
            ConfigUp["nachos.config.update"]
            ConfigDone["nachos.config.updated"]
        end
    end

    subgraph Impls["Bus Implementations"]
        NATS["NatsBusAdapter\nProduction (NATS server)"]
        InMem["InMemoryMessageBus\nTesting"]
    end

    BusClient --> Topics
    NATS --> BusClient
    InMem --> BusClient
```

### Message Envelope

```mermaid
%%{init: {'theme': 'base'}}%%

classDiagram
    class MessageEnvelope {
        +string id  "UUID"
        +string timestamp  "ISO 8601"
        +string source  "component name"
        +string type  "message type"
        +string correlationId  "request/reply match"
        +unknown payload  "message content"
    }

    class ChannelInboundMessage {
        +string channel
        +string conversationId
        +string userId
        +string content
        +Record metadata
    }

    class ChannelOutboundMessage {
        +string channel
        +string conversationId
        +string content
        +Record metadata
    }

    class LLMRequestType {
        +string sessionId
        +Message[] messages
        +string systemPrompt
        +LLMOptions options
    }

    class LLMResponseType {
        +string sessionId
        +string content
        +ToolCall[] toolCalls
        +TokenUsage usage
    }

    class ToolCall {
        +string id
        +string name
        +Record input
    }

    class ToolResult {
        +string toolCallId
        +boolean success
        +ContentBlock[] content
        +string error
        +Record metadata
    }

    MessageEnvelope --> ChannelInboundMessage : "payload (inbound)"
    MessageEnvelope --> ChannelOutboundMessage : "payload (outbound)"
    MessageEnvelope --> LLMRequestType : "payload (llm request)"
    MessageEnvelope --> LLMResponseType : "payload (llm response)"
    MessageEnvelope --> ToolCall : "payload (tool request)"
    MessageEnvelope --> ToolResult : "payload (tool response)"
```

---

## LLM Proxy

Provider selection, retry with backoff, cooldown management, failover chain, and streaming.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Input["Incoming Request"]
        LLMReq["LLMRequestType\nsessionId, messages[],\nsystemPrompt, options"]
    end

    subgraph Proxy["LLM Proxy Service"]
        subgraph Handler["handleRequest()"]
            BuildAttempts["Build attempt list\n1. Primary: config provider + model\n2. Fallbacks: fallback_order[]"]

            subgraph AttemptLoop["For each attempt"]
                GetAdapter["Get adapter from registry"]
                GetProfiles["Get profiles for provider\n(exclude cooled-down)"]
                Retry["retryWithBackoff()\nadapter.send(request)\n- Retry on rate_limit\n- Exponential backoff + jitter"]
            end

            Failover["On rate_limit/limit_reached:\n-> Try next fallback provider"]
        end

        subgraph Registry["AdapterRegistry"]
            AnthropicA["AnthropicAdapter\nAnthropic SDK"]
            OpenAIA["OpenAIAdapter\nOpenAI SDK"]
            OllamaAdpt["OllamaAdapter\nLocal REST API"]
        end

        subgraph Cooldowns["CooldownManager"]
            ProfileCD["Per-profile cooldowns\nExponential backoff:\nbase=60s, factor=2, max=900s"]
        end

        subgraph Metrics["Metrics Emission"]
            Usage["Token usage tracking\npromptTokens\ncompletionTokens\ntotalTokens"]
            Latency["Latency tracking\nrequestMs\ntimeToFirstChunk"]
            FailoverEvt["Failover events\nprovider, reason"]
        end
    end

    subgraph Streaming["Streaming Support"]
        StreamHandler["handleStream(request, onChunk)\n- adapter.stream() if available\n- Falls back to adapter.send()\n- Publishes chunks to\n  nachos.llm.stream.{sessionId}"]
    end

    subgraph Providers["External LLM APIs"]
        AnthropicAPI["api.anthropic.com"]
        OpenAIAPI["api.openai.com"]
        OllamaAPI["localhost:11434"]
    end

    Input --> BuildAttempts
    BuildAttempts --> GetAdapter --> GetProfiles --> Retry
    Retry -->|Rate limited| Cooldowns
    Cooldowns --> Failover --> GetAdapter
    Retry -->|Success| Metrics

    GetAdapter --> Registry
    AnthropicA --> AnthropicAPI
    OpenAIA --> OpenAIAPI
    OllamaAdpt --> OllamaAPI

    Input -->|stream=true| StreamHandler
    StreamHandler --> Registry
```

### Provider Failover Sequence

```mermaid
%%{init: {'theme': 'base'}}%%

sequenceDiagram
    participant R as LLM Proxy
    participant P1 as Primary Provider
    participant CD as CooldownManager
    participant P2 as Fallback Provider 1
    participant P3 as Fallback Provider 2

    R->>P1: adapter.send(request)
    P1-->>R: 429 Rate Limited

    R->>CD: cooldown(profile)
    CD-->>R: Exponential backoff set

    R->>R: retryWithBackoff()
    R->>P1: Retry attempt
    P1-->>R: 429 Rate Limited (again)

    R->>R: All retries exhausted
    R->>R: Emit failover event

    R->>P2: adapter.send(request)
    P2-->>R: 200 OK + Response

    R->>R: Emit metrics (latency, tokens)
```

---

## Channels Architecture

Base adapter, platform-specific implementations, and bus integration.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Base["Base Channel Adapter (@nachos/channel-base)"]
        Factory["createChannelBus()\nWraps NATS for channel topics"]
        DmPolicy["resolveDmPolicy()\nGet DM policy from config"]
        GroupPolicy["resolveGroupPolicy()\nGet group message policy"]
        PairingStore["createPairingStore()\nUser pairing for permissions"]
        PairCmd["parsePairingCommand()\nHandle /pair commands"]
    end

    subgraph SlackCh["Slack Channel"]
        SlackSDK["@slack/bolt SDK"]
        SocketMode["Socket Mode (WebSocket)\nor HTTP Webhooks"]
        SlackHandler["Message Handler\n-> Normalize to ChannelInboundMessage\n-> Publish to NATS"]
        SlackAllow["User allowlist\nper workspace"]
    end

    subgraph DiscordCh["Discord Channel"]
        DiscordSDK["discord.js SDK"]
        DiscordHandler["Message Handler\n-> Normalize\n-> Publish to NATS"]
        DiscordAllow["User allowlist\nper server"]
    end

    subgraph TelegramCh["Telegram Channel"]
        TelegramSDK["Telegram Bot API"]
        TelegramHandler["Message Handler\n-> Normalize\n-> Publish to NATS"]
    end

    subgraph WhatsAppCh["WhatsApp Channel"]
        WhatsAppSDK["WhatsApp Business API"]
        WhatsAppHandler["Message Handler\n-> Normalize\n-> Publish to NATS"]
    end

    Base --> SlackCh & DiscordCh & TelegramCh & WhatsAppCh

    subgraph BusCh["NATS Bus"]
        Inbound["nachos.channel.{name}.inbound"]
        Outbound["nachos.channel.{name}.outbound"]
    end

    SlackCh <--> BusCh
    DiscordCh <--> BusCh
    TelegramCh <--> BusCh
    WhatsAppCh <--> BusCh
```

---

## Skills System

SKILL.md loading, prompt injection, exec flow, and tool group policy gating.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph SkillFiles["SKILL.md Files (skills/)"]
        GP["goplaces/SKILL.md\n----------\nname: goplaces\ngroup: lookup\nbins: [goplaces]\nenv: [GOOGLE_PLACES_API_KEY]"]
        GG["gifgrep/SKILL.md\n----------\nname: gifgrep\ngroup: media\nbins: [gifgrep]"]
        SM["summarize/SKILL.md\n----------\nname: summarize\ngroup: summarize\nbins: [summarize]"]
        G["gog/SKILL.md\n----------\nname: gog\ngroup: workspace\nbins: [gog]"]
    end

    subgraph Loader["SkillLoader"]
        Parse["Parse YAML frontmatter\n+ markdown body"]
        Filter["Filter by tool groups\n(policy-controlled)"]
        Inject["Inject into LLM\nsystem prompt"]
    end

    subgraph Execution["Execution Flow"]
        LLM["LLM reads SKILL.md\nin system prompt"]
        Call["LLM calls exec()\n{command: 'goplaces search coffee'}"]
        TC["ToolCoordinator\nevaluates policy"]
        LTH["LocalToolHandler\nroutes to ShellTool"]
        STExec["ShellTool\nspawns subprocess"]
        ResultExec["Output returned\nto LLM context"]
    end

    subgraph Groups["Tool Groups (Policy)"]
        Lookup["lookup\ngoplaces"]
        Media["media\ngifgrep"]
        SumGroup["summarize\nsummarize"]
        Workspace["workspace\ngog"]
    end

    SkillFiles --> Parse --> Filter --> Inject
    LLM --> Call --> TC --> LTH --> STExec
    STExec --> ResultExec --> LLM

    Groups -.->|Policy gating| TC
```

---

## Audit System

Event sources, composite provider fan-out, and audit event schema.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Sources["Audit Event Sources"]
        GatewayA["Gateway\nSession events"]
        SalsaA["Salsa\nPolicy decisions"]
        ToolsA["ToolCoordinator\nTool executions"]
        StateLA["StateLayer\nState operations"]
        RouterA["Router\nMessage routing"]
    end

    subgraph Logger["AuditLogger"]
        Emit["emit(event)\nNormalize + timestamp"]
    end

    subgraph ProvidersA["Audit Providers"]
        Composite["CompositeAuditProvider\nFans out to multiple"]
        FileP["FileAuditProvider\nJSON lines to file"]
        SQLiteP["SQLiteAuditProvider\nStructured storage"]
        WebhookP["WebhookAuditProvider\nHTTP POST delivery"]
    end

    subgraph Event["AuditEvent"]
        Fields["eventId: UUID\ntimestamp: ISO\neventType: string\noutcome: allow|deny|error\nuserId: string\nsessionId: string\nresource: {type, id}\naction: string\nmetadata: Record\ndurationMs: number"]
    end

    Sources --> Logger
    Logger --> Composite
    Composite --> FileP & SQLiteP & WebhookP
    Logger -.-> Event
```

---

## Container Architecture

All containers with their base images, resource limits, and network assignments.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Compose["Docker Compose"]
        subgraph Internal["nachos-internal (isolated, no external access)"]
            gateway["gateway\n--------\nnode:22-alpine\nnon-root, read-only FS\n512MB\n--------\nRouter, Salsa,\nToolCoord, SubagentOrch,\nContextMgr, StateLayer,\nAudit, DLP, ShellTool"]
            bus["bus\n--------\nnats:alpine\nnon-root\n256MB\n--------\nNATS Server\nPorts: 4222, 8222"]
            filesystem["filesystem\n--------\nnode:22-alpine\nnon-root\n128MB\n--------\nFile read/write"]
            coderunner["code-runner\n--------\nnode:22-alpine\nsandboxed\n512MB\n--------\nJS/Python execution"]
        end

        subgraph EgressNet["nachos-egress (controlled external access)"]
            llmproxy["llm-proxy\n--------\nnode:22-alpine\n-> LLM APIs only\n256MB\n--------\nAnthropic, OpenAI, Ollama"]
            webchat["webchat\n--------\nnode:22-alpine\n-> port 8080\n256MB"]
            slack["slack\n--------\nnode:22-alpine\n-> slack.com\n256MB"]
            discord["discord\n--------\nnode:22-alpine\n-> discord.com\n256MB"]
            telegram["telegram\n--------\nnode:22-alpine\n-> api.telegram.org\n256MB"]
            browser["browser\n--------\nplaywright\n-> configured domains\n1GB"]
            webfetch["web-fetch\n--------\nnode:22-alpine\n-> allowed URLs\n256MB\nSSRF protection"]
        end
    end

    subgraph Volumes["Volumes"]
        natsdata["nats-data"]
        sessionsdb["sessions.db"]
        policies["./policies"]
        workspace["./workspace"]
    end

    bus --- natsdata
    gateway --- sessionsdb
    gateway --- policies
    filesystem --- workspace

    Internal <-->|NATS| EgressNet
```

---

## Network Topology

External services, egress network, internal network, and host volume mounts.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TB
    subgraph Internet["Internet"]
        anthropic["api.anthropic.com"]
        openai["api.openai.com"]
        slackapi["api.slack.com"]
        discordapi["discord.com/api"]
        telegramapi["api.telegram.org"]
        websites["Allowed websites"]
        googleapi["Google APIs"]
    end

    subgraph EgressNetDiag["nachos-egress (controlled external)"]
        llm["LLM Proxy\n-> anthropic, openai"]
        slackC["Slack\n-> slack.com"]
        discordC["Discord\n-> discord.com"]
        telegramC["Telegram\n-> telegram.org"]
        browserC["Browser\n-> configured domains"]
        webfetchC["Web Fetch\n-> allowed URLs"]
    end

    subgraph InternalNet["nachos-internal (no external access)"]
        gatewayN["Gateway"]
        busN["NATS Bus"]
        fs["Filesystem"]
        code["Code Runner"]
    end

    subgraph Host["Host Machine"]
        workspaceV["./workspace"]
        stateV["./state"]
        policiesV["./policies"]
    end

    llm <--> anthropic & openai
    slackC <--> slackapi
    discordC <--> discordapi
    telegramC <--> telegramapi
    browserC <--> websites
    webfetchC <--> websites
    gatewayN <-.-> googleapi

    EgressNetDiag <-->|NATS| InternalNet

    fs --- workspaceV
    gatewayN --- stateV & policiesV
```

---

## Configuration Flow

TOML loading, schema validation, env interpolation, hot-reload, and Docker Compose generation.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart LR
    subgraph Sources["Configuration Sources"]
        TOML["nachos.toml\n(primary config)"]
        ENV[".env file\n+ shell env vars"]
        CLI["CLI flags\n(overrides)"]
    end

    subgraph Loader["@nachos/config"]
        LoadTOML["loader.ts\nParse TOML from disk"]
        Schema["schema.ts\nNachosConfig schema"]
        Validate["validation.ts\nValidate against schema"]
        EnvOverlay["env.ts\nEnvironment variable\ninterpolation"]
        RuntimeOL["runtime-overlay.ts\nRuntime state persistence"]
        Merge["Merge all sources\nCLI > ENV > TOML"]
    end

    subgraph HotReload["Hot Reload"]
        Watch["hotreload.ts\nFile watcher"]
        RegistryHR["registry.ts\nNotify subscribers"]
        Publish["Bus: nachos.config.updated"]
    end

    subgraph Output["Validated Config"]
        NachosConfig["NachosConfig\n- nachos (name, version)\n- llm (provider, model, profiles)\n- channels (slack, discord, etc.)\n- security (mode, audit)\n- tools (browser, filesystem, etc.)\n- context (sliding_window, etc.)"]
    end

    subgraph Generated["Generated Artifacts"]
        ComposeOut["docker-compose.yml"]
        Networks["Network definitions"]
        VolumeDefs["Volume mounts"]
        SecretInject["Secret injection"]
    end

    TOML --> LoadTOML --> Validate
    Schema --> Validate
    ENV --> EnvOverlay
    CLI --> Merge
    Validate --> EnvOverlay --> Merge --> RuntimeOL --> NachosConfig

    NachosConfig --> ComposeOut & Networks & VolumeDefs & SecretInject

    TOML -->|File change| Watch --> RegistryHR --> Publish
```

---

## Security Gate Sequence

Every request passes through 8 security checkpoints in order.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TD
    Req["Incoming Request"] --> RL

    subgraph Gates["Security Gates (in order)"]
        RL["1. Rate Limiter\nToken bucket per user/action"]
        Policy["2. Salsa Policy Check\nresource + action + conditions"]
        DLPScan["3. DLP Scanner\nSensitive data detection"]
        ToolPolicy["4. Tool Policy Check\nTool-specific rules"]
        Approval["5. Approval Manager\n(RESTRICTED tier only)"]
        Cache["6. Tool Cache\nSkip if cached"]
        StatePolicy["7. State Layer Policy\nAccess control on state ops"]
        Audit["8. Audit Logger\nRecord all events"]
    end

    RL -->|Under limit| Policy
    RL -->|Over limit| Throttled["Throttled"]

    Policy -->|Allowed| DLPScan
    Policy -->|Denied| Denied1["Denied"]

    DLPScan -->|Clean| ToolPolicy
    DLPScan -->|Sensitive| Blocked["Blocked + Redacted"]

    ToolPolicy -->|Allowed| Approval
    ToolPolicy -->|Denied| Denied2["Denied"]

    Approval -->|Approved| Cache
    Approval -->|Denied| Denied3["Denied"]

    Cache -->|Hit| CacheResult["Cached Result"]
    Cache -->|Miss| ExecuteGate["Execute"]

    ExecuteGate --> StatePolicy
    StatePolicy -->|Allowed| Audit
    StatePolicy -->|Denied| Denied4["Denied"]

    Audit --> Done["Complete"]

    Throttled & Denied1 & Denied2 & Denied3 & Blocked & Denied4 --> Audit
```

---

## Session Lifecycle

Full state machine including the active sub-states (processing, waiting for LLM, executing tools).

```mermaid
%%{init: {'theme': 'base'}}%%

stateDiagram-v2
    [*] --> Created: getOrCreateSession()

    Created --> Active: First message

    Active --> Active: Message exchange
    Active --> Active: Tool execution
    Active --> Active: Context compaction
    Active --> Paused: User timeout

    Active --> Ended: /end command
    Active --> Ended: Session expired

    Paused --> Active: New message
    Paused --> Ended: Extended timeout

    Ended --> [*]

    state Active {
        [*] --> Processing
        Processing --> WaitingForLLM: sendLLMRequest()
        WaitingForLLM --> Processing: LLM response
        WaitingForLLM --> ExecutingTools: tool_calls in response
        ExecutingTools --> WaitingForLLM: Tool results follow-up LLM
        Processing --> [*]: Send response to channel
    }

    note right of Active
        Messages stored in SQLite
        Context budget tracked
        Tools available per policy
        State layer accessible
    end note

    note right of Paused
        State preserved in DB
        No active processing
        Can resume seamlessly
    end note
```

---

## Monorepo Package Graph

All `@nachos/*` packages and their dependency relationships.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart BT
    subgraph Shared["Shared Packages"]
        Types["@nachos/types\nSchemas, interfaces,\nerror definitions"]
        Utils["@nachos/utils\nUtility functions"]
        ConfigPkg["@nachos/config\nTOML loading,\nvalidation, hot-reload"]
        CtxMgrPkg["@nachos/context-manager\nBudget, sliding window,\nextraction, summarization"]
        ToolBasePkg["@nachos/tool-base\nTool interface\nabstractions"]
    end

    subgraph Core["Core Packages"]
        BusPkg["@nachos/bus\nNATS client wrapper"]
        GatewayPkg["@nachos/gateway\nRouter, Sessions, Salsa,\nTools, Subagents, State"]
        LLMProxyPkg["@nachos/llm-proxy\nProvider adapters,\nretry, cooldowns"]
    end

    subgraph ChannelsPkg["Channel Packages"]
        ChBase["@nachos/channel-base\nPolicy, pairing, factory"]
        ChSlack["@nachos/channel-slack"]
        ChDiscord["@nachos/channel-discord"]
        ChTelegram["@nachos/channel-telegram"]
        ChWhatsApp["@nachos/channel-whatsapp"]
    end

    subgraph ToolsPkg["Tool Packages"]
        TBrowser["@nachos/tool-browser"]
        TFS["@nachos/tool-filesystem"]
        TCode["@nachos/tool-code-runner"]
        TFetch["@nachos/tool-web-fetch"]
        TCopilot["@nachos/tool-copilot"]
        TMCP["@nachos/tool-claude-code-mcp"]
    end

    subgraph CLIPkg["CLI"]
        NachosCLI["@nachos/cli\nCommander.js"]
    end

    Types --> Utils
    Types --> ConfigPkg
    Types --> CtxMgrPkg
    Types --> ToolBasePkg
    Types --> BusPkg

    BusPkg --> GatewayPkg
    ConfigPkg --> GatewayPkg
    CtxMgrPkg --> GatewayPkg
    ToolBasePkg --> GatewayPkg

    BusPkg --> LLMProxyPkg

    ChBase --> ChSlack & ChDiscord & ChTelegram & ChWhatsApp
    BusPkg --> ChBase

    ToolBasePkg --> TBrowser & TFS & TCode & TFetch & TCopilot & TMCP
    BusPkg --> TBrowser & TFS & TCode & TFetch & TCopilot & TMCP

    ConfigPkg --> NachosCLI
    Types --> NachosCLI
```

---

## CLI Command Tree

Full command hierarchy including subagent and memory management.

```mermaid
%%{init: {'theme': 'base'}}%%

flowchart TD
    nachos["nachos"]

    nachos --> init["init\nInitialize config"]
    nachos --> up["up [services...]\nStart stack"]
    nachos --> down["down\nStop stack"]
    nachos --> restart["restart [service]\nRestart service"]
    nachos --> logs["logs [service]\nAggregated logs"]
    nachos --> status["status\nService health"]

    nachos --> add["add"]
    add --> add_channel["channel name"]
    add --> add_tool["tool name"]

    nachos --> remove["remove"]
    remove --> rm_channel["channel name"]
    remove --> rm_tool["tool name"]

    nachos --> list["list [type]"]
    nachos --> search["search type"]

    nachos --> config["config"]
    config --> config_edit["edit"]
    config --> config_validate["validate"]
    config --> config_show["show"]

    nachos --> policy["policy"]
    policy --> policy_validate["validate"]

    nachos --> doctor["doctor\nDiagnostics"]
    nachos --> version["version"]

    nachos --> create["create"]
    create --> create_channel["channel name"]
    create --> create_tool["tool name"]

    nachos --> chat["chat\nInteractive"]

    nachos --> subagents["subagents"]
    subagents --> sa_list["list"]
    subagents --> sa_info["info runId"]
    subagents --> sa_stop["stop runId"]

    nachos --> memory["memory"]
    memory --> mem_list["list"]
    memory --> mem_query["query"]
```
