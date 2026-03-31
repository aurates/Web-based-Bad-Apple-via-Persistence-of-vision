# Pretext Integration Plan for POV Breakout

## Understanding Pretext

**What Pretext Actually Is:**
- High-performance text measurement/layout library by Cheng Lou
- Avoids DOM reflow by pre-measuring text using Canvas `measureText()`
- Two-phase: `prepare()` (expensive, cached) + `layout()` (fast, pure arithmetic)
- **NOT about POV visual effects** - it's about text rendering performance

**Why It's Relevant:**
- Demonstrates efficient frame-based rendering patterns
- Shows how to avoid performance bottlenecks (DOM layout thrashing)
- Can be used for UI text (score, instructions) without slowing game loop
- Exemplifies the architecture we need: separate expensive setup from hot rendering

---

## Game Mechanics (As Requested)

### Controls
- **Left Arrow** - Move paddle left
- **Right Arrow** - Move paddle right

### Layout
- **Paddle** - Bottom of screen (moves left/right)
- **Ball** - Bounces around
- **Walls** - Top, Left, Right (ball bounces off these)
- **No Bottom Wall** - Ball falls = restart/lose

### Core Gameplay
1. Ball starts in center, moving at angle
2. User moves paddle to bounce ball back up
3. Ball bounces off top/left/right walls
4. If ball falls past paddle → restart

---

## Implementation Plan

### Phase 1: Basic Structure (No Pretext Yet)

**Files to Create:**
```
├── index.html           # Canvas + script loading
├── src/
│   ├── config.js        # Game constants
│   ├── input.js         # Keyboard handling (arrow keys)
│   ├── paddle.js        # Paddle state + movement
│   ├── ball.js          # Ball state + physics
│   ├── collision.js     # Wall + paddle collision
│   ├── renderer.js      # Canvas drawing + POV effects
│   └── game.js          # Main loop + coordination
└── package.json         # Dependencies (optional: pretext)
```

**config.js:**
```javascript
export const CONFIG = {
  CANVAS_WIDTH: 800,
  CANVAS_HEIGHT: 600,

  // Paddle
  PADDLE_WIDTH: 100,
  PADDLE_HEIGHT: 15,
  PADDLE_SPEED: 500,        // pixels per second
  PADDLE_Y: 550,            // Fixed Y position (bottom)

  // Ball
  BALL_RADIUS: 8,
  BALL_SPEED: 300,          // pixels per second

  // POV Effects
  TRAIL_LENGTH: 20,         // Number of trail positions
  TRAIL_FADE: 0.1,          // Canvas fade per frame
  MOTION_BLUR_THRESHOLD: 5, // Speed for blur effect

  FPS_TARGET: 60,
}
```

**input.js:**
```javascript
export class InputHandler {
  constructor() {
    this.keys = {
      ArrowLeft: false,
      ArrowRight: false,
    }

    this.setupListeners()
  }

  setupListeners() {
    window.addEventListener('keydown', (e) => {
      if (e.key in this.keys) {
        e.preventDefault()
        this.keys[e.key] = true
      }
    })

    window.addEventListener('keyup', (e) => {
      if (e.key in this.keys) {
        e.preventDefault()
        this.keys[e.key] = false
      }
    })
  }

  isLeftPressed() { return this.keys.ArrowLeft }
  isRightPressed() { return this.keys.ArrowRight }
}
```

**paddle.js:**
```javascript
import { CONFIG } from './config.js'

export class Paddle {
  constructor() {
    this.x = CONFIG.CANVAS_WIDTH / 2 - CONFIG.PADDLE_WIDTH / 2
    this.y = CONFIG.PADDLE_Y
    this.width = CONFIG.PADDLE_WIDTH
    this.height = CONFIG.PADDLE_HEIGHT
    this.velocity = 0
  }

  update(deltaTime, input) {
    // Update velocity based on input
    if (input.isLeftPressed()) {
      this.velocity = -CONFIG.PADDLE_SPEED
    } else if (input.isRightPressed()) {
      this.velocity = CONFIG.PADDLE_SPEED
    } else {
      this.velocity = 0
    }

    // Update position
    this.x += this.velocity * deltaTime

    // Clamp to screen bounds
    if (this.x < 0) this.x = 0
    if (this.x + this.width > CONFIG.CANVAS_WIDTH) {
      this.x = CONFIG.CANVAS_WIDTH - this.width
    }
  }

  getBounds() {
    return {
      left: this.x,
      right: this.x + this.width,
      top: this.y,
      bottom: this.y + this.height,
    }
  }
}
```

**ball.js:**
```javascript
import { CONFIG } from './config.js'

export class Ball {
  constructor() {
    this.reset()
    this.trail = [] // For POV effect
  }

  reset() {
    this.x = CONFIG.CANVAS_WIDTH / 2
    this.y = CONFIG.CANVAS_HEIGHT / 2

    // Random angle between 30-150 degrees (upward)
    const angle = (Math.random() * 120 + 30) * Math.PI / 180
    this.vx = Math.cos(angle) * CONFIG.BALL_SPEED
    this.vy = Math.sin(angle) * CONFIG.BALL_SPEED * -1 // Negative = up
  }

  update(deltaTime) {
    // Update position
    this.x += this.vx * deltaTime
    this.y += this.vy * deltaTime

    // Update trail for POV effect
    this.trail.push({ x: this.x, y: this.y, time: Date.now() })

    // Limit trail length
    if (this.trail.length > CONFIG.TRAIL_LENGTH) {
      this.trail.shift()
    }
  }

  getBounds() {
    return {
      x: this.x,
      y: this.y,
      radius: CONFIG.BALL_RADIUS,
    }
  }

  reverseX() { this.vx *= -1 }
  reverseY() { this.vy *= -1 }
}
```

**collision.js:**
```javascript
import { CONFIG } from './config.js'

export class CollisionDetector {
  checkWalls(ball) {
    const bounds = ball.getBounds()

    // Left wall
    if (bounds.x - bounds.radius < 0) {
      ball.x = bounds.radius
      ball.reverseX()
      return 'wall'
    }

    // Right wall
    if (bounds.x + bounds.radius > CONFIG.CANVAS_WIDTH) {
      ball.x = CONFIG.CANVAS_WIDTH - bounds.radius
      ball.reverseX()
      return 'wall'
    }

    // Top wall
    if (bounds.y - bounds.radius < 0) {
      ball.y = bounds.radius
      ball.reverseY()
      return 'wall'
    }

    return null
  }

  checkPaddle(ball, paddle) {
    const ballBounds = ball.getBounds()
    const paddleBounds = paddle.getBounds()

    // Simple AABB collision
    const ballBottom = ballBounds.y + ballBounds.radius
    const ballTop = ballBounds.y - ballBounds.radius
    const ballLeft = ballBounds.x - ballBounds.radius
    const ballRight = ballBounds.x + ballBounds.radius

    if (ballBottom >= paddleBounds.top &&
        ballTop <= paddleBounds.bottom &&
        ballRight >= paddleBounds.left &&
        ballLeft <= paddleBounds.right &&
        ball.vy > 0) { // Only collide when moving down

      // Bounce angle depends on hit position
      const hitPos = (ball.x - paddle.x) / paddle.width // 0 to 1
      const angle = (hitPos - 0.5) * 120 // -60 to +60 degrees
      const radians = angle * Math.PI / 180

      const speed = Math.sqrt(ball.vx * ball.vx + ball.vy * ball.vy)
      ball.vx = Math.sin(radians) * speed
      ball.vy = -Math.abs(Math.cos(radians) * speed) // Always upward

      return 'paddle'
    }

    return null
  }

  checkGameOver(ball) {
    return ball.y - ball.getBounds().radius > CONFIG.CANVAS_HEIGHT
  }
}
```

**renderer.js:**
```javascript
import { CONFIG } from './config.js'

export class Renderer {
  constructor(canvas) {
    this.canvas = canvas
    this.ctx = canvas.getContext('2d')

    // Set canvas size
    this.canvas.width = CONFIG.CANVAS_WIDTH
    this.canvas.height = CONFIG.CANVAS_HEIGHT
  }

  clear() {
    // POV Effect: Semi-transparent fill creates trails
    this.ctx.fillStyle = `rgba(0, 0, 0, ${CONFIG.TRAIL_FADE})`
    this.ctx.fillRect(0, 0, CONFIG.CANVAS_WIDTH, CONFIG.CANVAS_HEIGHT)
  }

  drawBall(ball) {
    // Draw motion trail (POV effect)
    ball.trail.forEach((point, i) => {
      const alpha = (i + 1) / ball.trail.length
      this.ctx.fillStyle = `rgba(255, 255, 255, ${alpha * 0.3})`
      this.ctx.beginPath()
      this.ctx.arc(point.x, point.y, CONFIG.BALL_RADIUS, 0, Math.PI * 2)
      this.ctx.fill()
    })

    // Draw current ball
    this.ctx.fillStyle = '#FFFFFF'
    this.ctx.beginPath()
    this.ctx.arc(ball.x, ball.y, CONFIG.BALL_RADIUS, 0, Math.PI * 2)
    this.ctx.fill()
  }

  drawPaddle(paddle) {
    // Motion blur effect (POV)
    const speed = Math.abs(paddle.velocity)
    if (speed > CONFIG.MOTION_BLUR_THRESHOLD) {
      const blurAmount = Math.min(speed / 100, 5)
      this.ctx.shadowBlur = blurAmount
      this.ctx.shadowColor = '#FFFFFF'
    }

    this.ctx.fillStyle = '#FFFFFF'
    this.ctx.fillRect(paddle.x, paddle.y, paddle.width, paddle.height)

    // Reset shadow
    this.ctx.shadowBlur = 0
  }

  drawWalls() {
    this.ctx.strokeStyle = '#333333'
    this.ctx.lineWidth = 2

    // Top wall
    this.ctx.beginPath()
    this.ctx.moveTo(0, 0)
    this.ctx.lineTo(CONFIG.CANVAS_WIDTH, 0)
    this.ctx.stroke()

    // Left wall
    this.ctx.beginPath()
    this.ctx.moveTo(0, 0)
    this.ctx.lineTo(0, CONFIG.CANVAS_HEIGHT)
    this.ctx.stroke()

    // Right wall
    this.ctx.beginPath()
    this.ctx.moveTo(CONFIG.CANVAS_WIDTH, 0)
    this.ctx.lineTo(CONFIG.CANVAS_WIDTH, CONFIG.CANVAS_HEIGHT)
    this.ctx.stroke()
  }
}
```

**game.js:**
```javascript
import { CONFIG } from './config.js'
import { InputHandler } from './input.js'
import { Paddle } from './paddle.js'
import { Ball } from './ball.js'
import { CollisionDetector } from './collision.js'
import { Renderer } from './renderer.js'

export class Game {
  constructor(canvas) {
    this.input = new InputHandler()
    this.paddle = new Paddle()
    this.ball = new Ball()
    this.collision = new CollisionDetector()
    this.renderer = new Renderer(canvas)

    this.lastTime = 0
    this.running = false
  }

  start() {
    this.running = true
    this.scheduleFrame()
  }

  scheduleFrame() {
    if (!this.running) return

    requestAnimationFrame((timestamp) => {
      this.update(timestamp)
      this.render()
      this.scheduleFrame()
    })
  }

  update(timestamp) {
    // Calculate delta time
    const deltaTime = this.lastTime === 0 ? 0 : (timestamp - this.lastTime) / 1000
    this.lastTime = timestamp

    // Skip first frame (delta time would be huge)
    if (deltaTime === 0) return

    // Update game objects
    this.paddle.update(deltaTime, this.input)
    this.ball.update(deltaTime)

    // Check collisions
    this.collision.checkWalls(this.ball)
    this.collision.checkPaddle(this.ball, this.paddle)

    // Check game over
    if (this.collision.checkGameOver(this.ball)) {
      this.ball.reset()
    }
  }

  render() {
    this.renderer.clear()
    this.renderer.drawWalls()
    this.renderer.drawBall(this.ball)
    this.renderer.drawPaddle(this.paddle)
  }
}
```

**index.html:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>POV Breakout - Pretext Proof of Concept</title>
  <style>
    body {
      margin: 0;
      padding: 20px;
      background: #000;
      display: flex;
      flex-direction: column;
      align-items: center;
      font-family: system-ui, sans-serif;
      color: #fff;
    }

    canvas {
      border: 2px solid #333;
      background: #000;
    }

    .info {
      margin-top: 20px;
      text-align: center;
      max-width: 800px;
    }

    h1 { margin: 0 0 10px 0; }
    p { margin: 5px 0; }
  </style>
</head>
<body>
  <h1>POV Breakout</h1>
  <p>Proof of Concept: Persistence of Vision Effects</p>

  <canvas id="game"></canvas>

  <div class="info">
    <p><strong>Controls:</strong> Left/Right Arrow Keys to move paddle</p>
    <p><strong>POV Effects:</strong> Motion trails, motion blur, 60+ fps animation</p>
    <p>Inspired by the <a href="https://github.com/chenglou/pretext" target="_blank">Pretext Project</a></p>
  </div>

  <script type="module">
    import { Game } from './src/game.js'

    const canvas = document.getElementById('game')
    const game = new Game(canvas)
    game.start()
  </script>
</body>
</html>
```

---

### Phase 2: Optional Pretext Integration (UI Text)

If you want to add score/instructions using Pretext:

**Install Pretext:**
```bash
npm install @chenglou/pretext
```

**Add UI Renderer (ui-renderer.js):**
```javascript
import { prepare, layout } from '@chenglou/pretext'

export class UIRenderer {
  constructor(ctx) {
    this.ctx = ctx
    this.preparedTexts = new Map()
  }

  // Prepare text once (expensive)
  prepareText(key, text, font) {
    if (!this.preparedTexts.has(key)) {
      this.preparedTexts.set(key, prepare(text, font))
    }
    return this.preparedTexts.get(key)
  }

  // Draw text (fast - no DOM layout)
  drawText(key, text, x, y, font = '20px Arial') {
    const prepared = this.prepareText(key, text, font)
    const { height } = layout(prepared, 800, 24)

    this.ctx.font = font
    this.ctx.fillStyle = '#FFFFFF'
    this.ctx.fillText(text, x, y)
  }

  drawScore(score) {
    this.drawText('score', `Score: ${score}`, 10, 30, '24px Arial')
  }

  drawFPS(fps) {
    this.drawText('fps', `FPS: ${fps}`, 10, 60, '16px Arial')
  }
}
```

**Why This Matters:**
- `fillText()` normally triggers DOM layout if you query text metrics
- Pretext pre-measures everything, avoiding reflow
- Keeps 60fps stable even with dynamic text

---

## POV Effects Explained

### 1. Motion Trails
- **Effect:** Ball leaves fading trail
- **Implementation:** Semi-transparent canvas clear + trail array
- **POV Principle:** Demonstrates eye retention (afterimage)

### 2. Smooth 60+ fps
- **Effect:** Imperceptible frame transitions
- **Implementation:** `requestAnimationFrame` + delta time
- **POV Principle:** Exceeds flicker fusion threshold (~50-60 Hz)

### 3. Motion Blur
- **Effect:** Paddle blurs when moving fast
- **Implementation:** Canvas `shadowBlur` based on velocity
- **POV Principle:** Temporal integration in visual system

### 4. Impact Flash (Optional)
- **Effect:** Brief glow on collision
- **Implementation:** Increase ball brightness for 2-3 frames
- **POV Principle:** Afterimage phenomenon

---

## Architecture Principles (From Pretext)

### Two-Phase Pattern
**PREPARE (Once):**
- Create canvas context
- Set up event listeners
- Pre-calculate constants

**HOT PATH (60+ fps):**
- Delta-time driven updates
- Pure arithmetic collision detection
- Canvas drawing (no DOM reads)

### Performance Rules
1. **Never read DOM during animation loop**
2. **Use delta time for frame-rate independence**
3. **Cache computed values when possible**
4. **Separate update from render**

---

## Testing the POV Effects

### Visual Confirmation
1. **Motion trails visible?** Ball should leave ghostly trail
2. **Smooth at 60fps?** No stuttering (check DevTools FPS counter)
3. **Paddle blur?** Move paddle fast → should blur
4. **Responsive input?** Arrow keys feel instant

### Browser DevTools
- **Performance tab:** Check for 60fps
- **Rendering:** Enable "FPS meter"
- **Memory:** Ensure no leaks (trail array stays bounded)

---

## Summary

**Direct Pretext Integration:** Optional, only for UI text rendering

**Pretext-Inspired Architecture:**
- Two-phase pattern (prepare vs. hot loop)
- Frame-based animation with `requestAnimationFrame`
- Avoid DOM layout during game loop
- Delta-time driven physics

**POV Effects (Core Feature):**
- Motion trails (your own implementation)
- 60+ fps smooth animation
- Motion blur
- These demonstrate persistence of vision

**Result:** A working Breakout game that demonstrates POV principles using efficient rendering patterns inspired by Pretext's architecture.
