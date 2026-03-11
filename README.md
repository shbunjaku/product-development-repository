# Product Development Repository

This is the knowledge base for a building intelligence platform for commercial real estate. It contains everything needed to define, design, and build the product — domain knowledge, personas, architecture, design system, and feature context files that drive each feature from idea to shipping.

The platform connects buildings — their sensors, equipment, and systems — into a graph-based digital model, then reasons over that data to detect faults, track energy performance, and surface actionable insights to facilities teams.

---

## Repository structure

Everything lives inside `blueprint/`.

```
blueprint/
├── core/                   ← Shared definitions. Stable. All teams read from here.
│   ├── domain/             ← Glossary, domain concepts
│   ├── product/            ← Personas, roadmap
│   ├── design/             ← Design system (tokens, components, patterns)
│   ├── engineering/        ← Platform architecture
│   └── ai-context.md       ← Compressed platform summary for fast context loading
│
├── context/                ← Feature context. Start here for any feature work.
│   ├── README.md           ← Feature tracker dashboard
│   └── [product]/
│       └── [feature]/
│           └── context.md  ← Primary context file (+ prd.md, design-spec.md, adr-*.md as needed)
│
├── _templates/             ← Copy these when creating new documents
│
└── [product]/              ← Deep domain knowledge and detailed specs per product
    ├── domain/concepts/
    ├── product/prd/
    ├── design/flows|specs/
    └── engineering/adr/
```

**Two layers:**

1. **`core/`** — what things mean, who the users are, how the platform works. Rarely changes.
2. **`context/`** — features grouped by product. Each feature has its own folder with a `context.md` and any supporting docs. This is where active work happens.

Product folders (`bms/`, `fdd/`, `energy/`, `datahub/`) hold deeper reference material — domain concepts, PRDs, design flows, and architecture decisions.

---

## Products

| Product | What it does |
|---------|-------------|
| **DataHub** | Data ingestion — integrations, pipeline, data quality, catalog |
| **BMS** | Building Graph construction — point mapping, equipment classification, twin completeness |
| **FDD** | Fault detection — rules, findings, issues, configuration |
| **Energy** | Energy management — metering, EUI, carbon, compliance, optimization |

Dependency chain: `core → DataHub → BMS → FDD / Energy`

---

## Quick navigation

| You want to... | Go to |
|----------------|-------|
| Understand the platform fast | `blueprint/core/ai-context.md` |
| See all features and progress | `blueprint/context/README.md` |
| Work on a feature | `blueprint/context/[product]/[feature]/context.md` |
| Start a new feature | Copy `blueprint/_templates/context.md` into `blueprint/context/[product]/[feature]/` |
| Look up a domain term | `blueprint/core/domain/glossary.md` |
| Understand who the users are | `blueprint/core/product/personas.md` |
| Understand the architecture | `blueprint/core/engineering/architecture.md` |
| See current priorities | `blueprint/core/product/roadmap.md` |

---

## Conventions

- **Use the templates.** Every document type has a template in `_templates/`. Copy it, don't start from scratch.
- **Reference, don't redefine.** If a term is in the glossary, link to it. Don't define it again in a feature file.
- **Every file has a status.** Draft / Review / Approved / Shipped for documents. Defining / Designing / Building / Shipped for features.
- **Open questions need owners.** Every unresolved question gets a named owner and a due date.
- **Decision log is append-only.** Add dated entries. Never delete or overwrite.
- **`core/` is shared.** If it applies to all products, it goes in `core/`. If one product, it goes in `[product]/`. If one feature, it goes in `context/`.

---

## Team workflows

A feature moves through phases. Each team member owns specific parts of the process and specific documents. Everyone works from the same feature folder — `blueprint/context/[product]/[feature]/`.

### Feature lifecycle

```
Defining → Designing → Building → Shipped
```

| Phase | Who leads | What happens |
|-------|-----------|-------------|
| **Defining** | Product Manager | Problem, users, scope, domain context, PRD |
| **Designing** | Designer + Tech Lead | Design spec, flows, architecture, ADRs |
| **Building** | Engineering | Implementation, API docs, decision log |
| **Shipped** | QA + Product Manager | Validation, rollout, sign-off |

### Product Manager

Owns the "why" and "what". Drives the Defining phase.

**What you do:**
1. Kick off the feature — create the context file (say "new feature" to the AI assistant, or copy from `_templates/context.md`)
2. Fill in Problem, Users, Scope, and Non-goals in the context file
3. Define domain context — work with AI to pull relevant terms from the glossary and identify constraints
4. Write the PRD if the feature needs formal requirements — copy from `_templates/prd.md` into the feature folder
5. Write user stories and acceptance criteria in the PRD
6. Assign owners to all open questions
7. Update the feature tracker (`blueprint/context/README.md`) as milestones complete

**Your documents:**
| Document | Template | When |
|----------|----------|------|
| `context.md` | `_templates/context.md` | Always — you create this |
| `prd.md` | `_templates/prd.md` | When requirements need formal tracking with user stories and acceptance criteria |

**With AI:** Say "new feature" to start. The assistant walks you through each section with questions. It reads the glossary, personas, and product-specific domain concepts so you don't have to look things up manually. It writes answers directly into the context file as you go.

---

### Designer

Owns the "how it looks and feels". Drives the design portions of Designing.

**What you do:**
1. Read the context file — Problem, Users, Scope, and Edge cases sections are your brief
2. Define design principles and key screens in the context file's Design approach section
3. Create a design spec if the feature has complex screens — copy from `_templates/design-spec.md`
4. Create a flow doc if the feature has a complex multi-step journey — copy from `_templates/flow.md`
5. Link Figma files back into the context file and design spec
6. Document all screen states: empty, loading, error, populated
7. Flag open design questions with your name as owner

**Your documents:**
| Document | Template | When |
|----------|----------|------|
| `design-spec.md` | `_templates/design-spec.md` | Complex screens with multiple states and interactions |
| `flow.md` | `_templates/flow.md` | Multi-step user journeys with branching paths |

**With AI:** Share the context file (`@context.md`) and ask the assistant to help draft screen states, edge case behaviours, or flow steps. It reads the design system (`blueprint/core/design/design-system.md`) to stay consistent with existing patterns.

---

### Tech Lead / Feature Lead

Owns the "how it's built". Drives the technical portions of Designing and oversees Building.

**What you do:**
1. Read the context file and PRD — understand scope, constraints, and edge cases
2. Fill in the Technical approach section of the context file — architecture touchpoints, key decisions
3. Create an ADR if any decision is irreversible or affects other teams — copy from `_templates/adr.md`
4. Identify all dependencies and their status — flag upstream blockers early
5. Break the feature into deliverable increments for engineering
6. Review that domain constraints from the context file are respected in implementation
7. Keep the decision log in the context file updated as technical decisions are made

**Your documents:**
| Document | Template | When |
|----------|----------|------|
| Technical approach section in `context.md` | (part of context template) | Always |
| `adr-NNN-*.md` | `_templates/adr.md` | Irreversible decisions, cross-team impact, lasting architectural choices |

**With AI:** Share the context file and ask the assistant to help map architecture touchpoints, draft an ADR, or identify missing dependencies. It reads the platform architecture (`blueprint/core/engineering/architecture.md`) and the product dependency chain to catch gaps.

---

### Engineering

Builds the feature. Drives the Building phase.

**What you do:**
1. Read the context file, PRD, design spec, and any ADRs — this is your full brief
2. Implement backend and frontend according to the documented scope and acceptance criteria
3. Update API docs as endpoints are created or changed
4. Add entries to the decision log when implementation decisions are made
5. Flag anything that deviates from the spec — update the context file, don't just code around it

**With AI:** Share the context file and relevant docs when asking for implementation help. The assistant has full context on domain terms, architecture, and constraints — it will use the right vocabulary and respect platform patterns.

---

### QA

Owns validation. Drives the Testing & validation phase.

**What you do:**
1. Read the PRD's acceptance criteria — these are your primary test cases
2. Read the context file's edge cases table — these are your secondary test cases
3. Read the design spec's states table — verify all states render correctly (empty, loading, error, populated)
4. Validate that domain constraints listed in the context file are enforced
5. Verify downstream impact — check that the feature doesn't break anything that depends on the same data
6. Sign off on the progress checklist in `blueprint/context/README.md`

**With AI:** Share the PRD and context file and ask the assistant to generate test scenarios from the acceptance criteria and edge cases. It can cross-reference constraints and identify gaps.

---

### Everyone

Regardless of role, every team member should:

- **Update the feature tracker** (`blueprint/context/README.md`) when completing checklist items
- **Add to the decision log** when making significant decisions — dated, append-only
- **Check Knowledge contributions** before closing out — new terms go to the glossary, new patterns go to the design system, new architecture goes to `architecture.md`

---

## Defining features with AI

This repository is built for AI-assisted product development. An AI assistant in Cursor can walk you through defining a feature from scratch using a structured conversational workflow.

**To start, say something like:**
- "new feature"
- "define a feature"
- "start a feature for energy"

The assistant activates a built-in skill that guides you through 8 phases:

| Phase | What you cover together |
|-------|------------------------|
| 1. Basics | Product, feature name, one-sentence summary, owner |
| 2. Problem & users | What pain this solves, which personas are affected |
| 3. Domain context | Relevant terms from the glossary, platform constraints |
| 4. Scope | What you're building (2-5 bullets) and explicit non-goals |
| 5. How it works | Happy path walkthrough, edge cases |
| 6. Design & tech | UX principles, key screens, architecture touchpoints |
| 7. Dependencies & questions | What must exist first, unresolved decisions with owners |
| 8. Wrap up | Knowledge contributions, additional docs if needed |

At each phase, the assistant asks questions, you answer, and the responses are written into a structured context file. By the end, you have:

```
blueprint/context/[product]/[feature-name]/
└── context.md          ← fully filled context file
```

Plus a tracked entry in `blueprint/context/README.md` with a progress checklist.

The assistant stays grounded by reading the glossary, personas, architecture, and product-specific domain concepts before asking questions — so it uses real platform vocabulary, not invented terms.

If a feature already has a context file, the assistant picks up where you left off instead of starting over.

See `blueprint/context/energy/weather-normalization/` for a fully worked example.

---

## AI assistants

If you are an AI assistant working in this repository, read `AGENTS.md` at the root. It contains navigation rules, domain vocabulary, and conventions you must follow. For fast platform context, start with `blueprint/core/ai-context.md`.
