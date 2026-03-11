# Feature Tracker

> The single view of all features in progress. Shows where each feature is in its lifecycle and what's been completed within each phase.
> Update this file whenever a feature changes phase or completes a milestone.

---

## Dashboard

Features are grouped by product. See `context/[product]/[feature]/context.md` for any feature.

### Energy

| Feature | Status | Owner | Started | Blockers |
|---------|--------|-------|---------|----------|
| [LL97 Compliance](energy/ll97-compliance/context.md) | Defining | — | — | — |
| [Budget](energy/budget/context.md) | Defining | — | — | — |
| [Weather Normalization](energy/weather-normalization/context.md) | Defining | — | 2026-03-11 | — |
| [Meters & Hierarchy](energy/meters/context.md) | Defining | — | — | — |
| [Utility Data](energy/utility-data/context.md) | Defining | — | — | — |

### BMS

| Feature | Status | Owner | Started | Blockers |
|---------|--------|-------|---------|----------|
| | | | | |

### FDD

| Feature | Status | Owner | Started | Blockers |
|---------|--------|-------|---------|----------|
| | | | | |

### DataHub

| Feature | Status | Owner | Started | Blockers |
|---------|--------|-------|---------|----------|
| | | | | |

---

## Progress by feature

### Energy

#### LL97 Compliance → Defining

Context: [energy/ll97-compliance/context.md](energy/ll97-compliance/context.md)

**1. Definition**
- [x] Context file created
- [x] Problem and users documented
- [x] Domain terms defined inline
- [x] Domain constraints captured
- [x] Scope and non-goals set
- [ ] Open questions have owners and due dates
- [ ] All dependencies identified with status

**2. Design**
- [ ] Happy path and edge cases documented
- [ ] Key screens / states identified
- [ ] Figma linked
- [ ] Architecture touchpoints mapped
- [ ] Cross-team review completed (if needed)
- [ ] ADR created (if irreversible decision)

**3. Implementation**
- [ ] Feature broken into deliverable increments
- [ ] Backend implemented
- [ ] Frontend implemented
- [ ] API docs updated
- [ ] Decision log current

**4. Testing & validation**
- [ ] Unit and integration tests passing
- [ ] Product-specific validations complete
- [ ] Edge cases covered
- [ ] Downstream impact verified

**5. Rollout**
- [ ] Staging validated
- [ ] Production deployed
- [ ] Monitoring period clean
- [ ] User confirmation received
- [ ] Context file set to `Shipped`

---

#### Budget → Defining

Context: [energy/budget/context.md](energy/budget/context.md)

**1. Definition**
- [x] Context file created
- [ ] Problem and users documented
- [ ] Domain terms defined inline
- [ ] Domain constraints captured
- [ ] Scope and non-goals set
- [ ] Open questions have owners and due dates
- [ ] All dependencies identified with status

**2–5.** *(begin after Definition is complete)*

---

#### Weather Normalization → Defining

Context: [energy/weather-normalization/context.md](energy/weather-normalization/context.md)

**1. Definition**
- [x] Context file created
- [x] Problem and users documented
- [x] Domain terms defined inline
- [x] Domain constraints captured
- [x] Scope and non-goals set
- [x] Open questions have owners and due dates
- [x] All dependencies identified with status

**2. Design**
- [x] Happy path and edge cases documented
- [x] Key screens / states identified
- [ ] Figma linked
- [x] Architecture touchpoints mapped
- [x] PRD created ([prd.md](energy/weather-normalization/prd.md))
- [x] Design flows created ([flow.md](energy/weather-normalization/flow.md))
- [ ] Cross-team review completed (if needed)
- [ ] ADR created (if irreversible decision)

**3–5.** *(begin after Design is complete)*

---

#### Meters & Hierarchy → Defining

Context: [energy/meters/context.md](energy/meters/context.md)

**1. Definition**
- [x] Context file created
- [ ] Problem and users documented
- [ ] Domain terms defined inline
- [ ] Domain constraints captured
- [ ] Scope and non-goals set
- [ ] Open questions have owners and due dates
- [ ] All dependencies identified with status

**2–5.** *(begin after Definition is complete)*

---

#### Utility Data → Defining

Context: [energy/utility-data/context.md](energy/utility-data/context.md)

**1. Definition**
- [x] Context file created
- [ ] Problem and users documented
- [ ] Domain terms defined inline
- [ ] Domain constraints captured
- [ ] Scope and non-goals set
- [ ] Open questions have owners and due dates
- [ ] All dependencies identified with status

**2–5.** *(begin after Definition is complete)*

---

## How to use this file

**Starting a new feature:**
1. Create a folder: `context/[product]/[feature-name]/`
2. Copy `_templates/context.md` into it as `context.md`
3. Add a row to the product's dashboard table above
4. Copy the progress template below
5. Check off items as they are completed

**Picking up a feature:**
1. Find it in the Dashboard under its product
2. Scroll to its progress section — first unchecked item is where to continue
3. Read the context file for full detail

**Adding more documents to a feature:**
When a feature needs a separate PRD, design spec, flow, or ADR — add it to the feature's folder:

```
context/energy/ll97-compliance/
├── context.md          ← always exists
├── prd.md              ← if requirements need formal tracking
├── design-spec.md      ← if complex screen specs
├── flow.md             ← if complex multi-step flow
└── adr-001-*.md        ← if irreversible decision
```

---

## Progress template

Copy this when adding a new feature.

```markdown
#### [Feature Name] → Defining

Context: [[product]/[feature]/context.md]([product]/[feature]/context.md)

**1. Definition**
- [ ] Context file created
- [ ] Problem and users documented
- [ ] Domain terms defined inline
- [ ] Domain constraints captured
- [ ] Scope and non-goals set
- [ ] Open questions have owners and due dates
- [ ] All dependencies identified with status

**2. Design**
- [ ] Happy path and edge cases documented
- [ ] Key screens / states identified
- [ ] Figma linked
- [ ] Architecture touchpoints mapped
- [ ] Cross-team review completed (if needed)
- [ ] ADR created (if irreversible decision)

**3. Implementation**
- [ ] Feature broken into deliverable increments
- [ ] Backend implemented
- [ ] Frontend implemented
- [ ] API docs updated
- [ ] Decision log current

**4. Testing & validation**
- [ ] Unit and integration tests passing
- [ ] Product-specific validations complete
- [ ] Edge cases covered
- [ ] Downstream impact verified

**5. Rollout**
- [ ] Staging validated
- [ ] Production deployed
- [ ] Monitoring period clean
- [ ] User confirmation received
- [ ] Context file set to `Shipped`
```
