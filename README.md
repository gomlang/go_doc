# Go documentation comments

`ecosystem::go_doc` parses Go documentation-comment text into a small AST and
renders it as canonical comment text, plain text, Markdown, or escaped HTML.
It is separate from GoML documentation and the `gomlgo` frontend. The source
syntax follows the [Go doc comment specification](https://go.dev/doc/comment).

`parse(input, Limits::standard())` returns a `Document` with paragraphs,
single-line `# ` headings, indented code blocks, bullet or numbered lists,
`[Text]: URL` definitions, URL links, and lexical symbol links. Use
`render(document, Format::Html, limits)` or another `Format` for output.
All failures return `Error` with a kind, line and message. Limits bound input
bytes, lines, blocks and link definitions separately, inline nodes, and
output bytes; render checks its budget
even for caller-constructed ASTs. HTML escapes text and URLs; only `http`,
`https`, and `mailto` URLs are accepted as links.

Markdown rendering escapes ASCII punctuation in ordinary text and link labels,
including entity ampersands and line-leading list/heading markers. Link destinations
escape ampersands, parentheses and backslashes. Symbol code spans choose a delimiter
longer than every backtick run and protect significant edge spaces in caller-built
ASTs. Code-span line endings are normalized to spaces before emission and padding
decisions, including all-whitespace labels and consecutive line breaks. These rules follow CommonMark's [backslash escapes](https://spec.commonmark.org/0.31.2/#backslash-escapes)
and [code spans](https://spec.commonmark.org/0.31.2/#code-spans); they prevent literal
comment text from becoming unintended Markdown formatting. Newlines in code spans
still follow CommonMark's whitespace normalization.

This module intentionally omits Go source extraction, package/import resolution,
automatic URL linking, note extraction, deprecation semantics, directives, and
`gofmt`'s legacy-heading and indentation heuristics. Symbol links render as
code in HTML and Markdown because a package resolver is not supplied. List
items support continuation text but no nested blocks. Unsupported bracketed
text remains ordinary text. This API has no runtime Go implementation.

## Development and examples

Requires GoML 0.1.56 or newer. The `examples/basic/` example shares the root manifest and its dependencies. From the library root, run:

```sh
goml run --example basic
goml test
goml verify --timeout 300s
```

`goml test` builds the example and runs its tests. `goml verify` repeats the example checks as an independent module against an isolated registry snapshot. `(cd ../verification && just ecosystem-test go_doc)` also retains the library-specific smoke and compatibility checks.
