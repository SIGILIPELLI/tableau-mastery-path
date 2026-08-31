# 08 · Career Growth: Analyst to BI Leader

This module maps the skill and responsibility progression from individual
Tableau author to BI leadership, tying each stage back to concrete
capabilities built across this course.

## 1. The progression stages

1. **Analyst**: builds individual worksheets/dashboards (Level 1–2 skills
   — connecting to data, calculated fields, basic dashboards). Success
   metric: can they build a correct, hand-verifiable dashboard like the
   Level 1 exercises unassisted?
2. **Senior Analyst / Developer**: handles advanced calculations (LOD,
   table calcs, Level 2 Module 1 and Level 3 Module 1), performance
   tuning (Level 3 Module 5), and RLS (Level 3 Module 3) — builds
   dashboards other people depend on, not just their own analysis.
3. **BI Lead / Architect**: owns governance (Level 3 Module 8, Level 4
   Module 3), platform decisions (Level 4 Module 1), and mentors
   Analysts/Developers — responsible for *other people's* dashboards being
   correct, not just their own.
4. **BI Leader / Head of BI**: owns the CoE (Level 4 Module 2), cost/
   license strategy (Level 4 Module 8), and cross-functional data
   strategy (Level 4 Module 5/6) — success is measured by organizational
   metrics (certification %, cost efficiency, self-service adoption), not
   individual dashboard output.

## 2. What actually changes at each transition

1. **Analyst → Senior/Developer**: the shift is from "can I make this
   chart" to "can I make this chart *correct and fast* under load" — the
   hand-verification habit built through Level 1–3 (always re-deriving
   totals like 6880/2210/3750/920 by hand) becomes second nature rather
   than a taught exercise.
2. **Senior/Developer → BI Lead**: the shift is from personally building
   correct dashboards to designing the *process* that makes correctness
   the default for everyone else — e.g. defining what "certified" means
   (Level 3 Module 8) rather than personally certifying every source.
3. **BI Lead → BI Leader**: the shift is from technical ownership to
   organizational ownership — budget conversations (Level 4 Module 8's
   cost-unit reasoning), platform architecture trade-offs (Level 4
   Module 1's HA/DR and single- vs. multi-site decisions) framed for
   non-technical executives, and staffing the CoE (Level 4 Module 2).

## 3. A skills checklist by stage

| Stage | Must demonstrate |
|---|---|
| Analyst | Level 1–2: dimensions/measures, calculated fields, basic dashboards, correct hand-verified numbers |
| Senior/Developer | Level 3: LOD nesting, RLS, Prep flows, performance diagnosis |
| BI Lead | Level 3 Module 8, Level 4 Modules 1–3: certification, architecture, content governance |
| BI Leader | Level 4 Modules 2, 5, 6, 8: CoE strategy, self-service enablement, cost governance |

## 4. Common growth-stalling mistakes

1. Staying purely technical past the Senior/Developer stage — a Developer
   who can write any LOD expression but never learns to explain *why* a
   number is trustworthy to a non-technical stakeholder struggles to
   become a BI Lead, since that role is largely about building and
   defending trust in data, not just building charts.
2. Skipping the hand-verification discipline once "experienced enough to
   not need it" — the Level 3 Module 8 governance failure (three
   conflicting totals) is exactly the failure mode a senior person who's
   stopped double-checking their own outputs can reintroduce at scale,
   now with more downstream dashboards depending on their work.

## 5. Building an individual growth plan

1. Identify the current stage honestly using Section 3's checklist against
   real recent work — e.g. "I can write nested LODs (Level 3 Module 1)
   but I've never led a certification review (Level 3 Module 8)" pinpoints
   the Senior→Lead gap precisely.
2. Seek the specific gap-closing experience, not just more of the current
   stage's work — a Developer aiming for BI Lead benefits more from
   running one real certification review end-to-end (Level 3 Module 8's
   process, on real conflicting sources) than from writing ten more LOD
   expressions.

## Exercise

An analyst has strong Level 1–2 skills and has recently learned LOD
nesting and RLS (Level 3 Modules 1 and 3) but has never been involved in
data source certification or governance decisions. Using Section 5's
approach, name the specific stage they're at, the specific gap to their
next stage, and one concrete experience that would close it. (They're at
Senior Analyst/Developer, approaching but not yet at BI Lead; the gap is
governance/certification experience; a concrete closing experience is
leading a real certification review — reproducing Level 3 Module 8's
process of re-deriving a source's true totals and deciding certify/reject
— rather than only continuing to build technical dashboards.)
