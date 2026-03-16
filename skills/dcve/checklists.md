# DCVE Phase Prompts & Checklists

Self-prompting questions for each phase, and checklists to determine phase completion.

---

## D — Devise

### Prompts

**Problem Definition**
- What real-world problem does this service/platform solve?
- Who experiences this problem, and in what context?
- What pain or loss occurs if this problem remains unsolved?

**Feature Ideation**
- What features could address this problem?
- If similar services already exist, what are they missing?
- If there were no constraints, what would the ideal solution look like?
- How do completely different domains solve similar problems?
- What would users say "it would be great if it could also do this" about?

**Perspective Shift**
- From the user's perspective, what is the most desired feature?
- From a technical perspective, what interesting possibilities exist?
- From a technical perspective, what aspects of this problem could be challenging to solve?
- From a business perspective, what elements could differentiate this product?

### Checklist

- [ ] Maximized quantity of ideas without evaluating them
- [ ] Included unconventional or seemingly unrealistic ideas
- [ ] Devised from all three perspectives: user, technical, and business
- [ ] Recorded every idea without omission
- [ ] Reached a state where no new ideas are emerging

---

## C — Curate

### Prompts

**Core Selection**
- Which ideas directly address the essence of the problem?
- Which features are essential — without them, the service cannot function?
- Which features are nice-to-have but not strictly necessary?

**Consolidation**
- Are there overlapping features that can be merged?
- Are there dependencies between features? (Feature A must exist for Feature B to work)

**Prioritization**
- Which features have the highest impact relative to technical difficulty?
- Given time and resource constraints, what should be built first?
- Where would the MVP (Minimum Viable Product) boundary be drawn?

### Checklist

- [ ] Clearly distinguished core features from supplementary ones
- [ ] Selected features consistently contribute to solving the original problem
- [ ] Prioritization criteria are explicit (impact, difficulty, dependencies, etc.)
- [ ] Dependencies between features have been identified
- [ ] MVP scope has been defined
- [ ] Discarded ideas have been recorded for potential future use

---

## V1 — Design & Feasibility Verification

### Prompts

**Architecture**
- What overall architecture realizes the selected specs?
- What are the major components/modules, and how do they interact?
- What technology stack is appropriate? What is the rationale for selection?

**Feasibility**
- Is this design actually implementable?
- Where are the technical bottlenecks?
- What are the risk factors in terms of performance, scalability, and security?
- Are the estimated resources (people, time, cost) realistic?
- Which areas have uncertain implementation? Are any PoCs needed?

**Alternatives**
- Are there alternative approaches to this design?
- What are the trade-offs? (performance vs development speed, flexibility vs simplicity, etc.)

### Checklist

- [ ] Overall system architecture diagram has been created
- [ ] Technology stack has been selected with documented rationale
- [ ] Technically uncertain areas have been identified
- [ ] Performance, scalability, and security risks have been assessed
- [ ] Resource estimation has been performed
- [ ] Ready to fall back to the C phase if infeasible specs are discovered

---

## V2 — Team Assignment & Detailed Design Verification

### Prompts

**Assignment**
- How will the overall system be divided into domains?
- Who owns each domain?
- Are domain boundaries clear? Are there any ambiguous areas?

**Detailed Design**
- Has each owner completed the detailed design for their domain?
- Has each owner performed a feasibility review of their domain?
- Have owners identified any additional risks or constraints?

**Task Derivation**
- Has each owner derived their task list?
- Have task priorities and execution order been determined?
- Has the Definition of Done been defined for each task?

### Checklist

- [ ] Every domain has an assigned owner
- [ ] Every owner has produced a detailed design document
- [ ] Every owner has performed a feasibility review of their domain
- [ ] Task lists and priorities have been derived
- [ ] Inter-task dependencies have been documented
- [ ] Definition of Done has been defined for each task
- [ ] Documentation is complete

---

## V3 — Cross-Team Consistency Verification

### Prompts

**Consistency Review**
- Have you reviewed the design documents of adjacent domain owners?
- Do the interfaces (APIs, data formats, protocols, etc.) match across domains?
- Are there any differing interpretations of the same spec?
- Are there points where data flow breaks or transforms at domain boundaries?

**Consensus & Escalation**
- If conflicts were found, has consensus been reached between the relevant owners?
- What items could not be resolved? What are the pros and cons of each option?
- Have items requiring escalation been documented with context and options?

**Change Incorporation**
- Have agreed or decided changes been reflected in each owner's documents?
- Has the change history (before → after) been recorded?
- Have the relevant owners re-reviewed the final revisions?
- Were any new conflicts found after re-review?

### Checklist

- [ ] Mutual design document reviews between all adjacent domains are complete
- [ ] Interface consistency (APIs, data formats, etc.) has been confirmed
- [ ] A list of conflicts has been compiled
- [ ] All conflicts have been resolved through consensus or escalation
- [ ] Agreed changes have been reflected in each owner's documents
- [ ] Change history (before → after) has been recorded in documents
- [ ] Items requiring escalation have been documented with context and options
- [ ] Affected owners have re-reviewed documents after decisions
- [ ] No additional conflicts found after re-review

---

## E — Execute

### Prompts

**Pre-Execution Final Check**
- Have all V-phase verifications been completed?
- Are there any unresolved conflicts or pending decisions?
- Has each owner's final task list and execution order been confirmed?
- Has execution approval been granted?

**In-Execution Monitoring**
- Is the work progressing according to plan?
- Have any unexpected technical issues arisen?
- Has a situation arisen that requires spec changes?

### Checklist

- [ ] Every owner's final design document is in confirmed state
- [ ] No unresolved conflicts or pending decisions remain
- [ ] Execution approval has been confirmed
- [ ] Re-entry criteria for DCVE cycle during execution have been pre-defined

### E → D Re-Entry Checklist

- [ ] The nature of the change has been identified (new feature / spec modification / technical constraint)
- [ ] The scope of impact has been identified (which domains, which owners)
- [ ] The re-entry starting point has been determined (from D / from V)
- [ ] Affected owners have been notified of the re-entry
