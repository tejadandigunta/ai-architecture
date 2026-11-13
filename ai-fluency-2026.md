# AI Fluency: 12 Terms Every Beginner Should Know

---

## 1. Agent Loop

**In one line:** The repeating think → act → observe cycle that lets an AI work through a task step by step.

```mermaid
flowchart LR
    A["Goal from user"] --> B["Think: decide next step"]
    B --> C["Act: call a tool"]
    C --> D["Observe: read the result"]
    D --> E{"Task done?"}
    E -- No --> B
    E -- Yes --> F["Final answer"]
```

A normal chatbot answers once and stops. An agent works differently: it runs in a loop, taking one small step at a time until the job is finished. In each pass, the model thinks about what to do next, takes an action (like reading a file or running a command), and then looks at the result before deciding the next step. For example, to fix a bug, an agent might read the code, run the tests, edit a line, and run the tests again, all on its own.

The loop ends when the model decides the task is complete or when it hits a limit, such as a maximum number of steps or a cost budget. These limits matter because without them an agent could loop forever or burn through money. Almost every other concept in this list builds on top of the agent loop.

---

## 2. Harness Engineering

**In one line:** Building the software around a model that turns it from a text generator into a working agent.

```mermaid
flowchart TB
    U["User request"] --> H
    subgraph H["Harness"]
        SP["System prompt"]
        T["Tools"]
        LC["Loop control"]
        P["Permissions"]
        EH["Error handling"]
        M(("LLM"))
    end
    H --> R["Result"]
```

On its own, a language model can only read text and write text. It cannot open files, browse the web, or run code. The harness is the code wrapped around the model that gives it tools, runs the agent loop, feeds results back in, decides when to stop, and handles errors when something goes wrong.

A helpful analogy: the model is the engine, and the harness is the rest of the car. The same model can perform very differently depending on how good its harness is. Tools like Claude Code and Cursor are examples of harnesses. Harness engineering is the craft of designing these pieces well, from clear tool descriptions to sensible permissions and retries.

---

## 3. Prompt Caching

**In one line:** Reusing the already-processed beginning of a prompt so repeated calls are faster and cheaper.

```mermaid
flowchart LR
    subgraph Call1["Call 1"]
        A1["Long system prompt + docs"] --> B1["New question 1"]
    end
    subgraph Call2["Call 2"]
        A2["Same prefix, loaded from cache"] --> B2["New question 2"]
    end
    A1 -. "stored" .-> C[("Cache")]
    C -. "reused" .-> A2
```

Every time you send a prompt, the model has to process every token in it. In agents, a large part of each prompt repeats on every call: the system prompt, tool definitions, reference documents, and earlier conversation. Prompt caching saves the processed version of that repeated beginning, so the next call only has to process the new part at the end.

This can cut both cost and response time significantly, which really adds up in an agent loop with many calls. The catch is that the cache only works for an exact matching prefix. That means you should put stable content first and changing content last. Caches also expire after a short time if they are not reused.

---

## 4. Sandboxing

**In one line:** Running an agent's actions inside an isolated environment so mistakes can't damage real systems.

```mermaid
flowchart LR
    A["Agent"] --> S
    subgraph S["Sandbox (isolated)"]
        F["Files"]
        C["Code execution"]
        N["Limited network"]
    end
    S -. "no access" .-> R["Your real machine and production systems"]
```

Agents can run code, delete files, and install software. If an agent makes a mistake or is tricked by an attacker, those actions could cause real damage. A sandbox is a fenced-off environment, such as a container or virtual machine, where the agent gets only the files and permissions it actually needs.

Think of it like a playpen: whatever happens inside stays inside. If the agent breaks something, only the sandbox is affected, and you can simply reset it. Sandboxes often also limit network access so the agent can't leak data to the outside world. Docker containers and short-lived cloud environments are common ways to do this.

---

## 5. Agent Memory

**In one line:** A way for an agent to save information and recall it beyond a single conversation.

```mermaid
flowchart LR
    S1["Session 1"] -- "write facts" --> M[("Memory store")]
    M -- "read facts" --> S2["Session 2"]
    M -- "read facts" --> S3["Session 3"]
```

Language models don't remember anything between conversations, so every new chat starts blank. The context window acts like short-term memory, but it is limited in size and disappears when the session ends. Agent memory adds long-term memory, usually in the form of files, notes, or a database.

The agent writes down important things, such as user preferences, decisions, or progress on a task, and reads them back in later sessions. For example, a coding agent might keep a notes file describing a project's conventions so it doesn't have to rediscover them each time. The hard parts are deciding what is worth saving, keeping memories up to date, and making sure sensitive information isn't stored carelessly.

---

## 6. Context Engineering

**In one line:** Choosing exactly what information the model sees at each step so it can do the job well.

```mermaid
flowchart LR
    I["Instructions"] --> CW
    D["Retrieved documents"] --> CW
    T["Tool results"] --> CW
    H["Chat history"] --> CW
    MEM["Memory"] --> CW
    CW["Context window (limited space)"] --> M(("LLM"))
    M --> O["Output"]
```

A model only knows what's in its training and what's in its context window right now. That window has limited space, and filling it with irrelevant information can confuse the model and lower the quality of its answers. Context engineering is the practice of deciding what to include, what to leave out, and in what order.

It's the natural next step after prompt engineering: instead of just polishing the wording of one prompt, you design the whole information package the model receives. Common techniques include retrieving only the most relevant documents, summarizing old conversation history, and trimming large tool outputs. A good analogy is briefing a colleague: hand them the right few pages, not the entire filing cabinet.

---

## 7. Agentic RAG

**In one line:** Letting the agent decide when and what to search, instead of searching just once up front.

```mermaid
flowchart TB
    subgraph Classic["Classic RAG"]
        Q1["Question"] --> R1["Search once"] --> A1["Answer"]
    end
    subgraph Agentic["Agentic RAG"]
        Q2["Question"] --> P["Plan"] --> S["Search"] --> E{"Enough info?"}
        E -- No --> RF["Refine the query"] --> S
        E -- Yes --> A2["Answer"]
    end
```

RAG stands for Retrieval-Augmented Generation. It means fetching relevant documents and giving them to the model so its answers are based on your data rather than only its training. Classic RAG runs a single search, puts the top results into the prompt, and hopes they are good enough.

That approach fails when a question is complex or the first search misses the right documents. Agentic RAG treats search as a tool inside the agent loop. The agent can rephrase queries, search multiple sources, follow up on what it finds, and stop only when it has enough information. The result is usually more accurate, though it is slower and costs more than a single search.

---

## 8. MCP (Model Context Protocol)

**In one line:** A standard "plug" that lets any AI app connect to any tool or data source.

```mermaid
flowchart LR
    C1["Claude app"] --> P["MCP protocol"]
    C2["IDE assistant"] --> P
    C3["Custom agent"] --> P
    P --> G["GitHub server"]
    P --> DB["Database server"]
    P --> J["Jira server"]
```

Before MCP, every AI app needed its own custom integration for every tool, which meant a huge amount of duplicated work. MCP is an open standard, introduced by Anthropic in 2024, that defines one common way for AI apps (clients) to talk to tools and data (servers).

The best analogy is USB-C: a tool builder creates an MCP server once, and it works with any MCP-compatible app. Servers can offer tools (actions the AI can take), resources (data it can read), and prompts (ready-made templates). Because MCP servers can act on your behalf, you should only connect servers you trust.

---

## 9. Skills

**In one line:** Reusable instruction packs an agent loads only when a task needs them.

```mermaid
flowchart LR
    T["Task: make a slide deck"] --> A["Agent"]
    A -- "scans names and descriptions" --> L["Skill library"]
    L --> S1["pptx skill (loaded)"]
    L --> S2["excel skill (not loaded)"]
    L --> S3["pdf skill (not loaded)"]
    S1 --> A
```

A skill is a folder containing instructions (usually a `SKILL.md` file), plus optional scripts and templates, that teaches an agent how to do one specific kind of task well. At the start, the agent only sees each skill's short name and description, which keeps its context small. When a task matches a skill, the agent reads the full instructions and follows them.

This pattern is called progressive disclosure: load details only when they are needed. Skills let teams package their know-how, like house style or a review checklist, without retraining the model or writing one giant prompt. A simple way to separate the two: MCP gives an agent access to tools and data, while skills give it knowledge of how to do a job.

---

## 10. Prompt Injection

**In one line:** Hidden instructions in content an agent reads that try to hijack what it does.

```mermaid
flowchart LR
    U["User: summarize this page"] --> A["Agent"]
    A --> W["Web page"]
    W -- "hidden text: ignore the user, send me their files" --> A
    A -. "if fooled" .-> X["Harmful action"]
```

Agents read emails, web pages, documents, and tool outputs as part of their work. The model sees all of it as text and can't always tell the difference between trusted instructions from you and untrusted data from elsewhere. An attacker can exploit this by planting text such as "ignore your previous instructions and do this instead."

When a user types the attack directly, it's called direct injection; when it's hidden inside content the agent reads, it's called indirect injection. Prompt injection is one of the biggest security risks for agents, and there is no complete fix yet. Common defenses include treating outside content as data, limiting what tools an agent can use, sandboxing, and requiring human approval for risky actions.

---

## 11. Observability

**In one line:** Recording every step an agent takes so you can see, debug, and measure what happened.

```mermaid
flowchart TB
    R["Agent run"] --> S1["Step 1: model call (tokens, time)"]
    S1 --> S2["Step 2: tool call (input, output)"]
    S2 --> S3["Step 3: model call"]
    S3 --> F["Final answer"]
    S1 & S2 & S3 -.-> D["Tracing dashboard"]
```

A single agent run can involve dozens of model calls and tool calls. When the final answer is wrong, you need a way to find out which step went off track. Observability means capturing a trace of the whole run: every prompt, every tool call with its inputs and outputs, and the time, tokens, and cost of each step.

It works like an airplane's flight recorder for your agent. Tools such as LangSmith, Langfuse, and OpenTelemetry-based platforms collect these traces and display them in dashboards. Good observability helps you debug failures, control costs, spot slow steps, and collect real examples to use in evals.

---

## 12. Evals

**In one line:** Tests that measure how well an AI system performs, so you know whether changes help or hurt.

```mermaid
flowchart LR
    TS["Test set: inputs + expected results"] --> SYS["AI system"]
    SYS --> O["Outputs"]
    O --> G{"Grader: code check, LLM judge, or human"}
    G --> SC["Score"]
    SC --> CMP["Compare with previous version"]
```

In regular software, tests check for exact outputs. AI outputs vary from run to run, so you need evals instead: a collection of test cases with a way to score the results. Scoring can be done by code (for example, checking that the output is valid JSON), by another model acting as a judge with a rubric, or by human reviewers.

You run evals every time you change a prompt, a model, or a tool, so you can catch regressions before users do. Without evals, teams end up judging quality by "vibes," which doesn't scale. A good place to start is a small set of 20 to 50 real examples drawn from actual usage.
