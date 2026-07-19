<picture><source media="(prefers-color-scheme: dark)" srcset="banner.svg"><img src="banner-light.svg" alt="The right format does not exist" width="100%"></picture>

# The right format doesn't exist (it depends who's reading)

[Français](README.fr.md) · **English**

> Markdown, JSON, XML, YAML, which one is best for talking to an LLM? The question keeps coming back, and it is badly framed in the same way as the one about language. There is no optimal format, there is a relation: who reads (a human, the model, some code), for which task (reasoning, extracting), with which model. Moving the problem means giving up the search for a universal winner and looking at the two moments separately, what you hand to the model and what it gives back.

<img src="monde.jpg" alt="The right format doesn't exist" width="100%">

Companion to the essay [Language doesn't exist (for an LLM)](https://github.com/ismael-joffroy-chandoutis/la-langue-n-existe-pas). Same logic, another scale.

## In short

- No universal format. On the same reasoning task, GPT-4 does better in Markdown than in JSON, and GPT-3.5 does the opposite. Up to a 40% gap on format alone.
- You judge on four axes that never point to the same format: token cost, interpretation by the model, human readability, reliable machine parsing.
- The real dividing line is input versus output. On input you optimize for the model's interpretation and for context. On output you optimize for whoever consumes it: a human or some code.
- For Claude, in one sentence: Markdown plus XML tags on input, Markdown on human output, JSON on machine output.

---

## 1. The wrong question

Intuition looks for an absolute ranking: "JSON is structured therefore reliable", "Markdown is light therefore efficient". Both are true and false depending on context, exactly as with language.

The clearest study on the subject (Microsoft, *Does Prompt Formatting Have Any Impact on LLM Performance?*, [arXiv 2411.10541](https://arxiv.org/html/2411.10541v1)) measures the same reasoning prompt under several formats. GPT-4 scores 81.2% in Markdown against 73.9% in JSON. On the same task, GPT-3.5 reverses the ranking: JSON at 59.7%, Markdown at 50.0%. The gap between formats can reach 40% on smaller models, and larger models are more robust but never insensitive. The study's conclusion: no universally superior format, the effect depends on the model and the task.

The right question is therefore not "which format", it is "which format, for whom, and at which moment".

## 2. Token cost

First axis, the most measurable. For the same content, the ranking from lightest to heaviest is robust: **Markdown, then YAML, then JSON, then XML**. JSON pays for every `{`, `}`, `"` and `:`; XML pays for every opening and closing tag. Independent benchmarks give consistent orders of magnitude: XML costs about 80% more tokens than Markdown for the same data, and an object rendered in formatted JSON weighs clearly more than the same one in YAML ([improvingagents](https://www.improvingagents.com/blog/best-nested-data-format), [ShShell](https://shshell.com/blog/token-efficiency-module-4-lesson-4-efficient-formatting)). Keep the ranking, not the decimal.

For tabular or uniform data, CSV is unbeatable on size, and an emerging format, TOON (Token-Oriented Object Notation), serializes uniform sets with 30 to 60% fewer tokens than JSON while keeping a structure the model can read ([TOON spec and benchmarks](https://github.com/toon-format/toon)).

## 3. Input: Markdown plus XML tags

On input, two goals: that the model interprets well, and that it does not devour the context. For Claude, the winning combination is not a single format, it is **Markdown for content and XML tags for structure**.

Anthropic explicitly recommends XML tags as delimiters ([prompting documentation](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)). The reason is mechanical: wrapping a 10,000-token document in `<document>...</document>` costs only a few tokens and cleanly isolates that block from the rest for the attention mechanism. So you write your instructions and your prose in Markdown, light and massively seen during training, and you bound your sections and documents with XML tags (`<context>`, `<examples>`, `<input>`). This is one of the rare places where adding a few tokens (the tags) buys you reliability.

For tabular data injected on input, switching to CSV or TOON saves context directly with no loss of understanding.

## 4. Output: whoever consumes it decides

On output, the criterion shifts to the recipient.

For a human, Markdown without hesitation: light, rendered everywhere, natural.

For downstream code or tool use, JSON, despite its cost. But with a measured trap. The study *Let Me Speak Freely?* ([arXiv 2408.02442](https://arxiv.org/abs/2408.02442), EMNLP 2024) shows that forcing a strict structured format (JSON mode, constrained decoding) **degrades reasoning**, while improving classification tasks. The typical mechanism: the model puts the answer before the reasoning because the schema forces it to. The countermeasure: let the model reason in prose first, then emit the JSON at the end. And a counterintuitive but sourced result: asking for plain JSON often gives better generation accuracy than imposing a strict grammar ([TOON vs JSON benchmark, arXiv 2603.03306](https://arxiv.org/abs/2603.03306)).

YAML on generated output is to be avoided: compact but its indentation is fragile, a single misplaced space breaks the parsing. XML is robust and good when the output mixes prose and nested structure, at the cost of token count.

HTML, for its part, is never a format for exchange with the model. It is a final rendering format for the human eye, heavy on tags, with no reasoning gain. You generate Markdown, you convert it to HTML at the very end. Do not ask a model to think in HTML.

## 5. The matrix

| Format | Token cost | LLM interpretation | Human editing | Machine parsing | Ideal use |
|---|---|---|---|---|---|
| Markdown | lightest | good on recent models | excellent | weak (ambiguous) | instructions on input, human output |
| XML (tags) | heavy | good (delimiting for Claude) | medium | robust | structuring and bounding sections and documents on input |
| JSON | verbose | drops if forced strict | medium | excellent | machine output, tool use, API |
| YAML | compact | good | good | fragile (indentation) | hand-written config, not generated output |
| CSV / TOON | very light (uniform data) | good for tabular reading | good | correct | tables and uniform data on input |
| HTML | very heavy | weak | bad raw, perfect rendered | medium | final human rendering only |

## 6. Conclusion

The right format is not a property of the format. It is a function of three variables: who reads (human, model, parser), for which task (reasoning, extracting), with which model. It is the same lesson as with language, moved up one notch. We thought we were comparing formats, we were in fact comparing reading situations.

For Claude, the rule fits in one line: Markdown plus XML tags on input, Markdown on output for the human, JSON on output for the machine, and the tabular in CSV or TOON when volume matters. The rest is folklore of absolute ranking, and absolute ranking doesn't exist.

---

## Sources

- Does Prompt Formatting Have Any Impact on LLM Performance? (Microsoft): https://arxiv.org/html/2411.10541v1
- Let Me Speak Freely? A Study on the Impact of Format Restrictions (EMNLP 2024): https://arxiv.org/abs/2408.02442
- Token-Oriented Object Notation vs JSON, benchmark: https://arxiv.org/abs/2603.03306
- TOON, spec and benchmarks: https://github.com/toon-format/toon
- TOON, InfoQ overview: https://www.infoq.com/news/2025/11/toon-reduce-llm-cost-tokens/
- Anthropic, prompting best practices: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Which nested data format do LLMs understand best: https://www.improvingagents.com/blog/best-nested-data-format
- Token-efficient formatting, Markdown vs XML vs JSON: https://shshell.com/blog/token-efficiency-module-4-lesson-4-efficient-formatting

## License

These texts are published under the [Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/) license. See [LICENSE](LICENSE).

By [Ismaël Joffroy Chandoutis](https://ismaeljoffroychandoutis.com).
