# Triage Labels

The factory CLI has no labels. These five roles are a vocabulary only. Each role maps to a factory stage or state.

|Role|Label|Factory equivalent|
|-|-|-|
|`needs-triage`|`needs-triage`|stage `backlog`, not yet refined|
|`needs-info`|`needs-info`|state `awaiting_input`|
|`ready-for-agent`|`ready-for-agent`|stage `spec_review` or `plan_review`, approved|
|`ready-for-human`|`ready-for-human`|state `blocked` or `failed`|
|`wontfix`|`wontfix`|state `cancelled`|

When a skill names a role, use the factory equivalent. Do not create GitHub labels.
