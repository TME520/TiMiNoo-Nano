# TiMiNoo (`timinoo.ino`) improvement ideas

## High-impact reliability fixes

1. **Replace `String` in game logic with integer pattern checks.**
   The slot result uses `String superReel = String(...) + ...`, which can fragment heap memory on AVR boards over time. Compare integer reel values directly instead. (`checkButton`, Catsino result logic).

2. **Use `>=` for time-step checks instead of `==`.**
   Status updates use exact equality like `if (frameCounter == lastCatHungerCheck + cat.hungerStep)`. If a frame is skipped, updates can be missed forever. Use `>=` and increment checkpoints safely.

3. **Add explicit parentheses in mixed `&&` / `||` win conditions.**
   Expressions such as `slotReel[0] == 0 && slotReel[1] == 0 || slotReel[1] == 0 && slotReel[2] == 0` rely on precedence and are easy to misread/modify incorrectly. Parenthesize pair checks for clarity and safety.

4. **Harden button input mode/electrical assumptions.**
   `pinMode(ButtonPin, INPUT)` with active-HIGH can float if no external pull resistor is present. Prefer `INPUT_PULLUP` + active-LOW logic (or clearly document required hardware resistor).

## Architecture & maintainability

5. **Replace magic game mode numbers with an enum.**
   `gameMode` values (`0,1,2,...,99`) are used in many branches and are hard to track. Introduce `enum GameMode { Idle=0, NeedAction=1, Feed=2, ... }`.

6. **Split monolithic `checkButton()` and `loop()` into smaller functions.**
   Input handling, state transitions, and reward calculations are deeply coupled. Extract `handleButtonInIdle`, `handleCatsinoSpinStop`, `updateNeeds`, `renderIdle`, etc.

7. **Centralize duplicated reward/stock update code.**
   Many branches repeat stock increments and score adds. Introduce helper methods (`addAllFood(int qty)`, `addScore(uint32_t delta)`, `awardFoodBySymbol(uint8_t s)`) to reduce bugs.

8. **Introduce compile-time constants for repeated limits.**
   Values like `3` (status max), `7` (slot icon count), and random bounds appear repeatedly. Consolidate into named constants.

## Data integrity & persistence

9. **Version EEPROM payload and validate ranges on load.**
   Signature-only checks can accept stale/corrupt data layouts. Add a struct version byte + simple checksum; clamp loaded fields to valid ranges.

10. **Use `EEPROM.update` or dirty-check writes.**
    Save currently writes full struct every time save is triggered. Compare and write only changed bytes to reduce EEPROM wear.

11. **Remove or implement unused autosave variables.**
    `lastSaveTime` / `saveInterval` are defined but not used in loop, which is confusing and suggests incomplete autosave behavior.

## Performance and memory footprint

12. **Move all static UI text to flash (`F(...)`).**
    Repeated literal strings in draw calls can consume RAM on AVR. Use flash-stored literals where APIs allow.

13. **Reduce RAM by narrowing scalar types.**
    Many counters/flags are `int` where `uint8_t`/`uint16_t` are enough. This can reclaim SRAM on constrained boards.

14. **Consider splitting huge bitmap assets into separate headers.**
    Keeping all art and logic in one file increases compile times and cognitive load. A dedicated `bitmaps.h` (PROGMEM only) improves readability.

## Suggested execution order

1. Reliability fixes (items 1–4).
2. Data integrity and EEPROM improvements (items 9–11).
3. Architecture cleanup (items 5–8).
4. Memory/readability refinements (items 12–14).
