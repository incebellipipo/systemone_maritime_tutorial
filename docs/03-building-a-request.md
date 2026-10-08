# 3. Building a request step by step

This page starts with the simplest request that works and improves it one step at a time. Each step shows only what changed and why. The finished request is at the end.

## Step 1: one yes/no question

```json
{
  "state": "Cargo ship on our starboard bow. We see her red sidelight. Bearing steady.",
  "questions": {
    "give_way": {"type": "noul", "instructions": "Must own ship give way to this vessel?"}
  }
}
```

The answer is one number, $p = P(\text{yes})$. It works, but it hides *which* situation the model thinks this is. A low $p$ could mean "we stand on" or "there's no encounter at all".

<details markdown="1">
<summary>Complete request after step 1</summary>

```json
{
  "state": "Cargo ship on our starboard bow. We see her red sidelight. Bearing steady.",
  "questions": {
    "give_way": {
      "type": "noul",
      "instructions": "Must own ship give way to this vessel?"
    }
  }
}
```

</details>

## Step 2: give the state structure

```json
"state": {
  "target": {
    "relative_position": "on our starboard bow",
    "aspect": "her port side: we see her red sidelight",
    "course_relation": "crossing courses",
    "bearing_trend": "steady or nearly steady"
  }
}
```

Each fact now has a name, so code can fill it in from the tracker ([page 4](04-encoding-state.md)) and questions can point at it.

<details markdown="1">
<summary>Complete request after step 2</summary>

```json
{
  "state": {
    "target": {
      "relative_position": "on our starboard bow",
      "aspect": "her port side: we see her red sidelight",
      "course_relation": "crossing courses",
      "bearing_trend": "steady or nearly steady"
    }
  },
  "questions": {
    "give_way": {
      "type": "noul",
      "instructions": "Must own ship give way to this vessel?"
    }
  }
}
```

</details>

## Step 3: ask which situation, not yes/no

```json
"encounter_type": {
  "type": "choice",
  "instructions": "Which COLREGs encounter situation does own ship have with `target`?",
  "criteria": {
    "head_on": null, "crossing_give_way": null, "crossing_stand_on": null,
    "overtaking": null, "being_overtaken": null, "no_encounter": null
  }
}
```

Now you get a probability for every situation. The give-way decision follows from the label, in code. `no_encounter` gives the model a way out, and `` `target` `` tells it what to look at.

<details markdown="1">
<summary>Complete request after step 3</summary>

```json
{
  "state": {
    "target": {
      "relative_position": "on our starboard bow",
      "aspect": "her port side: we see her red sidelight",
      "course_relation": "crossing courses",
      "bearing_trend": "steady or nearly steady"
    }
  },
  "questions": {
    "encounter_type": {
      "type": "choice",
      "instructions": "Which COLREGs encounter situation does own ship have with `target`?",
      "criteria": {
        "head_on": null,
        "crossing_give_way": null,
        "crossing_stand_on": null,
        "overtaking": null,
        "being_overtaken": null,
        "no_encounter": null
      }
    }
  }
}
```

</details>

## Step 4: separate the look-alike options

```json
"instructions": {
  "question": "Which COLREGs encounter situation does own ship have with `target`?",
  "assume": "The vessels are in sight of one another."
},
"criteria": {
  "crossing_give_way": {
    "what": "Courses cross and the target is on our starboard side, showing us her port side",
    "not_for": "A target on our port side, or one we are coming up on from astern"
  },
  "crossing_stand_on": {
    "what": "Courses cross and the target is on our port side, showing us her starboard side",
    "not_for": "A target on our starboard side, or one coming up on us from astern"
  }
}
```

The two crossing options differ only by side, so each one now says what it is *not*. The `assume` line makes explicit a condition Rules 13-15 depend on. Models read instructions literally.

<details markdown="1">
<summary>Complete request after step 4</summary>

```json
{
  "state": {
    "target": {
      "relative_position": "on our starboard bow",
      "aspect": "her port side: we see her red sidelight",
      "course_relation": "crossing courses",
      "bearing_trend": "steady or nearly steady"
    }
  },
  "questions": {
    "encounter_type": {
      "type": "choice",
      "instructions": {
        "question": "Which COLREGs encounter situation does own ship have with `target`?",
        "assume": "The vessels are in sight of one another."
      },
      "criteria": {
        "head_on": null,
        "crossing_give_way": {
          "what": "Courses cross and the target is on our starboard side, showing us her port side",
          "not_for": "A target on our port side, or one we are coming up on from astern"
        },
        "crossing_stand_on": {
          "what": "Courses cross and the target is on our port side, showing us her starboard side",
          "not_for": "A target on our starboard side, or one coming up on us from astern"
        },
        "overtaking": null,
        "being_overtaken": null,
        "no_encounter": null
      }
    }
  }
}
```

</details>

## Step 5: add a risk score

Add two facts to `target`, computed by code:

```json
"closest_approach": "inside own ship's safety domain",
"time_to_closest_approach": "within the planning horizon"
```

Then add a Score whose levels describe situations, not degrees:

```json
"collision_risk": {
  "type": "score",
  "instructions": "How much risk of collision with `target` exists?",
  "criteria": [
    "No risk: the target will pass well clear and nothing is developing",
    "Low: the target passes clear but should be kept under observation",
    "Developing: a close-quarters situation will develop if neither vessel acts",
    "Risk of collision exists: action under the rules is required now",
    "Immediate danger: action by the give-way vessel alone can no longer avoid collision"
  ]
}
```

The model sees *inside the safety domain*, never *0.36 nm*. The arithmetic stays in code.

<details markdown="1">
<summary>Complete request after step 5</summary>

```json
{
  "state": {
    "target": {
      "relative_position": "on our starboard bow",
      "aspect": "her port side: we see her red sidelight",
      "course_relation": "crossing courses",
      "bearing_trend": "steady or nearly steady",
      "closest_approach": "inside own ship's safety domain",
      "time_to_closest_approach": "within the planning horizon"
    }
  },
  "questions": {
    "encounter_type": {
      "type": "choice",
      "instructions": {
        "question": "Which COLREGs encounter situation does own ship have with `target`?",
        "assume": "The vessels are in sight of one another."
      },
      "criteria": {
        "head_on": null,
        "crossing_give_way": {
          "what": "Courses cross and the target is on our starboard side, showing us her port side",
          "not_for": "A target on our port side, or one we are coming up on from astern"
        },
        "crossing_stand_on": {
          "what": "Courses cross and the target is on our port side, showing us her starboard side",
          "not_for": "A target on our starboard side, or one coming up on us from astern"
        },
        "overtaking": null,
        "being_overtaken": null,
        "no_encounter": null
      }
    },
    "collision_risk": {
      "type": "score",
      "instructions": "How much risk of collision with `target` exists?",
      "criteria": [
        "No risk: the target will pass well clear and nothing is developing",
        "Low: the target passes clear but should be kept under observation",
        "Developing: a close-quarters situation will develop if neither vessel acts",
        "Risk of collision exists: action under the rules is required now",
        "Immediate danger: action by the give-way vessel alone can no longer avoid collision"
      ]
    }
  }
}
```

</details>

## Step 6: add context and checks on it

Add context to the state, and a fact that converts compass sides into our sides:

```json
"own_ship": {"starboard_side_faces": "east"},
"context": {
  "visibility_report": "Coastal forecast: wind NW 3, sea slight, visibility good.",
  "navigational_warnings": ["Dredger working on the east side of the fairway until further notice."]
}
```

Then add one Noul per condition, each with a clear boundary between yes and no:

```json
"restricted_visibility": {
  "type": "noul",
  "instructions": "Does `context.visibility_report` report restricted visibility around own ship?",
  "criteria": {
    "true": "Fog, mist, falling snow, heavy rain, or similar conditions reduce visibility",
    "false": "Visibility is reported as good or moderate"
  }
},
"starboard_constrained": {
  "type": "noul",
  "instructions": {
    "question": "Does a hazard limit the room own ship has to alter course to starboard?",
    "inspect": ["`context.navigational_warnings`"],
    "orientation": "`own_ship.starboard_side_faces` tells you which compass side is our starboard side."
  }
}
```

Without `starboard_side_faces`, the model would have to work out from our course that east is to starboard. That extra inference step costs accuracy.

<details markdown="1">
<summary>Complete request after step 6</summary>

```json
{
  "state": {
    "target": {
      "relative_position": "on our starboard bow",
      "aspect": "her port side: we see her red sidelight",
      "course_relation": "crossing courses",
      "bearing_trend": "steady or nearly steady",
      "closest_approach": "inside own ship's safety domain",
      "time_to_closest_approach": "within the planning horizon"
    },
    "own_ship": {
      "starboard_side_faces": "east"
    },
    "context": {
      "visibility_report": "Coastal forecast: wind NW 3, sea slight, visibility good.",
      "navigational_warnings": [
        "Dredger working on the east side of the fairway until further notice."
      ]
    }
  },
  "questions": {
    "encounter_type": {
      "type": "choice",
      "instructions": {
        "question": "Which COLREGs encounter situation does own ship have with `target`?",
        "assume": "The vessels are in sight of one another."
      },
      "criteria": {
        "head_on": null,
        "crossing_give_way": {
          "what": "Courses cross and the target is on our starboard side, showing us her port side",
          "not_for": "A target on our port side, or one we are coming up on from astern"
        },
        "crossing_stand_on": {
          "what": "Courses cross and the target is on our port side, showing us her starboard side",
          "not_for": "A target on our starboard side, or one coming up on us from astern"
        },
        "overtaking": null,
        "being_overtaken": null,
        "no_encounter": null
      }
    },
    "collision_risk": {
      "type": "score",
      "instructions": "How much risk of collision with `target` exists?",
      "criteria": [
        "No risk: the target will pass well clear and nothing is developing",
        "Low: the target passes clear but should be kept under observation",
        "Developing: a close-quarters situation will develop if neither vessel acts",
        "Risk of collision exists: action under the rules is required now",
        "Immediate danger: action by the give-way vessel alone can no longer avoid collision"
      ]
    },
    "restricted_visibility": {
      "type": "noul",
      "instructions": "Does `context.visibility_report` report restricted visibility around own ship?",
      "criteria": {
        "true": "Fog, mist, falling snow, heavy rain, or similar conditions reduce visibility",
        "false": "Visibility is reported as good or moderate"
      }
    },
    "starboard_constrained": {
      "type": "noul",
      "instructions": {
        "question": "Does a hazard limit the room own ship has to alter course to starboard?",
        "inspect": [
          "`context.navigational_warnings`"
        ],
        "orientation": "`own_ship.starboard_side_faces` tells you which compass side is our starboard side."
      }
    }
  }
}
```

</details>

## Step 7: make sure the right vessel is judged

The state now mentions two vessels: the target and a dredger. A dredger at work *is* restricted in her ability to manoeuvre, so a vague question may be answered about the wrong ship. Name the vessel, and say what to ignore:

```json
"target_ram": {
  "type": "noul",
  "instructions": {
    "question": "Is `target` itself a vessel restricted in her ability to manoeuvre?",
    "focus": "Judge only the vessel described in `target`, not other vessels mentioned in `context`."
  },
  "criteria": {
    "true": "The nature of her work restricts her ability to manoeuvre",
    "false": {"what": "Nothing says her work restricts her manoeuvring", "not_for": "A vessel that is only large, slow, or deep-laden"}
  }
}
```

`target_nuc` (not under command) follows the same pattern.

<details markdown="1">
<summary>Complete request after step 7</summary>

```json
{
  "state": {
    "target": {
      "relative_position": "on our starboard bow",
      "aspect": "her port side: we see her red sidelight",
      "course_relation": "crossing courses",
      "bearing_trend": "steady or nearly steady",
      "closest_approach": "inside own ship's safety domain",
      "time_to_closest_approach": "within the planning horizon"
    },
    "own_ship": {
      "starboard_side_faces": "east"
    },
    "context": {
      "visibility_report": "Coastal forecast: wind NW 3, sea slight, visibility good.",
      "navigational_warnings": [
        "Dredger working on the east side of the fairway until further notice."
      ]
    }
  },
  "questions": {
    "encounter_type": {
      "type": "choice",
      "instructions": {
        "question": "Which COLREGs encounter situation does own ship have with `target`?",
        "assume": "The vessels are in sight of one another."
      },
      "criteria": {
        "head_on": null,
        "crossing_give_way": {
          "what": "Courses cross and the target is on our starboard side, showing us her port side",
          "not_for": "A target on our port side, or one we are coming up on from astern"
        },
        "crossing_stand_on": {
          "what": "Courses cross and the target is on our port side, showing us her starboard side",
          "not_for": "A target on our starboard side, or one coming up on us from astern"
        },
        "overtaking": null,
        "being_overtaken": null,
        "no_encounter": null
      }
    },
    "collision_risk": {
      "type": "score",
      "instructions": "How much risk of collision with `target` exists?",
      "criteria": [
        "No risk: the target will pass well clear and nothing is developing",
        "Low: the target passes clear but should be kept under observation",
        "Developing: a close-quarters situation will develop if neither vessel acts",
        "Risk of collision exists: action under the rules is required now",
        "Immediate danger: action by the give-way vessel alone can no longer avoid collision"
      ]
    },
    "restricted_visibility": {
      "type": "noul",
      "instructions": "Does `context.visibility_report` report restricted visibility around own ship?",
      "criteria": {
        "true": "Fog, mist, falling snow, heavy rain, or similar conditions reduce visibility",
        "false": "Visibility is reported as good or moderate"
      }
    },
    "starboard_constrained": {
      "type": "noul",
      "instructions": {
        "question": "Does a hazard limit the room own ship has to alter course to starboard?",
        "inspect": [
          "`context.navigational_warnings`"
        ],
        "orientation": "`own_ship.starboard_side_faces` tells you which compass side is our starboard side."
      }
    },
    "target_ram": {
      "type": "noul",
      "instructions": {
        "question": "Is `target` itself a vessel restricted in her ability to manoeuvre?",
        "focus": "Judge only the vessel described in `target`, not other vessels mentioned in `context`."
      },
      "criteria": {
        "true": "The nature of her work restricts her ability to manoeuvre",
        "false": {
          "what": "Nothing says her work restricts her manoeuvring",
          "not_for": "A vessel that is only large, slow, or deep-laden"
        }
      }
    },
    "target_nuc": {
      "type": "noul",
      "instructions": {
        "question": "Is `target` itself a vessel not under command?",
        "focus": "Judge only the vessel described in `target`, not other vessels mentioned in `context`."
      },
      "criteria": {
        "true": "An exceptional circumstance, such as engine or steering failure, leaves her unable to manoeuvre as required",
        "false": "Nothing says she has lost the ability to manoeuvre"
      }
    }
  }
}
```

</details>

## Step 8: ask for a manoeuvre class

```json
"recommended_action": {
  "type": "choice",
  "instructions": {
    "question": "Which manoeuvre class should own ship take with `target` under the COLREGs?",
    "scope": "Choose the class of manoeuvre only. How much to alter or slow is decided elsewhere."
  },
  "criteria": {
    "maintain_course_and_speed": "Stand on: keep the present course and speed",
    "alter_course_to_starboard": "An early, substantial alteration of course to starboard",
    "reduce_speed": "Slacken speed while keeping the present course",
    "alter_to_starboard_and_reduce_speed": "Alter course to starboard and slacken speed together",
    "stop_or_reverse": "Take all way off by stopping or reversing propulsion",
    "alter_course_to_port": "An alteration of course to port"
  }
}
```

The model picks a *class* of manoeuvre. How far to turn is a number, so the planner decides it.

<details markdown="1">
<summary>Complete request after step 8</summary>

```json
{
  "state": {
    "target": {
      "relative_position": "on our starboard bow",
      "aspect": "her port side: we see her red sidelight",
      "course_relation": "crossing courses",
      "bearing_trend": "steady or nearly steady",
      "closest_approach": "inside own ship's safety domain",
      "time_to_closest_approach": "within the planning horizon"
    },
    "own_ship": {
      "starboard_side_faces": "east"
    },
    "context": {
      "visibility_report": "Coastal forecast: wind NW 3, sea slight, visibility good.",
      "navigational_warnings": [
        "Dredger working on the east side of the fairway until further notice."
      ]
    }
  },
  "questions": {
    "encounter_type": {
      "type": "choice",
      "instructions": {
        "question": "Which COLREGs encounter situation does own ship have with `target`?",
        "assume": "The vessels are in sight of one another."
      },
      "criteria": {
        "head_on": null,
        "crossing_give_way": {
          "what": "Courses cross and the target is on our starboard side, showing us her port side",
          "not_for": "A target on our port side, or one we are coming up on from astern"
        },
        "crossing_stand_on": {
          "what": "Courses cross and the target is on our port side, showing us her starboard side",
          "not_for": "A target on our starboard side, or one coming up on us from astern"
        },
        "overtaking": null,
        "being_overtaken": null,
        "no_encounter": null
      }
    },
    "collision_risk": {
      "type": "score",
      "instructions": "How much risk of collision with `target` exists?",
      "criteria": [
        "No risk: the target will pass well clear and nothing is developing",
        "Low: the target passes clear but should be kept under observation",
        "Developing: a close-quarters situation will develop if neither vessel acts",
        "Risk of collision exists: action under the rules is required now",
        "Immediate danger: action by the give-way vessel alone can no longer avoid collision"
      ]
    },
    "restricted_visibility": {
      "type": "noul",
      "instructions": "Does `context.visibility_report` report restricted visibility around own ship?",
      "criteria": {
        "true": "Fog, mist, falling snow, heavy rain, or similar conditions reduce visibility",
        "false": "Visibility is reported as good or moderate"
      }
    },
    "starboard_constrained": {
      "type": "noul",
      "instructions": {
        "question": "Does a hazard limit the room own ship has to alter course to starboard?",
        "inspect": [
          "`context.navigational_warnings`"
        ],
        "orientation": "`own_ship.starboard_side_faces` tells you which compass side is our starboard side."
      }
    },
    "target_ram": {
      "type": "noul",
      "instructions": {
        "question": "Is `target` itself a vessel restricted in her ability to manoeuvre?",
        "focus": "Judge only the vessel described in `target`, not other vessels mentioned in `context`."
      },
      "criteria": {
        "true": "The nature of her work restricts her ability to manoeuvre",
        "false": {
          "what": "Nothing says her work restricts her manoeuvring",
          "not_for": "A vessel that is only large, slow, or deep-laden"
        }
      }
    },
    "target_nuc": {
      "type": "noul",
      "instructions": {
        "question": "Is `target` itself a vessel not under command?",
        "focus": "Judge only the vessel described in `target`, not other vessels mentioned in `context`."
      },
      "criteria": {
        "true": "An exceptional circumstance, such as engine or steering failure, leaves her unable to manoeuvre as required",
        "false": "Nothing says she has lost the ability to manoeuvre"
      }
    },
    "recommended_action": {
      "type": "choice",
      "instructions": {
        "question": "Which manoeuvre class should own ship take with `target` under the COLREGs?",
        "scope": "Choose the class of manoeuvre only. How much to alter or slow is decided elsewhere."
      },
      "criteria": {
        "maintain_course_and_speed": "Stand on: keep the present course and speed",
        "alter_course_to_starboard": "An early, substantial alteration of course to starboard",
        "reduce_speed": "Slacken speed while keeping the present course",
        "alter_to_starboard_and_reduce_speed": "Alter course to starboard and slacken speed together",
        "stop_or_reverse": "Take all way off by stopping or reversing propulsion",
        "alter_course_to_port": "An alteration of course to port"
      }
    }
  }
}
```

</details>

## What you have now

Seven questions in **one request**. They are answered in parallel, so the extra questions add little time. Some only matter in some situations, and code ignores them otherwise ([page 6](06-using-the-answers.md)). The complete request is the one after step 8. To see the same request built for an ordinary LLM, see [page 8](08-building-an-llm-request.md).

Next: [Encoding state](04-encoding-state.md)
