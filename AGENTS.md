# PureO — Contributor Context

Read [SPECIFICATION.md](SPECIFICATION.md) for the language rules and
[README.md](README.md) for the pitch and roadmap; this file records only
what is not derivable from them.

## Positioning

PureO is the industrial implementation of the ideas explored by the
[EO](https://github.com/objectionary/eo) research language: same
object-thinking radicalism, familiar syntax. It complements EO, it does
not compete with it.

## Decision log

- Name: PureO. GitHub org: `pure-objects`. Domains: pureo.org,
  pure-objects.org. File extension: `.pureo`
- Target: JVM, by transpiling to Java source (bytecode backend possible
  later; architecture AST → IR → emitter)
- Compiler written in Java, hand-written recursive descent parser —
  error message quality is the priority
- License: MIT or Apache 2.0, to be decided
- Governance: BDFL model; design debates happen in `rfc` issues

## Working conventions

- The specification is the source of truth; change it before, not
  after, changing anything that implements it
- Unsettled design decisions are marked `OPEN:` in the specification
  and each deserves its own issue
- Code, issues, and pull requests are written in English
- Compiler code follows the same Elegant Objects principles the
  language enforces
