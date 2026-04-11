# task: example-copy-variants
provider: claude
cwd: /path/to/your/project
type: once
model: claude-haiku-4-5-20251001
allow_writes: true

## system
You are working on low-priority backlog preparation only.
Do not change core product logic.
Prefer docs, copy variants, eval assets, or small analysis tasks.

## prompt
Create a file called artifacts/copy-variants.md with 5 short alternative
taglines for a productivity CLI tool. Each variant should be one sentence,
punchy, and developer-focused.
