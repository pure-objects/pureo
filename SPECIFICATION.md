# PureO Language Specification

Version 0.1 — Draft

PureO is a statically typed programming language for the JVM that
structurally enforces the Elegant Objects principles. What other
languages leave to discipline — immutability, no null, no static
methods, no getters — PureO makes impossible to violate: the compiler
rejects such programs. Its syntax stays close to mainstream languages
so that the ideas of the [EO](https://github.com/objectionary/eo)
research project become usable in industrial codebases.

This document specifies version 0.1 of the language: declarations,
the type system, primitive objects, Java interoperability, modules,
error handling, and the entry point. Everything here is a direction,
not a contract; open questions are marked with `OPEN:`.

## 1. Design principles

Every rule in this specification derives from a small set of axioms:

1. **Everything is an object.** There are no primitives, no statics,
   no procedures. Numbers, text, and booleans are objects.
2. **Objects are immutable.** An object never changes state after
   construction. "Modification" means creating a new object.
3. **No null.** The type system has no bottom value. Absence is
   modeled explicitly (see §10).
4. **Types are interfaces.** The only public types are interfaces.
   Object declarations are implementation details.
5. **Composition over inheritance.** There is no implementation
   inheritance. Objects reuse behavior by decorating other objects.
6. **Constructors construct.** A constructor may only bind attributes;
   it never computes, validates eagerly, or performs I/O.
7. **Fail fast.** Errors surface immediately as exceptions; there is
   no defensive recovery deep inside objects.

## 2. Source files and lexical structure

### 2.1 Files

A PureO source file uses the `.pureo` extension and UTF-8 encoding.
A file contains one module declaration followed by imports and one or
more type declarations (interfaces and objects).

### 2.2 Comments

```pureo
// line comment

/*
 * Block comment, used for documentation of interfaces and objects.
 */
```

There are no special documentation markers in v0.1; a block comment
immediately preceding a declaration is its documentation.

### 2.3 Identifiers

Identifiers match `[a-zA-Z][a-zA-Z0-9]*`. Type names are
`UpperCamelCase`; attributes, methods, and parameters are
`lowerCamelCase`. Underscores are not permitted: names are words,
not sentences.

### 2.4 Keywords

```
module import interface object new if else true false
```

The keyword list is deliberately short. `OPEN:` `match` and `let`
are reserved but unspecified in v0.1.

### 2.5 Literals

| Literal    | Examples                  | Type      |
|------------|---------------------------|-----------|
| Integer    | `42`, `-7`, `1_000_000`   | `Int`     |
| Decimal    | `3.14`, `-0.5`            | `Decimal` |
| Text       | `"hello"`, `"line\nfeed"` | `Text`    |
| Boolean    | `true`, `false`           | `Bool`    |

Literals are syntactic sugar for primitive-object construction (§6).

## 3. Modules and imports

A module is a namespace, declared at the top of every file:

```pureo
module org.example.billing

import org.pureo.text.Formatted
import org.example.money.Money
```

- Module names are dot-separated identifiers, by convention rooted in
  a reversed domain name.
- The directory layout mirrors the module path, as in Java.
- An import brings one type into scope; there are no wildcard imports.
- All types declared in a module are visible to other modules.
  `OPEN:` module-private object declarations (e.g. a `local` modifier)
  are under consideration for v0.2.

The standard library lives under `org.pureo.*` and is imported
implicitly for the core types listed in §6.

## 4. Interfaces

An interface declares a type: a set of method signatures.

```pureo
/*
 * An amount of money in a single currency.
 */
interface Money {
  plus(other: Money): Money
  scaled(factor: Decimal): Money
  printed(): Text
}
```

Rules:

- Interfaces contain only method signatures — no default bodies, no
  constants, no static members.
- An interface may extend one or more interfaces:
  `interface Salary: Money, Comparable { ... }`. Extension is purely
  additive; diamond composition is legal because there are no bodies
  to conflict.
- Every method has an explicit return type. There is no `void`: a
  method that exists only for its effect returns the object that
  represents the effect's outcome (§5.4).
- Method names are single verbs or nouns, following CQRS: a noun
  names what the method returns; a verb names what it does.

Interfaces are the only public types. Client code declares parameters,
attributes, and return types with interfaces, never with object names
from another module. `OPEN:` whether the compiler enforces
"interfaces only across module boundaries" in v0.1 or defers to v0.2.

## 5. Objects

An object declaration provides an implementation of one or more
interfaces. It is the only way to create behavior.

```pureo
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

### 5.1 Primary constructor

The parenthesized list after the object name is the primary
constructor. It is the complete list of the object's attributes.

- Attributes are bound once, at construction, and never reassigned.
- The primary constructor contains no code: no validation, no
  computation, no I/O. If input must be validated, validation is an
  object (§10.3) or happens lazily inside methods.
- An object has between zero and four attributes. More than four is a
  compile-time error: the object is doing too much.
- Attributes are invisible outside the object. There is no syntax to
  read another object's attribute.

An object with zero attributes omits the parentheses:

```pureo
object ZeroMoney: Money { ... }
```

### 5.2 Secondary constructors

A secondary constructor, introduced by `new`, is pure delegation: its
entire body is a single call to the primary constructor or to another
secondary constructor.

```pureo
new(cents: Int) = UsdMoney(Decimal.of(cents, 2))
new() = UsdMoney(0)
```

Secondary constructors may not contain any other expression. In the
generated Java, they become static factory methods; that `static` is
a bytecode detail and is inexpressible in PureO source.

### 5.3 Methods

A method body is a single expression:

```pureo
printed(): Text = Formatted("$%s", amount)
```

- Every method declared by an object must implement a signature from
  one of its interfaces. Objects have no public methods of their own.
- Methods cannot be overridden: all objects are final. To vary
  behavior, decorate (§5.5).
- Recursion is the general iteration mechanism; collection types offer
  higher-order methods (`mapped`, `reduced`, `filtered`) for the
  common cases (§6.5).

`OPEN:` v0.1 restricts bodies to single expressions. Local named
sub-expressions (a `let`-like form) are under consideration for
readability in v0.2; multi-statement bodies with mutable locals are
permanently out.

### 5.4 Conditionals

`if`/`else` is an expression, and `else` is mandatory:

```pureo
signum(): Int = if amount.less(0) { -1 } else if amount.equals(0) { 0 } else { 1 }
```

There are no statements in the language, hence no loops, no `return`,
no `break`.

### 5.5 Decoration

Objects reuse behavior by wrapping other objects of the same
interface:

```pureo
/*
 * Money that logs every addition before delegating.
 */
object LoggedMoney(origin: Money, log: Log): Money {
  plus(other: Money): Money = LoggedMoney(origin.plus(logged(other)), log)
  scaled(factor: Decimal): Money = LoggedMoney(origin.scaled(factor), log)
  printed(): Text = origin.printed()
}
```

Decoration is manual in v0.1: every method is written out. `OPEN:` a
delegation shorthand (`object LoggedMoney(origin: Money, ...) : Money by origin`)
is under consideration; it must not become implementation inheritance
in disguise.

## 6. Primitive objects

PureO has no primitive values. The types below are ordinary interfaces
from `org.pureo.core`, imported implicitly; the compiler maps them to
efficient JVM representations where possible, but that optimization is
invisible at the language level.

### 6.1 `Int`

Arbitrary 64-bit signed integer. Methods include `plus`, `minus`,
`times`, `divided`, `less`, `greater`, `equals`, `printed`. Overflow
throws (§10) rather than wrapping.

### 6.2 `Decimal`

Exact decimal arithmetic (backed by `java.math.BigDecimal`). There is
no binary floating-point type in v0.1. `OPEN:` an `Ieee754` object
for scientific workloads, clearly named to expose its nature.

### 6.3 `Text`

Immutable Unicode text. Methods include `joined`, `length`, `part`,
`equals`, `printed`. There is no mutable builder; large concatenations
are objects (`Joined(parts)`) evaluated lazily.

### 6.4 `Bool`

`true` and `false`. Methods `and`, `or`, `negated`. The `if`
expression (§5.4) requires a `Bool`.

### 6.5 Collections

`List<T>`, `Mapped<K, V>`, and `Set<T>` are immutable interfaces with
structural operations returning new collections: `with`, `without`,
`mapped`, `filtered`, `reduced`, `each`. Generic parameters follow
Java-like syntax. `OPEN:` variance annotations are unspecified in
v0.1; type parameters are invariant.

## 7. Type system

- Nominal typing: an object is of type `T` iff it declares `: T`
  (directly or via interface extension).
- No subtyping between objects, only object-to-interface.
- No type introspection: there is no `instanceof`, no casting, no
  reflection API. If behavior must vary by kind, model the kinds as
  interfaces and dispatch by method call.
- Generics are erased at the JVM boundary, as in Java, but the PureO
  compiler checks them fully at compile time.
- Type inference is local only: attribute, parameter, and return
  types are always explicit; the types of sub-expressions are
  inferred.

## 8. Java interoperability

Interop is asymmetric by design: PureO objects are first-class Java
citizens, while Java values entering PureO are treated as suspect.

### 8.1 Outbound: PureO → Java

Each object compiles to a `final` Java class implementing the Java
interfaces generated from its PureO interfaces:

- `interface Money` → `public interface Money` with identical
  signatures.
- `object UsdMoney` → `public final class UsdMoney implements Money`
  with a package-private constructor and public static factories for
  each secondary constructor.
- Primitive objects map to stdlib-provided Java types
  (`org.pureo.core.Text`, etc.) with converters to and from
  `java.lang.String` and friends.

Any Java project can therefore consume a PureO library as a plain
JAR, with no runtime dependency beyond `org.pureo:core`.

### 8.2 Inbound: Java → PureO

Calling Java from PureO goes through the `java` construct, the only
place where the outside world's rules apply:

```pureo
object EnvText(name: Text): Text {
  printed(): Text =
    java(System.getenv(name.java())).or("")
}
```

- Every Java method return value is typed as `Nullable<T>` in PureO.
  It must be unwrapped explicitly: `.or(fallback)` supplies a default,
  `.orFail(message)` throws. There is no implicit conversion from
  `Nullable<T>` to `T`.
- Java static methods and fields are reachable only inside a
  `java(...)` expression. The construct is deliberately visible and
  greppable; a codebase audit for `java(` finds every impurity.
- Checked Java exceptions propagate as PureO failures (§10) without
  declaration.
- For major libraries the recommended pattern is an adapter module:
  a PureO interface plus one object whose methods each contain a
  single `java(...)` expression.

`OPEN:` the exact surface of `java(...)` — method references,
overload resolution against Java's primitive widening — is specified
in a dedicated interop document for v0.2.

## 9. Equality

`equals` is an ordinary interface method, not a JVM identity check.
Objects that need equality declare it in their interface. The
compiler generates `Object.equals`/`hashCode` in the emitted Java
class delegating to the PureO `equals` when present, and falls back
to attribute-wise structural equality otherwise. There is no `==`
operator in v0.1. `OPEN:` operator sugar (`==`, `+`) desugaring to
method calls is under consideration and would be sugar only.

## 10. Error handling

### 10.1 Failures

PureO follows fail-fast. A failure is an exception: it abandons the
current computation and propagates to the entry point unless caught
at an architectural boundary.

```pureo
fail("negative amount: %s", amount)
```

`fail` is an expression of every type, so it can appear in any branch:

```pureo
sqrt(): Decimal =
  if amount.less(0) { fail("no real root of %s", amount) } else { ... }
```

Failure messages are single sentences with context and no trailing
period.

### 10.2 Catching

There is no `try`/`catch` syntax. Recovery is an object:

```pureo
object OrZero(risky: Money): Money {
  plus(other: Money): Money = Rescued(risky, ZeroMoney()).plus(other)
  ...
}
```

`Rescued<T>` from the standard library evaluates its first argument
and falls back to the second on failure. Because recovery is an
object, it composes, decorates, and shows up in the type structure
instead of hiding in control flow. `OPEN:` whether `Rescued` can
observe the failure's message in v0.1.

### 10.3 Validation

Since constructors cannot validate, invalid state is caught at first
use:

```pureo
object PositiveInt(origin: Int): Int {
  plus(other: Int): Int = checked().plus(other)
  ...
}
```

where `checked()` is a private pattern: the object fails on first
method call if its origin violates the invariant. The standard
library provides `Checked<T>` decorators for common invariants.

## 11. Entry point

There is no static `main`. A program is an object implementing
`App`:

```pureo
module org.example.hello

interface App {
  exit(args: List<Text>): Int
}

object Hello: App {
  exit(args: List<Text>): Int =
    Printed(Stdout(), "Hello, %s!", args.at(0).or("world")).code()
}
```

- `App` is defined in `org.pureo.core`; the local declaration above
  is illustrative.
- The runtime instantiates the single `App` object named in the
  build manifest, passes the command-line arguments, and uses the
  returned `Int` as the process exit code.
- Standard streams are objects (`Stdout()`, `Stdin()`, `Stderr()`)
  injected nowhere implicitly: an object that prints must receive or
  construct the stream it prints to.

`OPEN:` the manifest format (how the runtime finds the `App` object)
belongs to the build-tool specification, not the language.

## 12. What PureO deliberately omits

For readers arriving from Java, the absences are the specification:

| Absent                     | Replacement                          |
|----------------------------|--------------------------------------|
| `null`                     | `Nullable<T>` at the Java boundary; explicit absence objects elsewhere |
| `static` methods/fields    | Objects; secondary constructors      |
| Getters/setters            | Behavior-revealing methods           |
| Mutable state              | New objects                          |
| Implementation inheritance | Decoration                           |
| `instanceof`, casts        | Interface dispatch                   |
| Reflection                 | Nothing; it is not needed            |
| Loops, `return`, `break`   | Recursion; collection methods        |
| `try`/`catch`              | `fail`; `Rescued` objects            |
| Utility classes            | Objects with real names              |

## 13. Open questions (v0.1 → v0.2)

Collected from the `OPEN:` markers above:

1. Module-private declarations (§3).
2. Compiler enforcement of interfaces-only across modules (§4).
3. `let`-like local bindings in method bodies (§5.3).
4. Delegation shorthand for decorators (§5.5).
5. Binary floating-point object (§6.2).
6. Generic variance (§6.5).
7. Full `java(...)` interop surface (§8.2).
8. Operator sugar desugaring to methods (§9).
9. Failure introspection in `Rescued` (§10.2).
