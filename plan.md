# BREAKOUT Game - POV Proof of Concept

**🎯 TOP PRIORITY:** Proof of concept for the [Pretext Project](https://github.com/chenglou/pretext) - demonstrating **Persistence of Vision (POV)** principles through interactive animation.

## Core Mission
- Demonstrate POV principles (not just a game)
- Educational demonstration of frame-based animation
- Visual effects over gameplay complexity
- Simple paddle + ball (NO bricks)

---

## Tech Stack

**Core:**
- HTML5 Canvas API (2D rendering)
- Vanilla JavaScript ES6+ (no frameworks)
- requestAnimationFrame (60+ fps timing)

**Build Tool:** Vite (or none - single HTML file works)

**Project Structure:**
```
├── index.html
├── src/
│   ├── main.js        # Initialization
│   ├── game.js        # Game loop
│   ├── paddle.js      # Paddle logic
│   ├── ball.js        # Ball physics
│   ├── renderer.js    # POV effects
│   └── input.js       # Input handling
```

---

## Game Loop

```javascript
function gameLoop(timestamp) {
  const deltaTime = (timestamp - lastTimestamp) / 1000;

  updatePaddle(deltaTime);
  updateBall(deltaTime);
  checkCollisions();

  clearCanvas();
  drawMotionTrails(); // POV effect
  drawPaddle();
  drawBall();

  requestAnimationFrame(gameLoop);
}
```

---

## POV Effects (ESSENTIAL)

1. **Motion Trails** - Ball leaves fading trail (demonstrates eye retention)
2. **60+ fps Animation** - Smooth motion (flicker fusion threshold)
3. **Motion Blur** - Paddle blur when moving (temporal integration)
4. **Impact Flash** - Collision glow (afterimage phenomenon)

**Optional:** Flicker fusion, Phi phenomenon, Beta movement, Afterimage effects

---

## Technical Config

```javascript
const CONFIG = {
  CANVAS_WIDTH: 800,
  CANVAS_HEIGHT: 600,

  PADDLE_WIDTH: 100,
  PADDLE_HEIGHT: 15,
  PADDLE_SPEED: 500,

  BALL_RADIUS: 8,
  BALL_SPEED: 300,

  TRAIL_LENGTH: 15,
  TRAIL_FADE_RATE: 0.06,
  FPS_TARGET: 60,
};
```

---

## Key Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| Variable frame rates | Delta time for physics |
| Motion trail performance | Limit trail history, use `globalAlpha` |
| Ball tunneling | Continuous collision detection |
| Input lag | Independent input state updates |
| Visual persistence | Semi-transparent canvas clearing |

---

## Implementation Phases

1. **Setup:** Project + canvas + game loop + delta time
2. **Mechanics:** Paddle + ball + collisions
3. **POV Effects:** Motion trails + fade + impact feedback
4. **Polish:** States + UI + responsive + optimization
5. **Docs:** README + comments + POV explanation

---

## Success Criteria

✓ 60fps in modern browsers
✓ Instant paddle response
✓ **Motion trails clearly show POV** (PRIMARY GOAL)
✓ Clean, educational code
✓ **Successfully demonstrates pretext concept** (TOP PRIORITY)

---

## Resources

- [Pretext Project](https://github.com/chenglou/pretext) - Primary reference
- [MDN: Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [MDN: requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
- Classic BREAKOUT (Atari, 1976)

---

**Remember:** This is an educational POV demonstration, not a complete game. Focus on visual effects that showcase how discrete frames create continuous motion through persistence of vision.
