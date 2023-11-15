Glob expressions resemble Unix paths, but are distinct from paths and support
patterns that can be matched against file paths and directory trees (i.e.,
search). Pattern syntax is similar to that found in Unix shells and tools like
`git`, though there are some important differences.

!!! info inline end
    Glob expressions **are not paths** (nor are they "paths but with meta
    characters"). Glob expressions are a distinct language with a consistent
    syntax on all platforms and do **not** support all native path features.

Here's an example of a glob expression:

```
**/*.{go,rs}
```

This glob matches any path with a final component ending with `.go` or `.rs`,
such as `./wax/src/lib.rs` or `glob.go`.

Glob syntax is opinionated and is **not** configurable. For example, with the
exception of [tree wildcards](#wildcards), patterns **never** match across
component boundaries (separators) and cannot be configured to do so.

## Wildcards

Wildcards match some amount of arbitrary text in paths and are the most
fundamental pattern provided by globs (and likely the most familiar).

The zero-or-more wildcards `*` and `$` match zero or more of any character
within a component (**never path separators**). Zero-or-more wildcards cannot be
adjacent to other zero-or-more wildcards. The `*` wildcard is eager and will
match the longest possible text while the `$` wildcard is lazy and will match
the shortest possible text. When followed by a literal, `*` stops at the last
occurrence of that literal while `$` stops at the first occurence.

The exactly-one wildcard `?` matches any single character within a component
(**never path separators**). Exactly-one wildcards do not group automatically,
so a pattern of contiguous wildcards such as `???` form distinct captures for
each `?` wildcard. [An alternative](#alternatives) can be used to group
exactly-one wildcards into a single capture, such as `{???}`.

The tree wildcard `**` matches any characters across zero or more components.
**This is the only pattern that implicitly matches across arbitrary component
boundaries**; all other patterns do **not** implicitly match across component
boundaries. When a tree wildcard participates in a match and does not terminate
the pattern, its captured text includes the trailing separator. If a tree
wildcard does not participate in a match, then its captured text is an empty
string.

Tree wildcards must be delimited by forward slashes or terminations (the
beginning and/or end of an expression). **Tree wildcards and path separators are
distinct** and any adjacent forward slashes that form a tree wildcard are parsed
together. Rooting forward slashes in tree wildcards are meaningful and the glob
expressions `**/*.txt` and `/**/*.txt` differ in that the former is relative
(has no root) and the latter has a root.

If a glob expression consists solely of a tree wildcard, then it matches any and
all paths and the complete contents of any and all directory trees, including
the root.

## Character Classes

Character classes match any single character from a group of literals and ranges
within a component (**never path separators**). Classes are delimited by square
brackets `[...]`. Individual character literals are specified as is, such as
`[ab]` to match either `a` or `b`. Character ranges are formed from two
characters separated by a hyphen, such as `[x-z]` to match `x`, `y`, or `z`.
Character classes match characters exactly and are always case-sensitive, so the
expressions `[ab]` and `{a,b}` are not necessarily the same.

Any number of character literals and ranges can be used within a single
character class. For example, `[qa-cX-Z]` matches any of `q`, `a`, `b`, `c`,
`X`, `Y`, or `Z`.

Character classes may be negated by including an exclamation mark `!` at the
beginning of the class pattern. For example, `[!a]` matches any character except
for `a`. **These are the only patterns that support negation.**

It is possible to escape meta-characters like `*`, `$`, etc., using character
classes though globs also support escaping via a backslash `\`. To match the
control characters `[`, `]`, and `-` within a character class, they must be
escaped via a backslash, such as `[a\-]` to match `a` or `-`.

Character classes have notable platform-specific behavior, because they match
arbitrary characters in native paths but never match path separators. This means
that if a character class consists of **only** path separators on a given
platform, then the character class is considered empty and matches nothing. For
example, in the expression `a[/]b` the character class `[/]` matches nothing on
Unix and Windows. Such character classes are not rejected, because the role of
arbitrary characters depends on the platform. In practice, this is rarely a
concern, but **such patterns should be avoided**.

Character classes have limited utility on their own, but compose well with
[repetitions](#repetitions).

## Alternatives

Alternatives match an arbitrary sequence of one or more comma separated
sub-globs delimited by curly braces `{...,...}`. For example, `{a?c,x?z,foo}`
matches any of the sub-globs `a?c`, `x?z`, or `foo`. Alternatives may be
arbitrarily nested and composed with [repetitions](#repetitions).

Alternatives form a single capture group regardless of the contents of their
sub-globs. This capture is formed from the complete match of the sub-glob, so if
the alternative `{a?c,x?z}` matches `abc`, then the captured text will be `abc`
(**not** `b`). Alternatives can be used to group captures using a single
sub-glob, such as `{*.{go,rs}}` to capture an entire file name with a particular
extension or `{???}` to group a sequence of exactly-one wildcards.

Alternatives must consider adjacency rules and neighboring patterns. For
example, `*{a,b*}` is allowed but `*{a,*b}` is not. Additionally, they may not
contain a sub-glob consisting of a singular tree wildcard `**` and cannot root a
glob expression as this could cause the expression to match or walk overlapping
trees.

## Repetitions

Repetitions match a sub-glob a specified number of times. Repetitions are
delimited by angle brackets with a separating colon `<...:...>` where a sub-glob
precedes the colon and an optional bounds specification follows it. For example,
`<a*/:0,>` matches the sub-glob `a*/` zero or more times. Though not implicit
like tree [wildcards](#wildcards), **repetitions can match across component
boundaries** (and can themselves include tree wildcards). Repetitions may be
arbitrarily nested and composed with [alternatives](#alternatives).

Bound specifications are formed from inclusive lower and upper bounds separated
by a comma `,`, such as `:1,4` to match between one and four times. The upper
bound is optional and may be omitted. For example, `:1,` matches one or more
times (note the trailing comma `,`). A singular bound is convergent, so `:3`
matches exactly three times (both the lower and upper bounds are three). If no
lower or upper bound is specified, then the sub-glob matches one or more times,
so `<a:>` and `<a:1,>` are equivalent. Similarly, if the colon `:` is also
omitted, then the sub-glob matches zero or more times, so `<a>` and `<a:0,>` are
equivalent.

Repetitions form a singular capture group regardless of the contents of their
sub-glob. The capture is formed from the complete match of the sub-glob. If the
repetition `<abc/>` matches `abc/abc/`, then the captured text will be
`abc/abc/`.

Repetitions compose well with [character classes](#character-classes). Most
often, a glob expression like `{????}` is sufficient, but the more specific
expression `<[0-9]:4>` further constrains the matched characters to digits, for
example. Repetitions may also be more terse, such as `<?:8>`. Furthermore,
repetitions can form tree expressions that further constrain components, such as
`<[!.]*/>[!.]*` to match paths that contain no leading dots `.` in any
component.

Repetitions must consider adjacency rules and neighboring patterns. For example,
`a/<b/**:1,>` is allowed but `<a/**:1,>/b` is not. Additionally, they may not
contain a sub-glob consisting of a singular separator `/`, a singular
zero-or-more wildcard `*` or `$`, nor a singular tree wildcard `**`. Repetitions
with a lower bound of zero may not root a glob expression, as this could cause
the expression to match or walk overlapping trees.

## Flags and Case Sensitivity

Flags toggle the matching behavior of globs. Importantly, flags are a part of a
glob expression rather than an API or behavior specific to an application.
Behaviors are toggled immediately following flags in the order in which they
appear in glob expressions. Flags are delimited by parenthesis with a leading
question mark `(?...)` and may appear anywhere within a glob expression so long
as they do not split tree wildcards (e.g., `a/*(?i)*` is not allowed). Each flag
is represented by a single character and can be negated by preceding the
corresponding character with a minus `-`. Flags are toggled in the order in
which they appear within `(?...)`.

The only supported flag is the case-insensitivty flag `i`. By default, glob
expressions use the same case sensitivity as the target platforms's file system
APIs (case-sensitive on Unix and case-insensitive on Windows), but `i` can be
used to toggle this explicitly as needed. For example,
`(?-i)photos/**/*.(?i){jpg,jpeg}` matches file paths beneath a `photos`
directory with a case-**sensitive** base and a case-**insensitive** extension
`jpg` or `jpeg`.
