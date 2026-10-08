# 6. Using the answers

## One request, read selectively

All seven questions go in one request. Code reads only what its path needs:

| Answer | Read when |
| - | - |
| `restricted_visibility` | Always, first |
| `encounter_type`, `recommended_action`, `collision_risk` | Always |
| `target_ram`, `target_nuc` | Encounter is `crossing_stand_on` |
| `starboard_constrained` | The action turns to starboard |

With several targets, send **one request per target**, all at the same time.

## The gate

Notation:

- Each Noul probability $p$ falls into a **band**: *yes* if $p \ge 0.8$, *no* if $p \le 0.2$, and *unsure* otherwise.
- The decision is only as certain as its weakest answer: $c_{\min} = \min(c_{\text{encounter}},\ c_{\text{action}},\ c_{\text{risk}})$.

Apply these checks in order. The first one that fails decides the outcome:

| # | Check | If it fails |
| - | - | - |
| 0 | A valid response arrived in time ([page 2](02-request-format.md#before-you-trust-a-response)) | Fall back, request takeover |
| 1 | $p_{\text{visibility}}$ is *no* | Fall back, advisory (Rule 19 applies, not Rules 13-17) |
| 2 | `encounter_type` agrees with the geometry: stern sector and closing $\to$ overtaking; on our starboard side $\to$ crossing, give way; and so on | Fall back, request takeover |
| 3 | Rule 18: stand-on, and $\max(p_{\text{RAM}}, p_{\text{NUC}})$ is *yes* $\to$ **become give-way** | (changes the role, then continues) |
| 4 | $c_{\min} \ge 0.5$ | Fall back, request takeover |
| 5 | The action is allowed for the role: give-way $\to$ starboard / slow / stop; stand-on $\to$ maintain | Fall back, request takeover |
| 6 | For a starboard action, $p_{\text{starboard}}$ is *no* | Fall back, request takeover |
| OK | $c_{\min} \ge 0.8$ and no warnings | **Act**: send the action class to the planner |
| OK | Otherwise | **Act**, and send an advisory to the operator |

What this gate guarantees:

- **"Alter course to port" never passes.** It isn't allowed for any role, so it always goes to the operator.
- **"Act" means the planner gets the class.** It sizes the manoeuvre, checks the predicted CPA, and can still refuse.
- **The fallback is your rule-based collision avoidance**: whatever the vessel would do with no model. (A placeholder: reduce speed if give-way, otherwise stand on.)

## Ranking several targets

The operator should see the most pressing target first. Combine the atomic answers with weights kept in code:

$$P = 0.5\,\frac{s_{\text{risk}}}{4} + 0.3\,u + 0.1\,\max(p_{\text{RAM}}, p_{\text{NUC}}) + 0.1\,p_{\text{starboard}}, \qquad u = \max\!\left(0,\ 1 - \frac{t^*}{30\ \text{min}}\right)$$

Urgency $u$ is pure arithmetic on TCPA, so the model is never asked for it.

For the example encounter, $t^* = 15.3$ min gives $u \approx 0.49$. Add **hypothetical** answers, chosen to make the arithmetic easy: $s = 3$, both $p_{\text{RAM}}$ and $p_{\text{NUC}} = 0$, $p_{\text{starboard}} = 1$.

$$P = 0.375 + 0.147 + 0 + 0.1 \approx 0.62$$

$P$ is only good for ranking targets against each other. If the order looks wrong, change the weights, not the questions.

## The worked encounter

The geometry is from [page 4](04-encoding-state.md): CPA 0.36 nm, TCPA 15.3 min. The target is on our starboard bow showing her red sidelight, so we give way. The textbook move is an alteration to starboard, but the dredger warning puts a hazard on that side.

| If the model answers... | Outcome |
| - | - |
| Crossing, give-way, with high confidence; a starboard action; $p_{\text{starboard}}$ *no* | Act: starboard action goes to the planner |
| The same, but $p_{\text{starboard}}$ *yes* or *unsure* | Fall back to reduce speed; request takeover |
| The same as the first row, but the action is reduce speed | Act: reduce speed |
| $c_{\min} < 0.5$, disagreement with geometry, or no valid response | Fall back; request takeover |

**The gate's limit:** if the model misreads the warning *confidently* ($p_{\text{starboard}}$ *no*), the starboard action reaches the planner. The gate catches uncertainty, not confident mistakes. That's why the planner must also check the manoeuvre against its own hazard data.

Next: [Limitations and safety](07-limitations.md)
