# Effort Tag Feature — Design Spec

**Date:** 2026-04-30

## Overview

Show a funny, sassy tag in the active day's section header that reflects how many sets the user has completed today. Updates live as sets are checked. No tag on rest days (Day 0).

## Placement

Appended to `.day-header` of the active day section, after the existing `.day-type-tag`. Same row, right-aligned via flex.

## Tiers

Progress = checked sets / total sets in active day section (simple ratio, no weighting).

| % Checked | EN | RU |
|---|---|---|
| 0% | 🦥 Lazy Sloth | 🦥 Диванный Боец |
| 1–20% | 🥱 Just Showed Up | 🥱 Зашёл Поглазеть |
| 21–50% | 🤷 Half-Hearted Hero | 🤷 Ну Типа Старался |
| 51–79% | 💦 Sweaty Mess | 💦 Уже Не Позор |
| 80–99% | 😤 One More Set, Coward | 😤 Ещё Подход, Слабак |
| 100% | 🔥 Absolute Unit | 🔥 Красавчик |

## Styling

- Pill tag matching `.day-type-tag` visual style
- Default: muted/neutral colors
- 100%: green glow to celebrate completion

## Logic

- `getEffortTag(ratio)` — returns `{en, ru}` object based on ratio
- `updateEffortTag()` — queries checkboxes in active day section, computes ratio, renders `#effort-tag` span in `.day-header`
- Called on `DOMContentLoaded` after `restoreCheckboxState()`
- Called on every checkbox `change` event
- Bilingual: respects current language setting, updates on `switchLanguage()`

## Constraints

- No tag on Day 0 (rest — no checkboxes)
- Single `index.html` file, no build step
- No new dependencies
