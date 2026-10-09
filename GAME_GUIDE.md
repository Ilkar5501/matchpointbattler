# HiQ Battler — Mechanics Guide

This document explains the core combat rules used by the HiQ Battler Discord bot. Individual character abilities can override these defaults; when a card explicitly changes a rule, the card text takes priority for that interaction.

## 1. Teams and shared HP

A complete 3v3 team contains **1 Frontline + 2 Backline** cards. A complete 6v6 team contains **3 Frontline + 3 Backline** cards.

All units on a side share one HP pool. A team loses when its shared HP reaches 0. Individual units do not have separate battle HP bars.

Up to ten saved teams can use either format. Teamcreate/Teamedit offer a format toggle: expanding preserves lanes, while shrinking resets the lineup (pending until Confirm in the editor). Incomplete teams remain saved but cannot battle. Battle and PvP support both formats; PvP requires matching formats. Training, Test and both Gauntlets use complete 3v3 teams. Tournaments use complete 6v6 teams.

A team cannot contain two versions of the same character name. For example, two different Hinata cards cannot be used together.

## 2. Offensive and defensive stats

Each character uses exactly one offensive stat:

- **POW** attackers use POW for damage and attack the target's **DEF**.
- **QIK** attackers use QIK for damage and attack the target's **REC**.

The unused offensive stat is 0 and is ignored for Boons, Banes, merge growth, and nature generation.

Other core stats:

- **SPD** controls action order and many follow-up conditions.
- **JMP** controls critical-hit checks and is referenced by several abilities.
- **HP** contributes to the team's shared HP pool.

## 3. Turn structure and action order

One **turn** consists of all six (3v3) or twelve (6v6) units receiving their normal action opportunity unless an ability adds, replaces, skips, or otherwise modifies an action.

There is no traditional "player phase" followed by "enemy phase." At the start of each turn, action order is determined primarily by SPD, so actions can occur in sequences such as Player → CPU → Player → Player → CPU → CPU.

### Timing terminology

Abilities use a common timing model:

1. **At start of match** — resolves once when the battle begins.
2. **At start of turn** — resolves once at the beginning of the turn. Start-of-turn effects resolve in unit SPD order where ordering matters.
3. **At start of unit's action** — resolves immediately before that unit takes its action.
4. **Target selection** — resolves when an attacker is choosing whom to attack.
5. **At start of combat** — checks and snapshots conditions immediately before that attacker/defender combat begins.
6. **During combat** — live combat modifiers. Conditions can respond to current in-combat stats.
7. **Ultimate trigger / damage calculation** — Ultimate modifiers are applied when the triggering attack is being calculated.
8. **Damage result** — effects that depend on the damage actually dealt resolve here.
9. **After combat** — resolves after the current attacker/defender combat finishes.
10. **End of turn** — temporary turn-scoped effects expire or resolve as appropriate.

When multiple effects occupy the same timing layer and do not directly depend on one another, they are treated as simultaneous modifiers before moving to the next layer.

## 4. Default targeting and both team sizes

Without targeting abilities, a three-unit defense is targeted as:

- **Frontline: 40%**
- **Backline 1: 30%**
- **Backline 2: 30%**

This comes from the shared-X formula:

```text
Frontline = X + 10
Backline 1 = X
Backline 2 = X

3X + 10 = 100
X = 30
```

Therefore the default distribution is 40 / 30 / 30. These percentages choose who enters combat; they do not divide incoming damage between units. The selected defender uses its defenses and damage reduces the shared team HP pool.

## 5. Targeting modifiers use a shared-X solver

Targeting abilities do **not** take an already-complete 40/30/30 distribution and then proportionally normalize it. Instead, all lane bonuses and applicable character targeting modifiers are collected first, and then one equal X value is solved so the three final probabilities add to 100%.

General form:

```text
Frontline = X + 10 + applicable modifiers
Backline 1 = X + applicable modifiers
Backline 2 = X + applicable modifiers

Final three weights must total 100%.
```

Targeting is maintained as **two separate defensive states**:

- targeting against incoming **POW** attackers
- targeting against incoming **QIK** attackers

An ability that only affects POW targeting does not change the team's QIK targeting state, and vice versa.

### Tsuki example

Tsuki adds +30 targeting points against POW attackers while he is Frontline.

```text
Tsuki Frontline = X + 10 + 30
Backline 1 = X
Backline 2 = X

3X + 40 = 100
X = 20
```

Against POW attackers:

- Tsuki: **60%**
- Backline 1: **20%**
- Backline 2: **20%**

Against QIK attackers, Tsuki contributes no targeting modifier, so the defense remains 40 / 30 / 30 unless another unit changes it.

### Komi example

Komi adds +30 targeting points against QIK attackers while he is Backline. With an ordinary Frontline unit and one ordinary Backline ally:

```text
Frontline = X + 10
Komi = X + 30
Other Backline = X

3X + 40 = 100
X = 20
```

Against QIK attackers:

- Frontline: **30%**
- Komi: **50%**
- Other Backline: **20%**

### Tsuki + Iwaizumi + Komi example

Formation:

- Frontline: Tsuki
- Backline: Iwaizumi
- Backline: Komi

Against POW attackers, only Tsuki's POW-specific modifier applies:

- Tsuki: **60%**
- Iwaizumi: **20%**
- Komi: **20%**

Against QIK attackers, Iwaizumi and Komi each add +30 while Tsuki contributes only the normal Frontline +10:

```text
Tsuki = X + 10
Iwaizumi = X + 30
Komi = X + 30

3X + 70 = 100
X = 10
```

Final QIK targeting:

- Tsuki: **20%**
- Iwaizumi: **40%**
- Komi: **40%**

Some abilities can override normal weighted targeting entirely, such as forced-target or target-restriction effects.

## 6. Target overrides and interception

Weighted targeting determines the intended target. Certain abilities can then override or replace that target.

Examples include:

- effects that force attacks toward a specific unit
- effects that restrict attacks to Backline units
- save/interception Ultimates that enter combat in an ally's place
- attacker replacement effects

When a defender intercepts, that defender receives the combat and any after-combat effects that the ability says are transferred to it.

## 7. Counterattacks

Units normally counterattack only when their attack type can answer the opponent under the game's standard counter rules. Some abilities explicitly allow a unit to counter regardless of attack type or modify which defensive stat is used.

Counterattacks occur inside the same combat and can interact with cooldown charge, critical hits, follow-ups, damage reduction, and after-combat effects.

## 8. SPD and follow-up attacks

SPD determines the order in which units act each turn and is also used by many ability conditions.

Follow-up attacks are not automatic solely because one unit is faster unless an ability or game rule explicitly grants the follow-up. Character text can grant guaranteed follow-ups, deny them, replace them with brave/twice attacks, or impose SPD checks.

Live "during combat" SPD values are used for conditions that are defined as in-combat comparisons.

## 9. Critical hits and JMP

Compare the two units' **live in-combat JMP for every hit**. The higher-JMP unit gets a critical chance of **5% × the difference**, capped at 100%; equal JMP gives no critical chance. A six-point advantage means 30% on each hit.

If the attacker wins the check, multiply active POW/QIK by **1.2** before subtracting defenses. If the defender wins, multiply the relevant DEF/REC by **1.2** for that hit. Iwaizumi raises allied offensive critical multipliers to **1.5**. Fukunaga's isolated combat disables both critical checks. Visible buffs, debuffs, merges, natures and live support (including Akaashi's JMP reversal) affect this comparison. Criticals do not simply multiply the final damage total.

## 10. Ultimates and cooldown

An Ultimate has a **maximum cost** and a **remaining count**. A 3-CD Ultimate starts with 3 remaining. At 0 it is ready, and activates only at its specified timing/conditions. Activation resets the remaining count to maximum cost.

| Wording | What changes | Example |
| --- | --- | --- |
| Normal hit | Both combatants normally gain 1 charge, reducing remaining count by 1 | Remaining 3 → 2 |
| Charge +1 per own attack | One extra point of progress on that attack | Remaining 3 → 1 instead of 2 |
| Foe charge −1 | Denies one point of normal progress | Normal 1 charge may become 0 |
| Foe count +1 | Increases current remaining count, up to maximum | Remaining 1 → 2; delayed by one |
| Count −1 once / after combat | Reduces current remaining count, maximum unchanged | Siphon: remaining 2 → 1 |
| Accelerates cooldown count −1 / permanent maximum-CD reduction | Reduces total cost and later resets | Shirabu: a designated 3-cost Ultimate becomes 2-cost |

The combatant activating an Ultimate does not also charge from that same triggering hit. Additional hits/counters can charge again according to their conditions. A count change is different from extra charge per hit: read the exact timing and whether the effect changes the maximum. Usuri's Siphon is a one-time change to remaining count after combat, not a permanent reduction in cost.

Kageyama activates after combat when ready and gives himself and allies POW/QIK +6 through their next actions. Offensive Ultimates apply their live damage effects before calculation; healing that depends on damage resolves afterward.

## 11. Buffs, debuffs, and statuses

**Visible bonuses/debuffs** are persistent temporary stat changes that last according to their stated duration, often through a unit's next action.

**In-combat modifiers** only apply inside the specified combat and do not become visible stat bonuses.

Some abilities:

- neutralize visible bonuses
- neutralize visible penalties
- grant positive statuses
- inflict negative statuses
- check whether any visible bonus/status is present

The exact wording on the card determines which category is used.

## 12. Damage reduction, mitigation, and battle statistics

DEF/REC and ability effects can prevent incoming damage. Battle results separately track:

- attacks made
- attacks tanked
- damage dealt
- actual damage tanked
- damage mitigated

**OPOM** (Offensive Player of the Match) primarily rewards the highest damage dealt.

**DPOM** (Defensive Player of the Match) primarily rewards total defensive pressure absorbed: actual damage tanked plus damage mitigated.

## 13. Same-school synergy

For every same-school ally a unit has, it gains **+1 to all stats**, without a cap. A full same-school 6v6 team gives every unit +5.

A three-card team from the same school therefore gives each card +2 to all applicable stats.

## 14. Boons and Banes

Every newly pulled pack card independently rolls a Boon and Bane from six legal stats:

- HP
- the character's active POW or QIK
- DEF
- REC
- SPD
- JMP

The zero offensive stat is never eligible.

If the two rolls differ:

- Boon: **+3**
- Bane: **−3**

If both rolls select the same stat, they cancel and the card is **Neutral**.

CPU, Gauntlet, Training, and Test-team cards are always Neutral.

## 15. Merges

Two copies with the same name + title can be merged if neither consumed card is locked.

Card 1 survives and retains its instance ID, Boon/Bane, original-owner metadata, and obtained timestamp. Card 2 is consumed.

```text
new merge = Card 1 merge + Card 2 merge + 1
```

Maximum merge is +10.

Merge stat progression:

```text
+1  active POW/QIK +1
+2  DEF +1
+3  REC +1
+4  SPD +1
+5  JMP +1 and HP +1
+6  active POW/QIK +1
+7  DEF +1
+8  REC +1
+9  SPD +1
+10 JMP +1 and HP +1
```

At +10, HP and every usable combat stat have gained +2 total.

## 16. Collection acquisition modes

Cards can be obtained through:

- Daily/Base packs
- Position packs
- School packs
- Crafting with school shards
- Trading
- One-time Neutral Suga for finishing all nine tutorial pages and clicking **Finish Tutorial**

Crafted cards are always Neutral +0. Traded cards keep their exact instance ID, merge, nature, original-owner data, and other instance history.

## 17. Pity

A player may designate one global pity target.

If a card of the same rarity is pulled but is not the target, pity becomes armed. The next pull of that rarity from a pack where the target is eligible becomes the pity target. Natural acquisition of the target also clears pity. Changing the pity target resets the armed state.

Crafting does not interact with pity. `;balance` / `/balance` shows your current target and whether pity is Active (guaranteed next eligible same-rarity pull) or Inactive (waiting for a same-rarity miss).

## 18. Training and school shards

Training uses the player's active team against a Neutral formation from the selected school's Regional/National Gauntlet pool.

Each school tracks weekly successful Training wins separately:

- wins 1–3: **Easy**, opponent +0
- wins 4–6: **Hard**, opponent +5
- wins 7–9: **Extreme**, opponent +10

A win gives exactly one shard for that school. A loss gives no shard and does not advance weekly Training progress. Each school allows at most 9 Training shard wins per week.

Shard inventory persists across weeks; only weekly Training progress resets.

Crafting costs:

- 1–2★: 3 school shards
- 3–4★: 6 school shards
- 5★: 9 school shards

## 19. Gauntlets

Regional Gauntlet order:

1. Date Tech
2. Fukurodani
3. Aoba Johsai
4. Kamomedai

Each school has G1–G3, for 12 required wins.

National Gauntlet order:

1. Shiratorizawa
2. Karasuno
3. Inarizaki
4. Nekoma

Each school has G1–G5, for 20 required wins.

The player's active team is used for the entire run. One loss or draw ends the run. A failed run can be retried immediately.

A successful clear declares the player Regional/National Champion. Only the **first successful clear of each mode each week** awards **450 coins**. You can replay successful gauntlets immediately, but repeat clears award no additional coins. Reward eligibility resets **Monday 00:00 America/New_York**.

The result interface provides school-by-school and game-by-game performance views showing damage dealt, damage tanked, mitigation, OPOM, and DPOM.

## 20. Weekly Battle rewards

Normal random CPU `battle` wins drive daily and weekly battle rewards.

After weekly random-battle win 50, each additional random-battle win awards +1 coin, up to 1,000 post-50 bonus coins that week. PvP, Test, Training, Gauntlet, and losses do not award this post-50 bonus.

---

When an individual card's wording conflicts with a generic rule, use the card's explicit effect text and timing for that interaction.


## 21. Editing teams and recycling

`;teamedit <slot>` lets you replace a player in Frontline, Backline 1 or Backline 2. Selecting someone already in the draft swaps their positions. Edits stay pending until **Confirm**; Cancel or timeout leaves the saved lineup untouched. Teamcreate hides selected character names, including alternate versions, until Reset.

`;recycle <ID>` requires confirmation and rejects locked cards. Coins awarded are **stars × 10 × (merge+1)**. A 4★ +5 card represents six copies and returns 240 coins. Recycling permanently removes the card; all saved team slots containing it become empty, making those teams invalid until fixed.

## 22. Usuri and skill definitions

**Heal Block** prevents all healing of the affected unit's shared team HP pool until that unit finishes its next action. This includes passive, Ultimate and ally-sourced team recovery. It does not prevent damage mitigation or the creation of new barriers.

**Siphon** applies once to each other ally's first initiation per turn. After that combat, the ally's remaining Ultimate count decreases by 1 and the actual target's increases by 1, capped to 0/maximum. It changes remaining count once, not maximum cost, and does not repeat for follow-ups or twice-hits. Counters and Usuri's own initiations do not receive it.

Usuri's **Feeding the Beast** gives POW/JMP `min(turn+2, 5)` to the other ally who dealt most damage in the previous complete turn; Kiryu always has priority. Ties use formation-list order; previous-turn damage is zero on turn 1. **Mental Break** converts foes' visible bonuses into equal penalties at turn start, through their next actions. **Tactical Dump** clears its target's barrier, applies Heal Block before calculation and adds 50% of Usuri's live POW for that hit.

**Excellence** deals bonus damage equal to 70% of a positive live SPD difference, rounded down, without a cap. **Fatigue** stacks have no cap: N stacks penalize SPD/JMP by N, POW/QIK by max(N−1,0) and DEF/REC by max(N−2,0); they persist across turns. Each relevant Cardview/Gallery skill includes its own extended explanation.


## 6v6 targeting and formations

The shared-X solver applies to all six defenders. With no targeting abilities, three fronts contribute +10 each: `6X + 30 = 100`, so X=11.6667. Each front has **21.6667%** targeting chance and each back **11.6667%**. Applicable POW/QIK bonuses still modify their individual slots and a new X is solved. Extreme combinations clamp negative slot weights to zero and renormalize the remaining probabilities. Forced targets, saves and swaps keep their existing timing. Formation swaps exchange one frontline with one backline, preserving the three/three counts.

## Mujinazaka Training and crafting

Mujinazaka is available alongside the existing Training schools. G1–G3 use **Unnan front / Usuri + Kiryu back**. The first three weekly shard wins use +0 opponents; the next three use +5; the final three use +10. The weekly maximum is nine shards per school; shards carry over. This also enables crafting Mujinazaka characters: Usuri (3★) costs six Mujinazaka shards. Mujinazaka School Packs remain unavailable under the existing five-character eligibility rule.

## Eight-player tournaments

Use **`;to` / `;tournament`** in a server channel, then Join with a complete 6v6 team. The private builder can create or expand a saved team. Eight unique entrants start automatically; the snapshot taken on entry stays fixed. Leave is available before start, and the host can cancel an open lobby. Each account can enter one live tournament at a time.

Two losses eliminate a player. The losers-bracket finalist must defeat the undefeated finalist twice: winning the first grand final creates a reset final. Match results and images arrive in DMs; **View my matches** retrieves them privately when DMs are closed. All entrants are pinged with the completed bracket.

Champion: **300 coins**, others: **50**. Daily champion cap: **900**; daily participation cap: **150**. These are independent allowances, shown in `;questview` / `;qv`, and reset at midnight in the configured game timezone. Tournaments remain playable after either cap. The champion earns only the champion reward. Ordinary battle quest counters are unaffected. Rewards use the completion day's allowance and cannot be awarded twice after a restart.

Tournament draws are rematched up to three simulations. A third draw uses remaining HP percentage, then a seeded coin toss for an exact tie. Brackets resume after a bot restart; `;to <ID>` reopens saved results from the originating channel. Discord deliveries can repeat after a crash between sending and recording a notification, while coin credits remain exactly once.


## v7 balance and personal tests

See [CHANGELOG_v7.md](CHANGELOG_v7.md) for the current character updates and exact Bokuto skill-counting rules. `;botset` supports 3v3 or 6v6; `;test` requires a matching complete active player team and shows the same results image and battle-log controls as `;battle`. Switching the bot format clears its lineup.


## Tournament CPU auto-fill

The host can click **Auto-fill (host)** on an open `;to` / `;tournament` lobby. It preserves entered player lineups, fills every remaining place with labeled Neutral +0 CPU teams and starts the eight-team bracket. Each CPU independently chooses one of the nine specified formations, so repeats are allowed. CPU champions earn no coins; humans retain the normal champion/participation rewards and daily limits. Players receive DMs and can replay their matches against CPUs using the existing results controls. See [CHANGELOG_v7_1.md](CHANGELOG_v7_1.md) for the exact formations.
