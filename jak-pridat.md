---
title: Jak přidat update
section: Šablona
nav: guide
permalink: /jak-pridat/
---

Nový update je jeden soubor ve složce `_updates`. GitHub Pages po pushi do `main` stránku sám přegeneruje.

## Postup

1. Zkopíruj `_updates/priklad.md`.
2. Pojmenuj kopii podle data a tématu, třeba `2026-10-07-slide.md`. Jméno souboru je adresa: `/updates/2026-10-07-slide/`.
3. Uprav hlavičku na začátku souboru. `area` je `client` nebo `server`.

~~~yaml
---
title: Název updatu
summary: Jedna věta, která se ukáže v seznamu.
date: 2026-10-07
area: client
---
~~~

4. Pod hlavičku napiš text v Markdownu. Nadpisy od `##`, odrážky, `kód` i bloky kódu.
5. Příklad `_updates/priklad.md` smaž, až budeš mít první skutečný zápis.
6. Commitni a pushni do větve `main`.

Stránka se za minutu až dvě objeví v levém menu pod **Poslední** a v seznamu podle `area`.
