---
title: Add an update
section: Template
nav: guide
permalink: /jak-pridat/
---

A new update is one file in `_updates`. GitHub Pages rebuilds the site after a push to `main`.

## Steps

1. Copy `_updates/priklad.md`.
2. Name the copy by date and topic, for example `2026-10-07-slide.md`. The file name is the address: `/updates/2026-10-07-slide/`.
3. Edit the header at the top of the file. `area` is `client`, `server`, or `both` when the change covers both.

~~~yaml
---
title: Update title
summary: One sentence shown in the list.
date: 2026-10-07
area: client
---
~~~

4. Write the text under the header in Markdown. Headings from `##`, lists, `code`, and code blocks.
5. Delete the `_updates/priklad.md` example once the first real write-up is in.
6. Commit and push to the `main` branch.

Within a minute or two the page shows in the left menu under **Latest**, and in the list for its `area`.
