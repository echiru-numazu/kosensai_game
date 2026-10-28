# タイ語であそぼう — Thai phrase booth (高専祭)

A small static website for a festival booth: visitors learn everyday Thai phrases from a phrase sheet, then play a word-ordering game. Anyone who clears every question gets a snack 🍬.

It has no build step and no server. Open `index.html` in a browser.

## Files

| File | What it is |
| --- | --- |
| `index.html` | Entrance page with two buttons: the game and the phrase sheet. |
| `thai-sentence-game.html` | **ならべかえゲーム.** Tap Thai word cards into the right order. Has score, combo, stars and an S/A/B/C rank. |
| `thai-phrase-matome.html` | **フレーズまとめ.** 19 phrases, each with Thai script, katakana, IPA, audio and a word-by-word breakdown. |
| `thai-phrasebook.html` | Older phrasebook page. Nothing links to it. |
| `audio/01.mp3` – `19.mp3` | Pronunciation audio. The number is the phrase number in the matome page. |
| `タイでよく使う表現.pdf` | The original handout the phrases come from. |
| `example.png` | Reference image. |

## Running it

- **Locally:** double-click `index.html`. The audio is embedded in the HTML as base64, so it plays from `file://` too.
- **At the booth:** use a tablet or phone in the browser. All of the pages are built for phone screens (tested at 320px and 360px wide).
- **Fonts:** pages load **Noto Sans JP** and **Noto Sans Thai Looped** from Google Fonts. Without internet the browser uses its default fonts, and everything still works.

## Booth operation

1. A visitor plays the game. The how-to popup appears the first time.
2. When all 12 questions are cleared, the clear screen says *「この画面をスタッフに見せて、お菓子をもらってね！」*. Check it and hand out the snack.
3. **Reset for the next visitor:** on the game board, tap **はじめから** twice (the first tap arms it, the second tap wipes it). You can also tap **🔄 もういちど** on the clear screen. Either one clears progress and shows the how-to popup again.

Progress is saved in the browser's `localStorage`:

- `thai-sentence-game/state/v2`: cleared questions, stars, score and combo
- `thai-sentence-game/howto-seen/v1`: whether the how-to popup has been shown
- `thai-sentence-game/mute/v1`: sound effects on or off (the 🔊 button)

## Game rules

- Placing the last card checks the answer automatically.
- **Wrong:** cards in the right position lock green and wrong cards bounce back. The combo resets.
- **Hint (💡):** locks the next correct card in place, but costs a star.
- **Stars per question:** 3 − mistakes − hints (minimum 1).
- **Points:** `(100 × cards + 200 no-miss bonus + speed bonus) × combo multiplier`
  - The speed bonus starts at 300 and goes down 15 points per second.
  - The combo grows only on answers with no mistakes and no hints. The multiplier is 1 + 0.5 × (combo − 1), up to ×3.
- **Rank** (based on total stars out of 36): S = all 36, A ≥ 80%, B ≥ 60%, C otherwise.
- **Effects:** sound effects are generated with Web Audio, so they need no files. Phones that support it also vibrate. Users who turn on "reduce motion" in their system settings don't see the confetti.

## Editing content

### Game questions
In `thai-sentence-game.html`, edit the `QUESTIONS` array:

```js
{tag:'レストラン', jp:'これをください', audio:10, note:'…shown after a correct answer…',
 words:[{t:'เอา',k:'アオ',m:'欲しい'},{t:'อันนี้',k:'アンニー',m:'これ'}]}
```

- `words`: list them **in the correct order**. The game shuffles them itself.
- Difficulty comes from the number of cards: 2 = ★, 3 = ★★, 4 = ★★★.
- `audio`: the phrase number of the mp3 to play after a correct answer.

### Phrase sheet
In `thai-phrase-matome.html`, edit the `PHRASES` array. Each phrase has:

- a group `g`
- Japanese `jp`
- `ipa`
- `words`, each with a role `r` (`s`, `v`, `o`, `neg`, `q`, `adj`, `aux`, `pol`, `req`, `greet`)
- an optional `lit` and `note`

Shared words are defined once in `W`.

### Audio
Both pages read audio from `window.AUDIO_DATA` (the `<script id="audio-data">` block) first. They fall back to `audio/NN.mp3` only when a number is missing there.

**If you replace an mp3, update the embedded base64 too.** Otherwise the old sound keeps playing. To get the new base64 string (PowerShell):

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("audio\10.mp3")) | Set-Clipboard
```

Then paste it after `data:audio/mpeg;base64,` for that number in **both** HTML files.

## Design notes

- **Colors:** light colors only. The game uses a bright arcade style. The phrase sheet and index use a mint and gold palette on a white or sand background.
- **Thai font:** a looped Thai font is used on purpose, because the loops make letters easier to tell apart for Japanese learners.
- **Buttons:** labels never wrap. Narrow screens get icon-over-label buttons in equal-width columns.
