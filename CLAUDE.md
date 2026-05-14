Global User Memory — James Buckett

Location: \~/.claude/CLAUDE.md Scope: Loaded for every Claude Code session on this machine, across all projects. Purpose: Cross-cutting preferences, role context, and conventions that apply to everything.



\## Role \& Context

* Executive Director, Product Management — Infrastructure Platforms, JP Morgan Chase \& Co.
* Work intersects enterprise security architecture, cloud infrastructure, and platform engineering.
* Strong orientation toward Zero Trust Architecture (NIST SP 800-207) and financial-services regulatory frameworks (PCI-DSS 4.0, SOX, DORA, SWIFT CSCF, MAS TRM, APRA CPS 234).
* Visual learner — prefer diagrams, tables, and structured artefacts over long prose where possible.



\## Response Format Preferences

* Always start with a TL;DR summary line.
* Use paragraphs with bullet points under bold titles, not flat bullet walls.
* For comparisons, prefer tables.
* For architecture, prefer Mermaid diagrams or inline SVG.
* Cite NIST, CISA ZTMM 2.0, FAPI 2.0, SPIFFE/SPIRE specs where they ground a claim.



\## Coding \& Architecture Defaults

* YAML over JSON for Kubernetes manifests; HCL for Terraform.
* Prefer declarative over imperative: Helm/Kustomize over raw kubectl apply, GitOps over ad-hoc.
* Container architectures: assume multi-arch (linux/amd64 + linux/arm64) by default — I run mixed Intel/Snapdragon hardware. Always check binary architecture before suggesting wget of a release artefact.
* Static binaries preferred when the option exists (e.g. opa\_envoy\_linux\_arm64\_static) — saves musl/glibc grief.
* Python: always use a virtualenv (python3 -m venv .venv). Never sudo pip install. Never fight Debian's RECORD-less system packages — go via venv or pipx.
* Node: use nvm or NodeSource, not Ubuntu's apt Node. Global npm installs go to a user-owned prefix (\~/.npm-global), never via sudo.



\## Security Defaults

* Treat every example as if it will end up in a regulated environment. 
* Default to:

  * mTLS everywhere (cert-manager + SPIRE-issued SVIDs, not long-lived certs).
  * Workload identity over IP allowlists (SPIFFE IDs, not 0.0.0.0/0).
  * Least-privilege RBAC, no cluster-admin in examples.
  * OIDC + short-lived tokens, no static API keys in code samples.
  * Image signing (cosign / Sigstore) when discussing supply chain.
* Never paste secrets into examples — use ${VAR} placeholders or reference an external secret manager (Vault, AWS Secrets Manager, External Secrets Operator).
* For financial services examples, assume data residency matters — call out APAC regional constraints (Singapore MAS, Australia APRA, Indonesia OJK PDP, India RBI) when relevant.



\## Architectural Frameworks (default to these)

* When describing, designing, or reviewing systems architecture, default to C4 + CALM as a complementary pair: 

  * C4 for the human-facing visual model 
  * CALM for the machine-readable source of truth. 
* Don't reach for UML, ArchiMate, or freeform Visio-style diagrams unless there's a specific reason.



\### C4 Model (visual structure)

* Use all four levels intentionally — System Context (L1), Container (L2), Component (L3), Code (L4). 
* Skip L4 unless it materially aids understanding; never skip L1.
* Diagrams as code — prefer Structurizr DSL when the same model is reused across L1–L3, C4-PlantUML for standalone diagrams, and Mermaid C4 for inline README/glossary use.
* Always label trust boundaries as enclosing rectangles with the regulatory regime named (e.g. "PCI-DSS scope", "MAS TRM applicable", "PDPA in-scope").
* Always label PDP and PEP components on policy/auth flows.
* Show identity provenance on every east-west call — SPIFFE ID, OIDC sub, or mTLS cert subject.



\### CALM — Common Architecture Language Model (machine-readable source of truth)

* CALM is the FINOS open-source specification developed under the Architecture as Code (AasC) community for defining, validating, and visualising system architectures in a standardized, machine-readable format. 
* It was open-sourced by Morgan Stanley through FINOS in August 2025 (v1.0), which makes it a particularly strong fit for Tier 1 bank reference architectures — it's already battle-tested in the same regulatory context.
* Treat CALM JSON as the source of truth; treat C4 diagrams as a rendered view of it. 
* When the two disagree, the CALM document wins.
* Use CALM for: reusable architectural patterns, automated compliance/security validation in CI, version-controlled architecture decisions, and embedding control requirements (FAPI 2.0, PCI-DSS 4.0, DORA, SWIFT CSCF) directly into the model.
* Toolchain: the CALM CLI (@finos/calm-cli) for validate, generate, and template workflows; CALM Hub UI for browsing patterns; the FINOS architecture-as-code monorepo for the spec and reference patterns.
* Particularly relevant for the AI-augmented controls work: the FINOS AI Governance Framework's 15 critical risks/controls map well onto CALM patterns, which is the most realistic path to operationalising AI governance in a regulated environment rather than leaving it as a slide deck.



\### How C4 and CALM compose

* Design phase — sketch in C4 (any level) to align humans on intent.
* Specification phase — encode the agreed design as a CALM document; this becomes the contract.
* Validation phase — run CALM CLI against the document to enforce schema, controls, and pattern conformance in CI.
* Communication phase — render C4 diagrams from the CALM document (or keep them in sync manually with a clear "generated from architecture.calm.json" note).
* ADRs — every architecture decision gets an ADR under docs/adr/, and where it changes the model it must reference the CALM document version it modifies.



\### Tooling Preferences

* Editor / IDE: VS Code with Claude Code extension primary; JetBrains for Java work.
* Shell: bash on Linux/WSL; PowerShell only on Windows when unavoidable.
* K8s local dev: Docker Desktop with Kubernetes enabled, plus kind for multi-node tests.
* MCP: Filesystem (built-in), GitHub MCP (with PAT, since DCR is broken on enterprise), Context7 for upstream-doc verification.
* Diagramming: Mermaid for inline / GitHub-native; D2 or draw.io when static export needed. C4 model for architecture diagrams, FINOS CALM for machine-readable architecture-as-code (Structurizr DSL or C4-PlantUML for rendered C4 views).



\### Communication Style

* Push back when something is wrong — don't soften technical disagreement.
* Skip excessive preambles ("Great question!", "Of course!") — get to the answer.
* Flag when an answer depends on assumptions; don't paper over uncertainty.
* When something is a known-broken upstream issue (e.g. Claude Code DCR bug, Falco on Docker Desktop kmod driver), say so explicitly with the workaround.



\### Out of Scope / Things to Avoid

* Don't suggest unsigned third-party MCP servers, marketplaces, or plugins for repos containing regulatory or financial-services content. 
* Stick to official Anthropic marketplace + Context7.
* Don't use emoji in technical output unless explicitly requested.
* Don't reproduce copyrighted spec text verbatim — paraphrase and cite.

