[![MIT][s2]][l2] [![Latest Version][s1]][l1] [![docs][s3]][l3] [![Chat on Miaou][s4]][l4]

[s1]: https://img.shields.io/crates/v/lazy-regex.svg
[l1]: https://crates.io/crates/lazy-regex

[s2]: https://img.shields.io/badge/license-MIT-blue.svg
[l2]: LICENSE

[s3]: https://docs.rs/lazy-regex/badge.svg
[l3]: https://docs.rs/lazy-regex/

[s4]: https://miaou.dystroy.org/static/shields/room.svg
[l4]: https://miaou.dystroy.org/3


# lazy-regex

Use the  `regex!` macro to build regexes:

* they're checked at compile time
* they're wrapped in `once_cell` lazy static initializers so that they're compiled only once
* they can hold flags as suffix: `let case_insensitive_regex = regex!("ab*"i);`
* regex creation is less verbose

This macro returns references to normal instances of `regex::Regex` so all the usual features are available.

You may also use shortcut macros for testing a match or capturing groups as substrings:

* `regex_is_match!`
* `regex_find!`
* `regex_captures!`

# Build Regexes

```rust
use lazy_regex::regex;

// build a simple regex
let r = regex!("sa+$");
assert_eq!(r.is_match("Saa"), false);

// build a regex with flag(s)
let r = regex!("sa+$"i);
assert_eq!(r.is_match("Saa"), true);

// you can use a raw literal
let r = regex!(r#"^"+$"#);
assert_eq!(r.is_match("\"\""), true);

// or a raw literal with flag(s)
let r = regex!(r#"^\s*("[a-t]*"\s*)+$"#i);
assert_eq!(r.is_match(r#" "Aristote" "Platon" "#), true);

// this line wouldn't compile because the regex is invalid:
// let r = regex!("(unclosed");

```
Supported regex flags: 'i', 'm', 's', 'x', 'U'.

# Test a match

```rust
use lazy_regex::regex_is_match;

let b = regex_is_match!("[ab]+", "car");
assert_eq!(b, true);
```

# Extract a value

```rust
use lazy_regex::regex_find;

let f_word = regex_find!(r#"\bf\w+\b"#, "The fox jumps.");
assert_eq!(f_word, Some("fox"));
```

# Capture

```rust
use lazy_regex::regex_captures;

let (_, letter) = regex_captures!("([a-z])[0-9]+"i, "form A42").unwrap();
assert_eq!(letter, "A");

let (whole, name, version) = regex_captures!(
    r#"(\w+)-([0-9.]+)"#, // a literal regex
    "This is lazy_regex-2.0!", // any expression
).unwrap();
assert_eq!(whole, "lazy_regex-2.0");
assert_eq!(name, "lazy_regex");
assert_eq!(version, "2.0");
```

There's no limit to the size of the tupple.
It's checked at compile time to ensure you have the right number of capturing groups.

# Shared lazy static

When a regular expression is used in several functions, you sometimes don't want
to repeat it but have a shared static instance.

The `regex!` macro, while being backed by a lazy static regex, returns a reference.

If you want to have a shared lazy static regex, use the `lazy_regex!` macro:

```rust
use lazy_regex::*;

pub static GLOBAL_REX: Lazy<Regex> = lazy_regex!("^ab+$"i);
```

Like for the other macros, the regex is static, checked at compile time, and lazily built at first use.

