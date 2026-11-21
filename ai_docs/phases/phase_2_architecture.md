**Goal**
Develop a comprehensible software architecture with AI support.

**TODOs**

* Investigate at least two architecture variants
* Use AI for architecture proposals
* Select and justify technologies
* Create an architecture diagram (e.g., C4)
* Define the data model
* Create interface/API design
* Document at least two Architecture Decision Records (ADRs)
* Critically question the AI-generated architecture
* Compare hosting options (costs, etc.)

**Deliverables**

* `docs/architecture.md`
* `docs/hosting.md`
* `docs/adr/`
* Architecture diagram

---

### Task Description (Details)

**Goal:** Develop a comprehensible, technically sound, and implementable software architecture with AI support. Architectural decisions should not only be made, but also justified and critically questioned.

#### 1. Key Points

* **Architecture variants:** Investigate and compare at least two different architectural approaches.
* **Technology selection:** Select and justify programming languages, frameworks, databases, and other components.
* **Architecture diagram:** Visualize the system structure and essential components (e.g., C4 model).
* **Data model:** Define central entities, relationships, and data structures.
* **API design:** Specify essential interfaces, endpoints, and data formats.
* **Architecture Decision Records:** Document at least two important architectural decisions.
* **Hosting:** Investigate options for hosting. Compare costs, etc.
* **AI support:** Use AI to develop, evaluate, and critically review the architecture.

#### 2. What to Pay Special Attention To?

* **Requirements First:** The architecture must support the requirements from Phase 1.
* **Keep it Simple:** No unnecessary complexity. A modular monolith makes more sense than microservices for many semester projects.
* **Trade-offs:** Every architectural decision comes with pros and cons. These must be understood.
* **Technology selection:** Do not select technologies based on their popularity, but based on concrete requirements.
* **Consistency:** Architecture diagram, data model, and API design must fit together.
* **Extensibility:** Consider sensible extensions without unnecessarily developing for hypothetical requirements.
* **AI quality:** Review AI-generated architecture proposals for technical correctness, complexity, and feasibility.

#### 3. Framework Conditions

**Architecture**

* Compare at least two architecture variants.
* Select one variant and justify the decision.
* Create at least one architecture diagram.
* Describe essential system components and their responsibilities.
* Create a basic data model and API design.
* Document at least two Architecture Decision Records (ADRs).

**Use of AI**

* Use AI to develop alternative architecture proposals.
* Use AI to identify weaknesses and potential risks.
* Critically question and evaluate at least one AI-generated recommendation.
* Document essential AI contributions and the resulting decisions.

**Quality Requirements**

* Architecture must fit the MVP and the available development time.
* No unnecessary technologies or infrastructure components.
* Consider important non-functional requirements (Security, Performance, Maintainability).
* The architecture must be explainable and defensible by the students.

#### 4. Deliverables

* `docs/architecture.md` – Architecture overview, comparison of variants, and technology decisions.
* `docs/hosting.md` – Hosting options.
* `docs/adr/` – At least two Architecture Decision Records.
* **Architecture diagram** – Visualization of the essential system components and their relationships.
* **Data model** – Central entities and relationships.
* **API design** – Essential interfaces and data formats.

#### 5. Avoid Typical Mistakes

* Overengineering through unnecessary microservices, message queues, or complex infrastructure.
* Adopting AI-generated architecture without verification.
* Using technologies that no one on the team understands sufficiently.
* Making architectural decisions without reference to the requirements.
* Creating diagrams that do not match the actual implementation.
* Comparing alternatives only superficially to fulfill formal requirements.

#### 6. Evaluation Focus Areas

* **Architecture quality:** Is the architecture sensible, understandable, and implementable?
* **Decision quality:** Are technology and architectural decisions comprehensibly justified?
* **Alternatives:** Were different approaches meaningfully compared?
* **AI competence:** Were AI proposals critically evaluated?
* **Consistency:** Do requirements, architecture, data model, and interfaces fit together?

#### 7. Definition of Done

Phase 2 is completed when:

* The architecture is understandably documented and visualized.
* Essential technology decisions have been justified.
* At least two architecture variants have been compared.
* At least two ADRs are available.
* The data model and API design are concrete enough to begin implementation.
* The students can comprehensibly explain their architectural decisions.

**Key Takeaway:**
AI can propose architectures, but engineers must understand and justify their decisions.

A good architecture is not the most complex one, but the simplest architecture that reliably fulfills the requirements.
