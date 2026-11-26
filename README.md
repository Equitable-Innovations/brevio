# Brevio (Token Optimized Software Harness)

> A CLI-based agentic software development harness. It uses structured local documentation that tools can parse, to increase speed and to decrease LLM token use.

## Concept
`brevio` is an AI development harness that puts token conservation and speed first. Token use is a large cost for all
organizations. A higher token count costs more money, and it also makes a run take more time. Fewer tokens give faster
runs and lower cost.

`brevio` is a CLI-based agentic software development harness. It does as much processing as possible locally to use
fewer tokens. This makes the development harness faster and decreases the token cost.


## Foundation
The token efficiency of this concept comes from project and development documentation that tools can parse. To get
this efficiency, the documentation must have a defined structure that gives the AI its context quickly. LLMs give good
results with `XML` tags.

All development pipeline documents use `XHTML` as their base format. `XHTML` is an effective polyglot format for this
use. A browser can render it as a rich UI. A tool can also parse and traverse it very quickly, and send only the
parts of the content that the task needs. Because the agent does not load full documents, token use decreases by a
large amount.

When `brevio` works together with a Code Graph tool, it can decrease token use more.


## Document Domain Breakdown

### [Work Item Documents](build-docs/docs/work-items/README.md)
Work item documents control the technical development phase, from story activation to the pull request. These
documents record the requirements, acceptance criteria, technical design, implementation plans, phases, review,
validation, summaries, and LLM conversations.

### [Wiki Documents (project)](build-docs/docs/wiki/README.md)
The wiki documents hold the general project information of the team. Examples are a project overview, project styles,
coding guidelines, a risk registry, design decisions, and a project setup guide.

### [OpenAPI Documents](build-docs/docs/open-api/README.md)
OpenAPI specifications are important to clients and LLMs. This document domain holds the specification of the service
app that the team builds (if it exposes endpoints). It also holds the specifications of the services that the
application calls as a client.



## Visualization
This framework supplies a base page that shows documents from all domains in a single pane to a developer.
When developers implement new features and fix bugs, they use the work item documents most. The wiki and OpenAPI
documents are optional context for the definition, design, and implementation of new code.

The harness creates and updates the work item documents in real time as it processes each stage.


## Components

### [CLI](build-docs/cli/README.md)
The CLI is the distributed application binary. The CLI initializes `brevio` and starts the other components that come
with the CLI. It can also do every action that `brevio` has. On every run, the CLI reads and writes project
documentation through a defined XHTML schema. Each document is then XML that a tool can parse. With the CLI, you
can query documents and get specific subsets of data from the documentation. The CLI also lets you manage checklists
in real time.

### [MCP](build-docs/mcp/README.md)
The MCP server comes bundled in the CLI. It lets the agentic LLM development tool use the CLI in a cleaner way, with
fewer messages.

### [UI](build-docs/static-ui/README.md)
The UI templates come bundled in the CLI, which uses them to build local files. In the initial phases, the UI will be
fully static, and the CLI will maintain it.


## Data Storage

### Local Data Storage
All the documentation that you need to implement application features and fix bugs is stored locally. Local storage
decreases cloud traffic and latency in a process where time is important.

### Repo Storage
When documentation is created or updated, it goes into source control. This lets teams synchronize documents, with a
history that you can trace and with effective merges. It also saves time for the team, because each team member does
not need to generate a local copy of the code.


## Partner Project
At the start, this harness will use `CodeGraph` to decrease token use for source code files.

Later, `brevio` can get its own knowledge graph for code, which will make it a more all-in-one solution.


## Project Management
`brevio` includes a basic [Project Manager](build-docs/project-mgr/README.md). It holds the product requirements, the
plan, and the release gates. It also holds the [Work Tracking System](build-docs/project-mgr/wts/README.md), which
tracks work items and their lifecycle. The Project Manager is intentionally small. It does not fully replace a
dedicated project management system.

`brevio` can also integrate with third-party project and work management systems. The CLI will let you add
capabilities through add-on modules. A module for Jira, Linear, ADO, or other systems is possible. A team that uses
Jira can download the Jira module and keep Jira as its system of record.
