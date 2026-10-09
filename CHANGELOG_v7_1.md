# v7.1 — tournament CPU auto-fill

The existing tournament lobby now includes **Auto-fill (host)**. Only the lobby creator can use it. It fills unoccupied places up to eight, preserving all existing players and their saved tournament lineups, and starts the bracket automatically. The operation commits atomically, so simultaneous joins or repeated clicks cannot overfill or start twice.

CPUs are labeled **CPU 1**, **CPU 2**, etc. Each independently selects one formation uniformly from the nine below. Repeated selections are allowed. All CPU cards are **Neutral +0** and use the current v7 abilities and stats. Hinata always means **Wings of Dawn** in these formations.

| Team | Frontline 1 | Frontline 2 | Frontline 3 | Backline 1 | Backline 2 | Backline 3 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Oikawa | Korai | Daichi | Atsumu | Osamu | Hinata — Wings of Dawn |
| 2 | Ushijima | Korai | Daichi | Oikawa | Akaashi | Bokuto |
| 3 | Ushijima | Korai | Daichi | Oikawa | Kiryu | Usuri |
| 4 | Hirugami | Korai | Ushijima | Oikawa | Kenma | Usuri |
| 5 | Hirugami | Korai | Ushijima | Oikawa | Kenma | Daichi |
| 6 | Hirugami | Korai | Ushijima | Kageyama | Oikawa | Hinata — Wings of Dawn |
| 7 | Ushijima | Korai | Yamagata | Oikawa | Shirabu | Reon |
| 8 | Aone | Koganegawa | Futakuchi | Oikawa | Daisho | Daichi |
| 9 | Korai | Nishinoya | Oikawa | Kageyama | Hinata — Wings of Dawn | Iwaizumi |

CPU entries and lineups persist across restart alongside the existing bracket. CPUs are not Discord accounts and do not receive DMs, mentions, wallet credits or reward-limit records. A CPU can win the tournament; human entrants then receive the existing participation reward, subject to its normal daily cap. A human champion receives the existing champion reward and other human entrants receive participation rewards. Finishing/reopening cannot pay twice.

The existing human match images, battle logs, DM delivery, replay controls, bracket rules and reward amounts remain unchanged. The bracket and champion display include CPU labels. An empty lobby can also be filled with eight CPUs; it completes normally without awarding player coins.

No character, combat, collection, economy, test-team or other UI changes were made in v7.1. Upgrading requires replacing the application files and restarting while retaining the existing `.env`, database and persistent directories. No database schema change is needed for this addition.
