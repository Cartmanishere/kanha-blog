---
title: "Test Note Shortcode"
date: 2026-01-12
draft: true
---

# Testing Note Shortcode

## Basic Note
{{< note >}}
This is a basic informational note.
{{< /note >}}

## Note with Title
{{< note title="Important" >}}
This note has a title and contains important information.
{{< /note >}}

## Warning Note
{{< note type="warning" title="Warning" >}}
This is a warning message. Be careful with this operation.
{{< /note >}}

## Success Note
{{< note type="success" title="Success" >}}
Operation completed successfully!
{{< /note >}}

## Danger Note
{{< note type="danger" title="Danger" >}}
This operation cannot be undone. Proceed with caution.
{{< /note >}}

## Tip Note
{{< note type="tip" title="Pro Tip" >}}
Here's a helpful tip to improve your workflow.
{{< /note >}}

## Note with Markdown
{{< note type="info" title="Markdown Example" >}}
This note supports **bold text**, *italic text*, and `inline code`.

- List item 1
- List item 2
- List item 3

[Link example](https://example.com)
{{< /note >}}
