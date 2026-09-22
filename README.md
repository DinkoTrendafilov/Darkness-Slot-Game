# 🦇 DARKNESS

A fully playable browser-based slot machine with 12 symbols, 20 paylines, free spins, and a bonus wheel. Built with vanilla HTML/CSS/JavaScript — no frameworks, no dependencies, single file.

## 🎮 Play Now

👉 **[▶️ PLAY DARKNESS](https://dinkotrendafilov.github.io/Darkness-Slot-Game/)**

Just click the link and the game opens directly in your browser. No install, no signup, no download.

## 🎰 Game Features

- **5×4 grid, 20 fixed paylines**
- **12 thematic symbols** — vampires, bats, coffins, blood moons, and more
- **Multiplier Streak system** — winning spins escalate from ×2 up to ×64
- **🦇 Bat Collection Bonus** — collect 5 bats on a line to trigger a weighted 15-segment wheel (×2 to ×64 multiplier, base ×500 bet/line)
- **⭐ Scatter Free Spins** — 4 scatters = 30 FS, 5 scatters = 200 FS, with retrigger capability
- **🎲 Gamble feature** — double-or-nothing dice minigame
- **Auto-spin, save/load, import/export** game state

## 📊 Math & RTP

| Metric | Value |
|---|---|
| RTP | **96.0%** (verified via 200M spin Monte Carlo) |
| Volatility | High (5.76x σ/μ index) |
| Max win (simulated) | 1,930× bet |
| NC bonus frequency | 1 in 1,454 spins |
| Scatter trigger | 1 in 702 spins |

All math verified with a Python simulation engine — **200,000,000 spins** tested, with 2,500 line patterns evaluated per spin.

## 🎮 How to Play

1. Open the game via the link above (or open `Darkness.html` locally)
2. Choose your bet (0.4 – 1,000 credits)
3. Press **SPIN** or **AUTO** for continuous play
4. Watch for bat symbols to complete the collection
5. Trigger scatters for free spins
6. Try the **GAMBLE** button after a win to double your credits

## 🛠 Tech Stack

- Pure vanilla JavaScript (ES6+)
- Web Audio API (procedural music & sound effects)
- Canvas 2D (bonus wheel, winlines, particle background)
- No build step, no dependencies, no frameworks

## 📁 Files

- `index.html` — the complete game (single file)
- `simulation.py` — Monte Carlo RTP verification engine

## 🔬 Simulation Results

The game's math has been verified with **200 million simulated spins**:

- **Simulated RTP:** 95.98%
- **Theoretical RTP:** 96.36%
- **Difference:** -0.38% (normal line-correlation effect)
- **Base line RTP:** 84.39%
- **Night Creatures (bat bonus) RTP:** 6.89%
- **Scatter / Free Spins RTP:** 4.70%
- **House Edge:** 4.02%

## ⚠️ Disclaimer

This is a **demo / portfolio project**. It uses virtual credits only and has no real-money gambling functionality. Not intended for commercial casino use without proper certification.

## 📜 License

MIT
