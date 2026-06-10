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

Coarse to Medium mesh relative error 0.0045949492391629595

Medium to Fine mesh relative error 0.006385039176709853

GCI fine = -0.028942710478639184

### Conclusion 

From the results, we can see that the findings from the Lid-Cavity simulation have been verified and validated against the Ghia et al. 1982 paper for a Reynolds number of 100.

## Subsonic Turbulent Boundary Layer over a Flat Plate with a Compressible Pressure Solver

In a CD nozzle, a high-temperature and high-pressure subsonic flow is converted to supersonic flow before being exhausted from the engine. A high-fidelity simulation must remain stable across two flow regimes (i.e., subsonic and supersonic) while capturing the physics acting on the nozzle wall, such as wall temperature. Using a density-based solver in the subsonic section of a CD nozzle makes it difficult to maintain stability, therefore, a pressure-based solver is used to produce an internal field for that specific section of the nozzle. To validate the internal field produced by the pressure solver, a unit cases using zero-pressure-gradient flat plate simulations in a incompressible (modelled as incompressible), subsonic, turbulent flow using the Spalart-Allmaras model, as a one equation model is computational efficient for large mesh sizes, such as those used for CD nozzles.
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
   *(Where* $$\epsilon_{32} = f_3 - f_2$$, $$\epsilon_{21} = f_2 - f_1$$ *, and* $$q(p)$$ *is a correction factor that equals zero if the grid refinement ratio is constant, i.e.,* $$r_{21} = r_{32}$$).

5. **Calculate Relative Error ($$\epsilon_{21}$$):** Determine the relative error between the two finest grids:
   $$\epsilon_{21} = \left| \frac{f_2 - f_1}{f_1} \right|$$

6. **Compute the GCI Error:** Finally, calculate the fine-grid GCI error by applying a safety factor ($$F_s$$), which is typically set to 1.25 for a rigorous three-grid study:
   $$GCI_{fine} = \frac{F_s \cdot \epsilon_{21}}{r_{21}^p - 1}$$

> **Note:** The resulting percentage represents your numerical uncertainty band. It quantifies how close your fine-grid solution is to the theoretical, asymptotic "grid-independent" solution.

### Case using Spalart-Allmaras

For the Spalart-Allmaras model, the inlet values where set to $$\tilde{\nu} = \nu \times 3 = 4.7\times10^{-5} \frac{m^2}{s}$$ and defined $$\nu_t = 0 \frac{m^2}{s}$$ at the inlet so that it could be calculated. 

### Verification
For the verification, the residuals for velocity, pressure, and temperature where recorded to see if they either drop over 3 orders of magnitude and leveled out or have drop significantly. Alongside the residuals, the values for those quanilities where measured in several key locations across the flate plate, as seen in Figure 9.

![SA Coarse Velocity Residuals](plots/SA/probeLocations)

*Figure 9: The probe locations in the domain.*

Probes 1 and 2 are based at the leading edge of the plate, this is where the wake forms as flow hits the plate, whcih is a point of potential unsteady behaviour, and hence, is worth monitoring. Probe 1 is at distance of 25 mm from the plate and probe 2 is at a distance of 25 cm, this such that there is one probe close to the boundary layer law-log reigion or wake center and one in the freestream, to make sure that the behaviour is stable. Subsequence probes are place at x = 0.5 and x = 1.5 m. 2 locations where pick along the boundary layer to monitor if those locations would become steady.

![SA Coarse Velocity Residuals](plots/SA/coarse/ResidualsOfVelocity.png)

*Figure 10: Convergence history of velocity residuals for the SA model on the coarse mesh.*

![SA Coarse Pressure Residuals](plots/SA/coarse/ResidualsOfPressure.png)

*Figure 11: Convergence history of pressure residuals for the SA model on the coarse mesh.*


![SA Medium Velocity Residuals](plots/SA/medium/ResidualsOfVelocity.png)

*Figure 12: Convergence history of velocity residuals for the SA model on the medium mesh.*


![SA Medium Pressure Residuals](plots/SA/medium/ResidualsOfPressure.png)

*Figure 13: Convergence history of pressure residuals for the SA model on the medium mesh.*


![SA Fine Velocity Residuals](plots/SA/fine/ResidualsOfVelocity.png)

*Figure 14: Convergence history of velocity residuals for the SA model on the fine mesh.*


![SA Fine Pressure Residuals](plots/SA/fine/ResidualsOfPressure.png)

*Figure 15: Convergence history of pressure residuals for the SA model on the fine mesh.*

The residuals and values from probes of quaility where shown in figures 9-14 for each level of mesh, i.e. coarse, medium, and fine, with coarse being the base mesh. While residuals for all 3 meshes have either leveled out or drop significantly, it is important also measure the values of velocity, pressure, and temperature, using the probe locaiton mentioned previously. Figures 16-18 shows the final values that each of the probes converged on by the end of the simulation plotted against the amount of cells used in the simulation. For all 3 variables, no sigificate change was found. 

![SA Fine Pressure Residuals](plots/SA/AllLevelsVelocityProbes.png)

*Figure 16: Velocity Probes.*

![SA Fine Pressure Residuals](plots/SA/AllLevelsPressureProbes.png)

*Figure 17: Pressure Probes.*

![SA Fine Pressure Residuals](plots/SA/AllLevelsTemperatureProbes.png)

*Figure 18: Temperature Probes.*


![SA Skin Friction Coefficient](plots/SA/SkinCoefficient.png)

*Figure 19: Local skin friction coefficient distribution along the surface calculated with the SA model.*


![SA Dimensionless Velocity Profile (u+ vs y+)](plots/SA/u+y+.png)

*Figure 20: Dimensionless boundary layer velocity profile (* $$u^+$$ *vs* $$y^+$$ *) plotted against the theoretical law of the wall using the SA model.*


![SA Momentum Thickness Reynolds Number vs X](plots/SA/ReThetaVsX.png)

*Figure 21: Development of the momentum thickness Reynolds number (* $$Re_{\theta}$$ *) along the streamwise direction (X) for the SA model.*

The GCI error calculated for the gradients of $$Re_{\theta}$$ from coarse, medium, and fine meshes in Figure 26 using methodlogy in the Error Calculation Methodology section, which is found to be 0.6%, comparing the $$Re_{\theta}$$ at 50% of the plate length. The value tabled in the Table 4.


| $$Re_{\theta}$$ | Base Mesh | x1.5 Mesh | x2 Mesh | Base → x1.5 (%) | x1.5 → x2 (%) |  $$GCI_{medium}$$ (%)  | $$GCI_{fine}$$ (%) | 
| ----------- | --------- | --------- | ------- | --------------- | ------------- | --------------- | ------------- |
| 4000        | 0.00295   | 0.00311   | 0.00312 |       5.11      |      0.42  |       0.66      |      0.12  |
| 6000        | 0.00276   | 0.00289   | 0.00291 |       4.49      |      0.45  |       0.72      |      0.15  |
| 8000        | 0.00265   | 0.00275   | 0.00276 |       3.83      |      0.48  |       0.81      |      0.21  |
| 10000       | 0.00256   | 0.00265   | 0.00266 |       3.47      |      0.29  |       0.44      |      0.08  |
| 11500       | 0.00250   | 0.00258   | 0.00259 |       3.28      |      0.25  |       0.20      |      0.07  |

*Table 2: Verification of mesh convergence using the skin-friction coefficient* $$C_f$$ *at selected momentum-thickness Reynolds numbers. Percentage differences are calculated relative to the finer mesh solution.*

| $$y^+$$ | Base Mesh | x1.5 Mesh | x2 Mesh | Base → x1.5 (%) | x1.5 → x2 (%) |  $$GCI_{medium}$$ (%)  | $$GCI_{fine}$$ (%) | 
| ----------- | --------- | --------- | ------- | --------------- | ------------- | --------------- | ------------- |
| 5        | 0.00295   | 0.00311   | 0.00312 |       5.11      |      0.42  |       0.66      |      0.12  |
| 30        | 0.00276   | 0.00289   | 0.00291 |       4.49      |      0.45  |       0.72      |      0.15  |
| 50        | 0.00265   | 0.00275   | 0.00276 |       3.83      |      0.48  |       0.81      |      0.21  |
| 100       | 0.00256   | 0.00265   | 0.00266 |       3.47      |      0.29  |       0.44      |      0.08  |
| 1000       | 0.00250   | 0.00258   | 0.00259 |       3.28      |      0.25  |       0.20      |      0.07  |

*Table 3: Verification of mesh convergence based on the u+ value relative to the y+ values. Percentage differences are calculated relative to the finer mesh solution.*


| $$\frac{x}{L_{plate}}$$ | Base Mesh | x1.5 Mesh | x2 Mesh | Base → x1.5 (%) | x1.5 → x2 (%) | $$GCI_{medium}$$ (%)  | $$GCI_{fine}$$ (%) | 
| ------------- | --------- | --------- | ------- | --------------- | ------------- | --------------- | ------------- |
| 0.2           | 3780 | 3620 | 3615 | 4.54 | 0.0161 | 0.0210 | 3.89e-04 |
| 0.4           | 6620 | 6490 | 6443 | 2.10 | 0.695 | 1.87 | 1.02 |
| 0.6           | 9320 | 9130 | 9070 | 2.08 | 0.707 | 1.95 | 1.08 |
| 0.8           | 11900 | 11600 | 11600 | 2.89 | 0.268 | 0.418 | 0.0847 |
| 1.0           | 14300 | 14000 | 13900 | 2.53 | 0.381 | 0.670 | 0.196 |

*Table 4: Verification of mesh convergence based on the momentum-thickness Reynolds number* $$Re_{\theta}$$ *at selected streamwise locations. Percentage differences are calculated relative to the finer mesh solution.*

### Validation
In Figure 19, we see that the u+ vs $$log_{10} (y+)$$ has a significant disagreement with Coles' theory for all levels of the mesh refinement when 5 < $$y^+$$ < 30, as this is the buffer zone between the viscous sublayer, where $$u^+ = y^+$$ applies, and the log-law region, $$y^+$$ > 30. In Figure 20, the error between K-S theory and the simulation results for the coarse mesh is incredibly small.
The data provided by Nasa BLTMR approximate an agreement with the NASA estimation of $$Re_{\theta}$$ increase along the flat plate boundary layer as seen in figure 21. 


| $$\frac{x}{L_{plate}}$$ | NASA TMR | x2 Mesh | Error (%) |
| ------------- | -------- | ------- | --------- |
| 0.2 | 3766 | 3615 | 4.00 |
| 0.4 | 6680 | 6443 | 3.54 |
| 0.6 | 9428 | 9067 | 3.83 |
| 0.8 | 11961 | 11578 | 3.21 |
| 1.0 | 14384 | 13941 | 3.08 |
*Table 5: Validation of the mesh-independent solution using the NASA Turbulence Modeling Resource (TMR) reference data. Percentage error is calculated relative to the reference values.*

| $y^+$ Location | Coles Theory $u^+$ | x2 Mesh $u^+$ | Relative Error (%) |
| :--- | :--- | :--- | :--- |
| 3 | 2.99 | 3.58 | 19.6 |
| 5 | 4.87 | 6.09 | 24.9 |
| 30 | 12.9 | 13.4 | 3.18 |
| 50 | 14.5 | 14.6 | 0.997 |
| 100 | 16.2 | 16.3 | 0.305 |
| 1000 | 22.4 | 21.9 | 1.95 |
*Table 6: Validation of the mesh-independent solution using the Coles theory NASA Turbulence Modeling Resource (TMR) reference data. Percentage error is calculated relative to the reference values.*


| $$Re_{\theta}$$ | Kármán–Schoenherr | x2 Mesh | Error (%) |
| ----------- | ----------------- | ------- | --------- |
| 4000 | 0.00314396 | 0.003119266854151661 | 0.7854153948631256 |
| 6000 | 0.00290117 | 0.0029059643548560075 | 0.16525590903006943 |
| 8000 | 0.00274561 | 0.0027640285431899227 | 0.6708361052706979 |
| 10000 | 0.00263344 | 0.0026559303475700026 | 0.8540292381828611 |
| 11500 | 0.00257075 | 0.0025905867904628204 | 0.771634365956249 |
*Table 7: Validation of the mesh-independent solution using the Kármán–Schoenherr skin-friction correlation. Percentage error is calculated relative to the reference correlation.*


### Conclusion
Considering the results produced from this simulation using a pressure-based solver for subsonic, incompressible flow, i.e. Ma < 0.3, agrees with the validation data from Nasa TMR Zero Pressure Gradient flate plate case, means that it can produce the initial internal field for subsonic pre-inlet for a CD nozzle.

## References 
### In-Text Citations
* (Ghia et al., 1982)
* (AIAA TMBWG, 2026)

### Reference List

Ghia, U., Ghia, K.N. and Shin, C.T., 1982. High-Re solutions for incompressible flow using the Navier-Stokes equations and a multigrid method. *Journal of Computational Physics*, 48(3), pp. 387–411.

AIAA Turbulence Model Benchmarking Working Group (TMBWG), 2026. *Turbulence Modeling Resource: Zero Pressure Gradient Flat Plate Validation Case*. Available at: <https://tmbwg.github.io/turbmodels/flatplate_val.html> [Accessed 9 June 2026].
