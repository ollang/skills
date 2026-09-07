# Ollang agent skill library

Prompt-based skills that teach agents to use the Ollang integration API. The root SKILL.md routes intent to operation-specific skill directories; this is not a Node application.

## Structure and editing

- `SKILL.md` is the routing entrypoint; `README.md` documents installation and the available operations.
- Each `ollang-*/SKILL.md` owns a bounded operation. Inspect its frontmatter, request examples, response handling, and links before editing.
- When adding a skill, update the router and README index together. Keep names, trigger descriptions, and relative links consistent with the directory actually shipped.
- Preserve progressive disclosure: common routing belongs at the root, detailed API behavior in the owning skill.

## API and execution rules

- Validate methods, paths, payloads, pagination, and required fields against the integration backend and public API docs. Distinguish project creation/uploads from order creation and asynchronous processing.
- Read `OLLANG_API_KEY` from the environment in examples; never substitute an actual credential into source, tool arguments, or transcripts.
- Use safe quoting and curl's glob handling for bracketed query parameters. Check HTTP failures before parsing a success response.
- Separate read-only discovery from operations that create paid orders, rerun jobs, request review, or delete data. Honor existing user authorization; do not add blanket confirmation requirements that interrupt already-authorized work.
- Describe truthful tool capabilities and required inputs. Do not promise unavailable tools, automatic notifications, or completed processing after merely submitting a job.

## Validation

- There is no package manager, build, or test runner. Check Markdown frontmatter, every local link, router coverage, and examples against the API schema.
- Walk through one representative input/output per changed skill, including missing inputs and error responses. Validate shell syntax using placeholders; do not execute paid or destructive API examples just to test documentation.
- Keep the docs site's skills pages in sync when behavior or installation instructions change.

## Working agreement

- Inspect the current diff and nearby implementation before editing; preserve unrelated work. Keep changes focused on the requested behavior.
- Treat manifests, source, and CI configuration as the authority when this guide drifts. Update guidance alongside changes to commands or architecture.
- Keep credentials, customer data, generated caches, and local environment files out of diffs and logs.
- Run checks appropriate to the change, then `git diff --check`. Report what passed, existing failures, environment blockers, and any unverified runtime behavior separately.
- Maintain shared instructions here; `CLAUDE.md` imports this file.
