$Skill Creator 

I want to create a modular, codebase-aware agent workflow that produces high-quality **<< Technical Jira Epics and Jira Tasks >>** through a phased, skill-based process.

The system will be composed of specialized skills (agents), each responsible for a single phase of the workflow and producing one structured document as output. These documents are designed to be session-independent, so they can be pasted into a new conversation and reliably understood by the system.

At the core, we will have an Orchestrator skill responsible for:

- Starting the workflow from a raw **<< technical initiative >>**
- Parsing previously generated documents
- Determining the current phase and what is missing
- Deciding which skill to run next
- Requesting only the minimum required input when information is incomplete

The workflow will:
- ** << define the wanted workflow >> ** ex:
- Navigate and reason about the codebase
- Gather evidence and constraints from real system structure
- Frame bounded technical decisions safely
- Generate a Lean Technical Epic focused on outcomes, scope, and success signals
- Apply a Quality Gate review before execution
- Derive Jira-ready tasks/stories aligned with the epic goals
- The end result is a repeatable, auditable, and resumable process that turns vague technical initiatives into clear epics and executable Jira work, while preserving engineering judgment, traceability, and long-term maintainability.

Use the superpowers SKILL.md files as references:
- superpowers/skills/brainstorming
- superpowers/skills/using-superpowers
- superpowers/skills/writing-skills
- superpowers/skills/writing-plans
- superpowers/.codex/superpowers-bootstrap.md