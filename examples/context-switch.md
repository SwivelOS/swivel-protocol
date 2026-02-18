# Context Switch Example

This example shows how to switch contexts using the swivel protocol.

## Scenario

You're working on a trading bot (oakmont-ab-test.swivel.md) and want to switch to a blog post (blog-post.swivel.md).

## Before the Switch

Your root `.swivel.md` looks like:

```markdown
## Active Topics
- [oakmont-ab-test](./trading/oakmont-ab-test.swivel.md) ← **RUNNING**

## Paused Topics
- [blog-post](./blog/blog-post.swivel.md)
```

## The Switch

**You say:** "swivel to blog post"

**Agent does:**

1. **Stops** referencing oakmont-ab-test
2. **Reads** `./blog/blog-post.swivel.md`
3. **Updates** root `.swivel.md`:

```markdown
## Active Topics
- [blog-post](./blog/blog-post.swivel.md) ← **ACTIVE**

## Paused Topics
- [oakmont-ab-test](./trading/oakmont-ab-test.swivel.md)
```

4. **Confirms:** "Switched to blog post. Status: drafting. Word count: 1,247. Next: finish 'Why This Works' section."

## What NOT to Do

❌ **Wrong:** "If I recall, we were testing something with Oakmont... let me search my memory..."

✅ **Right:** Read the swivel file and report only what's there.

## Key Principle

Context switches are atomic. No reconstruction. No drift. Just the file.