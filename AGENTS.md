# AGENTS.md

## Project purpose

EnterpriseOps AI is a production-oriented enterprise operations assistant that investigates customer operations using grounded retrieval, controlled business tools, and auditable workflows. The system supports enterprise knowledge retrieval, structured investigation, safe operational actions, and evaluation-driven improvement.

The system must be:
- reliable and production-ready
- security-conscious by default
- understandable to a senior engineer
- aligned with enterprise operating constraints
- evaluable for behavior, safety, cost, latency, and correctness

This repository is not a general-purpose autonomous agent playground. It is a controlled operations platform for enterprise investigation and safe business execution.

## Architecture principles

1. Clean architecture and clear boundaries.
2. Strong typing and explicit contracts.
3. Dependency inversion where it improves testability and safety.
4. Testability before convenience.
5. Security by default.
6. No secrets in source code.
7. No unnecessary framework complexity.
8. Prefer deterministic workflows over autonomous agent behavior when deterministic systems are sufficient.
9. All write operations require explicit authorization.
10. The LLM must never have unrestricted SQL or database access.
11. Every important agent behavior must be evaluable.
12. The application must run locally using Docker.
13. Code must stay readable and maintainable by another senior engineer.

## Technology stack

- Python
- FastAPI
- Pydantic
- PostgreSQL
- pgvector
- LangGraph
- MCP
- Langfuse
- Docker
- pytest
- GitHub Actions

Additional technology choices may be added only when necessary, and only with a clear justification in the change description or PR notes.

## Coding conventions

- Prefer explicit, typed, readable code over cleverness.
- Keep modules small and responsibility-focused.
- Favor dependency injection for infrastructure and external systems.
- Write small functions with clear contracts and narrow responsibilities.
- Avoid broad abstractions that hide important behavior.
- Do not add framework complexity without a demonstrated need.
- Use domain models and service boundaries; do not let API code, orchestration code, and persistence code become entangled.
- Keep business policy logic separate from infrastructure concerns.
- Validate input at boundaries using Pydantic and clear error handling.
- Avoid silent failures and hidden retries.
- Use structured logging and correlation IDs for operational visibility.
- Preserve deterministic behavior whenever possible.
- Do not implement fake functionality or placeholder logic as if it were complete.
- Any new dependency must be justified in writing: why it is required, what problem it solves, and why the repository should depend on it.

## Testing requirements

Every feature, fix, and meaningful behavior change must include appropriate tests.

Required testing practices:
- Unit tests for domain logic, validation, policy decisions, and pure transforms.
- Integration tests for API endpoints, persistence flows, retrieval workflows, and tool adapters.
- Contract tests for external interfaces, especially MCP and tool boundaries.
- End-to-end tests for critical business flows, including safe write workflows and approval enforcement.
- Security tests for prompt injection attempts, privilege escalation, permission misuse, and unsafe tool behavior.
- Evaluation tests for retrieval quality, groundedness, tool selection, approval correctness, and operational safety.
- Performance and reliability tests for latency-sensitive or costly paths.

Testing rules:
- Do not write tests that only validate mocks or mock-only behavior.
- Prefer real behavior over mocks when feasible.
- Run the smallest relevant test scope first, then broader validation when necessary.
- Do not merge code that fails required tests or reduces coverage below project thresholds.
- New behavior without tests is not complete.

## Security requirements

Security is a requirement, not a feature.

- Never commit secrets, tokens, API keys, private keys, or credentials to source control.
- Use environment variables, secret managers, or local configuration files that are excluded from git.
- Treat all external input as untrusted.
- Validate and sanitize prompt inputs, tool arguments, and user-provided content.
- Prevent prompt injection and instruction hijacking through retrieval isolation, context control, and tool-output handling rules.
- Enforce least privilege for every user, tenant, role, and tool capability.
- Require explicit authorization for every write operation and sensitive read.
- The LLM must never receive unrestricted SQL, raw database access, or unrestricted persistence capabilities.
- All data access must pass through controlled service-layer abstractions and policy checks.
- Require audit logs for sensitive operations, approvals, and high-impact actions.
- Prefer allowlists for tools, actions, and data sources.
- Use defensive defaults for timeouts, retries, validation, and exception handling.

## Database rules

- PostgreSQL is the source of truth for transactional and operational data.
- pgvector is used only for retrieval and search use cases, not as a substitute for policy enforcement.
- The LLM must never have unrestricted SQL access.
- Database access must be mediated by repositories, service boundaries, and authorization checks.
- All write paths must be explicit, auditable, and permission-checked.
- Use migration-based schema evolution rather than ad hoc schema changes.
- Keep schemas explicit, versioned, and reviewable.
- Enforce tenant and account boundaries in queries and repository logic.
- Do not bypass business rules by writing directly to tables from orchestration or API layers.
- Do not add DB access patterns that fail the principle of least privilege.
- Database rules are core security constraints; violations are not acceptable.

## Agent design rules

- Prefer deterministic workflows over autonomous behavior when deterministic logic is sufficient.
- Treat the LLM as a reasoning component, not as an unrestricted operator.
- Use graph-based or workflow-based orchestration for multi-step business flows.
- Separate investigation, reasoning, approval, execution, and verification into explicit stages.
- Every mutating action must pass through an approval gate and policy enforcement.
- Keep tool calls narrow, typed, and observable.
- Require trace IDs and audit references for action flows.
- Do not allow the model to choose arbitrary database commands or dangerous tool invocations without guardrails.
- When agent behavior is important, design it so it is measurable and evaluable.
- Prefer a small set of high-confidence, controlled actions over broad autonomy.
- Degrade gracefully when confidence or authorization is insufficient.

## MCP rules

- MCP is a standard integration mechanism, not a bypass for security.
- Every MCP server or tool must be treated as a policy-controlled integration.
- Tool schemas must be explicit and validated.
- Limit tool capabilities to the minimum needed for the task.
- Do not add broad or unscoped tool access to satisfy convenience.
- Verify tool outputs before acting on them.
- Never expose unrestricted file-system or database access through MCP without explicit approval and policy enforcement.
- Tool invocations must be traceable to the user request, workflow step, and approval context.
- Any new MCP integration must document its purpose, security model, and operational boundaries.

## Evaluation rules

- Every important agent behavior must eventually be evaluable.
- Maintain reproducible evaluation datasets for retrieval quality, groundedness, tool-use correctness, policy compliance, and safety.
- Build evaluation into the development process rather than treating it as an afterthought.
- Track metrics relevant to business correctness, safety, latency, and cost.
- Prompt and tool changes must be evaluated against regression datasets.
- Failure modes must be identified, measured, and improved systematically.
- Production-critical workflows should have explicit acceptance criteria and evaluation coverage.
- Do not ship behavior that cannot be explained or measured.

## Observability rules

- Instrument critical flows with trace IDs, structured logs, timing, and status metadata.
- Use Langfuse or equivalent tracing for research, debugging, and operational visibility.
- Record costs, latency, token usage, tool call timing, and retrieval health where relevant.
- Monitor workflow success, failure, authorization denials, retrieval quality, and tool execution quality.
- Keep logs structured and privacy-aware.
- Correlate user actions, approval events, tool actions, and downstream system outcomes.
- Do not hide failures behind generic errors; preserve enough context for diagnosis.
- Observability is part of correctness, not an optional add-on.

## Dependency management rules

- Dependencies must be justified, minimal, and necessary.
- Do not add a dependency without explaining why it is needed and what problem it solves.
- Prefer the smallest dependency that satisfies the requirement.
- Avoid dependency sprawl, hidden frameworks, or “just in case” packages.
- Keep dependency boundaries clear and monitor transitive risk.
- Keep versions reviewable and aligned with the project’s support needs.
- Security-sensitive dependencies must be reviewed and updated as needed.
- New packages are not allowed as “nice to haves” without explicit need.

## Git and pull request rules

- Keep changes focused and limited to the task at hand.
- Do not modify unrelated files.
- Commit in a way that reflects the actual scope of the change.
- Ensure code and tests are passing before opening or approving a PR.
- Include a clear summary of what changed, why it changed, and how it was validated.
- PRs must describe any security, data, permissions, or workflow impact.
- If a change introduces a new dependency, explain why in the PR description.
- Do not merge or approve work with unreviewed security-sensitive changes.
- Preserve a clear history of architectural decisions and significant workflow changes.
- Keep PRs small enough to review responsibly.

## Repository operating expectations

- Follow the architecture and principles in this document.
- Keep code understandable and maintainable.
- Favor correctness, safety, and evaluability over hype or unnecessary automation.
- When in doubt, choose the narrow, explicit, auditable path.
- Do not implement TODO placeholders as fake functionality.
- Do not claim completion without verification from relevant tests or checks.

## Final instruction for AI agents

Before making code changes, read the relevant files and keep the scope narrow. Prefer deterministic, reviewable, and permissioned designs. Do not bypass safety constraints, security rules, or authorization requirements. Treat this repository as a production-grade systems project with strong operational and audit expectations.
