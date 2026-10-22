# json2types

**Paste a few sample JSON payloads. Get clean TypeScript interfaces and Python Pydantic models, merged across every sample.**

Most type converters look at one payload and guess. Real APIs are messier than that: the field that is a number today is a string tomorrow, and half your responses are missing the email. json2types infers from *several* samples at once, so the types it emits are the honest ones: optionals where fields go missing, unions where types disagree.

```bash
# No install. No build. No server.
open index.html
```

Everything runs locally in one HTML file. Nothing you paste leaves your browser, which is more than the converter sites can say.

## Why this exists

Type generation from JSON is an entire economy of ad-funded converter sites, and the serious tool in the space, quicktype, is powerful but widely described as overkill for small structures, and it ships runtime converters by default when most people just want the types. json2types is the unoccupied middle: tiny, multi-sample, two languages, fully offline, and the output is only the types, nothing else.

## Features

- **Multi-sample inference.** Add as many samples as you like, or paste several payloads into one box separated by a line containing only `---`.
- **Two outputs, one click.** TypeScript interfaces and Python Pydantic models, side by side, each with a copy button.
- **Optional detection.** A field missing from any sample at its level becomes optional (`?` in TypeScript, `Optional[...] = None` in Python).
- **Union detection.** A field whose type differs between samples becomes a union (`string | number` in TypeScript, `Union[int, str]` in Python).
- **Named nested types.** Nested objects become their own named interfaces and classes, derived from the field name in PascalCase, with automatic suffixes on collisions.
- **int vs float.** Whole numbers stay `int` in Python; a single float anywhere makes the field `float`. Both map to `number` in TypeScript, as they should.
- **Array merging.** Element types are merged across every item in every sample. Mixed arrays become unions; empty arrays fall back to `unknown[]` / `List[Any]`.
- **Python-safe field names.** Keys that are not valid Python identifiers are sanitized and keep the original key as a Pydantic `Field` alias.
- **An inference report, not a black box.** Every optional field, union, and date-like string is listed with the reason it was inferred that way.
- **Errors that name the sample.** Invalid JSON reports which sample failed and what the parser said, instead of a generic failure.
- **Zero dependencies, zero network.** One HTML file. Google Fonts is the only external request, and the tool works without it.

## Inference rules, precisely

1. Samples are parsed in order. Each sample may itself contain several payloads separated by a `---` line.
2. At every node, values from all samples (and, for arrays, all items) are grouped by kind: `string`, `number`, `boolean`, `null`, `array`, `object`.
3. Integers and floats are one group, `number`, but the group remembers whether every value was whole. All whole: Python `int`. Any float: Python `float`. TypeScript is always `number`.
4. One kind at a node: that kind wins. More than one kind: a union of the per-kind schemas, each built from only the values of that kind.
5. For objects, a field's occurrence count is compared with the number of objects at that level. Missing anywhere means optional.
6. `null` inside a union becomes part of the union in TypeScript (`string | null`) and an `Optional[...]` wrapper in Python.
7. Strings matching an ISO date or datetime shape are reported as date-like. They are emitted as `string` (with a comment) and `str`, never silently converted.
8. Empty arrays contribute an `unknown` element type. An array empty in one sample and populated in another takes the populated element type.

## Demo

Hit **Load demo samples** in the tool. Three user payloads demonstrate everything at once: one is missing `email` (optional), `id` is a number in two and a string in one (union), and `address` is nested (its own named type in both outputs).

## Honest limits

- Inference is only as good as your samples. A field that is always a string in your samples will be typed as a string even if the API sends a number on the fifth Tuesday of the month.
- No enum detection in v1: a field that only ever holds `"admin"` or `"user"` is typed as `string`, not a literal union.
- Date-like strings stay strings by design. Converting them to `Date` is a modeling decision the tool refuses to make for you.
- Root payloads that are not objects get a type alias instead of an interface, which is the correct shape but surprises people the first time.
- Python output targets Pydantic (v1 and v2 compatible for the constructs emitted). Dataclass output is on the roadmap, not in v1.

## Roadmap

- Dataclass and attrs output toggle
- Zod schemas and Go structs
- Enum detection from low-cardinality string fields
- JSON Schema output
- Drag and drop `.json` files onto the samples panel

## License

MIT. See [LICENSE](LICENSE).

---

Built by Amir Hussain. Part of a small family of sharp, single-purpose tools that respect your data.
