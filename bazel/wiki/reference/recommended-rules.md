---
title: "Recommended Rules & Quality Standards"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["Recommended Rules.md"]
tags: ["rules", "quality", "standards", "ecosystem"]
graph-group: "reference"
related: ["[[reference/builtin-rules]]"]
last_updated: "2026-07-19"
---

# Recommended Rules & Quality Standards

## What Are Recommended Rules?

**Recommended rules** are a curated set of high-quality rulesets officially endorsed by the Bazel team.

**Purpose:** Help users find reliable, well-maintained rules among hundreds available on the internet.

**Status:** Rules must maintain high quality standards to be recommended; those that fall below standards may be demoted.

## Requirements for Rule Maintainers

To nominate a ruleset for recommendation, maintainers must demonstrate:

### Functional Requirements
1. **Useful to many users** — Addresses an important feature or widely-used language
2. **Well maintained** — At least two active maintainers
3. **Well documented** — Examples, guides, easy to use
4. **Best practices** — Follows Bazel performance and design guidelines
5. **High test coverage** — Comprehensive test suite

### Testing Requirements
6. **Tested on BuildKite** — Continuous integration with latest Bazel version
7. **Pre-submit checks** — All tests pass before each commit
8. **Upcoming changes** — Tested with Bazel's incompatible changes flag
9. **Quick issue resolution** — Breakages fixed within two weeks
10. **Rapid reporting** — Migration issues reported to Bazel team quickly

## Requirements for Bazel Core

The Bazel team maintains recommended rules by:

- **Frequent testing** — At least daily testing at Bazel HEAD (development version)
- **No regressions** — Bazel changes don't break recommended rules (with default flags)
- **Quick fixes** — If a Bazel change breaks a rule, it's fixed or rolled back immediately

## Nomination Process

1. File a [GitHub issue](https://github.com/bazelbuild/bazel/)
2. Demonstrate compliance with requirements above
3. Review by Bazel core team
4. Listed on the [Bazel website](https://bazel.build/rules) if approved

## Demotion

If a ruleset falls below quality standards:

1. GitHub issue filed to raise concern
2. Rule maintainers contacted (2-week response window)
3. Bazel core team decides based on maintainer response
4. Demotion may be decided if standards not met

## Finding Recommended Rules

Visit [bazel.build/rules](https://bazel.build/rules) for the complete list of recommended rulesets by language and domain.

---

See also: [[reference/builtin-rules]]
