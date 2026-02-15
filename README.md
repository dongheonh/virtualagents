# Virtual Agents — Microrobot Swarm Simulator

> **Live demo:** [dongheonh.github.io/virtualagents](https://dongheonh.github.io/virtualagents)

<img width="1512" height="826" alt="virtualagent" src="https://github.com/user-attachments/assets/e60f2580-eb1f-4e6f-b5ff-cb6354d00888" />

A real-time, browser-based swarm simulator where every particle represents an **autonomous microrobot**. A grid of **virtual electromagnets** (the blue glowing lamps) observes the local density of microrobots and lights up when enough of them are nearby — giving visual evidence of the control layer that could drive such a system in the physical world.

Built with vanilla HTML5 Canvas. No dependencies.

---

## Table of Contents

1. [What This Simulates](#what-this-simulates)
2. [Physics Principles](#physics-principles)
3. [How the Blue Lamps (Virtual Electromagnets) Work](#how-the-blue-lamps-virtual-electromagnets-work)
4. [Comparison with Existing Swarm Simulators](#comparison-with-existing-swarm-simulators)
5. [Parameters](#parameters)
6. [Controls](#controls)

---

## What This Simulates

Each moving dot is a **microrobot** — a miniature agent that:

- has a **position** $(x, y)$ and **velocity** $(v_x, v_y)$
- has an internal **oscillation phase** $\theta$ that can synchronise with neighbours
- responds to short-range repulsion, medium-range attraction, and an external mouse attractor

The system is a 2-D **Swarmalator** — a model where spatial swarming and phase synchronisation are coupled together. Clusters that form are not just spatial; they are also *phase-coherent*, meaning the microrobots inside them oscillate in lockstep.

---

## Physics Principles

### State Variables

Each microrobot $i$ carries:

| Symbol | Meaning |
|--------|---------|
| $(x_i, y_i)$ | Position on the canvas |
| $(v_{x,i},\ v_{y,i})$ | Velocity |
| $\theta_i \in [0, 2\pi)$ | Internal oscillation phase |

---

### Force Equations (per frame, $\Delta t = 1$)

#### 1. Mouse Attraction

$$
\mathbf{f}_{\text{mouse},i} = F_{\mu} \left(1 - \frac{d_{\mu,i}}{R_\mu}\right) \hat{\mathbf{r}}_{\mu,i}
\quad \text{if } d_{\mu,i} < R_\mu
$$

| Symbol | Value | Meaning |
|--------|-------|---------|
| $F_\mu$ | 0.25 (default) | Mouse force coefficient |
| $R_\mu$ | 200 px | Mouse interaction radius |
| $d_{\mu,i}$ | — | Distance from robot $i$ to cursor |
| $\hat{\mathbf{r}}_{\mu,i}$ | — | Unit vector toward cursor |

Force is zero at the edge of the range and maximal directly under the cursor — a **linear radial attractor**.

---

#### 2. Short-Range Repulsion (Collision Avoidance)

For any pair $(i, j)$ with $d_{ij} < d_{\text{sep}}$:

$$
\mathbf{f}_{\text{rep},ij} = F_s \left(1 - \frac{d_{ij}}{d_{\text{sep}}}\right) \hat{\mathbf{r}}_{ij}
$$

Applied **symmetrically**: robot $i$ is pushed away and robot $j$ is pushed toward. This prevents overlapping while keeping the interaction purely local.

| Symbol | Value | Meaning |
|--------|-------|---------|
| $F_s$ | 0.15 (default) | Separation force coefficient |
| $d_{\text{sep}}$ | $R + 12R = 2r_{\min}$ | Separation activation distance |

---

#### 3. Phase-Coupled Attraction (Swarmalator Core)

For pairs with $d_{\text{sep}} \le d_{ij} \le d_{\text{att}}$, the attraction force is **modulated by phase alignment**:

$$
\text{sync}(\theta_i, \theta_j) = \frac{1 + \cos(\theta_j - \theta_i)}{2} \in [0,\ 1]
$$

$$
\mathbf{f}_{\text{att},ij} = F_a \cdot \text{sync}(\theta_i, \theta_j) \cdot \hat{\mathbf{r}}_{ij}
$$

- When $\theta_i = \theta_j$ (in phase): $\text{sync} = 1$ → full attraction
- When $|\theta_i - \theta_j| = \pi$ (antiphase): $\text{sync} = 0$ → no attraction

This means **only phase-aligned microrobots attract each other**, driving the emergent formation of synchronised clusters.

| Symbol | Value | Meaning |
|--------|-------|---------|
| $F_a$ | 0.014 | Attraction force magnitude |
| $d_{\text{att}}$ | 50 px | Maximum attraction distance |

---

#### 4. Phase Synchronisation (Kuramoto-style)

Simultaneously with positional coupling, phases are updated:

$$
\dot{\theta}_i = K \sin(\theta_j - \theta_i), \quad \dot{\theta}_j = -K \sin(\theta_j - \theta_i)
$$

| Symbol | Value | Meaning |
|--------|-------|---------|
| $K$ | 0.0008 | Phase coupling strength |

This is the **Kuramoto coupling rule** applied pairwise. Over time, spatially close robots converge to the same phase — reinforcing the spatial cohesion through the sync term above.

---

#### 5. Velocity Update

$$
\mathbf{v}_i \leftarrow \gamma \, \mathbf{v}_i + \sum \mathbf{f}
$$

$$
\mathbf{v}_i \leftarrow \mathbf{v}_i \cdot \min\!\left(1,\ \frac{v_{\max}}{|\mathbf{v}_i|}\right)
$$

| Symbol | Value | Meaning |
|--------|-------|---------|
| $\gamma$ | 0.96 | Damping factor (4% energy loss per frame) |
| $v_{\max}$ | 4 px/frame | Speed cap |

---

#### 6. Position Update (Toroidal Topology)

$$
\mathbf{x}_i \leftarrow (\mathbf{x}_i + \mathbf{v}_i) \bmod (W, H)
$$

Microrobots that leave one edge reappear on the opposite side, eliminating boundary effects.

---

## How the Blue Lamps (Virtual Electromagnets) Work

The blue glowing circles are a **grid of virtual electromagnets** — the "virtual agents" of the title. They represent the external control infrastructure (e.g., a PCB coil array underneath a working surface) that a real microrobot system would use.

### Grid Layout

Electromagnets are arranged in a **hexagonally-offset grid**:

- Horizontal spacing: 88 px
- Vertical spacing: 68 px
- Odd rows are shifted right by 44 px (half-pitch offset)

This produces a layout that matches the natural packing density of a uniform magnetic coil array.

### Density Sensing

Every frame, each electromagnet counts how many microrobots are within its detection radius:

$$
c_m = \left| \left\{ i \ :\ (x_i - x_m)^2 + (y_i - y_m)^2 < R_d^2 \right\} \right|
$$

| Symbol | Value | Meaning |
|--------|-------|---------|
| $R_d$ | 62 px | Detection radius |
| $c_m$ | — | Local microrobot count for magnet $m$ |

### Brightness Activation

The raw activation level is normalised and passed through a power curve:

$$
\text{raw}_m = \text{clip}\!\left(\frac{c_m - c_{\min}}{c_{\max} - c_{\min}},\ 0,\ 1\right)
$$

$$
\text{target}_m = \text{raw}_m^{\ 0.55}
$$

The exponent $< 1$ compresses the response — the lamp reacts quickly at low densities and saturates gracefully at high densities.

| Symbol | Value | Meaning |
|--------|-------|---------|
| $c_{\min}$ | 5 | Minimum count to activate |
| $c_{\max}$ | 14 | Count at full brightness |

### Smooth Brightness Transition

To avoid flickering, brightness is low-pass filtered each frame:

$$
b_m \leftarrow b_m + (\text{target}_m - b_m) \times 0.08
$$

This is a simple **exponential moving average** with a time constant of about 12 frames (~200 ms at 60 fps).

### Visual Rendering

When $b_m > 0.015$, a radial gradient is drawn:

| Radius fraction | Alpha |
|-----------------|-------|
| 0% (centre) | $0.92 \cdot b_m$ |
| 45% | $0.55 \cdot b_m$ |
| 80% | $0.18 \cdot b_m$ |
| 100% (edge) | 0 |

The resulting soft blue glow is colour `rgb(80, 200, 255)` in dark mode and `rgb(30, 100, 255)` in light mode — chosen to evoke the visible emission spectrum of a magnetic field indicator lamp.

### Role in the System

The electromagnets are **reactive observers**, not directive actuators in this simulation. Each one independently decides its brightness based only on local particle count. This models the first stage of a real closed-loop magnetic control system:

```
Microrobots move  →  Electromagnet senses density  →  Lamp activates  →  (future: coil energises  →  field applied)
```

---

## Comparison with Existing Swarm Simulators

| Feature | Reynolds Boids (1987) | Vicsek Model (1995) | Standard Swarmalator (Lee 2019) | **This Simulator** |
|---|---|---|---|---|
| **Agent model** | Heading + speed | Heading only | Position + phase | Position + phase + microrobot radius |
| **Attraction rule** | Distance-based cohesion | None (alignment only) | $\frac{J(1+K\cos\Delta\theta)}{r}$ | $F_a \cdot \frac{1+\cos\Delta\theta}{2}$ (pairwise, capped range) |
| **Repulsion rule** | Separation zone | Hard excluded volume | $-\frac{1}{r}$ (inverse distance) | Linear: $F_s(1-d/d_{\text{sep}})$ |
| **Phase dynamics** | None | None | Kuramoto mean-field | Pairwise Kuramoto: $K\sin(\Delta\theta)$ |
| **Phase effect on motion** | None | None | Modulates attraction | Modulates attraction via $\text{sync}=\frac{1+\cos\Delta\theta}{2}$ |
| **External control layer** | None | None | None | **Electromagnet grid (virtual agents)** |
| **Topology** | Bounded or periodic | Periodic | Periodic | **Toroidal wrap** |
| **Interactive input** | Rarely | No | No | **Live mouse attractor** |
| **Real-time parameter tuning** | Rarely | No | No | **Yes (4 sliders)** |
| **Target application** | Animation / games | Statistical physics | Theory | **Microrobot swarm visualisation** |

### Key Differences from the Standard Swarmalator (Lee et al.)

The canonical swarmalator uses a mean-field (all-pairs, distance-weighted) coupling:

$$
\dot{x}_i = \frac{1}{N}\sum_{j\neq i}\left[\frac{x_j-x_i}{|x_j-x_i|}\left(A + J\cos(\theta_j-\theta_i)\right) - \frac{x_j-x_i}{|x_j-x_i|^2}\right]
$$

This simulator uses a **range-limited, linearly-capped** formulation instead:

$$
\mathbf{f}_{ij} = \begin{cases}
-F_s\left(1 - \dfrac{d_{ij}}{d_{\text{sep}}}\right)\hat{\mathbf{r}}_{ij} & d_{ij} < d_{\text{sep}} \\[8pt]
+F_a \cdot \dfrac{1+\cos(\theta_j-\theta_i)}{2} \cdot \hat{\mathbf{r}}_{ij} & d_{\text{sep}} \le d_{ij} \le d_{\text{att}} \\[4pt]
\mathbf{0} & d_{ij} > d_{\text{att}}
\end{cases}
$$

This choice:
- is **O(N²)** but with early-exit once $d > d_{\text{att}}$, keeping it real-time for $N \leq 600$
- avoids the $1/r$ singularity at zero distance
- produces **visually stable**, discrete clusters rather than continuous density waves
- is easier to tune interactively

---

## Parameters

| Parameter | Default | Range | Effect |
|-----------|---------|-------|--------|
| Number of agents | 400 | 50 – 3000 | Total microrobot count; respawns on change |
| Agent diameter (mm) | 3 | 1 – 8 | Visual size and separation distance $d_{\text{sep}}$ |
| Mouse force $F_\mu$ | 0.25 | 0 – 2.0 | Strength of cursor attraction |
| Repulsion coefficient $F_s$ | 0.15 | 0 – 3.0 | Short-range repulsion strength |

---

## Controls

- **Mouse / trackpad hover** — acts as a mobile attractor; microrobots within 200 px are pulled toward the cursor
- **Theme toggle** (top-right button) — switches between dark and light mode
- **Sliders** (top-right panel) — adjust agent count, size, mouse force, and repulsion in real time

---

*DH Han · SAM Lab*
