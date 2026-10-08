# 2. The request format

```http
POST /v1/systemone
Authorization: Bearer <key>
Content-Type: application/json
```

```json
{
  "model": "<model name>",
  "state": "the content to judge: a string, object, or array",
  "questions": {
    "<id you choose>": {"type": "choice | score | noul", "instructions": "...", "criteria": "..."}
  }
}
```

- **`state`** is what the model judges. Use an object so each part has a name.
- **`questions`** maps IDs to questions. The model **never sees the IDs**, so the whole question goes in `instructions`.
- Every question sees the same state and is answered **independently**.

## Question types

| Type | `criteria` | Answer |
| - | - | - |
| **Choice** | Object: option $\to$ description (or `null`) | `choice`, `probabilities`, `confidence` |
| **Score** | Array of levels, low $\to$ high | `score`, `probabilities`, `legend`, `confidence` |
| **Noul** | Optional `{"true": ..., "false": ...}` | `noul` = P(yes) |

`instructions` and the descriptions can be strings, objects or arrays. Point at parts of the state with backticked paths such as `` `target.aspect` ``.

## Reading the answers

For a Score with levels $i = 0, \dots, n-1$ and probabilities $p_i$:

$$s = \sum_i i \, p_i$$

So $s$ can land between levels: $s = 1.4$ means split between levels 1 and 2, leaning toward 1.

**Confidence** of a Choice with $n$ options, where $p_{\max}$ is the top probability:

$$c = \frac{p_{\max} - 1/n}{1 - 1/n}$$

That gives $c = 1$ when all the probability sits on one option, and $c = 0$ when it is spread evenly.

**Confidence of a Score** also counts how far the probability sits from the most likely level $m$:

$$c = \max\!\left(0,\; 1 - \frac{\sum_i p_i \,\lvert i - m\rvert}{\tfrac{1}{n}\sum_i \lvert i - \tfrac{n-1}{2}\rvert}\right)$$

**A Noul has no confidence field.** $p$ near 0.5 *is* the uncertainty. If you want a matching number, use $\lvert 2p - 1 \rvert$.

> These formulas are the ones published for the first System One model. Other models may compute `confidence` differently, so check yours.

## Before you trust a response

Check that:

- every question has an answer;
- each `choice` is one of your options;
- $\sum p = 1$ and every value is in $[0, 1]$;
- $0 \le s \le n - 1$.

Treat a response that fails these checks like a timeout. Servers that share the format don't all implement it equally well.

Next: [Building a request step by step](03-building-a-request.md)
