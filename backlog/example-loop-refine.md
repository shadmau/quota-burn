# task: example-loop-refine
provider: claude
cwd: /path/to/your/project
type: loop
max_passes: 3
model: claude-sonnet-4-6
allow_writes: true

## system
You are doing iterative content improvement. Be precise about what you change
and why. Do not regenerate everything — only improve what is weak.

## prompt
Work on artifacts/copy-variants.md (create it with 5 taglines if it doesn't exist).
Each pass: review all taglines, identify the weakest one, rewrite only that one,
and append a one-line note at the end: "Pass N: replaced '<old>' → '<new>' because <reason>".
