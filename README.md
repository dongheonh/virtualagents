virtual swarmalator + electromagnet-density activation (README.txt)
DH Han, SAM Lab

1) what you are seeing
- Small gray circles: microrobots (agents).
- Blue glow circles behind them: electromagnets (a virtual hex/offset grid of coils).
- The blue intensity is NOT a direct “mouse attraction field”.
  It is activated by local microrobot density near each electromagnet center.
- The mouse applies an external attraction force to microrobots (for interaction),
  but electromagnet activation is computed from microrobot density only.

2) state variables (per agent i)
Position:
  x_i, y_i   (wrapped on a torus via modulo screen width/height)
Velocity:
  v_{x,i}, v_{y,i}
Phase:
  θ_i        (called phase in code)

3) interaction radii (from code)
- Mouse influence range: MOUSE_RANGE = 200 px
- Swarm interaction range: ATT_DIST = 50 px
- Repulsion (separation) range: SEP_DIST = MIN_DIST
  where
    R = agent radius in pixels (slider “agent diameter (mm)” is treated as pixel radius)
    MIN_GAP  = 12 R
    MIN_DIST = 2R + MIN_GAP = 14R
    SEP_DIST = 14R

Important note:
- This makes repulsion range grow linearly with agent radius.
- Attraction + phase coupling only occur for pairs with distance d_ij <= ATT_DIST.

4) simplified swarmalator dynamics implemented

4.1 mouse external forcing (per agent i)
Let m = (mouseX, mouseY), r_i = (x_i, y_i).
If d_i = ||m - r_i|| < MOUSE_RANGE and d_i > 1:

  f_mouse(d_i) = MOUSE_FORCE * (1 - d_i / MOUSE_RANGE)

Velocity update:
  v_i ← v_i + f_mouse(d_i) * (m - r_i) / d_i

This is a purely spatial external field; it does not change phase θ_i.

4.2 pairwise local coupling (for each unordered pair i<j)
Let r_ij = r_j - r_i, d_ij = ||r_ij||, n_ij = r_ij / d_ij.

Pairs are processed only if:
  1 < d_ij <= ATT_DIST

(A) short-range repulsion (piecewise linear in distance)
If d_ij < SEP_DIST:

  f_rep(d_ij) = SEP_F * (1 - d_ij / SEP_DIST)

Apply equal and opposite impulses:
  v_i ← v_i - f_rep(d_ij) * n_ij
  v_j ← v_j + f_rep(d_ij) * n_ij

This is a “soft” collision-avoidance force with finite range SEP_DIST
and finite maximum magnitude SEP_F as d_ij -> 0.

(B) phase-dependent attraction + phase synchronization (swarmalator-like)
Else (i.e., SEP_DIST <= d_ij <= ATT_DIST):

Define a phase-coherence weight:
  sync_ij = (1 + cos(θ_j - θ_i)) / 2
so sync_ij ∈ [0,1].

Spatial attraction magnitude:
  f_att = ATT_F * sync_ij
where ATT_F is a constant (ATT_F = 0.014 in code).

Velocity impulses:
  v_i ← v_i + f_att * n_ij
  v_j ← v_j - f_att * n_ij

Phase coupling (Kuramoto-like, local, no distance weighting):
  Δθ = 0.0008 * sin(θ_j - θ_i)

  θ_i ← θ_i + Δθ
  θ_j ← θ_j - Δθ

Key properties vs classical swarmalator:
- Local neighborhood only (cutoff at ATT_DIST).
- No explicit 1/d or 1/d^2 distance weighting inside the attraction/phase coupling.
- Repulsion is not singular; it is bounded and piecewise linear.
- Phase natural frequency is effectively ω_i = 0 (no ω_i term exists).

4.3 discrete-time integration used (velocity-damped update)
After accumulating forces from mouse and neighbors:

Damping:
  v_i ← DAMP * v_i     (DAMP = 0.96)

Speed clamp:
  if ||v_i|| > MAX_SPD then v_i ← (MAX_SPD / ||v_i||) v_i
  (MAX_SPD = 4 px/frame)

Position update:
  r_i ← r_i + v_i

Periodic boundary conditions (toroidal wrap):
  x_i ← (x_i + W) mod W
  y_i ← (y_i + H) mod H

This is not a continuous-time ODE integrator; it is a stable, real-time,
discrete-time, damped “velocity + impulse” scheme.

5) electromagnet grid (blue circles) and activation rule

5.1 electromagnet placement
Electromagnets are placed on a staggered (offset) grid:

- Horizontal spacing: MAG_SX = 88 px
- Vertical spacing:   MAG_SY = 68 px
- Each row is offset by MAG_SX/2 for a hex-like packing:
    offset = (row is odd) ? MAG_SX/2 : 0
- Magnet visual radius: MAG_R = 20 px

Magnets are regenerated on resize and cover the viewport with margins.

5.2 density detection around each magnet
Each magnet k has center c_k = (x_k, y_k).
You count microrobots within a fixed detection radius:

- Detection radius: DETECT_R = 62 px
- Condition: ||r_i - c_k||^2 < DETECT_R^2

Let count_k = number of agents inside this radius.

5.3 thresholding + normalization
Activation uses a thresholded, normalized density:

Parameters:
  DENSITY_THRESHOLD = 5
  DENSITY_MAX       = 14

Raw normalized density:
  raw_k = clamp( (count_k - DENSITY_THRESHOLD) / (DENSITY_MAX - DENSITY_THRESHOLD), 0, 1 )

Then apply a gamma-like nonlinearity (makes activation rise faster near onset):
  target_k = raw_k^(0.55)

5.4 temporal smoothing (low-pass filter)
Each magnet has a persistent brightness b_k in [0,1] updated as:

  b_k ← b_k + 0.08 * (target_k - b_k)

So brightness responds smoothly rather than flickering.

5.5 rendering meaning
- A faint ring is always drawn for each electromagnet.
- When b_k is above a small value, a blue radial glow is drawn with opacity ~ b_k.
- Therefore, “blue behind the swarm” indicates regions where local microrobot density
  exceeded the threshold near electromagnet centers.

Important: In the current code, microrobots do NOT feel forces from electromagnets.
Electromagnets are a visualization layer driven by swarm density.

6) simplified swarmalator model summary (code-faithful)

Your implementation can be summarized as:

For each frame t:
1) For each agent i:
   Add external mouse impulse if within MOUSE_RANGE.

2) For each pair (i,j) with d_ij <= ATT_DIST:
   If d_ij < SEP_DIST:
     apply bounded repulsion proportional to (1 - d_ij/SEP_DIST)
   else:
     apply attraction proportional to (1 + cos(θ_j - θ_i))/2
     update phases via sin(θ_j - θ_i) (local Kuramoto-like)

3) Dampen velocities, clamp speed, update positions with wrap-around.

4) Electromagnets:
   For each magnet center, count agents within DETECT_R, convert count to brightness
   using threshold + normalization + power law + smoothing, then draw blue glow.

7) computational note (important if you scale up)
- Density computation currently loops over (agents × magnets), i.e. O(N*M),
  with M being number of magnet nodes.
- Pairwise interaction loops over all pairs O(N^2) but early-exits when d > ATT_DIST;
  however the distance is still computed for each pair you iterate over.
For very large N (e.g., 3000), consider spatial hashing / uniform grid to reduce cost.

8) controls (sliders)
- number of agents: reseeds positions and velocities uniformly over the screen.
- agent diameter (mm): used as pixel radius R; it also scales SEP_DIST = 14R.
- mouse force: scales external attraction to mouse cursor.
- repulsion coefficient: scales separation strength SEP_F.

9) terminology mapping for your project
- “virtual swarmalator”: microrobots with coupled space-phase dynamics (Sections 4.2–4.3).
- “electromagnets activate”: blue circles whose brightness is driven by local microrobot density
  around each magnet center (Section 5). This is a visualization of where coils would turn on.

end of file





dongheonh.github.io/virtualagents

<img width="1512" height="826" alt="virtualagent" src="https://github.com/user-attachments/assets/e60f2580-eb1f-4e6f-b5ff-cb6354d00888" />
