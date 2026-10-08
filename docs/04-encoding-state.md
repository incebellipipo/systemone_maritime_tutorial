# 4. Encoding state

**Code computes the geometry. The model sees words, not numbers.** System One models judge well and calculate badly, so comparing 0.36 nm with 0.5 nm, or working out TCPA, never goes to the model.

## The geometry

Work in a local flat frame: east and north, in nautical miles. A vessel with course $\chi$ and speed $V$ has velocity

$$\mathbf{v} = V(\sin\chi,\ \cos\chi).$$

For own ship $o$ and target $t$, the relative position and relative velocity are

$$\mathbf{r} = \mathbf{p}_t - \mathbf{p}_o, \qquad \mathbf{v} = \mathbf{v}_t - \mathbf{v}_o.$$

From these:

| Quantity | Formula |
| - | - |
| Range | $R = \lVert\mathbf{r}\rVert$ |
| True bearing | $\beta = \operatorname{atan2}(r_E, r_N)$ |
| Relative bearing ($0^\circ$ = dead ahead) | $\beta_{rel} = (\beta - \chi_o) \bmod 360^\circ$ |
| Aspect (where own ship sits, seen from the target's bow) | $\alpha = (\beta + 180^\circ - \chi_t) \bmod 360^\circ$ |
| Time to CPA | $t^* = -\dfrac{\mathbf{r}\cdot\mathbf{v}}{\lVert\mathbf{v}\rVert^2}$ |
| Distance at CPA | $d = \lVert\mathbf{r} + \mathbf{v}\,\max(t^*, 0)\rVert$ |
| Bearing rate | $\dot\beta = \dfrac{r_N v_E - r_E v_N}{R^2}$ |

If $t^* < 0$, the closest approach has already passed.

## Worked example

Own ship: $\chi_o = 000^\circ$, $V_o = 12$ kn. The target is 4 nm away bearing $045^\circ$, with $\chi_t = 270^\circ$ and $V_t = 10$ kn.

$$\mathbf{r} = (2.83,\ 2.83), \quad \mathbf{v} = (-10,\ 0) - (0,\ 12) = (-10,\ -12)$$

$$t^* = -\frac{2.83(-10) + 2.83(-12)}{100 + 144} = 0.255\ \text{h} = 15.3\ \text{min}$$

$$d = \lVert (2.83 - 2.55,\ 2.83 - 3.06) \rVert = \lVert (0.28,\ -0.23) \rVert = 0.36\ \text{nm}$$

$$\beta_{rel} = 045^\circ, \quad \alpha = (45 + 180 - 270) \bmod 360 = 315^\circ, \quad \dot\beta \approx 0.34\ ^\circ/\text{min}$$

## From numbers to words

Thresholds are placeholders; your safety case sets the real ones.

| Field | Rule | Value here |
| - | - | - |
| `relative_position` | $\beta_{rel}$ in $6^\circ$ to $67.5^\circ$ $\to$ "on our starboard bow" (and so on round the compass) | on our starboard bow |
| `aspect` | $\alpha$ in $247.5^\circ$ to $354^\circ$ $\to$ "her port side: red sidelight" | red sidelight |
| `course_relation` | $\chi_t - \chi_o$ within $174^\circ$ to $186^\circ$ $\to$ reciprocal; within $\pm 22.5^\circ$ $\to$ same direction; otherwise crossing | crossing courses |
| `bearing_trend` | $\lvert\dot\beta\rvert < 0.5\ ^\circ/\text{min}$ $\to$ steady | steady or nearly steady |
| `closest_approach` | $d < 0.5$ $\to$ inside domain; $< 1$ $\to$ close; $< 2$ $\to$ clear but watched; else well clear | inside own ship's safety domain |
| `time_to_closest_approach` | $t^* < 6$ $\to$ imminent; $< 12$ $\to$ soon; $< 30$ min $\to$ within horizon | within the planning horizon |
| `starboard_side_faces` | compass point nearest $\chi_o + 90^\circ$ | east |

## Rules for the state

- **One target per request.** Unrelated content distracts the model.
- **No raw numbers or timestamps.** Pass the bucket, and keep the number in code.
- **Convert frames in code.** "Starboard side faces east" saves the model an inference step.
- **Leave out AIS free text** such as name and destination. Anyone can write anything there.
- **Skip the request entirely** when $d \ge 2$ nm or $t^* < 0$. There is nothing to decide.

The resulting state is the `state` object in the [complete request on page 3](03-building-a-request.md#step-8-ask-for-a-manoeuvre-class).

Next: [Writing clear requests](05-writing-clear-requests.md)
