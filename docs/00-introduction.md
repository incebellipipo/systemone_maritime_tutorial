# 0. What is a System One model?

A **System One model** is an AI decision model. You send it some content (the **state**) and a set of **typed questions**, and it returns a **typed answer with probabilities** for each one. It doesn't generate text.

The name comes from Daniel Kahneman's *Thinking, Fast and Slow* (2011). **System 1** is fast, intuitive judgement; **System 2** is slow, deliberate reasoning. A System One model is built for the first kind: quick, focused judgements that software can branch on.

## Key facts

- **Three question types.** A *Choice* picks one option from a set. A *Score* places the state on ordered levels. A *Noul* gives the probability that a statement is true.
- **Answers stay inside your options.** A Choice returns one of the options you supplied, never something else, so there is no free text to parse or to hallucinate.
- **A probability for every option.** Choice and Score answers also carry a `confidence` value computed from those probabilities ([page 2](02-request-format.md)).
- **Calibrated by design.** The models are trained so that, over many answers given at 80%, about 80% are right. This holds on average, not for every single answer.
- **Questions are answered independently and in parallel.** Many questions about the same state go in one request.
- **Fast and cheap.** For Jev, the documentation says most queries complete in about 100 ms, and pricing is per input token, with output tokens free.
- **No explanations, no writing.** Drafting text, summarizing and open-ended reasoning stay with LLMs.

Limits differ by model. As one example, Jev 1.13 documents up to 255 Choice options, 2-10 Score levels, a 64k-token context, and text input only, with English as its strongest language.

## History

| Date | Event |
| - | - |
| 2011 | Kahneman's *Thinking, Fast and Slow* popularizes "System 1" and "System 2". |
| 15 Sep 2026 | TypeSafe AI announces **Jev**, the first System One model, as a hosted API at `POST /v1/systemone`. |
| Sep 2026 | **Kev** (Apache 2.0, 0.8B-27B, built on Qwen) reimplements Jev's API as open weights. |
| 23 Sep 2026 | **CLM** (8B, Apache 2.0) is released with a compatible API. |
| 24 Sep 2026 | Fastino releases **GLiNER2.5-Decide** (340M). |
| 29-30 Sep 2026 | Liquid AI's **d1** and Inception's **Mercury Decide** become available free. |
| 1 Oct 2026 | Cloudflare launches **Clef** (27B and 9B, open weights under Apache 2.0, text and images). |

## The landscape today

Since Jev, the `/v1/systemone` request format has become a de facto standard. Three kinds of model speak it:

| Kind | Examples |
| - | - |
| Hosted APIs | Jev, Clef, Mercury Decide, d1 |
| Open weights, run on your own hardware | Kev, Laya, CLM |
| Wrappers that serve the format from an ordinary LLM | AnyJev, simple-jev |

Not everyone follows it. Perplexity's decider model uses its own `/v1/decisions` format. Some models answer only Choice questions (Together AI's Tev1, OpenAI's Decisions API in early access), and some only Noul questions. Even among compatible servers, calibration and response details differ, so evaluate each model on your own data.

> **Sources and caveats:** The model facts come from the [Jev documentation](https://docs.typesafe.ai/llms.txt). The history and landscape come from community directories ([systemonemodels.org](https://systemonemodels.org/), [awesome-system-one](https://github.com/SamratDuttaOfficial/awesome-system-one-1)) as of October 2026. The field moves fast; check each model's own documentation before relying on a detail.

Next: [System One vs LLMs](01-system-one-vs-llms.md)
