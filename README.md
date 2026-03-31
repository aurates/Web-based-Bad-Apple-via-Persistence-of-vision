# POS-Breakout Demo

A demonstration of the [Pretext](https://github.com/chenglou/pretext) text measurement library using a simple Breakout game.

## What is Pretext?

Pretext is a high-performance text measurement and layout library that avoids expensive DOM reflow operations by using Canvas `measureText()` internally. It features a two-phase architecture:
- **prepare()**: One-time text analysis (expensive, cached)
- **layout()**: Fast arithmetic calculations (pure arithmetic, no DOM reads)

## What This Demo Shows

This interactive demo demonstrates:
1. **Real-time text measurement** using Pretext's APIs
2. **Performance metrics** showing prepare() and layout() timing
3. **A playable Breakout game** rendered using text characters
4. **Live bounce counter** tracking successful paddle hits

The game displays performance metrics at 60fps, showing how Pretext can efficiently measure and layout text without triggering DOM reflow.

## Running the Demo

### Prerequisites
- Node.js 16+ and npm

### Setup
```bash
# Install dependencies
npm install

# Start development server
npm run dev
```

Then open your browser to `http://localhost:5173`

### Build for Production
```bash
npm run build
npm run preview
```

## Game Controls
- **Left Arrow (←)**: Move paddle left
- **Right Arrow (→)**: Move paddle right
- **Objective**: Bounce the ball ● back up using the paddle ━

## Mobile Device Support
- **Desktop browsers**: Full support, game starts immediately
- **Mobile devices with keyboard**: Supported! Press any arrow key or click "I Have a Keyboard - Continue" button to play
- **Mobile devices without keyboard**: Not recommended (touch controls not available)
- The game automatically detects arrow key presses and enables gameplay on mobile devices with attached keyboards

## Technical Details

- **Layout**: 90% game area, 10% metrics display
- **Characters**: U+25A0 (■) background, U+25CF (●) ball, ━ paddle, ▓ walls
- **Performance**: 60fps with requestAnimationFrame
- **Mobile Support**: Desktop browsers recommended, mobile devices with attached keyboards supported

## License
See LICENSE file for details.

## Links
- [Pretext Repository](https://github.com/chenglou/pretext)
- [Pretext Demos](https://chenglou.me/pretext/)
