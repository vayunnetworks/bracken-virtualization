# Contributing to BRACKEN

Thanks for helping build BRACKEN. This guide applies across the BRACKEN repositories. Each repository may add a short component-specific section in its README as implementation details become established. If a repository instruction conflicts with this guide, follow the more specific instruction and raise the conflict for maintainers to resolve.

BRACKEN is an early-stage systems project. Treat repository READMEs as design and implementation notes, and distinguish clearly between shipped behavior, experimental work, and plans. Do not describe a proposed capability as implemented or supported unless the code and validation evidence justify that claim.

## 1. Before you start

1. Read the repository README and this guide.
2. Search existing issues and pull requests for related work.
3. For a substantial feature, behavior change, security-sensitive change, or cross-repository contract, open or join an issue before investing in implementation. Explain the user problem and intended outcome. Small typo fixes and clear documentation corrections can usually proceed directly.
4. Check which repository owns the change. Keep packet handling, management workflows, host networking, VM lifecycle, DNS, delivery, analytics, and build responsibilities in their respective repositories. Propose shared contracts only when multiple components need the same semantics.
5. If the design is uncertain, open a discussion or draft pull request early. Mark unresolved choices as open questions instead of silently making them permanent.

Before making changes, confirm that your checkout is the repository you intend to modify and that you are on a feature branch. Do not commit secrets, personal data, production traffic captures, private keys, tokens, customer configuration, or generated artifacts unless the repository explicitly requires them and maintainers have approved their handling.

## 2. Set up a local checkout

Install Git and the prerequisites documented by the repository. The project is still establishing build and test commands; do not assume a command works just because another component uses it. If setup instructions are missing, record the operating system, tool versions, exact error, and steps attempted in your issue or pull request. Do not invent a successful build status.

Clone the repository and create a dedicated branch:

```sh
git clone https://github.com/vayun-networks/REPOSITORY.git
cd REPOSITORY
git switch main
git pull --ff-only origin main
git switch -c docs/short-description
```

Replace `REPOSITORY` with the repository name and use a branch prefix appropriate to the work, such as `feat/`, `fix/`, `docs/`, `test/`, `build/`, or `security/`. Use lowercase words separated by hyphens. Keep the branch focused on one change. Do not develop directly on `main`.

If you already have a checkout, inspect it first:

```sh
git status --short --branch
git remote -v
git branch --show-current
```

If the working tree contains changes you do not recognize, stop and inspect them before switching branches or running cleanup commands. Never use a reset, clean, or force-push command to discard work unless you have confirmed exactly what it affects.

## 3. A focused contribution workflow

### Understand and plan

Write down the problem, intended behavior, owning component, affected users, relevant contracts, failure modes, and how you will validate the change. For a cross-repository change, list every affected repository and propose a compatible rollout order. Avoid changing multiple components in one pull request unless the change is an inseparable contract migration.

### Implement

Make the smallest coherent change that solves the problem. Prefer explicit behavior and simple interfaces. Validate input at trust boundaries, bound retries and resource use, and make failure visible. Keep implementation details private unless another component needs a stable contract. Add comments for non-obvious reasoning rather than narrating each line.

For code changes, add or update tests alongside the behavior. For documentation changes, verify commands, links, names, and status statements. For configuration changes, document defaults, units, allowed values, secret handling, migration behavior, and rollback. For performance work, include a reproducible baseline and measurement method.

### Review your local diff

Before committing, inspect the complete diff:

```sh
git status --short
git diff --check
git diff --stat
git diff
```

Confirm that the diff contains only the intended files, no secrets, no accidental generated output, and no unrelated formatting changes. Run the repository's documented formatter, build, tests, or validation commands if they exist and are relevant. If a command is unavailable or fails for an unrelated environment reason, report that accurately; do not claim it passed.

### Commit and publish

Use the commit format in the next section. Then update your branch from the current project base and push:

```sh
git fetch origin
git rebase origin/main
git push -u origin docs/short-description
```

If the rebase has conflicts, resolve them carefully, run relevant validation again, and inspect the final diff. Do not force-push a shared branch. Open a pull request against `main` using GitHub. Link the issue, explain the behavior change, summarize validation, and list any remaining risks or follow-up work. Use a draft pull request when the design or implementation is still under review.

## 4. Commit message format

Use this format:

```text
type(scope): imperative summary

Optional explanation of why the change is needed and important implementation details.

Refs: #123
```

Keep the subject concise, specific, and in the imperative mood (for example, “document recovery behavior”, not “documented recovery behavior”). Use lowercase type and scope. Omit the scope when it adds no clarity.

Common types:

- `feat`: add a user-visible capability.
- `fix`: correct a defect.
- `docs`: change documentation.
- `test`: add or adjust tests without changing product behavior.
- `refactor`: restructure code without changing intended behavior.
- `perf`: improve performance with evidence.
- `build`: change build, packaging, or dependency configuration.
- `ci`: change continuous integration.
- `chore`: maintenance that does not fit the other types.
- `revert`: revert an earlier change.

Examples:

```text
docs(os): explain image recovery workflow
feat(dns): validate zone updates before activation
fix(data-plane): reject rules that exceed map capacity
test(control-plane): cover retry after controller restart
build(build): pin image assembly dependency
```

For an incompatible public API, configuration, data format, or behavior change, call it out in the pull request and use a `BREAKING CHANGE:` footer when appropriate. Explain who must migrate, what order is safe, and whether old and new versions can run together. A commit title does not replace the migration note.

## 5. Engineering guidelines

### Architecture and interfaces

- Keep each repository within its documented responsibility.
- Make producers and consumers of a contract explicit.
- Define units, defaults, versioning, error behavior, timeouts, and retry safety.
- Distinguish desired state from observed state.
- Make asynchronous operations and partial completion visible.
- Avoid circular dependencies and hidden coupling through log parsing or private files.
- Record cross-repository decisions and migration steps.

### Reliability and safety

- Validate before applying changes with external effects.
- Keep retries bounded and safe; never retry a permanent validation error indefinitely.
- Preserve known-good state when an update fails, where the architecture permits it.
- Document rollback, recovery, and data-retention behavior.
- Report resource limits and degraded operation clearly.
- Do not claim availability, performance, or scale beyond measured evidence.

### Security and privacy

- Use least privilege and explicit authentication/authorization for administrative operations.
- Never commit credentials, tokens, private keys, customer data, or production traffic captures.
- Redact secrets from logs, metrics, traces, examples, and error messages.
- Review new dependencies for maintenance, license, security, and build impact.
- Test only on systems you own or are authorized to use. Network and attack-like simulations must run in isolated labs or against explicitly authorized targets.
- For suspected vulnerabilities, do not publish exploit details in a public issue. Use GitHub's private vulnerability reporting for the repository if enabled; otherwise ask a maintainer for a secure reporting route without posting sensitive details publicly.

### Tests and documentation

- Add tests for normal behavior, invalid input, boundaries, failures, retries, restart/recovery, and compatibility as applicable.
- Prefer deterministic tests with synthetic fixtures.
- Keep test claims precise: list what ran and what did not.
- Document setup, configuration, commands, limitations, and migration behavior.
- Mark roadmap items as planned. Do not turn a design idea into a feature claim.
- Keep examples safe and reproducible; do not use real secrets or real customer data.

## 6. Repository-specific validation focus

These are review prompts, not claims that the corresponding tooling already exists:

| Repository | Validation focus |
| --- | --- |
| `bracken-os` | Boot, install, service profiles, storage boundaries, upgrade interruption, and recovery in declared environments. |
| `bracken-build` | Pinned inputs, clean builds, artifact manifests, integrity checks, image validation, and failure-gated publication. |
| `bracken-core` | Schema compatibility, generated-code consistency, consumer tests, versioning, and deprecation behavior. |
| `bracken-data-plane` | Packet correctness, policy semantics, resource limits, update behavior, counters, and isolated performance measurement. |
| `bracken-control-plane` | Authentication, authorization, idempotency, durable operations, partial rollout, audit, and restart recovery. |
| `bracken-virtualization` | Host capability checks, VM resource admission, storage lifecycle, network isolation, and restart reconciliation. |
| `bracken-network-engine` | Interface/route validation, safe remote changes, partial apply, rollback, observed state, and isolated network tests. |
| `bracken-dns` | Recursion access, zone validation, cache behavior, DNSSEC lifecycle if implemented, health freshness, and safe query tests. |
| `bracken-cdn` | Cache-key correctness, private-content safety, origin failures, invalidation, TLS handling, and cache-capacity behavior. |
| `bracken-ddos` | Synthetic traffic evaluation, telemetry freshness, false positives, observe/dry-run mode, action bounds, expiry, and rollback. |

## 7. Pull request expectations

A useful pull request includes:

1. **Problem:** what user or operator problem this solves.
2. **Approach:** what changed and why this design was chosen.
3. **Scope:** affected components, contracts, configuration, and deployment assumptions.
4. **Validation:** commands and tests run, results, environment, and anything not tested.
5. **Safety:** security impact, failure behavior, rollback, and data or traffic impact.
6. **Compatibility:** migration needs and supported old/new combinations.
7. **Documentation:** updated README, guide, examples, or design record.
8. **Follow-up:** known limitations and separate work that remains.

Keep the pull request reviewable. Split unrelated changes. Respond to review comments with an update or a clear explanation. Mark threads resolved only after the requested change or rationale is present. Do not merge your own change unless repository policy explicitly permits it.

## 8. License and contribution ownership

The existing BRACKEN repositories use the MIT License. Only contribute code, documentation, designs, or test material that you created or have permission to contribute. Preserve required notices for third-party material and identify copied or generated content with its source and license. Check dependency licenses before adding a dependency. No separate contributor license agreement is established by this guide; maintainers must state any future contributor agreement explicitly.

## 9. Current project limitations

Some repositories may not yet have working build, test, release, or deployment commands. Treat this as an open project task: document reproducible commands when they are implemented and exercised. Do not submit placeholder commands as if they were validated. If a contribution needs an unsettled architecture choice, submit a proposal or draft pull request and label the decision clearly.

Contributions should help BRACKEN become easier to understand, reproduce, validate, and operate. Clear boundaries, honest status, safe failure behavior, and reviewable changes matter as much as adding features.
