# Go documentation comments

`ecosystem::go_doc` parses Go documentation-comment text into a small AST and
renders it as canonical comment text, plain text, Markdown, or escaped HTML.
It is separate from GoML documentation and the `gomlgo` frontend. The source
syntax follows the [Go doc comment specification](https://go.dev/doc/comment).

`parse(input, Limits::standard())` returns a `Document` with paragraphs,
single-line `# ` headings, indented code blocks, bullet or numbered lists,
`[Text]: URL` definitions, URL links, and lexical symbol links. Use
`render(document, Format::Html, limits)` or another `Format` for output.
Symbol links require adjacent Unicode punctuation, ASCII spaces/tabs/newlines,
or a text boundary. Letters, numbers, emoji and other symbols next to the brackets
keep the text literal. Defined URL links can appear directly beside other text.
Duplicate link labels resolve to their first definition. Labels are case sensitive,
and `Document.links` retains all definitions in source order.
When looking up a URL link, each newline or tab in its reference label becomes
one ASCII space. Other whitespace and repeated spaces remain significant; the
AST retains the reference label's original text.
Indented code removes the longest common space/tab prefix of its nonblank lines,
following [Go comment code blocks](https://go.dev/doc/comment#code). Relative
indentation and interior blank lines are retained in one code block; trailing
blank lines separate it from later prose or lists. Whitespace-only interior lines
normalize to empty lines. Mixed tab/space prefixes are compared literally rather
than expanded to display columns.

All failures return `Error` with a kind, line and message. Limits bound input
bytes, lines, blocks and link definitions separately, inline nodes, and
output bytes; render checks its budget
even for caller-constructed ASTs. HTML escapes text and URLs; only `http`,
`https`, and `mailto` URLs are accepted as links.

HTML rendering automatically links bare `http://` and `https://` URLs in prose,
headings and list items. Recognition follows Go doc-comment URL heuristics: an
ASCII host, no URL recognition inside identifiers, balanced parentheses/brackets/braces
in paths, and sentence punctuation
excluded at the end. URLs inside code blocks or existing links are not linked
again. Other schemes and bare email addresses remain text. Automatic linking is
an HTML rendering operation; the public AST and Comment/Text/Markdown output keep
the original plain text. Escaping and the output-byte limit apply to generated
anchors, including caller-built documents.

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
note extraction, deprecation semantics, directives, and
`gofmt`'s legacy-heading and indentation heuristics. Symbol links render as
code in HTML and Markdown because a package resolver is not supplied. List
items support continuation text but no nested blocks. Inline links can span
continuation lines within an item, but cannot cross item boundaries. Unsupported bracketed
text remains ordinary text. This API has no runtime Go implementation.

## Development and examples

Requires GoML 0.1.56 or newer. The `examples/basic/` example shares the root manifest and its dependencies. From the library root, run:

```sh
goml run --example basic
goml test
goml verify --timeout 300s
```

`goml test` builds the example and runs its tests. `goml verify` repeats the example checks as an independent module against an isolated registry snapshot. `(cd ../verification && just ecosystem-test go_doc)` also retains the library-specific smoke and compatibility checks.

## Native dependency setup

The HTML dependency includes a managed Go adapter for document parsing and
sanitization. Projects using this module need a module-root `go.mod`, even when
they use only the existing text APIs. A minimal Go module with `go 1.26.0` is
sufficient; GoML generates the adapter requirements and replacements. This
repository includes that manifest. Fetch the declared native Go dependencies
before building with readonly module resolution; ecosystem verification and CI
do this automatically.
