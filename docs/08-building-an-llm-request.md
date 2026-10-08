# 8. Building the same request for an LLM

This page repeats [page 3](03-building-a-request.md) step by step, using an ordinary chat LLM instead of a System One model. It uses the chat-completions format with JSON-schema structured output, which most LLM APIs and open-model servers accept. Comparing the two pages shows what you gain and what you lose.

## Step 1: one yes/no question

```json
{
  "model": "<any chat model>",
  "messages": [
    {
      "role": "user",
      "content": "Cargo ship on our starboard bow. We see her red sidelight. Bearing steady. Must own ship give way to this vessel?"
    }
  ]
}
```

What comes back is prose (illustrative):

```text
Yes. She is on your starboard side in a crossing situation, so under Rule 15 you
are the give-way vessel and should alter course to starboard.
```

Code has to find "yes" in the sentence. The System One version returned $p = P(\text{yes})$ instead, and nothing else.

<details markdown="1">
<summary>Complete request after step 1</summary>

```json
{
  "model": "<any chat model>",
  "messages": [
    {
      "role": "user",
      "content": "Cargo ship on our starboard bow. We see her red sidelight. Bearing steady. Must own ship give way to this vessel?"
    }
  ]
}
```

</details>

## Step 2: give the input structure

An LLM has no separate `state` field. Facts and question both go into the messages, so structure means formatting the text, and a system message sets the role.

```json
"messages": [
  {
    "role": "system",
    "content": "You classify vessel encounters for an autonomous ship. Answer only in the requested format."
  },
  {
    "role": "user",
    "content": "Target:\n- relative_position: on our starboard bow\n- aspect: her port side: we see her red sidelight\n- course_relation: crossing courses\n- bearing_trend: steady or nearly steady\n\nMust own ship give way to this vessel?"
  }
]
```

The model still sees one stream of text. Facts, instructions and the question all compete for its attention.

<details markdown="1">
<summary>Complete request after step 2</summary>

```json
{
  "model": "<any chat model>",
  "messages": [
    {
      "role": "system",
      "content": "You classify vessel encounters for an autonomous ship. Answer only in the requested format."
    },
    {
      "role": "user",
      "content": "Target:\n- relative_position: on our starboard bow\n- aspect: her port side: we see her red sidelight\n- course_relation: crossing courses\n- bearing_trend: steady or nearly steady\n\nMust own ship give way to this vessel?"
    }
  ]
}
```

</details>

## Step 3: ask which situation, constrained to a set

A JSON schema with an `enum` forces the answer into your option set, the LLM counterpart of `criteria`.

```json
"response_format": {
  "type": "json_schema",
  "json_schema": {
    "name": "encounter",
    "strict": true,
    "schema": {
      "type": "object",
      "properties": {
        "encounter_type": {
          "type": "string",
          "enum": [
            "head_on",
            "crossing_give_way",
            "crossing_stand_on",
            "overtaking",
            "being_overtaken",
            "no_encounter"
          ]
        }
      },
      "required": [
        "encounter_type"
      ],
      "additionalProperties": false
    }
  }
}
```

What comes back (illustrative):

```json
{"encounter_type": "crossing_give_way"}
```

That's one label, with **no probabilities**. You can't see the runner-up, or how close the call was.

<details markdown="1">
<summary>Complete request after step 3</summary>

```json
{
  "model": "<any chat model>",
  "messages": [
    {
      "role": "system",
      "content": "You classify vessel encounters for an autonomous ship. Answer only in the requested format."
    },
    {
      "role": "user",
      "content": "Target:\n- relative_position: on our starboard bow\n- aspect: her port side: we see her red sidelight\n- course_relation: crossing courses\n- bearing_trend: steady or nearly steady\n\nWhich COLREGs encounter situation does own ship have with the target?"
    }
  ],
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "encounter",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {
          "encounter_type": {
            "type": "string",
            "enum": [
              "head_on",
              "crossing_give_way",
              "crossing_stand_on",
              "overtaking",
              "being_overtaken",
              "no_encounter"
            ]
          }
        },
        "required": [
          "encounter_type"
        ],
        "additionalProperties": false
      }
    }
  }
}
```

</details>

## Step 4: separate the look-alike options

The descriptions and the assumption move into the system message. The schema can hold only the option names.

```json
"messages": [
  {
    "role": "system",
    "content": "You classify vessel encounters for an autonomous ship. Answer only in the requested format.\nencounter_type options:\n- crossing_give_way: courses cross and the target is on our starboard side, showing us her port side. Not for a target on our port side, or one we are coming up on from astern.\n- crossing_stand_on: courses cross and the target is on our port side, showing us her starboard side. Not for a target on our starboard side, or one coming up on us from astern.\nAssume the vessels are in sight of one another."
  }
]
```

Same idea as page 3, but in prose. Nothing marks which line describes which option, apart from your formatting.

<details markdown="1">
<summary>Complete request after step 4</summary>

```json
{
  "model": "<any chat model>",
  "messages": [
    {
      "role": "system",
      "content": "You classify vessel encounters for an autonomous ship. Answer only in the requested format.\nencounter_type options:\n- crossing_give_way: courses cross and the target is on our starboard side, showing us her port side. Not for a target on our port side, or one we are coming up on from astern.\n- crossing_stand_on: courses cross and the target is on our port side, showing us her starboard side. Not for a target on our starboard side, or one coming up on us from astern.\nAssume the vessels are in sight of one another."
    },
    {
      "role": "user",
      "content": "Target:\n- relative_position: on our starboard bow\n- aspect: her port side: we see her red sidelight\n- course_relation: crossing courses\n- bearing_trend: steady or nearly steady\n\nWhich COLREGs encounter situation does own ship have with the target?"
    }
  ],
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "encounter",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {
          "encounter_type": {
            "type": "string",
            "enum": [
              "head_on",
              "crossing_give_way",
              "crossing_stand_on",
              "overtaking",
              "being_overtaken",
              "no_encounter"
            ]
          }
        },
        "required": [
          "encounter_type"
        ],
        "additionalProperties": false
      }
    }
  }
}
```

</details>

## Step 5: add a risk level

The levels need names for the `enum`, with descriptions in the system message.

```json
"collision_risk": {
  "type": "string",
  "enum": [
    "no_risk",
    "low",
    "developing",
    "risk_exists",
    "immediate_danger"
  ]
}
```

The LLM picks **one** level. A System One Score returns $s = \sum_i i\,p_i$, which can land between two levels and shows when the model is split. Here, "split between `developing` and `risk_exists`" can't be expressed.

<details markdown="1">
<summary>Complete request after step 5</summary>

```json
{
  "model": "<any chat model>",
  "messages": [
    {
      "role": "system",
      "content": "You classify vessel encounters for an autonomous ship. Answer only in the requested format.\nencounter_type options:\n- crossing_give_way: courses cross and the target is on our starboard side, showing us her port side. Not for a target on our port side, or one we are coming up on from astern.\n- crossing_stand_on: courses cross and the target is on our port side, showing us her starboard side. Not for a target on our starboard side, or one coming up on us from astern.\nAssume the vessels are in sight of one another.\ncollision_risk levels:\n- no_risk: the target will pass well clear and nothing is developing\n- low: the target passes clear but should be kept under observation\n- developing: a close-quarters situation will develop if neither vessel acts\n- risk_exists: risk of collision exists; action under the rules is required now\n- immediate_danger: action by the give-way vessel alone can no longer avoid collision"
    },
    {
      "role": "user",
      "content": "Target:\n- relative_position: on our starboard bow\n- aspect: her port side: we see her red sidelight\n- course_relation: crossing courses\n- bearing_trend: steady or nearly steady\n- closest_approach: inside own ship's safety domain\n- time_to_closest_approach: within the planning horizon\n\nWhich COLREGs encounter situation does own ship have with the target, and how much risk of collision exists?"
    }
  ],
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "encounter",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {
          "encounter_type": {
            "type": "string",
            "enum": [
              "head_on",
              "crossing_give_way",
              "crossing_stand_on",
              "overtaking",
              "being_overtaken",
              "no_encounter"
            ]
          },
          "collision_risk": {
            "type": "string",
            "enum": [
              "no_risk",
              "low",
              "developing",
              "risk_exists",
              "immediate_danger"
            ]
          }
        },
        "required": [
          "encounter_type",
          "collision_risk"
        ],
        "additionalProperties": false
      }
    }
  }
}
```

</details>

## Step 6: add context and checks on it

Context goes into the user message; each check becomes a boolean field.

```json
"restricted_visibility": {
  "type": "boolean"
},
"starboard_constrained": {
  "type": "boolean"
}
```

A boolean is **true or false, nothing in between**. A Noul's $p = 0.5$ tells the gate on [page 6](06-using-the-answers.md) "unsure, fall back". Here, an unsure model still has to pick a side.

<details markdown="1">
<summary>Complete request after step 6</summary>

```json
{
  "model": "<any chat model>",
  "messages": [
    {
      "role": "system",
      "content": "You classify vessel encounters for an autonomous ship. Answer only in the requested format.\nencounter_type options:\n- crossing_give_way: courses cross and the target is on our starboard side, showing us her port side. Not for a target on our port side, or one we are coming up on from astern.\n- crossing_stand_on: courses cross and the target is on our port side, showing us her starboard side. Not for a target on our starboard side, or one coming up on us from astern.\nAssume the vessels are in sight of one another.\ncollision_risk levels:\n- no_risk: the target will pass well clear and nothing is developing\n- low: the target passes clear but should be kept under observation\n- developing: a close-quarters situation will develop if neither vessel acts\n- risk_exists: risk of collision exists; action under the rules is required now\n- immediate_danger: action by the give-way vessel alone can no longer avoid collision\nrestricted_visibility: true if the visibility report describes fog, mist, falling snow, heavy rain or similar.\nstarboard_constrained: true if a reported hazard lies on our starboard side, where a starboard alteration would go."
    },
    {
      "role": "user",
      "content": "Target:\n- relative_position: on our starboard bow\n- aspect: her port side: we see her red sidelight\n- course_relation: crossing courses\n- bearing_trend: steady or nearly steady\n- closest_approach: inside own ship's safety domain\n- time_to_closest_approach: within the planning horizon\n\nOwn ship:\n- starboard_side_faces: east\n\nContext:\n- visibility_report: Coastal forecast: wind NW 3, sea slight, visibility good.\n- navigational_warnings: Dredger working on the east side of the fairway until further notice.\n\nAnswer every field in the schema."
    }
  ],
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "encounter",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {
          "encounter_type": {
            "type": "string",
            "enum": [
              "head_on",
              "crossing_give_way",
              "crossing_stand_on",
              "overtaking",
              "being_overtaken",
              "no_encounter"
            ]
          },
          "collision_risk": {
            "type": "string",
            "enum": [
              "no_risk",
              "low",
              "developing",
              "risk_exists",
              "immediate_danger"
            ]
          },
          "restricted_visibility": {
            "type": "boolean"
          },
          "starboard_constrained": {
            "type": "boolean"
          }
        },
        "required": [
          "encounter_type",
          "collision_risk",
          "restricted_visibility",
          "starboard_constrained"
        ],
        "additionalProperties": false
      }
    }
  }
}
```

</details>

## Step 7: make sure the right vessel is judged

The same fix as page 3: name the vessel and say what to ignore. Now it's one more line in an ever longer system message.

```json
"target_ram": {
  "type": "boolean"
},
"target_nuc": {
  "type": "boolean"
}
```

**The answers are no longer independent.** An LLM writes the JSON one token at a time, so `target_ram` is generated *after* `encounter_type` and conditioned on it. An early mistake can carry forward, and reordering the schema fields can change the answers.

<details markdown="1">
<summary>Complete request after step 7</summary>

```json
{
  "model": "<any chat model>",
  "messages": [
    {
      "role": "system",
      "content": "You classify vessel encounters for an autonomous ship. Answer only in the requested format.\nencounter_type options:\n- crossing_give_way: courses cross and the target is on our starboard side, showing us her port side. Not for a target on our port side, or one we are coming up on from astern.\n- crossing_stand_on: courses cross and the target is on our port side, showing us her starboard side. Not for a target on our starboard side, or one coming up on us from astern.\nAssume the vessels are in sight of one another.\ncollision_risk levels:\n- no_risk: the target will pass well clear and nothing is developing\n- low: the target passes clear but should be kept under observation\n- developing: a close-quarters situation will develop if neither vessel acts\n- risk_exists: risk of collision exists; action under the rules is required now\n- immediate_danger: action by the give-way vessel alone can no longer avoid collision\nrestricted_visibility: true if the visibility report describes fog, mist, falling snow, heavy rain or similar.\nstarboard_constrained: true if a reported hazard lies on our starboard side, where a starboard alteration would go.\ntarget_ram: true if the target itself is restricted in her ability to manoeuvre by the nature of her work. A vessel that is only large, slow or deep-laden is not.\ntarget_nuc: true if the target itself is not under command, for example after engine or steering failure.\nJudge only the target vessel for target_ram and target_nuc, not other vessels mentioned in the context."
    },
    {
      "role": "user",
      "content": "Target:\n- relative_position: on our starboard bow\n- aspect: her port side: we see her red sidelight\n- course_relation: crossing courses\n- bearing_trend: steady or nearly steady\n- closest_approach: inside own ship's safety domain\n- time_to_closest_approach: within the planning horizon\n\nOwn ship:\n- starboard_side_faces: east\n\nContext:\n- visibility_report: Coastal forecast: wind NW 3, sea slight, visibility good.\n- navigational_warnings: Dredger working on the east side of the fairway until further notice.\n\nAnswer every field in the schema."
    }
  ],
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "encounter",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {
          "encounter_type": {
            "type": "string",
            "enum": [
              "head_on",
              "crossing_give_way",
              "crossing_stand_on",
              "overtaking",
              "being_overtaken",
              "no_encounter"
            ]
          },
          "collision_risk": {
            "type": "string",
            "enum": [
              "no_risk",
              "low",
              "developing",
              "risk_exists",
              "immediate_danger"
            ]
          },
          "restricted_visibility": {
            "type": "boolean"
          },
          "starboard_constrained": {
            "type": "boolean"
          },
          "target_ram": {
            "type": "boolean"
          },
          "target_nuc": {
            "type": "boolean"
          }
        },
        "required": [
          "encounter_type",
          "collision_risk",
          "restricted_visibility",
          "starboard_constrained",
          "target_ram",
          "target_nuc"
        ],
        "additionalProperties": false
      }
    }
  }
}
```

</details>

## Step 8: ask for a manoeuvre class

```json
"recommended_action": {
  "type": "string",
  "enum": [
    "maintain_course_and_speed",
    "alter_course_to_starboard",
    "reduce_speed",
    "alter_to_starboard_and_reduce_speed",
    "stop_or_reverse",
    "alter_course_to_port"
  ]
}
```

What comes back (illustrative):

```json
{
  "encounter_type": "crossing_give_way",
  "collision_risk": "risk_exists",
  "restricted_visibility": false,
  "starboard_constrained": true,
  "target_ram": false,
  "target_nuc": false,
  "recommended_action": "reduce_speed"
}
```

That's seven answers and no probabilities.

<details markdown="1">
<summary>Complete request after step 8</summary>

```json
{
  "model": "<any chat model>",
  "messages": [
    {
      "role": "system",
      "content": "You classify vessel encounters for an autonomous ship. Answer only in the requested format.\nencounter_type options:\n- crossing_give_way: courses cross and the target is on our starboard side, showing us her port side. Not for a target on our port side, or one we are coming up on from astern.\n- crossing_stand_on: courses cross and the target is on our port side, showing us her starboard side. Not for a target on our starboard side, or one coming up on us from astern.\nAssume the vessels are in sight of one another.\ncollision_risk levels:\n- no_risk: the target will pass well clear and nothing is developing\n- low: the target passes clear but should be kept under observation\n- developing: a close-quarters situation will develop if neither vessel acts\n- risk_exists: risk of collision exists; action under the rules is required now\n- immediate_danger: action by the give-way vessel alone can no longer avoid collision\nrestricted_visibility: true if the visibility report describes fog, mist, falling snow, heavy rain or similar.\nstarboard_constrained: true if a reported hazard lies on our starboard side, where a starboard alteration would go.\ntarget_ram: true if the target itself is restricted in her ability to manoeuvre by the nature of her work. A vessel that is only large, slow or deep-laden is not.\ntarget_nuc: true if the target itself is not under command, for example after engine or steering failure.\nJudge only the target vessel for target_ram and target_nuc, not other vessels mentioned in the context.\nrecommended_action: the class of manoeuvre own ship should take under the COLREGs. Choose the class only; how much to alter or slow is decided elsewhere."
    },
    {
      "role": "user",
      "content": "Target:\n- relative_position: on our starboard bow\n- aspect: her port side: we see her red sidelight\n- course_relation: crossing courses\n- bearing_trend: steady or nearly steady\n- closest_approach: inside own ship's safety domain\n- time_to_closest_approach: within the planning horizon\n\nOwn ship:\n- starboard_side_faces: east\n\nContext:\n- visibility_report: Coastal forecast: wind NW 3, sea slight, visibility good.\n- navigational_warnings: Dredger working on the east side of the fairway until further notice.\n\nAnswer every field in the schema."
    }
  ],
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "encounter",
      "strict": true,
      "schema": {
        "type": "object",
        "properties": {
          "encounter_type": {
            "type": "string",
            "enum": [
              "head_on",
              "crossing_give_way",
              "crossing_stand_on",
              "overtaking",
              "being_overtaken",
              "no_encounter"
            ]
          },
          "collision_risk": {
            "type": "string",
            "enum": [
              "no_risk",
              "low",
              "developing",
              "risk_exists",
              "immediate_danger"
            ]
          },
          "restricted_visibility": {
            "type": "boolean"
          },
          "starboard_constrained": {
            "type": "boolean"
          },
          "target_ram": {
            "type": "boolean"
          },
          "target_nuc": {
            "type": "boolean"
          },
          "recommended_action": {
            "type": "string",
            "enum": [
              "maintain_course_and_speed",
              "alter_course_to_starboard",
              "reduce_speed",
              "alter_to_starboard_and_reduce_speed",
              "stop_or_reverse",
              "alter_course_to_port"
            ]
          }
        },
        "required": [
          "encounter_type",
          "collision_risk",
          "restricted_visibility",
          "starboard_constrained",
          "target_ram",
          "target_nuc",
          "recommended_action"
        ],
        "additionalProperties": false
      }
    }
  }
}
```

</details>

## What you have now

| | LLM request (this page) | System One request ([page 3](03-building-a-request.md)) |
| - | - | - |
| Facts and question | Mixed in the messages | `state` and `questions` kept apart |
| Option descriptions | Prose in the system message | `criteria`, one per option |
| Answer | One generated JSON object | One typed answer per question |
| Uncertainty | None | `probabilities`, `confidence`, $p$ |
| Answers depend on each other | Yes, through generation order | No, each is independent |
| Unsure yes/no | Forced to `true` or `false` | $p$ near 0.5 |
| Level between two levels | Not possible | $s$ between levels |

The gate on [page 6](06-using-the-answers.md) needs probabilities, and this request has none. The usual workarounds:

- **Sample $k$ times** at a temperature above 0, and use the vote share of the top label as $p_{\max}$. That costs $k$ calls per encounter.
- **Read token log-probabilities**, where the API exposes them, for the first token of each enum value. This is fragile: labels that share a first token can't be told apart.
- **Add a `confidence` field.** Don't. The model writes it like any other text, so it isn't a measured probability.

An LLM is still the right tool for what a person reads, such as the operator's alert text ([page 1](01-system-one-vs-llms.md)).

Back to the [index](index.md).
