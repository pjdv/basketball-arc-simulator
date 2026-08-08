# Basketball Arc Simulator

An interactive, single-file simulator that demonstrates **why a higher shooting arc gives a
basketball more room for error** — and what that arc costs you in force precision.

Open `index.html` in any modern browser. No build step, no dependencies, no network access.

```bash
open index.html
```

---

## The core idea

A regulation rim is **18 in** across; a men's basketball is **9.4 in** across. That looks like a
generous 8.6 in of slack — but only if the ball falls straight down.

A ball arriving at entry angle **θₑ** doesn't see a circle. It sees the rim *projected onto the
plane perpendicular to its flight path*: an **ellipse**, still 18 in wide side-to-side, but only
`18·sin θₑ` deep along the direction of travel. Flatten the shot and that ellipse collapses.

| Entry angle | Effective depth of opening | Depth window for the ball's center |
|---|---|---|
| 90° (straight down) | 18.0 in | 8.6 in |
| 60° | 15.6 in | 7.1 in |
| 45° | 12.7 in | 4.7 in |
| 35° | 10.3 in | 1.6 in |
| **31.5°** | **9.4 in** | **0 in** |

Below **θₑ = asin(9.4/18) ≈ 31.5°** the ball physically cannot pass through without touching iron.
That is the origin of the coaching rule of thumb that a shot needs at least ~32–35° of entry angle.

---

## The physics, as implemented

All quantities are computed live from the slider values — nothing is table-driven.

### 1. Required launch velocity

With release height `h₀`, rim height `H`, ground distance `L`, and `Δy = H − h₀`, a projectile
launched at angle θ passes through the rim center when

```
v² = g·L² / ( 2·cos²θ · (L·tan θ − Δy) )
```

with `g = 9.8 m/s²`. The solution exists only when `L·tan θ > Δy` — i.e. you must out-angle the
straight line to the rim. As θ approaches that limit, required velocity diverges.

**Two solutions per speed.** Solving the same relation for θ instead of v gives a quadratic in
`tan θ` with two roots — the familiar low-arc / high-arc pair that reach the same target with the
same force. The app reports the companion angle for the current shot.

**Minimum-force angle:** `θ* = 45° + ½·atan(Δy / L)`.

### 2. Entry angle ≠ launch angle

Horizontal velocity is conserved, vertical velocity is not. At the rim plane:

```
t     = ( v·sin θ + √( v²sin²θ − 2g·Δy ) ) / g      (descending root)
θₑ    = atan( (g·t − v·sin θ) / (v·cos θ) )
```

**Entry is *shallower* than launch** whenever the rim sits above the release point. The ball is
still shedding vertical speed as it climbs while horizontal speed is untouched, so
`tan θₑ = √(v²sin²θ − 2gΔy) / (v·cos θ) < tan θ`. The relationship inverts only for a release
above rim height.

From a 2.15 m release at 5 m out: a **52° launch arrives at 42.6°**, and a **45° launch arrives at
just 32.6°** — barely inside the 31.5° limit. This is why "shoot at 45°" is ambiguous: 45° *at the
rim* takes roughly **54° at the hand**.

### 3. The swish window

Model the trajectory as locally straight where it crosses the rim plane (over the ~24 cm of the
ball's passage, parabolic curvature is negligible). The ball clears both rim edges when its
center crosses the rim plane within

```
depth window   W = D_rim − d_ball / sin θₑ      (shrinks as the shot flattens)
```

### 3b. Left and right — the full 2-D region

Treating left/right as a second independent window is wrong. For a ball centre crossing the rim
plane at `(x, y)` (depth along travel, lateral), the distance to the rim point at angle `φ` is

```
d²(φ) = sin²θₑ·(R·cos φ − x)² + (R·sin φ − y)²
swish ⇔ the centre crosses inside the ring AND min over φ of d(φ) ≥ ball radius
```

So the acceptance region is the rim's **ellipse (semi-axes `R·sinθₑ × R`) eroded by the ball
radius** — a lens, not a rectangle. Depth and lateral error **trade off**: miss slightly long and
you have less room left or right. On the depth axis this reduces exactly to `W` above.

On the lateral axis it equals `D − d` **only while `sin²θₑ ≥ d/D`**, i.e. above

```
θ_lateral = asin(√(d/D)) = asin(√(9.4/18)) = 46.3°
```

Above that, arc genuinely does nothing for left/right. Below it, a wide ball starts clipping the
near rim on the diagonal and the lateral window collapses too:

| Entry angle | Lateral window | vs. `D − d` = 21.8 cm |
|---|---|---|
| ≥ 46.3° | 21.84 cm | at its maximum |
| 42.6° | 21.41 cm | −0.4 cm |
| 40° | 20.42 cm | −1.4 cm |
| 35° | 15.49 cm | −6.4 cm |
| 33° | 10.89 cm | −11.0 cm |

The practical point: lateral room is a fixed **distance**, so the aim *precision* it demands scales
as `1/L`. The **Aim error left/right** slider sprays the shot by an azimuth `α`, which lands the
ball wide by `L·sin α` and long by `L(1 − cos α)`:

| Distance | Aim tolerance | Lateral room at the rim |
|---|---|---|
| 1 m | ±5.94° | ±10.4 cm |
| 3 m | ±2.02° | ±10.6 cm |
| 5 m | ±1.22° | ±10.6 cm |
| 7.24 m (three) | ±0.84° | ±10.7 cm |
| 10 m | ±0.61° | ±10.7 cm |

Arc cannot buy any of that back. Beyond mid-range, **left/right aim — not arc — is the binding
constraint.**

### 4. Room for error

Rather than linearizing, the app searches numerically:

- **Angular tolerance** — hold `v` at the required value, sweep θ outward in both directions, and
  find where the crossing point leaves the swish window. Each perturbed shot is re-evaluated with
  *its own* entry angle and *its own* window.
- **Velocity tolerance** — hold θ fixed and bisect on `v` for the same condition, reported as a
  percentage of required speed.

### 5. The tradeoff you can see

The three error measures do not peak in the same place — that disagreement is the whole point of
the chart.

- **Depth window** grows monotonically with arc. More arc is always a bigger target.
- **Velocity tolerance** tracks the window. Since range goes as `v²`, a fractional speed error
  moves the ball by `dx/x ≈ 2·dv/v`, so tolerance ≈ `W / 4L` — it rises with arc too.
- **Angular tolerance** does *not*. At the minimum-force angle `θ*`, range is stationary in angle
  (`∂x/∂θ = 0`), so **`θ*` is simultaneously the cheapest shot and the one most forgiving of
  launch-angle error**. Either side of it, angular tolerance falls away.

Computed by the app for a 2.15 m release at 5 m (`θ* = 50.1°`):

| Launch | Entry | Depth window | Angular tol. | Velocity tol. | Speed needed |
|---|---|---|---|---|---|
| 45° | 32.6° | 1.4 cm | ±0.12° | ±0.05% | 7.73 m/s |
| **50°** | 39.8° | 8.4 cm | **±2.33°** | ±0.34% | **7.66 m/s** |
| 55° | 46.9° | 13.0 cm | ±1.44° | ±0.56% | 7.72 m/s |
| 65° | 60.7° | 18.4 cm | ±0.66° | ±0.83% | 8.36 m/s |
| 75° | 73.5° | 20.8 cm | ±0.35° | ±0.98% | 10.15 m/s |

So the price of a rainbow is **not** force tolerance — it is aim precision and raw strength (75°
needs 32% more speed than 50°). The price of a flat shot is everything at once: below ~44° launch
here, the window is shut and no amount of precision helps. The practical sweet spot sits at or a
little above `θ*`, which lands entry angles in the 40–48° band that coaching data converges on.

---

## What's in the app

- **Side-view trajectory** — animated ball, apex marker, release geometry, backboard/rim, the
  shaded **cone of acceptance** (every launch angle that still swishes at the current speed), and
  the swish window drawn to scale at the rim.
- **Through the rim, actual scale** — a true-relative-size cross-section of the ball passing
  between the two rim edges, with ghost balls drawn at their exact grazing positions, shown side
  by side against a straight-down drop.
- **Ball's-eye view** — the rim as the ball sees it: full circle vs. foreshortened ellipse, with
  the ball's cross-section drawn where this shot actually goes.
- **Landing map** — the 2-D swish region (the eroded lens) with this shot's crossing point on it,
  so the depth/lateral tradeoff is visible directly.
- **Error-margin chart** — angular tolerance (°), velocity tolerance (%), and depth window (cm)
  across launch angles 20–80°, with the current angle and `θ*` marked.
- **Top-down court** — click or drag anywhere to reposition the shooter; distance and lateral
  offset update together.
- **Live metrics** — entry angle, effective opening, both clearances, required velocity, apex
  height, flight time, companion angle, minimum entry angle, and both tolerances.

### Parameters

| Group | Controls |
|---|---|
| Player | height 150–220 cm, release height above head 10–40 cm, distance 1–10 m, lateral offset ±7 m |
| Shot | launch angle 20–80°, aim error left/right ±3°, animation speed 0.1–3× |
| Equipment | rim height 2.0–3.5 m (10 ft = 3.05), rim diameter 14–24 in (18), ball diameter 6–12 in (9.4) |

---

## What's left out, and how much it matters

Stated plainly, because it bounds what the numbers mean.

### Air drag — the biggest omission

Not negligible. For a size-7 ball (m = 0.62 kg, r = 0.119 m, A = 0.045 m², C_D ≈ 0.54) at a
7.7 m/s release:

```
F_drag = ½·ρ·C_D·A·v² = 0.88 N   ≈ 14% of the ball's weight (6.1 N)
```

That shortens the range by a few percent, so a real shooter needs roughly **2–3% more launch speed
than this model reports** — which is larger than the ±0.34% velocity tolerance at `θ*`. Read the
absolute velocity figures as indicative, not as a target to train against. Drag also *steepens* the
descent slightly, which marginally *widens* the swish window — the model is conservative there.

### Backspin (Magnus) — real, but small in flight

A typical shot carries 2–3 rev/s of backspin. Spin parameter `S = ωr/v ≈ 0.23`, giving a lift
coefficient around `C_L ≈ 0.10`:

```
F_Magnus = ½·ρ·C_L·A·v² ≈ 0.16 N   ≈ 2.7% of weight, directed upward
```

So backspin generates **roughly a fifth of the drag force**, pointing up, partially cancelling
drag's range loss and flattening the descent by well under a degree of entry angle. Including it
would move the swish windows by millimetres.

**Crucially, it cannot change the central result.** The window `W = D − d/sin θₑ` is pure
projective geometry — it depends only on the entry angle, never on how the ball acquired it. Spin
shifts `θₑ` a fraction of a degree; it does not alter the relationship being demonstrated.

Where backspin genuinely earns its reputation is at **contact**, not in flight: a ball with
backspin that strikes the rim or backboard sheds energy and deflects downward into the cylinder —
the "shooter's roll" that converts near-misses into makes. That is a rim-interaction effect and
sits entirely outside a swish-only model. It is a good reason to treat these windows as a *lower
bound* on real forgiveness.

### Everything else

- **Swishes only.** No backboard bank, no rim contact. Real make rates exceed these windows.
- **Point-mass parabola**, with the ball's finite diameter reintroduced only at the rim.
- **Rim is an ideal circle** of zero thickness; the net is decorative.
- The shooting plane is the vertical plane through shooter and hoop center, so lateral offset
  changes shot distance, not the physics.
- Constant `g = 9.8 m/s²`; no wind.

## License

MIT
