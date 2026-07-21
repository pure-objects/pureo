# PureO — Project Context

## Vision
PureO is a JVM programming language that structurally enforces the
Elegant Objects principles (Yegor Bugayenko), with a familiar syntax —
unlike the EO language (objectionary/eo), whose φ-calculus-based syntax
hinders adoption. Positioning: the "industrial implementation" of the
EO ideas, complementary to the EO research project.

## Decisions made
- Name: PureO. GitHub org: pure-objects. Domains: pureo.org, pure-objects.org
- Target: JVM, by transpiling to Java source code (bytecode backend
  possible later; architecture AST → IR → emitter)
- File extension: .pureo
- Open source; license: MIT or Apache 2.0 (to be decided)
- Compiler written in Java, hand-written recursive descent parser
  (priority on error message quality)

## Language constraints (enforced by the compiler, not conventions)
- No null; no static methods; no getters/setters
- Full immutability; small objects; no implementation inheritance
  (composition/decoration only)
- Primary constructor = attribute list only (no code);
  secondary constructors = pure delegation
- Public types = interfaces only
- No instanceof/cast, no reflection

## JVM interop
- Outbound: PureO objects → final Java classes implementing regular
  Java interfaces (consumable by any Java project)
- Inbound: Java method return values typed as nullable, explicit
  unwrapping required; Java statics accessible only through a dedicated,
  visible construct; adapters recommended for major libraries
- Secondary constructors → generated static factories (static exists
  in the bytecode but is inexpressible in the language)

## Syntax sketch (direction, not final)
interface Money {
  plus(other: Money): Money
  printed(): Text
}

object UsdMoney(amount: Decimal): Money {
  new(cents: Int) = UsdMoney(Decimal.of(cents, 2))
  plus(other: Money): Money = UsdMoney(amount.plus(other.decimal()))
  printed(): Text = Formatted("$%s", amount)
}

## Entry point
No static main: an App object instantiated by the runtime (to be specified).

## Next steps
1. SPECIFICATION.md v0.1 (declarations, types, primitive-objects, interop,
   modules, error handling, entry point)
2. Compelling README + examples/*.pureo (hello world, business object,
   Java interop)
3. LICENSE + CONTRIBUTING.md (BDFL model, rfc issues)
4. Compiler built in public
