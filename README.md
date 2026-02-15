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
4. [Comparison with Related Work](#comparison-with-related-work)
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

### Force Equations (per frame)

#### 1. Mouse Attraction

$$\mathbf{f}_{\text{mouse},i} = F_{\mu} \left(1 - \frac{d_{\mu,i}}{R_\mu}\right) \hat{\mathbf{r}}_{\mu,i} \quad \text{if } d_{\mu,i} < R_\mu$$

| Symbol | Value | Meaning |
|--------|-------|---------|
| $F_\mu$ | 0.25 (default) | Mouse force coefficient |
| $R_\mu$ | 200 px | Mouse interaction radius |
| $d_{\mu,i}$ | — | Distance from robot $i$ to cursor |
| $\hat{\mathbf{r}}_{\mu,i}$ | — | Unit vector toward cursor |

Force falls off linearly to zero at $R_\mu$ — a **linear radial attractor**.

---

#### 2. Short-Range Repulsion

For any pair $(i, j)$ with $d_{ij} < d_{\text{sep}}$:

$$\mathbf{f}_{\text{rep},ij} = F_s \left(1 - \frac{d_{ij}}{d_{\text{sep}}}\right) \hat{\mathbf{r}}_{ij}$$

Applied symmetrically to both robots.

| Symbol | Value | Meaning |
|--------|-------|---------|
| $F_s$ | 0.15 (default) | Separation force coefficient |
| $d_{\text{sep}}$ | $2r_{\min}$ | Repulsion activation distance |

---

#### 3. Phase-Coupled Attraction (Swarmalator Core)

For pairs with $d_{\text{sep}} \le d_{ij} \le d_{\text{att}}$:

$$\text{sync}_{ij} = \frac{1 + \cos(\theta_j - \theta_i)}{2} \in [0,\ 1]$$

$$\mathbf{f}_{\text{att},ij} = F_a \cdot \text{sync}_{ij} \cdot \hat{\mathbf{r}}_{ij}$$

- $\theta_i = \theta_j$ (in phase): $\text{sync} = 1$ → full attraction
- $|\theta_i - \theta_j| = \pi$ (antiphase): $\text{sync} = 0$ → no attraction

Only phase-aligned microrobots attract each other, driving emergent phase-coherent clusters.

| Symbol | Value | Meaning |
|--------|-------|---------|
| $F_a$ | 0.014 | Attraction force magnitude |
| $d_{\text{att}}$ | 50 px | Maximum attraction distance |

---

#### 4. Phase Synchronisation (Kuramoto)

$$\dot{\theta}_i = K \sin(\theta_j - \theta_i), \qquad \dot{\theta}_j = -K \sin(\theta_j - \theta_i)$$

| Symbol | Value | Meaning |
|--------|-------|---------|
| $K$ | 0.0008 | Phase coupling strength |

Pairwise Kuramoto coupling: spatially close robots converge to the same phase, reinforcing spatial cohesion through the $\text{sync}$ term above.

---

#### 5. Velocity Update

$$\mathbf{v}_i \leftarrow \gamma \, \mathbf{v}_i + \sum \mathbf{f}$$

$$\mathbf{v}_i \leftarrow \mathbf{v}_i \cdot \min\left(1,\ \frac{v_{\max}}{|\mathbf{v}_i|}\right)$$

| Symbol | Value | Meaning |
|--------|-------|---------|
| $\gamma$ | 0.96 | Damping (4% energy loss per frame) |
| $v_{\max}$ | 4 px/frame | Speed cap |

---

#### 6. Position Update (Toroidal Topology)

$$\mathbf{x}_i \leftarrow (\mathbf{x}_i + \mathbf{v}_i) \operatorname{mod} (W,\, H)$$

Robots leaving one edge reappear on the opposite side, eliminating boundary effects.

---

## How the Blue Lamps (Virtual Electromagnets) Work

The blue glowing circles are a **grid of virtual electromagnets** — the "virtual agents" of the title. They represent the external control infrastructure (e.g., a PCB coil array underneath a working surface) that a real microrobot system would use.

### Grid Layout

Electromagnets are arranged in a **hexagonally-offset grid**:

- Horizontal spacing: 88 px
- Vertical spacing: 68 px
- Odd rows shifted right by 44 px (half-pitch offset)

This matches the natural packing density of a uniform magnetic coil array.

### Density Sensing

Every frame, each electromagnet counts how many microrobots are within its detection radius:

$$c_m = \lvert \{ i : (x_i - x_m)^2 + (y_i - y_m)^2 < R_d^2 \} \rvert$$

| Symbol | Value | Meaning |
|--------|-------|---------|
| $R_d$ | 62 px | Detection radius |
| $c_m$ | — | Local microrobot count for magnet $m$ |

### Brightness Activation

The raw activation level is normalised and passed through a power curve:

$$\text{raw}_m = \operatorname{clip}\left(\frac{c_m - c_{\min}}{c_{\max} - c_{\min}},\ 0,\ 1\right)$$

$$\text{target}_m = \text{raw}_m^{0.55}$$

The exponent $< 1$ compresses the response: the lamp reacts quickly at low densities and saturates gracefully at high densities.

| Symbol | Value | Meaning |
|--------|-------|---------|
| $c_{\min}$ | 5 | Minimum count to activate |
| $c_{\max}$ | 14 | Count at full brightness |

### Smooth Brightness Transition

To avoid flickering, brightness is low-pass filtered each frame:

$$b_m \leftarrow b_m + (\text{target}_m - b_m) \times 0.08$$

Exponential moving average with a time constant of ~12 frames (~200 ms at 60 fps).

### Visual Rendering

When $b_m > 0.015$, a radial gradient is drawn centred on the electromagnet:

| Radius fraction | Alpha |
|-----------------|-------|
| 0 % (centre) | $0.92 \cdot b_m$ |
| 45 % | $0.55 \cdot b_m$ |
| 80 % | $0.18 \cdot b_m$ |
| 100 % (edge) | 0 |

Colour: `rgb(80, 200, 255)` in dark mode, `rgb(30, 100, 255)` in light mode.

### Role in the System

The electromagnets are **reactive observers**, not directive actuators in this simulation. Each one independently decides its brightness from local particle count alone — the first stage of a real closed-loop magnetic control pipeline:

```
Microrobots move
  → Electromagnet senses local density
  → Lamp activates (blue glow)
  → (physical system: coil energises → field applied → robots steered)
```

---

## Comparison with Related Work

This simulator is most closely related to two lines of work.

### O'Keeffe, Hong & Strogatz (2017) — *Oscillators that sync and swarm*

The foundational swarmalator paper ([Nature Communications, 2017](https://www.nature.com/articles/s41467-017-01190-3)) introduced coupled equations for position and phase:

$$\dot{\mathbf{x}}_i = \frac{1}{N}\sum_{j \neq i} \left[ \frac{\mathbf{x}_j - \mathbf{x}_i}{|\mathbf{x}_j - \mathbf{x}_i|} \left(A + J\cos(\theta_j - \theta_i)\right) - \frac{\mathbf{x}_j - \mathbf{x}_i}{|\mathbf{x}_j - \mathbf{x}_i|^2} \right]$$

$$\dot{\theta}_i = \frac{K}{N}\sum_{j \neq i} \frac{\sin(\theta_j - \theta_i)}{|\mathbf{x}_j - \mathbf{x}_i|}$$

This yields five collective states (static sync, static async, static phase wave, splintered phase wave, active phase wave) depending on $J$ and $K$.

### Ceron, O'Keeffe et al. (2023) — *Diverse behaviors in non-uniform chiral and non-chiral swarmalators*

The extended model ([Nature Communications, 2023](https://www.nature.com/articles/s41467-023-36563-4)) introduces:
- **Non-uniform natural frequencies** $\omega_i$ (robots oscillate at different rates)
- **Chirality** (robots have intrinsic CW or CCW circular orbits)
- **Local coupling** within a finite radius $R_c$ (no global mean-field)
- Physical validation with magnetic microrobots of different sizes

This dramatically expands the observable state space and directly models the heterogeneous microrobot collectives studied in the lab.

### Feature Comparison

| Feature | O'Keeffe et al. 2017 | Ceron et al. 2023 | **This Simulator** |
|---|---|---|---|
| **Coupling range** | Global mean-field | Local radius $R_c$ | Local $d \le d_{\text{att}}$ |
| **Attraction law** | $(A + J\cos\Delta\theta)/r$ | Size-dependent, local | $F_a(1+\cos\Delta\theta)/2$ |
| **Repulsion law** | $1/r^2$ (singular at 0) | Size-dependent exclusion | $F_s(1 - d/d_{\text{sep}})$ (linear) |
| **Phase coupling** | $K\sin\Delta\theta / r$ (distance-weighted) | Local Kuramoto + chirality | $K\sin\Delta\theta$ (pairwise) |
| **Natural frequency** | $\omega_i = 0$ | Non-uniform $\omega_i$ | $\omega_i = 0$ |
| **Agent heterogeneity** | Identical | Different sizes & chirality | Identical (uniform) |
| **Self-propulsion** | Yes ($\dot{\mathbf{x}}_i$ has $\mathbf{v}_i$ term) | Yes (magnetic actuation) | No (damped passive) |
| **External control layer** | None | Physical magnetic field | Virtual electromagnet grid |
| **Implementation** | Theoretical | Physical microrobot experiment | Interactive web simulation |

### Key Design Choice vs. O'Keeffe

The O'Keeffe model has a $1/r^2$ repulsion term that diverges as $r \to 0$, requiring careful numerical integration. This simulator replaces it with a **linear soft repulsion**:

$$\mathbf{f}_{ij} = \begin{cases} -F_s\left(1 - \dfrac{d_{ij}}{d_{\text{sep}}}\right)\hat{\mathbf{r}}_{ij} & d_{ij} < d_{\text{sep}} \\ +F_a \cdot \dfrac{1+\cos(\theta_j - \theta_i)}{2} \cdot \hat{\mathbf{r}}_{ij} & d_{\text{sep}} \le d_{ij} \le d_{\text{att}} \\ \mathbf{0} & d_{ij} > d_{\text{att}} \end{cases}$$

This avoids the singularity, enables stable real-time simulation at $N \le 600$ without an ODE integrator, and produces visually discrete clusters suitable for interactive exploration.

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

## References

- O'Keeffe, K. P., Hong, H., & Strogatz, S. H. (2017). Oscillators that sync and swarm. *Nature Communications*, 8, 1504. https://doi.org/10.1038/s41467-017-01190-3
- Ceron, S., O'Keeffe, K., Petersen, K., et al. (2023). Diverse behaviors in non-uniform chiral and non-chiral swarmalators. *Nature Communications*, 14, 940. https://doi.org/10.1038/s41467-023-36563-4

---

*DH Han · SAM Lab*
