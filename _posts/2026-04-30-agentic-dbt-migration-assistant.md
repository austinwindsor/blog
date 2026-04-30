---
layout: post
title: "Building an Agentic dbt Migration Assistant for Post-Merger Analytics Reconciliation"
author: austin
categories: [ data, analytics-engineering, dbt, ai, portfolio ]
image: assets/images/data_science.jpg
featured: false
---

{:.image-caption}
*Image courtesy of medium.com*

### Why I Built This Project

One of the most interesting problems in analytics engineering is not building a greenfield data model from scratch. It is inheriting two systems that were never designed to work together and then creating a reliable, documented, analytics-ready layer on top of both.

That problem shows up constantly during mergers and acquisitions. A company acquires another business, then suddenly needs to answer simple questions like:

- What is revenue across both businesses?
- Which tables describe the same business entity?
- Are these metrics actually comparable?
- What assumptions are hidden in the old reporting layer?

Those questions are much harder than they look. The source systems may use different schemas, naming conventions, metric definitions, reporting tables, views, and stored procedures. Documentation is often partial, inconsistent, or missing entirely.

That is the motivation behind this portfolio project:

**an agentic dbt migration assistant for post-merger analytics reconciliation.**

The goal was to build a prototype that can take two messy legacy-style relational systems, convert each into an interpretable dbt project, enrich that project with business semantics from documentation, compare the projects against each other, and propose a reconciled canonical layer.

### The Business Problem

I framed the project around a common post-merger analytics integration scenario.

Company A represents a more normalized Brazil-based e-commerce schema inspired by the Olist dataset. Company B represents a messier India-based marketplace reporting environment built from sales reports, P&L extracts, expense files, and warehouse comparisons.

Each source has overlapping business concepts:

- orders or sales lines
- products
- revenue-like measures
- reporting views
- operational reporting tables

But they do not define them in the same way.

For example:

1. In Brazil, **recognized revenue** is based on delivered order items.
2. In India, **reported sales** is based on valid non-cancelled sales lines.
3. Brazil has a clear **GMV** concept.
4. India does not expose an equally clear GMV metric.
5. Brazil has a usable customer identity.
6. India does not provide an equivalent customer identity in the imported reporting layer.

That makes the real challenge semantic, not just structural.

### What I Wanted the Project to Demonstrate

I wanted this project to show more than “I can call an LLM from Python.”

The skills I wanted to surface were:

1. **Analytics engineering**
   building source, staging, and legacy view layers in dbt-compatible form
2. **Metadata and lineage thinking**
   extracting tables, fields, views, and procedures into structured artifacts
3. **Documentation-first modeling**
   using business documentation to enrich raw schema outputs
4. **Semantic reconciliation**
   comparing similar concepts across systems without pretending they are automatically equivalent
5. **Appropriate AI orchestration**
   using deterministic code where possible and agentic reasoning only where ambiguity actually exists

### The Architecture

The workflow ended up becoming a hybrid system:

1. deterministic code parses MySQL dumps
2. deterministic code generates dbt starter projects
3. LLM-assisted extraction turns prose documentation into structured semantic evidence
4. deterministic enrichment applies that evidence to source, staging, and legacy view YAML
5. deterministic comparison exports normalized project semantics
6. deterministic reconciliation generates mapping candidates, semantic conflicts, and a reconciliation plan
7. a LangGraph workflow orchestrates the whole process and introduces human review where needed

That separation was important. Early on, I had to make a decision about what should be rule-based and what should be agentic.

My conclusion was:

- **tables, schemas, views, dbt generation, validation, and file writing** should be deterministic
- **documentation interpretation, ambiguous stored procedures, and semantic reconciliation decisions** are the right places for agentic workflows

That made the project much more credible than trying to force every step into a general-purpose agent loop.

### What the System Produces

For each source system, the workflow generates artifacts such as:

- `schema_inventory.csv`
- `views.json`
- `procedures.json`
- `procedure_lineage.yml`
- `semantic_evidence.yml/json`
- `project_semantics.yml/json`
- a dbt-style project with enriched `sources.yml`, staging model YAML, and legacy view YAML

Then, across the two projects, it generates:

- `mapping_candidates.yml/json`
- `semantic_conflicts.yml/json`
- `reconciliation_plan.yml/json`

Those final reconciliation artifacts are where the post-merger analytics story becomes clear.

### Why dbt Was the Right Target

I chose dbt because it is a strong intermediate representation for analytics migration work.

dbt gives you:

1. a structured project layout
2. YAML-based documentation and metadata
3. model lineage
4. tests
5. a natural place to express semantic transformations

Even if a legacy system does not currently use dbt, translating it into a dbt-like structure makes the logic more inspectable and easier to compare against another system.

That makes dbt not just a transformation framework here, but a migration and interpretation layer.

### What Worked Well

Several parts of the project ended up feeling especially strong.

#### 1. Converting raw SQL dumps into interpretable artifacts

The first pass focused on transforming:

- tables
- fields
- views
- stored procedures

into structured metadata and dbt-friendly outputs.

Views were a particularly good fit for deterministic processing because they are bounded, declarative SQL statements. Stored procedures were treated more conservatively: preserve them, preserve referenced objects, and postpone deeper interpretation when necessary.

That distinction ended up being one of the better design decisions in the project.

#### 2. Enriching the dbt project with business semantics

The documentation layer added a lot of value. Instead of leaving the dbt project as a purely structural scaffold, the workflow enriched:

- `sources.yml`
- `_staging__models.yml`
- `_legacy_views__models.yml`

with business definitions, grains, caveats, metric logic, join keys, and field-level meanings.

That made the outputs feel much more like analytics engineering deliverables and much less like code generation artifacts.

#### 3. Separating mapping candidates from conflicts

This was important philosophically and practically.

Many migration projects fail because teams jump too quickly from “these look related” to “these are the same thing.”

By separating:

- candidate mappings
- semantic conflicts
- canonical reconciliation plans

the workflow can show overlap without overstating certainty.

That is much closer to how real migration work should be done.

### Pros of This Approach

There are several advantages to this style of system.

#### Pro 1: It is realistic about ambiguity

The workflow does not assume that all similar-looking fields or metrics are comparable. It explicitly captures gaps, caveats, and human review points.

#### Pro 2: It produces useful intermediate artifacts

Even before the final reconciliation plan, the generated dbt projects, semantic evidence, and lineage files are already valuable for understanding a legacy system.

#### Pro 3: It uses AI where AI is genuinely helpful

The agentic layer is focused on:

- documentation interpretation
- semantic ambiguity
- policy questions
- human-readable reasoning

That is a much better fit than using an LLM to do everything indiscriminately.

#### Pro 4: It is portfolio-friendly and enterprise-relevant

This project demonstrates analytics engineering, platform design, metadata thinking, documentation systems, and AI orchestration in one coherent story.

### Cons and Limitations

I also think it is important to be honest about where this kind of system is limited.

#### Con 1: Documentation quality still matters

If the source documentation is incomplete or misleading, the semantic enrichment step becomes weaker. The workflow can help structure ambiguity, but it cannot conjure true business meaning out of nothing.

#### Con 2: Stored procedure interpretation is still hard

Views are tractable. Stored procedures are much more variable. They may contain control flow, temp tables, side effects, and dialect-specific logic that do not translate cleanly into dbt or deterministic lineage.

That is why I treat them conservatively and reserve deeper interpretation for explicit LLM-assisted review.

#### Con 3: LLM extraction introduces operational dependencies

Once documentation extraction becomes LLM-backed, the workflow depends on model quality, quotas, and prompt robustness. That is manageable, but it means productionizing the system requires fallback behavior and human oversight.

#### Con 4: Canonical metrics are often policy choices, not just technical mappings

A system can identify that two metrics are similar, different, or partially comparable. But deciding whether they should be rolled into a shared enterprise KPI is often a governance decision, not merely a data engineering task.

### Why I Used LangGraph

This project ended up being a very natural fit for LangGraph.

The key reason is that it is not a single “chatbot.” It is a stateful, multi-step workflow with:

- deterministic nodes
- LLM-assisted nodes
- long-running execution
- human review checkpoints
- resumable state

That is the sort of use case where a graph-based orchestration model makes more sense than a single free-form agent loop.

In other words, this project is best understood as a **hybrid agentic workflow**:

- workflow backbone for reliability
- agentic reasoning where ambiguity exists

### How I Would Improve It Next

If I continue developing this project, the next improvements I would prioritize are:

1. stronger structured-output enforcement for documentation extraction
2. more robust fallback behavior when LLM extraction fails
3. persistent LangGraph checkpointing for terminal resume flows
4. better field-level semantic extraction and cleaner generated documentation
5. richer stored procedure interpretation with explicit confidence scoring
6. a small UI or report layer to make the reconciliation outputs easier to review

### Final Thoughts

What I liked most about this project is that it mirrors a real analytics engineering challenge: not just moving data, but making inherited systems understandable, comparable, and governable.

A lot of AI projects are framed as replacing human judgment. I think a more realistic and useful framing is different:

**use deterministic systems to structure what is knowable, and use agentic reasoning to surface and explain what is ambiguous.**

That is the idea behind this project.

The final result is not a magic migration engine. It is a migration assistant that helps teams:

- understand legacy data systems
- translate them into a dbt-oriented analytics layer
- document business semantics
- compare two independently evolved systems
- make better reconciliation decisions with clearer evidence

For analytics engineering, data infrastructure, semantic layer, governance, and AI-ready data platform roles, that felt like exactly the kind of problem worth building around.
