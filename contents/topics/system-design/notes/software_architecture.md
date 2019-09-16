# Introduction

- Desirability: requirements
- Viability: time, resources, costs
- Aesthetics: simplicity, elegance
- Feasibility: technology

## Context

Software Architecture and Component-based Software Engineering: how to design large systems out of reusable components.

## Why Software architecture?

- manage complexity through abstraction
- communicate, remember and share global decisions
- visualize and represent relevant aspects of a software system
- understand, predict and control how the design impacts quality attributes of a system
- define a flexible foundation for maintenance and future evolution

# Quality attributes

## Different types

Functional:
- Correctness
- Compliance
- Completeness

Extra-Functional:
- Internal vs. External:
  - External qualities: fitness for purpose of the product, for satisfaction of stakeholder concerns; affected by deployment env.
  - Internal qualities: developer's perception of state and change during desing and development.
- Static vs. Dynamic:
  - Static qualities: structural properties that can be assessed before deployment in prod
  - Dynamic qualities describe behaviour during: normal op, failures, under attack, responding to change, long term

Meta qualities:
- Measurability
- Repeatability (Jitter)
- Predictability
- Auditability
- Testability

Many quality attributes categorized:

![quality_attributes](/LambdaFalcon/665f2f98f085f0c3aadc7870e02648d8/raw/quality_attributes.png)

## Design Quality Attributes

### Feasibility
What's the likelihood of success for your new project?
- Affordability: resources (money for hw/people, time, slack)
- Time to market: how soon can we start learning from our users?

### Modularity
Is there a structural decomposition of the architecture?
- Prerequisite for: Code Reuse, Separate Compilation, Incremental Build, Distributed
Deployment, Separation of Concerns, Dependency Management, Parallel Development in
Larger Teams

### Reusability
Can we use this software many times for different purposes?
- Mechanism: fork vs. reference
- Origin: internal vs. external
- Scope: General-purpose vs. Domain-specific
- Pre-requisites for reuse: trusted "quality" components, standardized and documented interfaces, marketplaces

### Design Consistency
What is the design's conceptual integrity and coherence?
- Understanding a part helps understanding the whole
- Avoid unexpected surprises: use naming conventions, follow architectural style constraints

### Simplicity
What is the complexity of a design?
- A simple solution for a complex problem
- One general solution vs. many specific: less duplication, minimal variability
- Conciseness
- Resist changes that compromise simplicity
- Refactor to simplify
- Complexity arises from:
  - number of components
  - number of connections between components

### Clarity
Is the design easy to understand?
- Distill most essential aspects into simple primitive elements that can be combined to solve the important problems of the system
- Freedom from ambiguity and irrelevant details
- Definitive, precise, explicit and undisputed decisions
- Opposite of clarity: clutter, confusion, obscurity

### Stability
How likely to change is your design?

| Unstable | Stable |
| --- | --- |
| prototype | product |
| implementation | interface |
| likely to break clients | platform to build upon |
| experimental spike, throw-away code | worthy of further investment: building, testing, documenting |

### Composability
How easy is it to assemble the architecture from its constituent parts?
- Assuming all components are ready, putting them together is fast, cheap and easy
- Cost of composition < cost of components
- Components can be easily recomposed in different ways

### Deployability
How difficult is it to deploy the system in production?

| Hard | Easy |
| --- | --- |
| manual release | automated release |
| scheduled updates | continuous updates |
| unplanned downtime | planned or no downtime |
| wait for dependencies | no synchronization |
| changes cannot be undone | rollback possible |

## Normal Operation Qualities

### Performance
How timely are the external interactions of the system?
- Latency
- Throughput

### Scalability
Is the performance guaranteed with an increasing workload?
- Design architecture for growth: requests, users, data, nodes, components

### Capacity
How much can the system perform?
- Ensure spare capacity (no saturation)

### Usability
Is the user interface intuitive and convenient to use?
- Learnability, Memorability, Efficiency, Satisfaction, Accessibility, Internationalization

### Ease of Support
Can users be effectively helped in case of problems?

| Hard | Easy |
| --- | --- |
| cryptic error messages | self-correcting errors |
| heisen-bugs | reproducible bugs |
| unknown config | remotely visible config |
| no error logs | stack traces and debug logs |
| user in the loop | remote screen |

### Serviceability
How convenient is the ordinary maintenance of the system?

| Hard | Easy |
| --- | --- |
| complete operational stop | service running system |
| reboot to upgrade | transparent upgrade |
| install wizard | unattended installation script |
| restart to apply config change | hot/live config |
| manual bug reports | automatic crash report |

### Visibility
Is it possible to monitor runtime events and interactions?
- Extent of possible observation of sys behaviour and internal state during op
- Are there logs to debug, detect errors or audit the sys in prod?
- Is the sys self-aware?
- Process Visibility: can the progress of the project be measured and tracked?

## Attack Qualities

### Availability
How likely is it that the system is functioning correctly?

### Reliability
How long can the system keep running?

### Recoverability
How long does it take to repair the system?

### Safety
Is damage prevented during use within the operating range?

### Security
Is damage prevented during intentional/hostile use outside the operating range?

### Defensibility
Is the system protected from attacks?

### Survivability
Does the system survive the mission?

### Privacy
How to keep personal information secret?


## Change Qualities
What changes are expected in the future?  
No change: put it in hardware.  
Software is expected to change.

### Flexibility
- Configurability: Can architectural decisions be delayed until after deployment?
- Customizability: Can the architecture be specialized to address the needs of individual customers?
- Modifyability: old (can func be removed from sys?)
- Extensibility: new (can func be added to sys?)
- Elasticity: Can workload changes be absorbed by dynamically re-allocating resources?
- Resilience: temporary changes. Can the architecture return to the original design after the change is reverted?
- Adaptability: permanent changes. Can the architecture evolve to adapt to the changed requirements?

### Compatibility
Does X work together with Y?
- Interfaces
- Interoperability: Protocols and Data formats. Can two systems exchange information to successfully interact?
- Portability: Platforms. Can the software run on multiple execution platforms without modifycation?
- Source vs. Binary
- Versioning
- Ease of Integration: How expensive is it to integrate our system with others?

## Long Term Qualities

### Durability
How permanent is the data?

### Maintainability
How to deal with software entropy?

### Sustainability
- Technical: How to avoid your software becomes obsolete in the long term?
- Economic: How to ensure your software development organization does not go bankrupt in the long term?
- Growth: How to bootstrap the growth of your startup?



# Definitions

## Architecture tribes
- Enterprise Architect
- Business Architect
- Software Architect
- Data Architect
- Information Architect
- Solution Architect
- Product Architect
- Security Architect

## Waterfall model
... where is the architecture?

Challenge requirements, avoid making assumptions.  
Someone will have to live with your architecture: make sure it is good.

Architectural decisions are made early in the dev lifecycle, but affect the whole lifecycle of the system.

A system's architecture may be visualized and represented using models that are somehow related to the code.

## Architecture vs. Design
All architecture is design.  
Not all design is architecture.

Architecture:
- What? Functional, Extra-functional, Requirements
- How? Design
- Why? Rationale

A software system’s architecture is the set of principal design decisions made about the system. It is the blueprint for a software system’s construction and evolution.

## Types of Architecture
- Prescriptive: specifies intent (TO BE)
- Descriptive: specifies realization (AS IS)
- Presumptive: default architecture for a given application domain (e.g. 3-Tier for web apps), variations need to be justified
- Reference: predefined architecture for a given application domain with explicit variation points, choices need to be justified
- Solution: architecture to solve problems of one customer, with many products
- Product: architecture of one product aimed at satisfying requirements of multiple customers
- M(arketing): architecture that describes how to market and sell the software system to customers (business model, licensing model, terms of use)
- T(echnology): architecture that prescribes how to build deploy and configure the system (styles, patterns, components, connectors)

### Green/Brown field development
- Green: building from scratch, only Prescriptive architecture exists, no Descriptive
- Brown: building by reuse, Descriptive architecture of components already exists

### Drift and Erosion
- Drift: new Descriptive decisions do not violate Prescriptive
- Erosion: Descriptive and Prescriptive are inconsistent

Causes of drift: ad-hoc changes, quick fixes.

# Modeling
One month of coding can save you one hour of architecting.

## Definitions
An architectural **model** is an artifact that captures some or all of the design decisions that comprise a system’s architecture.  
Architectural **modeling** is the reiѹcation and documentation of those design decisions.

### Abstraction and interpretation
- Abstraction (real sys -> model): some information is intentionally left out
- Interpretation (model -> real sys): solve ambiguities, add missing decisions

### Solving problems with models
Problem -> Abstraction -> Solve model -> Interpretation  
vs.
Solving directly (often harder).

### Views
Various parts of the architecture (or views) may have to be modeled with a different:
- Notation
- Level of detail
- Target audience

A view is a set of design decisions related by common concerns (the viewpoint).

There are a lot of different views. Usually more than one is used at a time because there is too much information to model.  
Views are not always orthogonal and should not become inconsistent.

### Canonical models
- Domain model: irrefutable truths about the real world, outside control, the system will be tested against it
- Code model: complete set of design commitments, represents the executable, ready-to-run source code
- Design model:
  - Boundary: visible interfaces of the system architecture...
  - Internal: ...refined with details about the implementation

## Notations

### Domain Model

Use Case Scenarios

Feature Models

### Design Model

C5: Context, Containers, Components, Connectors, Classes

4+1: Logical, Process, Development, Physical (and Use Cases)







