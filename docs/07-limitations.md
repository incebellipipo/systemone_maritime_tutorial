# 7. Limitations and safety

This is **decision support inside a safety-bounded architecture**, not a certified collision-avoidance system. A real deployment needs a safety case, verification and validation, and approval from the flag state and classification society.

**The key property:** everything below the model works without it. If the model is slow, unreachable, wrong or switched off, the vessel uses its rule-based fallback.

## Model limits

- **Confident mistakes pass the gate.** Confidence measures how decisive the model is, not whether it's right.
- **Calibration may not carry over to your data.** Measure it on your own encounters, for each model and each version.
- **No arithmetic, counting or time comparisons.** Keep all of them in code.
- **A Score is a position, not a measurement.** Don't interpolate between its levels.
- **Instructions are read literally, and Choice option order can bias the answer.**
- **State text can be adversarial.** AIS fields, spoofed messages and VHF transcripts are all written by someone else.
- **Most models are strongest in English and accept only text.**

## Format and setting

- **One format, many behaviours.** Servers that share the format differ in which question types they support, their limits and their calibration. Validate every response.
- **Model versions change.** Pin a version, log the `model` field of each response, and re-tune thresholds when you switch.
- **A hosted model needs a link**, and at sea that may be slow or absent. A timeout means fall back, never wait. An open-weight model on board removes the link but brings hardware and update management of its own.

## Simplifications here

- **The geometry check covers only Rules 13-15.** It ignores narrow channels and traffic separation schemes, other vessel categories, action by the stand-on vessel, and encounters with several vessels at once.
- **Relative bearing uses course over ground instead of heading.**
- **All thresholds, buckets and weights are placeholders.**

Next: [Building the same request for an LLM](08-building-an-llm-request.md)
