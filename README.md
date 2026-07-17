# validate

A data/input validation library for [Ecko](https://ecko.sh), written in Ecko.
Validate a form submission, an API body, or a config map against a schema of
composable rules, and get back every error at once.

Built on Ecko's first-class functions: **a validator is just a function**
`|value| -> null (pass) | message (fail)`. No native code, no capabilities —
pure computation over `std.re` and the builtins.

## Install

```bash
ecko add https://github.com/ecko-sh/validate
```

`ecko add` vendors the package into `./vendor/validate/` and pins it by SHA-256
in `ecko.lock`. It needs no capabilities.

## Usage

```ecko
import validate
v = validate                       # `import ... as` is not supported yet

schema = {
    name:  [v.required(), v.string(), v.min_len(2)],
    email: [v.required(), v.email()],
    age:   [v.int(), v.min(18)],
    role:  [v.one_of(["admin", "member", "guest"])],
    tags:  [v.each(v.string())],
}

errors = v.check(data, schema)     # {} = valid, else { field: [messages] }

if v.is_valid(data, schema) { ... }

# validate() returns the data, or raises a kind-"validation" error you can catch
clean = try { v.validate(data, schema) } catch (e) { get(e, "errors") }
```

### Errors are collected, not fail-fast

```ecko
v.check(
    { name: "A", email: "nope", age: 15 },
    { name: [v.required(), v.min_len(2)], email: [v.email()], age: [v.min(18)] },
)
# {
#   name:  ["must be at least 2 characters"],
#   email: ["must be a valid email"],
#   age:   ["must be at least 18"],
# }
```

## How it works

Each validator checks **exactly one** concern and passes (`null`) on a `null` or
wrong-typed value — so `[v.required(), v.int(), v.min(18)]` on a missing field
yields exactly `["is required"]` (no cascade), on `"abc"` yields
`["must be a whole number"]`, and never crashes by running `<` on a string.
**Fields are optional by default**; `required()` is the only rule that fails on
`null`. There is **no coercion** — validation checks, it never transforms.

A custom rule is just a function:

```ecko
even = |x| if x == null { null } else if x % 2 == 0 { null } else { "must be even" }
schema = { n: [v.int(), even] }
```

## Validators

| Group | Validators |
|---|---|
| Presence | `required()`, `optional()` |
| Types | `string()`, `int()`, `float()`, `number()`, `bool()`, `list()`, `map()` |
| Numbers | `min(n)`, `max(n)`, `between(lo, hi)`, `positive()`, `negative()` |
| Strings | `min_len(n)`, `max_len(n)`, `len_between(lo, hi)`, `pattern(re, msg?)`, `email()`, `url()` |
| Membership | `one_of(options)` |
| Collections | `non_empty()`, `each(validator)` |
| Nested | `shape(schema)` |

## Runner

| Function | Description |
|---|---|
| `check(data, schema)` | `{ field: [messages] }` — empty means valid |
| `is_valid(data, schema)` | `Bool` |
| `validate(data, schema)` | Returns `data`, or raises a kind-`"validation"` error carrying the error map |
| `check_value(value, rules)` | Run a rule list against a single value → `[messages]` |

## Testing

```bash
ecko test tests/
```

Offline and deterministic. `example.ecko` is a runnable signup-form demo.

## License

MIT — see [LICENSE](LICENSE).
