# 1. System One vs LLMs

Ask both kinds of model the same question: *a cargo ship is on our starboard bow, showing her red sidelight, on a steady bearing. Which COLREGs situation is this?*

**An LLM writes an answer** (illustrative):

```text
This is a crossing situation under Rule 15. You are the give-way vessel and
should alter course to starboard to pass astern of her.
```

That's fine for a person to read. Code, though, has to dig the label out of the sentence, can't tell how sure the model was, and gets advice nobody asked for. With structured output you can get `{"encounter_type": "crossing_give_way", "confidence": 0.95}`, but that `confidence` is a number the model *wrote*, not a measured probability.

**A System One model picks from your options** (illustrative):

```json
{
  "choice": "crossing_give_way",
  "probabilities": {"head_on": "p1", "crossing_give_way": "p2", "crossing_stand_on": "p3", "...": "..."},
  "confidence": "c"
}
```

- The `choice` is always one of the options you supplied.
- Every option gets a probability, and they sum to 1, so you can see the runner-up.
- `confidence` is computed from those probabilities, not written by the model.

| | LLM | System One |
| - | - | - |
| Output | Text | Typed values |
| Answer space | Open | Exactly your options |
| Uncertainty | Missing, or written as text | A probability for every option |
| Calibration | Not a training goal | A training goal |
| Explanations | Yes | No |
| Code that uses it | Parses and validates | Branches and thresholds |

**Calibrated** means that, over many answers given at 80%, about 80% are right. That's a claim about averages on some set of inputs. Measure it on your own encounters.

**Use an LLM** wherever a person reads the output, such as the operator's alert text or a voyage summary. **Use System One** wherever code branches on the answer.

Next: [The request format](02-request-format.md)
