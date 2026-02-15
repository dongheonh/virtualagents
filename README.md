## simplified swarmalator model

This simulation is based on the classical swarmalator framework but uses a simplified formulation for real-time large-scale interaction.

---

### comparison with classical swarmalator

<table>
<tr>
<th>feature</th>
<th>classical swarmalator</th>
<th>simplified model</th>
</tr>

<tr>
<td>spatial dynamics</td>
<td>dx_i/dt = (1/N) Σ_j [(x_j − x_i)(1 + J cos(θ_j − θ_i)) − (x_j − x_i)/||x_j − x_i||²]</td>
<td>piecewise local force</td>
</tr>

<tr>
<td>interaction range</td>
<td>global</td>
<td>d_ij &lt; R_int</td>
</tr>

<tr>
<td>repulsion</td>
<td>~ 1/||x_j − x_i||²</td>
<td>k_r (1 − d_ij / R_sep)</td>
</tr>

<tr>
<td>attraction</td>
<td>continuous</td>
<td>k_a (1 + cos(θ_j − θ_i))</td>
</tr>

<tr>
<td>phase dynamics</td>
<td>dθ_i/dt = ω_i + (K/N) Σ_j sin(θ_j − θ_i) / ||x_j − x_i||</td>
<td>dθ_i/dt = K Σ_{j∈N_i} sin(θ_j − θ_i)</td>
</tr>

<tr>
<td>distance weighting</td>
<td>yes</td>
<td>no</td>
</tr>

<tr>
<td>natural frequency</td>
<td>heterogeneous ω_i</td>
<td>ω_i = 0</td>
</tr>

<tr>
<td>integration</td>
<td>first-order ODE</td>
<td>v_i(t+1) = λ v_i + F_i</td>
</tr>

<tr>
<td>position update</td>
<td>direct</td>
<td>x_i(t+1) = x_i + v_i</td>
</tr>

<tr>
<td>stability</td>
<td>sensitive</td>
<td>robust</td>
</tr>

<tr>
<td>scalability</td>
<td>limited</td>
<td>real-time large-scale</td>
</tr>

</table>

---

This simplified formulation preserves the key idea of coupled spatial and phase synchronization while improving numerical stability and computational efficiency for interactive swarm simulations.




dongheonh.github.io/virtualagents

<img width="1512" height="826" alt="virtualagent" src="https://github.com/user-attachments/assets/e60f2580-eb1f-4e6f-b5ff-cb6354d00888" />
