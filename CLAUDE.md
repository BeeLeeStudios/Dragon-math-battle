# CLAUDE.md

`index.html` is the whole game (single file, no build step). Edit it directly; keep it fully offline (no CDN/network). The APK/AAB are PWABuilder wrappers around the Netlify site (dragon-math.netlify.app), which must be redeployed with the new index.html to update the app.


Read this first, then open `index.html` (the current, final build — ~370 KB, single file).

## What it is
A single-file, fully offline HTML/JS math combat game for kids ages 5–14. The player picks a boy or girl rider per save slot; dragons are mounts bought in the shop. Enemies are pixel art. A boss arrives every 5 waves.

## Hard rules from Brandon (the owner)
- Name is "Dragons vs Math". Nothing else ("no Dragon Reckoning").
- **Must work offline**: no CDN, no network. It gets downloaded and tested on a network-disabled device. (Phaser was removed for this reason.)
- **Icon-first UI**: kids who can't read yet. Use real emoji/icons, not generic coin icons everywhere.
- HUD numbers stay in their original form (e.g. `❤️ 84/120`).
- Answer buttons: big blocky squares in a 2×2 grid.
- No enemy dragons (too confusing).
- Show picture examples/options before building anything visual; let him pick (often "1 and 3" style picks). Work in phases.
- Saves may be wiped freely (dev build; only Brandon has access). Bump the localStorage key to wipe.
- Don't post the full file mid-work; deliver when a phase is done and tested.

## Architecture
- **Renderer**: custom canvas-2D `BattleStage` with a mini tween engine (`tweenTo`, `killTweensOf`, `updateTweens`, `EASE`), `makeSprite`/`drawSprite`, rAF `frameLoop`, `syncBattleStage`, `BattleFX` facade.
- **Pixel art**: strings where each pixel = slot char + shade digit. `PIX_SLOT_KEY` maps chars → palette slots (b body, d dark, l belly, w wing, h horn, s skin, m/f hair, a armor, c/C cloth, L leather, g gold, k dark2, B blade, H hilt, e eye, W white). `paintPixelFrame`, `pixRamp` (3-step), `pixScaleFor` (integer scale only).
- **Enemy art**: `PIX_ENEMY_ART` keyed by enemy NAME (`sizePct` for minions). A missing name silently falls back to emoji, so `validateArtCoverage()` warns in the console for any enemy, boss, or summon without art. Keep it passing.
- **Rider**: profile-facing seated sprites `PIX_HERO_BOY_SEAT` (sword) / `PIX_HERO_GIRL_SEAT` (spear), 28×36, placed at `SADDLE_X = 11, SADDLE_Y = -14`. `getRiderPalette()` applies sword tier colors; the tier is part of the sprite cache key.
- **Math**: 8 mastery bands (`MATH_BANDS`), `MASTERY` constants (minSeen 24, window 26, promote at 85%, demote below 55% after 10 seen, fast-track at 100% accuracy + speed, retry cooldowns). 20% review chance. Brandon wants the ramp slow: big numbers come late.
- **Balance**: damage is decoupled from answer size (`getProblemDamage`); enemy HP is anchored to expected player damage (`hitsToKillForWave` = min(7, 2.5 + wave×0.09), `MIN_HITS_TO_DIE = 5`).
- **Spawning**: shuffled bags for enemies (`S.enemyBag`) and bosses (`S.bossBag`). Wave 5 is always the Ooze Monarch.
- **Bosses (3 of a planned 5)**:
  - Ooze Monarch: hpMult 1.8; summons 2 Ooze Spawn at 66% and 33% HP. Spawns shield the boss, absorb heals after 4 rounds, and each spawn adds +35% enemy damage. Weak to "−" (×2).
  - Rime Warden: armor (slow answers are cut to 35%), cracks below 30% HP. Weak to "+".
  - Ancient Effigy: charges every 2 rounds (×2.2, capped at maxHP/3). Weak to "−".
  - Boss intro: fade to black, "FIGHT" button (`bossIntroHTML`, `beginBossFight`, `S.bossIntro`).
- **Screens**: journey title screen with forest background (pick boy/girl → begin), icon save slots (portrait + chips, ✅/❌ delete confirm), tabbed shop (⚔️ Power / 🐾 Pets / 🐉 Mounts / ✨ Deco, 3-column tiles, tabs at the bottom), level-up cards with icon chips, game over with 🚩/🏆 and 🥇 NEW BEST.
- **Saves**: localStorage `dragonReckoningSaves_v2`. Per slot: hero, mathBand, mastery, bestWave, bag state.

## Testing approach that works
Run the game in Node with DOM/rAF/setTimeout stubs (the rAF stub must actually call its callback) and simulate full runs: 40+ waves, all bosses, chest, mount purchase, sword tiers, game over, level-up, save/reload. The last full sweep passed with zero art warnings and no Phaser reference.

## Open items / next ideas
- The standing (unmounted) hero sprite still faces the viewer; the mounted rider is profile-facing.
- 2 more bosses planned (5 total; 3 exist). Brandon liked the weakness and phase ideas; minions only for 2 bosses.
- The pixel-art reference (in the claude.ai project chat) is the target art style (16-bit indie, chibi hero, forest).
