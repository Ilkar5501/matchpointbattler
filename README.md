# HiQ Battler Discord Bot — v7.1

v7.1 adds a host-only **Auto-fill** tournament button: empty places become CPU 1, CPU 2, etc., with each independently choosing one of the nine requested Neutral +0 6v6 formations. Filling the lobby starts the bracket. CPUs do not earn coins or receive DMs; human rewards and match-result displays remain unchanged. See [CHANGELOG_v7_1.md](CHANGELOG_v7_1.md).

v7 adds the requested Bokuto, Hirugami, Kiryu, Daisho, Shirabu, Tendo, Miracle Maker Hinata and Unnan balance changes; uncapped school bonds; and personal test teams in both 3v3 and 6v6 with the regular Battle image and log display. The roster remains 48 characters. See [CHANGELOG_v7.md](CHANGELOG_v7.md) for mechanics, skill counting and upgrade details.

## Wispbyte upgrade from v6.1 or earlier

Keep these live files/directories exactly as they are:

```text
.env
hiq_battler.sqlite3
.local/
.cache/
```

Replace the application files/folders with v7.1:

```text
bot.py
data/
hiq/
scripts/
tests/
Dockerfile
requirements.txt
README.md
GAME_GUIDE.md
VERSION
CHANGELOG_v7.md
CHANGELOG_v7_1.md
.env.example       reference only; do not overwrite live .env
```

`__pycache__/` and `.pyc` files are not deployment data and can be deleted. Python recreates them.

Install/update dependencies with:

```bash
pip install -r requirements.txt
```

Use `bot.py` as the startup file. Database migrations are transactional/idempotent; existing v1/v4.1 cards, instance IDs, owners, merges, teams, locks, and legacy Neutral natures remain persistent. v7 never rolls existing cards on startup. The saved_teams table is rebuilt in one transaction to expand its SQLite CHECK constraint to ten slots; old lineups default to 3v3.

## `.env`

Keep the existing token. Recommended values:

```env
DISCORD_TOKEN=your_real_token
BOT_PREFIX=;
GAME_TIMEZONE=America/New_York
POW_EMOJI_NAME=pow
QIK_EMOJI_NAME=qik
POW_EMOJI_ID=1555445373219242086
QIK_EMOJI_ID=1555445374691315873
MECHANICS_GUIDE_URL=
```

`MECHANICS_GUIDE_URL` is optional. Paste the public Google Doc URL there after copying `GAME_GUIDE.md` into Google Docs. The tutorial will show a Mechanics Guide link button.

Message Content Intent must remain enabled for `;` commands. Public commands are now hybrid commands, so their canonical names also register as `/slash` commands.

## Command surface

```text
;start /start
;help /help

;daily /daily                    aliases: ;d
;balance /balance                alias: ;bal
;shop /shop
;pack base /pack base            shorthand: ;p b
;pack position /pack position    shorthand: ;p p
;pack school /pack school        shorthand: ;p s

;collection /collection          aliases: ;col ;co
;cardview /cardview              alias: ;cv
;cardsearch /cardsearch
;gallery /gallery
;merge /merge
;lock /lock
;recycle /recycle
;pity /pity

;teamcreate /teamcreate
;teamedit /teamedit                stage swaps/replacements; Confirm saves
;teamview /teamview
;teamuse /teamuse
;botset /botset
;test /test

;battle /battle
;pvp /pvp
;questview /questview            aliases: ;qv ;quest ;quests
;gauntlet /gauntlet
;training /training
;tournament /tournament          alias: ;to
;craft /craft
;trade /trade
;data /data
```

Prefix fuzzy matching remains available, but the important shorthand above is explicit/deterministic.

## Collection filters

The existing syntax remains:

```text
;cardsearch name:oikawa spd:>30 position:S school:"Aoba Johsai"
```

v5 also accepts shorthand in Collection/Card Search:

```text
;col -n oikawa
;col -spd > 30
;col -pos S
;col -tm Aoba Johsai
;col -n oikawa -jmp >= 35 -tm Aoba Johsai
```

Supported dash stats: `-hp -pow -qik -def -rec -spd -jmp`, with `> < >= <= =`.

Gallery results and Gallery dropdown selections are public/persistent channel embeds. Teamview is also always public.

## Targeting — separate POW/QIK defensive states

Every defending team maintains a targeting distribution for incoming POW attacks and a separate distribution for incoming QIK attacks.

For a 3v3 defending formation:

```text
Frontline = X + 10 + applicable targeting modifiers
Backline1 = X + applicable targeting modifiers
Backline2 = X + applicable targeting modifiers
```

One equal `X` is solved so the final weights total exactly 100%.

No modifier:

```text
40 / 30 / 30
```

Examples now implemented in the engine and card text:

- Tsuki Front vs POW: `60 / 20 / 20`
- Tsuki Front + Komi Back + ordinary Back vs QIK: Tsuki `30`, Komi `50`, other `20`
- Tsuki Front + Iwaizumi Back + Komi Back:
  - vs POW: `60 / 20 / 20`
  - vs QIK: `20 / 40 / 40`

Forced targeting/restriction abilities can still override the weighted selection stage as their kits specify.

## v5 roster updates

The bundled workbook and generated `characters.json` include only these requested base-stat buffs:

```text
Bokuto: +1 POW
Shirabu: +10 HP, +3 POW
```

Tsuki, Komi, and Iwaizumi targeting text was updated to match the shared-X system. POW/QIK card/gallery lines use the configured Discord custom emoji IDs rather than sword/diamond fallbacks.

## Battle results

Normal Battle/PvP keeps the public result embed plus the native paginated text Battle Log. The attached summary shows three or six cards per side, with lane/name and combat production. For 6v6, the embed omits duplicate individual text lines; the larger image carries all twelve readable cards. A trophy identifies the winner in either size.

Per-unit tracked values include:

- attacks made
- attacks tanked
- damage dealt
- damage tanked
- damage mitigated

Awards remain:

- **OPOM:** highest damage dealt, with crit/efficiency tiebreaks
- **DPOM:** highest damage tanked + mitigated, with mitigation/attacks-tanked tiebreaks

`teamuse` is still the account's active team for both PvP offense and PvP defense.

## Gauntlets

Gauntlets always use the player's active complete team.

Regional — 12 wins:

```text
Date Tech -> Fukurodani -> Aoba Johsai -> Kamomedai
```

National — 20 wins:

```text
Shiratorizawa -> Karasuno -> Inarizaki -> Nekoma
```

A loss or draw ends the run and the player may retry immediately. A successful clear:

- declares Regional/National Champion
- awards **450 coins only on the first clear of that mode that week**
- allows immediate replays; subsequent clears award **0 extra coins**

Reward eligibility resets Monday at **00:00 America/New_York**, including daylight-saving transitions. Regional and National rewards are tracked separately.

v5 removes the giant combined Gauntlet log. The final public message shows the run record grouped by school and includes an interactive performance panel:

1. choose the school
2. view G1
3. use Previous/Next arrows through G2/G3 and, for Nationals, G4/G5

Each game shows both teams' lane/player, Damage Dealt, Damage Tanked, Damage Mitigated, turns/HP, and OPOM/DPOM markers.

## Training and school shards

```text
;training <school>
/training <school>
```

Schools:

```text
Inarizaki
Date Tech
Shiratorizawa
Karasuno
Aoba Johsai
Kamomedai
Fukurodani
Nekoma
```

Training uses one random existing Gauntlet formation from that school. All Training opponents are Neutral and the whole opponent formation is overridden to the current difficulty merge level.

Each school separately allows at most 9 successful Training wins per week:

```text
weekly successes 1-3: Easy    +0 opponent
weekly successes 4-6: Hard    +5 opponent
weekly successes 7-9: Extreme +10 opponent
```

A win gives exactly 1 shard for that school. A loss/draw gives nothing and does not advance weekly Training progress. The weekly progress resets Monday 00:00 New York time, but shard inventory never resets. The DB award is transactional, so simultaneous commands cannot create a tenth weekly shard.

## Crafting

```text
;craft <player name>
/craft <player name>
```

Craft cost uses that player's school shards:

```text
1-2★: 3 shards
3-4★: 6 shards
5★:   9 shards
```

Crafted cards are new unique **Neutral +0** instances. Crafting does not interact with pity.

## Recycling

```text
;recycle <exact instance ID>
/recycle <exact instance ID>
```

A destructive confirmation button is required. Locked cards cannot be recycled.

```text
payout = rarity × 10 × (merge + 1)
```

Examples:

```text
1★ +0  -> 10 coins
1★ +1  -> 20 coins
5★ +5  -> 300 coins
5★ +10 -> 550 coins
```

The recycled card is permanently deleted. Any saved-team slot containing that exact instance becomes empty; incomplete teams remain saved but cannot battle.

## Trading

```text
;trade @user
/trade @user
```

Flow:

1. recipient Accepts/Declines the request
2. accepted request opens a shared public trade builder
3. each player can offer exact card instance IDs, coins, and school shard quantities
4. locked cards cannot be offered
5. editing either offer clears previous confirmations
6. both players independently press Confirm
7. the exchange executes in one DB transaction after balances/ownership/locks are revalidated

Traded cards keep their exact instance ID, merge, Boon/Bane, original-owner metadata, and obtained metadata. Any team slot containing a transferred card becomes empty.

## Pity

```text
;pity <player name>
/pity <player name>
;pity clear
```

One global target is stored per account. Redesignating a target resets the armed state.

- same-rarity miss -> pity becomes armed
- while armed, the next pull of that rarity from a pack where the target is eligible becomes the pity target
- natural acquisition of the target also clears pity
- crafting does not use pity

Pity applies to Base/Daily, Position, and School packs only when the target is eligible for that pack pool.

## Personal Test Bot

```text
;botset /botset
;test /test
```

`botset` builds a saved per-user CPU test formation from the full roster, not the owned collection. Choose **3v3 (one Frontline + two Backline)** or **6v6 (three Frontline + three Backline)**. Switching format clears the saved bot lineup. Specific-slot selection supports replacements. Both formats obey the no-duplicate-character-name rule.

Test units are always Neutral +0. `test` requires an active complete player team and a complete Test Bot with the same format. Results use the shared regular Battle text/image and interactive battle log. It records analytics but awards no coins, quest wins, shards, Training progress, or Gauntlet progress.

## Post-50 Battle bonus

Only random CPU `battle` wins count.

After weekly Battle win #50, every additional Battle win awards **+1 coin**, capped at **1,000 bonus coins per week**. PvP, Training, Test, Gauntlet, and losses do not contribute.

`questview` displays:

```text
Post-50 Battle Bonus: X / 1000 bonus coins
```

The counter resets with the normal Monday 00:00 America/New_York weekly reset.

## Data / analytics

`data` is no longer owner-only. Any player may run queries such as:

```text
;data Suna front
;data Suna with Atsumu front
;data Suna vs Oikawa back
;data Suna with Atsumu front mode:regional
```

The command returns Discord statistics, not a file.

## Mechanics guide

`GAME_GUIDE.md` is included for copying into Google Docs. It explains the battle system, shared HP, timing layers, 40/30/30 targeting, shared-X POW/QIK states, counters, SPD, JMP/crit, Ultimates, buffs/debuffs, damage reduction, interception, school synergy, Boons/Banes, merges, acquisition, Training, and Gauntlets.

After publishing the Google Doc, set its public URL in `MECHANICS_GUIDE_URL` and restart the bot to show the link in `start`.

## Source-of-truth roster

```text
data/HiQ Battler Abilities.xlsx
data/characters.json
data/gauntlets.json
data/game_config.json
```

`scripts/build_data.py` regenerates runtime roster/config data from the workbook. The v5 workbook already contains the Bokuto/Shirabu changes and revised Tsuki/Komi/Iwaizumi targeting text.

## v5.1 patch

Akaashi — Architect of Calm's B-slot, **Weakness Whisperer**, now also reverses visible **JMP** penalties using the same twice-penalty in-combat bonus as POW/QIK/SPD/DEF/REC. No other gameplay behavior was changed in this patch.


## v6 changes

This release keeps v5.1's other systems and Akaashi's JMP penalty reversal intact.

- `;teamedit <slot>` / `/teamedit slot:<slot>` opens an editor. Choose Frontline, Backline 1, or Backline 2, then an owned card. Existing members swap positions; other cards replace that position. Confirm commits the whole draft atomically. Cancel/timeout leaves the saved team unchanged. Reset Changes restores the original draft. Cards must still be owned at confirmation; stale drafts are rejected if the saved team changed elsewhere. Editing does not switch active team slots.
- Teamcreate hides all copies/variants of already selected character names. Selected players cannot be reused until Reset; use Teamedit for swaps.
- Shiratorizawa is available in Training through both command forms, using the existing five National formations. Its shard cap and Easy/Hard/Extreme rules are unchanged.
- Osamu: QIK 51 and C-slot JMP +4.
- Atsumu: Excellence has no damage cap (70% of the positive in-combat SPD difference, rounded down). His self/allied SPD aura is turn number +1, with no cap.
- Kenma: Fatigue stacks have no maximum; penalties still use the existing stack formulas.
- Kageyama: HP 162, JMP 32, Ultimate CD 3. Ultimate grants himself and allies POW/QIK +6 through their next actions. A retains POW +6 on initiation and adds the post-combat target SPD −3. B grants initiating allies follow-ups at team HP ≤75% and SPD ≥ foe SPD+3. C remains unchanged.
- Card art replaced for Kageyama, Kenma, Oikawa, and Hinata — Wings of Dawn. Their icons remain unchanged.
- Cardview/Gallery show extended skill definitions immediately under relevant skills, including Excellence, Fatigue, Free Ball, Siphon and Heal Block.
- Gauntlets are replayable after any clear; only the first successful Regional and National clears each week pay coins. Weekly claim and coin credit are atomic.
- Balance displays the global pity target and Active/Inactive guarantee state, with pack eligibility explained.
- Tutorial expands to nine pages (targeting, JMP, cooldown terminology, Training/Crafting/Recycling). Finish Tutorial on the last page grants Neutral +0 Suga once per account. An additive `tutorial_rewards` table preserves that claim through restarts, trading or recycling. Existing accounts can earn this newly introduced reward on their first complete v6 tutorial; the old `tutorial_started` flag continues to unlock commands, but did not prove a full tutorial completion.
- Usuri — Calculated Distraction: Mujinazaka, Setter, 3★, Support / Control; HP 154, POW 47, QIK 0, DEF 30, REC 27, SPD 36, JMP 32; Ultimate CD 3. His art/icon, kit, and glossary are included in the runtime roster and workbook. He is available in Base/Setter packs, roster lookup, pity and test-team selection under existing eligibility rules. In v6.1, Mujinazaka Training supplies shards and enables crafting Usuri, Unnan and Kiryu under the existing crafting costs. Mujinazaka still has fewer than five players, so it is not eligible for school packs.

### Usuri effect details

Tactical Dump clears the targeted defender's barrier and applies Heal Block before damage calculation; adds 50% of live POW, rounded down. Heal Block blocks recovery of that team's shared HP pool until the affected unit finishes its next action, including healing sourced from another ally. Barriers on other units are unaffected.

Mastermind's Strings grants Siphon once on **each other ally's first initiated combat per turn**. It does not apply to Usuri's own initiation or counters and does not repeat on follow-ups/twice-hits. After that combat, remaining CD decreases by 1 for the initiator and increases by 1 for the actual defending target (clamped to 0/maximum); maximum Ultimate costs do not change. Support suppression respects Fukunaga's existing rules.

Feeding the Beast uses actual damage dealt during the **previous complete turn**, including counters. Kiryu always has priority; otherwise it selects the highest-damage other ally. Ties use formation-list order, and turn 1 starts with zero previous damage. POW/JMP bonus is `min(turn+2, 5)`. Mental Break converts current visible bonuses at turn start using the engine's existing start-of-turn resolution order.

Runtime JSON and the source workbook match. `scripts/build_data.py` now also reads Ultimate extended skill information.


## v6.1 — Mujinazaka, team formats and tournaments

- Mujinazaka Training uses **Unnan frontline / Usuri + Kiryu backline** for G1–G3. The first three weekly shard wins fight +0, the next three +5, the final three +10. Nine shard wins per school per week; shard inventory carries over. Regional and National schedules are unchanged.
- Save **10 teams**. `;teamcreate <1-10>` and `;teamedit <slot>` offer a 3v3 / 6v6 toggle. 6v6 is **3 front / 3 back**. Expanding preserves each current lane. Shrinking clears the lineup; in Teamedit this clears the pending draft and only commits on Confirm. You can refill the empty draft before confirming. Reset Changes restores the original draft and format. Existing teams stay 3v3 after migration.
- `;battle [slot]` uses that saved team's format and generates a matching neutral CPU. `;pvp @user` requires matching active team formats. `/pvp` mismatch notices are ephemeral; prefix `;pvp` sends the mismatch privately by DM because Discord prefix messages cannot be ephemeral. Closed DMs prompt use of `/pvp`.
- Training, Regional/National Gauntlets and the existing custom Test mode remain **3v3**. Use `;teamuse <slot>` to choose a complete 3v3 team for these modes.
- In 6v6 the same shared-X targeting formula applies to every defender: each front has X+10 plus applicable modifiers; each back has X plus applicable modifiers. Baseline X=70/6, so each front is 21.6667%, each back 11.6667%. POW/QIK remain separate; the existing zero-clamp/normalization fallback handles extreme stacked targeting modifiers. School bonds are now uncapped (+5 per unit for a full same-school 6v6 team). Damage/crit rules, cooldowns, action order and the 30-turn limit otherwise retain the documented behavior, with v7 character changes listed in CHANGELOG_v7.md.

### Tournament flow

1. Run **`;to` / `;tournament`** (or `/tournament`) in a server channel. One open/running lobby is shared in that channel. The host can cancel before play starts.
2. Click **Join** and choose a complete saved 6v6 team, or create/expand one with the private builder and return to Join. Six different character names are required. Players can leave before the eighth entrant; each account can enter one live tournament at a time.
3. Eight unique entrants start automatically with shuffled, persisted seeds. Team instances, merge levels, natures and character definitions are snapshotted at entry. Editing, trading, merging or recycling afterward does not change the tournament entry.
4. The winners and losers brackets run in the background. Each player is eliminated on their second loss. A losers-bracket finalist who wins the first grand final triggers a reset final. The event has **14 matches**, or **15** with the reset.
5. Each resolved match sends both players its full 6v6 image, awards, a paginated battle log and a win / first loss / knockout notice by DM. If DMs are unavailable, **View my matches** opens private results from the lobby or final announcement. Historical panels remain available after restart; `;to <tournament ID>` reopens this channel's saved event.
6. After all matches, the channel announcement pings all eight entrants, shows the bracket and coins actually awarded. The champion earns **300**, everyone else **50**. Champion rewards have a separate **900/day** cap; participation rewards have a separate **150/day** cap. A champion does not also receive participation coins. Caps reset at midnight in the configured game timezone (Eastern by default); the completion day determines the allowance. `;qv` displays both.

Tournament rewards do not count as ordinary Battle wins or spend ordinary Battle quest rewards. Repeated tournaments remain playable after reaching a reward cap, with zero coins for the capped category. A draw is replayed twice; after three drawn simulations the higher remaining HP percentage wins, with a persisted-seed coin toss for an exact tie.

Bracket state, lineups, match results, DM delivery state and rewards survive restart. Wallet credit, caps and completion commit atomically, so restarting/reopening cannot credit rewards twice. Discord message delivery is best effort: an abrupt crash between sending a DM/announcement and recording its message state can repeat that message on recovery. Coin credit remains exactly once.

The bot needs permission to send messages, embeds and attachments in the tournament channel. Members with closed DMs can use the private result controls. Live Discord delivery still requires testing with your bot account.
