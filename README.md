<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/svg/logo-dark.svg">
  <img src="assets/svg/logo-light.svg" height="56" alt="PureO">
</picture>

# PureO

**A JVM language where object thinking is not a discipline — it is the grammar.**

PureO structurally enforces the [Elegant Objects](https://www.elegantobjects.org)
principles. What other languages leave to code reviews — immutability, no null,
no static methods, no getters — PureO makes impossible to write: the compiler
rejects such programs. Its syntax stays close to Java and Kotlin, so the ideas
pioneered by the [EO](https://github.com/objectionary/eo) research project
become usable in industrial codebases.

> **Status: design phase.** The [language specification](SPECIFICATION.md)
> is at v0.1 (draft). The compiler does not exist yet; it will be built in
> public, in Java, with a hand-written recursive descent parser.

## A taste

```pureo
module org.example.billing

/*
 * An amount of money in a single currency.
 */
interface Money {
  plus(other: Money): Money
  scaled(factor: Decimal): Money
  printed(): Text
}

/*
 * Money kept in US dollars with two-digit precision.
 */
object UsdMoney(amount: Decimal): Money {
  new(cents: Int) = UsdMoney(Decimal.of(cents, 2))
  plus(other: Money): Money = UsdMoney(amount.plus(other.scaled(1.0).decimal()))
  scaled(factor: Decimal): Money = UsdMoney(amount.times(factor))
  printed(): Text = Formatted("$%s", amount)
}
```

Everything is an object. Every object is immutable. Every public type is an
interface. A method body is a single expression. There are no statements, no
loops, no `return` — and nothing to unlearn if you already think in objects.

## Hello, world

There is no static `main`. A program is an object implementing `App`,
instantiated by the runtime:

```pureo
module org.example.hello

object Hello: App {
  exit(args: List<Text>): Int =
    Printed(Stdout(), "Hello, %s!", args.at(0).or("world")).code()
}
```

Even the standard output is an object you receive or construct — never an
ambient global.

## What the compiler forbids

For readers arriving from Java, the absences are the point:

| Absent in PureO            | Replaced by                          |
|----------------------------|--------------------------------------|
| `null`                     | Explicit absence objects; `Nullable<T>` at the Java boundary |
| `static` methods/fields    | Objects; secondary constructors      |
| Getters/setters            | Behavior-revealing methods           |
| Mutable state              | New objects                          |
| Implementation inheritance | Decoration                           |
| `instanceof`, casts, reflection | Interface dispatch              |
| Loops, `return`, `break`   | Recursion; collection methods        |
| `try`/`catch`              | `fail`; `Rescued` objects            |
| Utility classes            | Objects with real names              |

These are not lint warnings or conventions. Each one is a compile-time error.

## Java interoperability

Interop is asymmetric by design.

**Outbound**, every PureO object compiles to a `final` Java class implementing
plain Java interfaces. Any Java project consumes a PureO library as an
ordinary JAR, with no runtime dependency beyond `org.pureo:core`.

**Inbound**, Java values entering PureO are treated as suspect. Every Java
return value arrives as `Nullable<T>` and must be unwrapped explicitly, and
Java statics are reachable only inside the visible, greppable `java(...)`
construct:

```pureo
object EnvText(name: Text): Text {
  printed(): Text =
    java(System.getenv(name.java())).or("")
}
```

A codebase audit for `java(` finds every impurity.

## Why not EO itself?

[EO](https://github.com/objectionary/eo) proved that a language built on pure
object composition works, but its φ-calculus-based syntax hinders adoption.
PureO takes the opposite trade: keep the semantics radical, keep the syntax
familiar. Think of PureO as the industrial implementation of the EO ideas,
complementary to the research project.

## Roadmap

1. ~~[Language specification](SPECIFICATION.md) v0.1~~ — drafted
2. `examples/*.pureo` — hello world, a business object, Java interop
3. Governance: LICENSE, CONTRIBUTING.md, RFC process
4. The compiler, built in public: AST → IR → Java source emitter

## Contributing

The language design is open for debate: the specification marks every
unsettled decision with `OPEN:`, and each one deserves an issue. If you have
built software the Elegant Objects way and hit the limits of mainstream
languages, your experience is exactly what this project needs.
