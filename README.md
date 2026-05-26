# Build a Flappy Bird Clone with Pygame
### A Step-by-Step Tutorial for Beginners


---


## Before You Start


**What you need:**
- Python 3 installed on your computer
- Pygame installed — open a terminal and run:
 ```
 pip3 install pygame
 ```
- A text editor (e.g. VS Code, IDLE, Thonny)


**What you will build:** 
A working Flappy Bird game, one small piece at a time. Each step adds something new and the game runs at the end of every step. The goal of this is to get you thinking about how a game engine works, how a game loop works, and how might you build another game in the future if you wanted to.


**The finished file** is `flappy_bird.py` — you can look at it any time, but try to type each step yourself first.


---


## Table of Contents


- [Step 1 — Open a Window](#step-1--open-a-window)
- [Step 2 — Draw the Bird](#step-2--draw-the-bird)
- [Step 3 — Make the Bird Fall (Gravity)](#step-3--make-the-bird-fall-gravity)
- [Step 4 — Flap on Spacebar](#step-4--flap-on-spacebar)
- [Step 5 — Add the Ground and a Ceiling Check](#step-5--add-the-ground-and-a-ceiling-check)
- [Step 6 — Add One Pipe](#step-6--add-one-pipe)
- [Step 7 — Collision with the Pipe](#step-7--collision-with-the-pipe)
- [Step 8 — Keep Score](#step-8--keep-score)
- [Step 9 — Random Gap Position](#step-9--random-gap-position)
- [Step 10 — Game Over Screen](#step-10--game-over-screen-instead-of-quitting)
- [The Complete File](#the-complete-file)
- [Pygame Concepts Summary](#pygame-concepts-summary)
- [Extension Challenges](#extension-challenges)


---


## Step 1 — Open a Window


Create a new file called **`my_flappy.py`** and type this:


```python
import pygame
import sys


pygame.init()                                      # start pygame


screen = pygame.display.set_mode((400, 600))       # create a 400×600 window
pygame.display.set_caption("Flappy Bird")          # window title


clock = pygame.time.Clock()                        # used to control speed


# --- Game loop ---
while True:
   # 1. Handle events
   for event in pygame.event.get():
       if event.type == pygame.QUIT:              # X button clicked
           pygame.quit()
           sys.exit()


   # 2. Draw
   screen.fill((112, 197, 206))                   # paint sky blue background


   # 3. Show the frame
   pygame.display.flip()
   clock.tick(60)                                 # run at 60 frames per second
```


**Run it:** `python3 my_flappy.py` 
You should see a blue window. Close it with the ✕ button.


### What's happening?
| Line | Explanation |
|---|---|
| `pygame.init()` | Starts all pygame systems — always the first line |
| `set_mode((400, 600))` | Creates the window. The numbers are width × height in pixels |
| `while True:` | The **game loop** — runs 60 times per second forever |
| `pygame.event.get()` | Collects anything that happened (key press, mouse click, close) |
| `screen.fill(colour)` | Wipes the canvas before drawing. Colour is `(red, green, blue)`, each 0–255 |
| `pygame.display.flip()` | Sends the finished drawing to the screen |
| `clock.tick(60)` | Waits until 1/60th of a second has passed so the loop doesn't run too fast |


---


## Step 2 — Draw the Bird


Add these lines **above** the `while True:` line:


```python
# Bird position
bird_x = 80
bird_y = 300
```


Then, **inside the game loop**, replace `# 2. Draw` with:


```python
   # 2. Draw
   screen.fill((112, 197, 206))                   # sky


   # Draw the bird as a yellow circle
   pygame.draw.circle(screen, (255, 220, 50), (bird_x, bird_y), 18)
```


**Run it** — you should see a yellow circle on the left side of the window.


### What's happening?
- `pygame.draw.circle(surface, colour, centre, radius)`
- `centre` is an `(x, y)` tuple. In pygame, **x=0 is the left edge** and **y=0 is the top edge**. y increases *downward*.


```
(0,0) ──────────── x increases →
 |
 |   (80, 300) ← that's where the bird is
 y
 increases
 downward
```


---


## Step 3 — Make the Bird Fall (Gravity)


Add two variables **above** `while True:`:


```python
bird_vel = 0          # current vertical speed (pixels per frame)
GRAVITY  = 0.45       # added to bird_vel every frame
```


Inside the game loop, **before** the draw section, add:


```python
   # --- Update ---
   bird_vel += GRAVITY        # gravity pulls the bird down
   bird_y   += bird_vel       # move the bird by its velocity
```


**Run it** — the bird should fall off the bottom of the screen.


### What's happening?
Every frame we add `GRAVITY` to `bird_vel`, making the bird speed up as it falls — just like real gravity. 
`bird_y += bird_vel` moves the bird by however fast it's currently going.


---


## Step 4 — Flap on Spacebar


Add a constant **above** `while True:`:


```python
FLAP_STRENGTH = -8.5   # negative = upward (remember y=0 is at the top!)
```


Inside the event loop, **after** the `pygame.QUIT` check, add:


```python
       if event.type == pygame.KEYDOWN:
           if event.key == pygame.K_SPACE:
               bird_vel = FLAP_STRENGTH   # kick the bird upward
```


**Run it** — press Space to flap. The bird still falls off the screen eventually, but now you can fight gravity!


### What's happening?
When Space is pressed we set `bird_vel` to a **negative** number. Because y increases downward, a negative velocity moves the bird *upward*.


---


## Step 5 — Add the Ground and a Ceiling Check


Add this constant **above** `while True:`:


```python
GROUND_Y = 540   # y-position where the ground starts
```


In the **draw section**, add the ground after drawing the sky but before `pygame.display.flip()`:


```python
   pygame.draw.rect(screen, (210, 180, 140), (0, GROUND_Y, 400, 600 - GROUND_Y))
```


In the **update section**, add collision checks:


```python
   # Kill the bird if it hits the ground or flies off the top
   if bird_y + 18 >= GROUND_Y or bird_y < 0:
       print("Game Over!")
       pygame.quit()
       sys.exit()
```


**Run it** — the game now ends when the bird leaves the play area.


### `pygame.draw.rect`
```
pygame.draw.rect(surface, colour, (x, y, width, height))
```
The rectangle starts at `(x, y)` and extends `width` pixels right and `height` pixels down.


---


## Step 6 — Add One Pipe


Add these variables **above** `while True:`:


```python
pipe_x     = 500    # start off the right edge
pipe_gap   = 155    # size of the gap the bird flies through
gap_centre = 300    # y-position of the middle of the gap
PIPE_SPEED = 3      # pixels the pipe moves left each frame
```


In the **update section**, move the pipe:


```python
   pipe_x -= PIPE_SPEED
```


In the **draw section** (after the sky fill, before drawing the bird), draw the pipe:


```python
   # Top pipe
   pygame.draw.rect(screen, (83, 166, 81),
                    (pipe_x, 0, 60, gap_centre - pipe_gap // 2))
   # Bottom pipe
   pygame.draw.rect(screen, (83, 166, 81),
                    (pipe_x, gap_centre + pipe_gap // 2, 60, 600))
```


**Run it** — one green pipe scrolls past. The bird can fly through the gap!


---


## Step 7 — Collision with the Pipe


Add this to the **update section** (after moving the pipe):


```python
   # Build rectangles for the pipes
   top_pipe_rect    = pygame.Rect(pipe_x, 0,
                                  60, gap_centre - pipe_gap // 2)
   bottom_pipe_rect = pygame.Rect(pipe_x, gap_centre + pipe_gap // 2,
                                  60, 600)
   bird_rect = pygame.Rect(bird_x - 14, bird_y - 14, 28, 28)


   if bird_rect.colliderect(top_pipe_rect) or bird_rect.colliderect(bottom_pipe_rect):
       print("Hit a pipe! Game Over.")
       pygame.quit()
       sys.exit()
```


**Run it** — flying into the pipe now ends the game.


### `pygame.Rect` and `colliderect`
```python
rect = pygame.Rect(x, y, width, height)
rect.colliderect(other_rect)    # returns True if they overlap
```
This is the standard way to check if two objects touch each other.


---


## Step 8 — Keep Score


Add a score variable and a font **above** `while True:`:


```python
score  = 0
scored = False
font   = pygame.font.SysFont("Arial", 48, bold=True)
```


In the **update section**, award a point when the bird passes the pipe:


```python
   if pipe_x + 60 < bird_x and not scored:
       score  += 1
       scored  = True


   if pipe_x < -60:
       pipe_x     = 500
       gap_centre = 300    # same gap for now
       scored     = False
```


In the **draw section**, show the score (add this just before `pygame.display.flip()`):


```python
   score_surf = font.render(str(score), True, (255, 255, 255))
   screen.blit(score_surf, (200 - score_surf.get_width() // 2, 20))
```


**Run it** — the score goes up each time you pass a pipe!


### `font.render` and `screen.blit`
```python
surf = font.render("text", True, colour)   # draw text onto a surface
screen.blit(surf, (x, y))                  # stamp that surface onto the screen
```


---


## Step 9 — Random Gap Position


Make the game interesting by changing the gap each time a new pipe arrives.


Add `import random` at the very **top** of the file (next to `import pygame`):


```python
import pygame
import sys
import random        # ← add this line
```


When the pipe resets (inside `if pipe_x < -60:`), change the `gap_centre` line to:


```python
       gap_centre = random.randint(150, 400)
```


**Run it** — each pipe now has the gap in a different place.


---


## Step 10 — Game Over Screen Instead of Quitting


Instead of quitting immediately on collision, show a "Game Over" message and wait for Space to restart.


**First**, add `game_over = False` with your other variables above the loop.


**Second**, replace **both** `pygame.quit() / sys.exit()` crash blocks with:


```python
       game_over = True
```


**Third**, wrap the entire update section so it only runs when the game is not over:


```python
   if not game_over:
       # ... paste all the update code here, indented one extra level ...
```


**Fourth**, in the **draw section** (just before `pygame.display.flip()`), add:


```python
   if game_over:
       msg      = font.render("Game Over!", True, (220, 50, 50))
       hint_fnt = pygame.font.SysFont("Arial", 22)
       hint     = hint_fnt.render("Press SPACE to restart", True, (255, 255, 255))
       screen.blit(msg,  (200 - msg.get_width()  // 2, 250))
       screen.blit(hint, (200 - hint.get_width() // 2, 320))
```


**Fifth**, in the **event section**, handle restart:


```python
       if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
           if game_over:
               bird_y     = 300
               bird_vel   = 0
               pipe_x     = 500
               gap_centre = 300
               score      = 0
               scored     = False
               game_over  = False
           else:
               bird_vel = FLAP_STRENGTH
```


> **Tip:** remove the old `if event.key == pygame.K_SPACE: bird_vel = FLAP_STRENGTH` line — it is now inside the `else` branch above.


**Run it** — you now have a complete, restartable Flappy Bird game!


---


## Pygame Concepts Summary


| Concept | What it does |
|---|---|
| `pygame.init()` | Start pygame — always first |
| `set_mode((w, h))` | Open a window |
| `clock.tick(60)` | Limit to 60 frames per second |
| `pygame.event.get()` | Get all input events this frame |
| `screen.fill(colour)` | Wipe the screen before drawing |
| `pygame.draw.circle(...)` | Draw a circle |
| `pygame.draw.rect(...)` | Draw a rectangle |
| `pygame.Rect(x, y, w, h)` | A rectangle with collision helpers |
| `rect.colliderect(other)` | Check if two rectangles overlap |
| `font.render(text, aa, colour)` | Turn text into a drawable surface |
| `screen.blit(surf, (x, y))` | Stamp a surface onto the screen |
| `pygame.display.flip()` | Show the finished frame |


---


## Extension Challenges


Try these once the basic game is working:


1. **Speed up over time** — increase `PIPE_SPEED` by 0.5 every 5 points.
2. **Two pipes on screen** — keep a list of pipe positions instead of one variable.
3. **Sound** — add `pygame.mixer.Sound("flap.wav").play()` on every flap.
4. **High score** — save the best score to a `.txt` file between sessions.
5. **Sprite image** — load a PNG with `pygame.image.load("bird.png")` and draw it with `screen.blit()` instead of a circle.

---

## Grading

- Code complete and running (10 marks)
- Comments done ( 5 marks)
- Choose and complete **one extension challenge** from above. (5 marks)
