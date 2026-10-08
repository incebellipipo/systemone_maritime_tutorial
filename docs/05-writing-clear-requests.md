# 5. Writing clear requests

**Test for any question:** could a competent watch officer answer it in a second, given this state? If not, split it up or move it into code.

| # | Before | Problem | After |
| - | - | - | - |
| 1 | "Is it safe to keep our course?" | Hides four judgements in one number | Separate questions: encounter type, collision risk, starboard room |
| 2 | "Is `target.cpa_nm` below 0.5?" | Arithmetic | Code compares; state says "inside own ship's safety domain" |
| 3 | Score levels `["0","1","2","3"]` or low/medium/high | Nothing for the model to match against | Levels that describe situations: "Risk of collision exists: action is required now" |
| 4 | "Is the vessel restricted in her ability to manoeuvre?" | Which vessel? The state also mentions a dredger | "Is `target` itself ...", plus "judge only `target`" |
| 5 | "Is visibility not restricted and the target not under command?" | Two conditions, both negated | Two Nouls, each phrased so that yes means the condition holds |
| 6 | `crossing_give_way: null`, `crossing_stand_on: null` | Look-alike options | `what` / `not_for` descriptions |
| 7 | Encounter types without a way out | Forces a label onto a target that will pass well clear | Add `no_encounter` |
| 8 | "Which COLREGs situation is this?" | Unstated assumption; extra inference step for "east side" | `"assume": "in sight of one another"`; state says `starboard_side_faces: east` |

## Did a rewrite help?

Higher confidence doesn't prove the rewrite helped. A question can make the model more decisive without making it more right. Instead:

1. **Label a set of encounters**: simulator runs, plus recorded traffic reviewed by navigators. Include the awkward cases.
2. **Compare accuracy** before and after, overall and within each confidence band. Use encounters you didn't look at while writing the rewrite.
3. **Reverse the option order.** Some models lean toward the first option; the answer shouldn't change.
4. **Paraphrase the state.** "Starboard bow" and "fine on the starboard bow" should get the same answer.
5. **Try a second model.** If a rewrite helps only one model, it may be exploiting that model's quirk rather than making the question clearer.

Next: [Using the answers](06-using-the-answers.md)
