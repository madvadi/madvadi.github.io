# madvadi.github.io
# Portfolio of Cenk Tekin CFD Projects

Introduction: This is a presentation of a number of CFD projects that I have been using to keep my skills up to date with advancements in OpenFOAM 13 and CFD in general.

## Lid-cavity Demonstration
To verify and validate my own CFD skills in setting up, running, and post-processing results, I had a go at creating a simulation that I can easily validate against well-established results from Ghia, Ghia, and Shin, 1982.
The model is a 2D square cavity of unit dimensions (1m x 1m) containing a Newtonian fluid with a moving lid.
The goal is to determine the velocity profile halfway along the unit cavity at a Reynolds number of 100 with a baseline grid size of 129 x 129, and validate the results against my source.
By setting the reference pressure to 0 at the bottom-left corner of the domain, I configured the boundary conditions as:

![MeshAndBC](plots/MeshAndBC.png)

*Figure 1: The baseline mesh with the boundary conditions highlighted.*

The initial field for pressure was set to a uniform 0, and the velocity was set to a uniform 0 m/s in all directions, as seen in Figure 1.

The results compared with the validation data in Ghia et al., 1982 are presented in Figure 2:

![Results](plots/DataVnV.png)

*Figure 2: Comparison of normalized horizontal velocity ($U_x$) along the vertical centerline (x=0.5) against Ghia et al. (1982) benchmark data.*

### Verification 

For the purpose of verification, 4 probes were placed at locations along the vertical centerline of the cavity (x = 0.5 m) to measure changes in velocity and pressure.
Probe 1: y = 0.0 m, Probe 2: y = 0.25 m, Probe 3: y = 0.5 m, Probe 4: y = 0.75 m.

The data in Figures 3 and 4 show that the velocities and pressure converge to a steady value. The residuals for both velocity and pressure also drop down several orders of magnitude, supporting the case that the simulation has converged, as can be seen in Figures 5 and 6.

![Velocity X Results](plots/Velocity_Probes.png)

*Figure 3: Velocity X data at 4 probe locations over 4000 iterations.*

![Pressure Results](plots/Pressure_Probes.png)

*Figure 4: Pressure data at 4 probe locations over 4000 iterations.*

![Velocity X Residuals](plots/residuals_for_u.png)

*Figure 5: Velocity X residual over 4000 iterations.*

![Pressure Residuals](plots/Residual_of_Pressure.png)

*Figure 6: Pressure residual over 4000 iterations.*

![Convergance Mesh](plots/ConvMesh.png)

*Figure 7: Grid convergence study comparing 129 x 129, 258 x 258, and 516 x 516 grid sizes.*

Figure 7 shows that the results from all levels of refinement do not change significantly. The resulting errors are compiled in Table 1.

| Mesh Resolution | Total Cells | Max $U_x$ | % Change |
| :--- | :--- | :--- | :--- |
| 129 x 129 | 16,641 | -0.216 | — |
| 258 x 258 | 66,564 | -0.217 | 0.463% |
| 516 x 516 | 266,256 | -0.216 | 0.175% |

*Table 1: Grid convergence error when refined results are compared against the baseline 129 x 129 mesh results.*

### Conclusion 

From the results, we can see that the findings from the Lid-Cavity simulation have been verified and validated against the Ghia et al. 1982 paper for a Reynolds number of 100.

## Subsonic Turbulent Boundary Layer over a Flat Plate with a Compressible Pressure Solver

In a CD nozzle, a high-temperature and high-pressure subsonic flow is converted to supersonic flow before being exhausted from the engine. A high-fidelity simulation must remain stable across two flow regimes (i.e., subsonic and supersonic) while capturing the physics acting on the nozzle wall, such as wall temperature. Using a density-based solver in the subsonic section of a CD nozzle makes it difficult to maintain stability; therefore, a pressure-based solver is used to produce an internal field for that specific section of the nozzle. I produced 2 unit cases using zero-pressure-gradient flat plate simulations in a compressible, subsonic, turbulent flow using the $k$-$\omega$ SST and Spalart-Allmaras models.
The NASA Turbulent Modeling Resource provides a validation case for my simulation to be compared against, which can be found at https://tmbwg.github.io/turbmodels/flatplate_val.html.

The baseline mesh used features a 175 by 90 grid size with the boundary conditions shown in Figure 8.

![Flate Plate Mesh](plots/Mesh_N_BCs.png)
*Figure 8: Mesh with a 175 x 90 resolution and its corresponding boundary conditions.*

For these turbulence models, it is crucial that the mesh spacing near the wall remains under $y^+ = 1$ in order to fully resolve the viscous sub-layer. Using the calculated $y$-coordinate value for $y^+ = 1$, I applied a simple expansion ratio to cluster the nodes closely along the y-axis.

### Error Calculation Methodology

To calculate the Grid Convergence Index (GCI) error using the representative grid size, you follow a structured, multi-step verification process:

1. **Calculate Representative Grid Sizes ($$h$$):** Generate three distinct meshes—coarse, medium, and fine—and compute the characteristic grid size ($$h_1, h_2, h_3$$) for each using the total domain area and total cell count:
   $$h = \sqrt{\frac{A_{total}}{N}}$$

2. **Determine Grid Refinement Ratios ($$r$$):** Calculate the refinement ratios between the grid pairs. It is highly recommended to keep these ratios above 1.3 to ensure the grid resolutions are distinct enough to capture discretization changes:
   $$r_{21} = \frac{h_2}{h_1}, \quad r_{32} = \frac{h_3}{h_2}$$

3. **Extract Solution Variables ($$f$$):** Run the CFD simulations and extract a critical target scalar variable (such as drag coefficient, lift coefficient, or peak velocity) from all three grids, yielding $$f_1$$ (fine), $$f_2$$ (medium), and $$f_3$$ (coarse).

4. **Solve for the Local Order of Accuracy ($$p$$):** Solve iteratively for the apparent order of accuracy ($$p$$) using the grid refinement ratios and the differences between the solutions:
   $$p = \frac{1}{\ln(r_{21})} \left| \ln\left| \frac{\epsilon_{32}}{\epsilon_{21}} \right| + q(p) \right|$$
   *(Where $$\epsilon_{32} = f_3 - f_2$$, $$\epsilon_{21} = f_2 - f_1$$, and $$q(p)$$ is a correction factor that equals zero if the grid refinement ratio is constant, i.e., $$r_{21} = r_{32}$$).*

5. **Calculate Relative Error ($$\epsilon_{21}$$):** Determine the relative error between the two finest grids:
   $$\epsilon_{21} = \left| \frac{f_2 - f_1}{f_1} \right|$$

6. **Compute the GCI Error:** Finally, calculate the fine-grid GCI error by applying a safety factor ($$F_s$$), which is typically set to 1.25 for a rigorous three-grid study:
   $$GCI_{fine} = \frac{F_s \cdot \epsilon_{21}}{r_{21}^p - 1}$$

> **Note:** The resulting percentage represents your numerical uncertainty band. It quantifies how close your fine-grid solution is to the theoretical, asymptotic "grid-independent" solution.

### Case using Spalart-Allmaras

For the Spalart-Allmaras model, the inlet values where set to $$\tilde{\nu} = \nu \times 3 = 4.7\times10^{-5} \frac{m^2}{s}$$ and defined $$\nu_t = 0 \frac{m^2}{s}$$ at the inlet so that it could be calculated. 

### Verification
For the verification, the residuals for velocity, pressure, and temperature where recorded to see if they either drop over 3 orders of magnitude and leveled out or have drop significantly. Alongside the residuals, the values for those quanilities where measured in 4 key places to monitor the change in those values, the locations pick where in the developed boundary layer and in the freestream. Probe locations are at the leading edges, half way throught the plate, 75% of the plate. 


![SA Coarse Velocity Residuals](plots/SA/coarse/ResidualsOfVelocity.png)

*Figure 9: Convergence history of velocity residuals for the SA model on the coarse mesh.*


![SA Coarse Pressure Residuals](plots/SA/coarse/ResidualsOfPressure.png)

*Figure 10: Convergence history of pressure residuals for the SA model on the coarse mesh.*


![SA Coarse Velocity Probability](plots/SA/coarse/ProbVelocity.png)

*Figure 11: Velocity profiles extracted from the 4 probe locations across the control volume domain using the SA model on the coarse mesh.*


![SA Coarse Pressure Probability](plots/SA/coarse/ProbPressure.png)

*Figure 12: Pressure distributions extracted from the 4 probe locations across the control volume domain using the SA model on the coarse mesh.*


![SA Coarse Temperature Probability](plots/SA/coarse/ProbTemperature.png)

*Figure 13: Temperature profiles extracted from the 4 probe locations across the control volume domain using the SA model on the coarse mesh.*


![SA Medium Velocity Residuals](plots/SA/medium/ResidualsOfVelocity.png)

*Figure 14: Convergence history of velocity residuals for the SA model on the medium mesh.*


![SA Medium Pressure Residuals](plots/SA/medium/ResidualsOfPressure.png)

*Figure 15: Convergence history of pressure residuals for the SA model on the medium mesh.*


![SA Medium Velocity Probability](plots/SA/medium/ProbeVelocity.png)

*Figure 16: Velocity profiles extracted from the 4 probe locations across the control volume domain using the SA model on the medium mesh.*


![SA Medium Pressure Probability](plots/SA/medium/ProbePressure.png)

*Figure 17: Pressure distributions extracted from the 4 probe locations across the control volume domain using the SA model on the medium mesh.*


![SA Medium Temperature Probability](plots/SA/medium/ProbeTemperature.png)

*Figure 18: Temperature profiles extracted from the 4 probe locations across the control volume domain using the SA model on the medium mesh.*


![SA Fine Velocity Residuals](plots/SA/fine/ResidualsOfVelocity.png)

*Figure 19: Convergence history of velocity residuals for the SA model on the fine mesh.*


![SA Fine Pressure Residuals](plots/SA/fine/ResidualsOfPressure.png)

*Figure 20: Convergence history of pressure residuals for the SA model on the fine mesh.*


![SA Fine Velocity Probability](plots/SA/fine/ProbVelocity.png)

*Figure 21: Velocity profiles extracted from the 4 probe locations across the control volume domain using the SA model on the fine mesh.*


![SA Fine Pressure Probability](plots/SA/fine/ProbPressure.png)

*Figure 22: Pressure distributions extracted from the 4 probe locations across the control volume domain using the SA model on the fine mesh.*


![SA Fine Temperature Probability](plots/SA/fine/ProbTemperature.png)

*Figure 23: Temperature profiles extracted from the 4 probe locations across the control volume domain using the SA model on the fine mesh.*


![SA Skin Friction Coefficient](plots/SA/SkinCoefficient.png)

*Figure 24: Local skin friction coefficient distribution along the surface calculated with the SA model.*


![SA Dimensionless Velocity Profile (u+ vs y+)](plots/SA/u+y+.png)

*Figure 25: Dimensionless boundary layer velocity profile (* $$u^+$$ * vs * $$y^+$$ *) plotted against the theoretical law of the wall using the SA model.*


![SA Momentum Thickness Reynolds Number vs X](plots/SA/ReThetaVsX.png)

*Figure 26: Development of the momentum thickness Reynolds number (* $$Re_{\theta}$$ *) along the streamwise direction (X) for the SA model.*

The residuals and values from probes of quaility where shown in figures 9-23 for each level of mesh, i.e. coarse, medium, and fine, with coarse being the base mesh. Residuals for all 3 meshes have either leveled out or drop significantly, while the probe values for temperature, velocity, and pressure, have shown no significant charge for more than 3000 iterations minimum.

The GCI error calculated for the gradients of $$Re_{\theta}$$ from coarse, medium, and fine meshes in Figure 26 using methodlogy in the Error Calculation Methodology section, which is found to be 0.6%, comparing the $$Re_{\theta}$$ at 50% of the plate length. The value tabled in the table 2.


| Mesh Resolution | $$Re_{\theta}$$ at x = 1.0 m | % Change |
| :--- | :--- | :--- |
| 175 x 90 | 8014.5 | — |
| 263 x 135 | 7806.7 | 2.661 |
| 350 x  180| 7754.2 | 0.651 |

*Table 2: Grid convergence error when refined results are compared against the baseline 175 x 90 mesh results.*

### Validation
In Figure 25, we see that the u+ vs $$log_{10} (y+)$$ has a significant disagreement with Coles' theory for all levels of the mesh refinement when 5 < $$y^+$$ < 30, as this is the buffer zone between the viscous sublayer, where $$u^+ = y^+$$ applies, and the log-law region, $$y^+$$ > 30. In Figure 24, the error between K-S theory and the simulation results for the coarse mesh is incredibly small.
The data provided by Nasa BLTMR approximate an agreement with the NASA estimation of $$Re_{\theta}$$ increase along the flat plate boundary layer as seen in figure 26. 

### Conclusion
Conclusion

### Case using k- $$\omega$$ SST

For the k- $$\omega$$ SST case, the following initial conditions were set using $$k = \frac{3}{2} (U I)^2$$, where I is the turbulence intensity set to I = 0.039, and $$\epsilon = C_{\mu}^{3/4} \frac{k^{3/2}}{L}$$. Here, $$L \sim 0.07\delta$$, where $$\delta$$ is the boundary layer thickness estimated to be 0.029 meters using reference [X, X], resulting in $$\nu_{T} = C_{\nu}\frac{k^2}{\epsilon}=2894.3$$.

### Verification
![k-omega SST Coarse Velocity Residuals](plots/k_omegaSST/coarse/ResidualsOfVelocity.png)

*Figure 27: Convergence history of velocity residuals for the k-omega SST model on the coarse mesh.*


![k-omega SST Coarse Pressure Residuals](plots/k_omegaSST/coarse/ResidualsOfPressure.png)

*Figure 28: Convergence history of pressure residuals for the k-omega SST model on the coarse mesh.*


![k-omega SST Coarse Velocity Probes](plots/k_omegaSST/coarse/ProbeVelocity.png)

*Figure 29: Velocity profiles extracted from the 4 probe locations across the control volume domain using the k-omega SST model on the coarse mesh.*


![k-omega SST Coarse Pressure Probes](plots/k_omegaSST/coarse/ProbePressure.png)

*Figure 30: Pressure distributions extracted from the 4 probe locations across the control volume domain using the k-omega SST model on the coarse mesh.*


![k-omega SST Coarse Temperature Probes](plots/k_omegaSST/coarse/ProbeTemperature.png)

*Figure 31: Temperature profiles extracted from the 4 probe locations across the control volume domain using the k-omega SST model on the coarse mesh.*


![k-omega SST Medium Velocity Residuals](plots/k_omegaSST/medium/ResidualsOfVelocity.png)

*Figure 32: Convergence history of velocity residuals for the k-omega SST model on the medium mesh.*


![k-omega SST Medium Pressure Residuals](plots/k_omegaSST/medium/ResidualsOfPressure.png)

*Figure 33: Convergence history of pressure residuals for the k-omega SST model on the medium mesh.*


![k-omega SST Medium Velocity Probes](plots/k_omegaSST/medium/ProbeVelocity.png)

*Figure 34: Velocity profiles extracted from the 4 probe locations across the control volume domain using the k-omega SST model on the medium mesh.*


![k-omega SST Medium Pressure Probes](plots/k_omegaSST/medium/ProbePressure.png)

*Figure 35: Pressure distributions extracted from the 4 probe locations across the control volume domain using the k-omega SST model on the medium mesh.*


![k-omega SST Medium Temperature Probes](plots/k_omegaSST/medium/ProbeTemperature.png)

*Figure 36: Temperature profiles extracted from the 4 probe locations across the control volume domain using the k-omega SST model on the medium mesh.*

![k-omega SST Fine Velocity Residuals](plots/k_omegaSST/fine/ResidualsOfVelocity.png)

*Figure 37: Convergence history of velocity residuals for the k-omega SST model on the fine mesh.*


![k-omega SST Fine Pressure Residuals](plots/k_omegaSST/fine/ResidualsOfPressure.png)

*Figure 38: Convergence history of pressure residuals for the k-omega SST model on the fine mesh.*


![k-omega SST Fine Velocity Probes](plots/k_omegaSST/fine/ProbVelocity.png)

*Figure 39: Velocity profiles extracted from the 4 probe locations across the control volume domain using the k-omega SST model on the fine mesh.*


![k-omega SST Fine Pressure Probes](plots/k_omegaSST/fine/ProbPressure.png)

*Figure 40: Pressure distributions extracted from the 4 probe locations across the control volume domain using the k-omega SST model on the fine mesh.*


![k-omega SST Fine Temperature Probes](plots/k_omegaSST/fine/ProbTemperature.png)

*Figure 41: Temperature profiles extracted from the 4 probe locations across the control volume domain using the k-omega SST model on the fine mesh.*


![k-omega SST Skin Friction Coefficient](plots/k_omegaSST/SkinCoefficient.png)

*Figure 42: Local skin friction coefficient distribution along the surface calculated with the k-omega SST model.*


![k-omega SST Dimensionless Velocity Profile (u+ vs y+)](plots/k_omegaSST/u+y+.png)

*Figure 43: Dimensionless boundary layer velocity profile ($u^+$ vs $y^+$) plotted against the theoretical law of the wall using the k-omega SST model.*


![k-omega SST Momentum Thickness Reynolds Number vs X](plots/k_omegaSST/ReThetaVsX.png)

*Figure 44: Development of the momentum thickness Reynolds number ($Re_\theta$) along the streamwise direction ($X$) for the k-omega SST model.*

### Validation
Validation

### Conclusion
Conclusion
