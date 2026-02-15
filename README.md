simplified swarmalator model

This simulation is based on the classical swarmalator framework but uses a simplified formulation for real-time large-scale interaction.

comparison with classical swarmalator
feature	classical swarmalator	simplified model
spatial dynamics	dx_i/dt = (1/N) Σ_j [(x_j − x_i)(1 + J cos(θ_j − θ_i)) − (x_j − x_i)/‖x_j − x_i‖²]	piecewise local force
interaction range	global	d_ij < R_int
repulsion	~ 1/‖x_j − x_i‖²	k_r (1 − d_ij / R_sep)
attraction	continuous	k_a (1 + cos(θ_j − θ_i))
phase dynamics	dθ_i/dt = ω_i + (K/N) Σ_j sin(θ_j − θ_i) / ‖x_j − x_i‖	dθ_i/dt = K Σ_{j ∈ N_i} sin(θ_j − θ_i)
distance weighting	yes	no
natural frequency	heterogeneous ω_i	ω_i = 0
integration	first-order ODE	v_i(t+1) = λ v_i + F_i
position update	direct	x_i(t+1) = x_i + v_i
stability	sensitive	robust
scalability	limited	real-time large-scale

This simplified formulation preserves the key idea of coupled spatial and phase synchronization, while improving numerical stability and computational efficiency for interactive swarm simulations.





dongheonh.github.io/virtualagents

<img width="1512" height="826" alt="virtualagent" src="https://github.com/user-attachments/assets/e60f2580-eb1f-4e6f-b5ff-cb6354d00888" />
