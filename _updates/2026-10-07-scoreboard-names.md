---
title: Scoreboard player names
category: Engine improvements
summary: Joining a server could leave scoreboard names blank or show whoever had that slot before. Names now match the players who are actually in the game.
date: 2026-10-07
area: both
---

## Overview

**The problem.** You connect, open the scoreboard, and some rows are empty — or still show a name from a previous match or another player who used that slot. The game already knew who was online; the scoreboard just failed to show it.

**What we improved.** On join, the scoreboard keeps the name list that came with the connection. The server no longer rewinds and resends an older list that the client then ignores. You should see the real names of everyone already in the server.

## Stock

On a CoD4x server the scoreboard does not print the name from the normal player configstring. Each slot has its own name and clan tag, and the row reads that. The server sends it as `svc_configclient`. The gamestate you get when you connect carries one of those records for every player who is already connected, and it ends with a sequence number, `configDataSequence`. Later name changes continue from that number. The client stores the last sequence it applied and keeps a name update only when the new sequence is exactly the next one.

That name table is not cleared when you connect. A slot that never receives a new record stays as it was: an empty string, or the name of whoever had the slot before.

The acknowledgement of that sequence could also arrive too early. The server had already marked you caught up, because the gamestate contained the full list. A packet could then report an older acknowledgement, from before you had parsed that gamestate. The server used to move its marker backward and replay the name list. Those replayed records were not the next sequence the client was waiting for, so the client threw them away. The slot kept a blank name, or the previous player's name.

## What changed

A lower acknowledgement no longer moves the marker backward. The server leaves it on the sequence it wrote with the gamestate and does not replay the older list. The comment on that check is the connect case: the client has not parsed the gamestate yet, so the older number is ignored.

The gamestate still carries the current name and clan tag of every connected player, and that is the list the scoreboard shows when you join. A later rename is still the next sequence after that. A real gap in the sequence is still dropped, so one lost update does not shift every name that follows it onto the wrong slot.
