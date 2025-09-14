# Recurrence Markup Language (RML)

**RML** is a JSON-based standard for defining recurring dates and schedules. It
precisely models everything from simple weekly patterns to complex business
logic—including holiday adjustments and non-Gregorian calendars—in a portable,
implementation-agnostic format.

## Key Features

- **Expressive:** Model complex rules like "the last business day of the month"
  or lunisolar festivals with clear, unambiguous semantics.
- **Portable:** A pure, schema-validated JSON format with zero runtime
  dependencies.
- **Deterministic:** Explicit definitions for edge cases like time zones and
  holidays ensure consistent behavior across any system.
- **Composable:** Build powerful schedules by layering simple rules,
  constraints, and exceptions.

## Why RML?

Existing tools are often not enough for modern scheduling needs. RML provides a
superior alternative:

- **Beyond iCalendar RRULE:** Offers first-class support for business-day logic
  and non-Gregorian calendars where RRULE can be ambiguous or insufficient.
- **Smarter than Cron:** Provides a fully calendar-aware model that understands
  holidays and complex date calculations, not just fixed intervals.
- **More than a Library DSL:** A standardized data format ensures that rules are
  portable and behave identically across different systems and programming
  languages.

## Use Cases

RML is ideal for critical scheduling systems that require auditable and
deterministic logic, such as:

- Financial systems (trading days, billing cycles)
- Payroll and HR platforms
- Global applications requiring cultural and religious calendar support

## Resources (Version 1.0)

- **Specification:** `versions/1.0/spec.md`
- **JSON Schema:** `versions/1.0/schema.json`
- **Examples:** `versions/1.0/examples/*`

The 1.0 specification is stable and ready for implementation. Get started by
reading the spec and validating your RML documents against the official schema.
