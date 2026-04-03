Q: If with Codex, I opened up ClassicUO, would it be very hard to modify it so it would basically load art files that are twice the dimensions of everything? so basic tiles 88x88 instead of 44x44 [disclaimer, the following might be wrong]

A: **no, not very hard conceptually… but there are a few exact places you must get right** or it will break visually.

## What ClassicUO assumes (the important part)

ClassicUO is built around a **fixed tile system**, not just “images of a certain size.”

Key assumptions baked into the client:

1. Ground tiles are **44×44**
2. The projection math assumes that size (diamond grid)
3. Z-height, offsets, and draw order depend on that size
4. Art assets are not just scaled images, they are **positioned relative to tile bounds**

So if you simply swap in 88×88 PNGs, it will **not align correctly** without changes.

---

## What you would actually need to change

### 1. Tile size constant(s)

Somewhere in rendering code (often called things like):

* `TileWidth`
* `TileHeight`
* `CELL_SIZE`

You would:

* Change 44 → 88

This is the easy part.

---

### 2. World → screen projection math (this is critical)

ClassicUO uses the classic formula:

```
screen_x = (x - y) * (tile_width / 2)
screen_y = (x + y) * (tile_height / 2)
```

If you double tile size:

* You must ensure **every place using 44-derived math updates consistently**

Otherwise:

* tiles overlap wrong
* gaps appear
* movement offsets feel off

---

### 3. Art offsets (the part people underestimate)

UO art is not centered cleanly. It often uses:

* “draw offsets”
* “texture bounds”
* “origin points”

If your new tiles are just scaled versions:

* offsets must also scale ×2

Otherwise:

* objects float
* seams appear between tiles
* cliffs and walls break visually

---

### 4. Hitboxes / selection / pathfinding

These may rely on:

* tile size
* pixel bounds

If untouched:

* clicking tiles may feel “off”
* movement might still work (grid-based), but selection could mismatch visuals

---

### 5. UI zoom vs true resolution (important design choice)

You actually have two paths:

#### Option A — True 88×88 system (what you're asking)

* Modify engine
* Clean, native result
* More work, but correct foundation

#### Option B — Keep 44×44, scale rendering (much easier)

* Render at 2×
* No logic changes
* Works immediately

But:

* Less precise control
* Not ideal for editing tools

---

## Difficulty level (honest assessment)

If you are comfortable with:

* reading C# (ClassicUO codebase)
* searching for constants and render paths

Then:

* **Basic prototype:** 2–4 hours
* **Visually correct system:** 1–2 days
* **Fully polished (no edge bugs):** several days

---

## The hidden opportunity 

What you are proposing is actually powerful:

* 88×88 tiles = **modernized UO pipeline**
* Better for:

  * Godot integration
  * higher fidelity assets
  * map editor tooling 

You are essentially creating a **“UO-HD coordinate system”**

That aligns very well with:

* Godot 4.6 work
* custom editor idea
* a tool 

---

## Recommendation (decisive)

Do this:

1. Fork ClassicUO
2. Create a **branch: `tile88-experiment`**
3. Change:

   * tile constants
   * projection math
4. Test with:

   * simple flat tiles first (no statics)

Then iterate.
