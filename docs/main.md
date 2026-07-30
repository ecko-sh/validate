# validate

## `check(data, schema)`

check(data, schema) -> { field: [messages] }, empty when the data is valid.

Every rule of every field runs, so one call reports all the problems rather
than the first one. A field absent from `data` is validated as null, which
only `required()` objects to.

```ecko
errors = validate.check(form, { email: [v.required(), v.email()] })
```

## `check_value(value, rules)`

check_value(value, rules) -> [messages] for one value against one rule list.

The building block `check` uses per field. Useful on its own for a value that
is not part of a map.

## `is_valid(data, schema)`

is_valid(data, schema) -> whether `data` passes every rule.

The same work as `check`, when the messages are not wanted.

## `validate(data, schema)`

validate(data, schema) -> `data` when valid, otherwise raises.

The raised error has kind `"validation"` and carries the same map `check`
returns under `errors`, so a handler can render per-field messages.

```ecko
try { user = validate.validate(body, schema) }
catch e { respond(422, e.errors) }
```

## `required()`

required() -> fails on null. The only rule that does: every other validator
passes a null value, so fields are optional unless you say otherwise.

## `optional()`

optional() -> never fails. A marker for readers, since optional is already
the default.

## `string()`

string() -> fails unless the value is text.

## `int()`

int() -> fails unless the value is a whole number.

## `float()`

float() -> fails unless the value is a decimal number.

## `number()`

number() -> fails unless the value is an int or a float.

## `bool()`

bool() -> fails unless the value is true or false.

## `list()`

list() -> fails unless the value is a list.

## `map()`

map() -> fails unless the value is a map.

## `min(n)`

min(n) -> fails when a number is below `n`. Non-numbers pass, so the type
error is reported once by `number()` rather than by every rule.

## `max(n)`

max(n) -> fails when a number is above `n`.

## `between(lo, hi)`

between(lo, hi) -> fails when a number is outside `lo`..`hi`, inclusive.

## `positive()`

positive() -> fails unless a number is greater than zero.

## `negative()`

negative() -> fails unless a number is less than zero.

## `min_len(n)`

min_len(n) -> fails when text is shorter than `n` characters.

## `max_len(n)`

max_len(n) -> fails when text is longer than `n` characters.

## `len_between(lo, hi)`

len_between(lo, hi) -> fails when text length is outside `lo`..`hi`.

## `pattern(re_pat, msg = null)`

pattern(regex, msg?) -> fails when text does not match `regex`.

`msg` replaces the default "has an invalid format", which is worth setting -
a regex in an error message helps nobody.

## `email()`

email() -> fails unless the text looks like an email address.

A deliberately loose shape check (something@something.something). Whether an
address exists is a question only sending to it can answer.

## `url()`

url() -> fails unless the text is an http or https URL.

## `one_of(options)`

one_of(options) -> fails unless the value is one of `options`.

Works for any type, and lists the allowed values in the message.

## `non_empty()`

non_empty() -> fails on empty text or an empty list.

## `each(validator)`

each(validator) -> applies `validator` to every element of a list.

Element failures are numbered from 1 and joined into one message, so the
reader knows which item was wrong.

```ecko
{ tags: [v.list(), v.each(v.min_len(2))] }
```

## `shape(schema)`

shape(schema) -> validates a nested map against its own schema.

Sub-errors are flattened into a single `field: message` summary rather than a
nested map. A deeper error tree is a future change.
