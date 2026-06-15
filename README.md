# Portfolio of Cenk Tekin CFD Projects

Introduction: This is a presentation of a number of CFD projects that I have been using to keep my skills up to date with advancements in OpenFOAM 13 and CFD in general.

### Error Calculation Methodology

In the following projects, calculation of the Grid Convergence Index (GCI) error is as follows:

1. **Calculate Representative Grid Sizes ($$h$$):** Generate three distinct meshes—coarse, medium, and fine—and compute the characteristic grid size ($$h_1, h_2, h_3$$) for each using the total domain area and total cell count:
   $$h = \sqrt{\frac{A_{total}}{N}}$$

2. **Determine Grid Refinement Ratios ($$r$$):** Calculate the refinement ratios between the grid pairs. It is highly recommended to keep these ratios above 1.3 to ensure the grid resolutions are distinct enough to capture discretization changes:
   $$r_{21} = \frac{h_2}{h_1}, \quad r_{32} = \frac{h_3}{h_2}$$

3. **Extract Solution Variables ($$f$$):** Run the CFD simulations and extract a critical target scalar variable (such as drag coefficient, lift coefficient, or peak velocity) from all three grids, yielding $$f_1$$ (fine), $$f_2$$ (medium), and $$f_3$$ (coarse).

4. **Solve for the Local Order of Accuracy ($$p$$):** Solve iteratively for the apparent order of accuracy ($$p$$) using the grid refinement ratios and the differences between the solutions:
   $$p = \frac{1}{\ln(r_{21})} \left| \ln\left| \frac{\epsilon_{32}}{\epsilon_{21}} \right| + q(p) \right|$$
   *(Where* $$\epsilon_{32} = f_3 - f_2$$, $$\epsilon_{21} = f_2 - f_1$$ *, and* $$q(p)$$ *is a correction factor that equals zero if the grid refinement ratio is constant, i.e.,* $$r_{21} = r_{32}$$).

5. **Calculate Relative Error ($$\epsilon_{32}$$ and $$\epsilon_{21}$$):** Determine the relative error between the two finest grids:
   $$\epsilon_{21} = \left| \frac{f_2 - f_1}{f_1} \right|$$ and $$\epsilon_{32} = \left| \frac{f_3 - f_2}{f_2} \right|$$

6. **Compute the GCI Error:** Finally, calculate the fine- and medium-grid GCI error by applying a safety factor ($$F_s$$), which is typically set to 1.25 for a rigorous three-grid study:
   $$GCI_{fine} = \frac{F_s \cdot \epsilon_{21}}{r_{21}^p - 1}$$

> **Note:** The resulting percentage represents your numerical uncertainty band. It quantifies how close your fine-grid solution is to the theoretical, asymptotic "grid-independent" solution.


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

| Mesh Resolution | Max $U_x$ | % Relative Error | % GCI Error |
| :--- | :--- | :--- | :--- |
| 129 x 129 | -0.216309 | — | -- |
| 258 x 258 | -0.217237 | 0.463 | 0.744 |
| 516 x 516 | -0.216975 | 0.175 | 0.594 |

*Table 1: Grid convergence error when refined results are compared against the baseline 129 x 129 mesh results.*

coarse value = -0.216309, medium value = -0.217237, fine value = -0.216975

Relative Error 32 = 0.004271832146457611, GCI medium = 0.007440428363199541

Relative Error 21 = 0.001207512386219667, GCI fine = 0.0005937842439744134


### Conclusion 

From the results, we can see that the findings from the Lid-Cavity simulation have been verified and validated against the Ghia et al. 1982 paper for a Reynolds number of 100.

## Subsonic Turbulent Boundary Layer over a Flat Plate with a Compressible Pressure Solver

In a CD nozzle, a high-temperature, high-pressure subsonic flow is converted into a supersonic flow before being exhausted from the engine. A high-fidelity simulation must remain stable across two flow regimes (i.e., subsonic and supersonic) while capturing the boundary layer at the nozzle wall to determine quantities such as wall temperature. Using a density-based solver in the subsonic section of a CD nozzle makes it difficult to maintain stability. Therefore, a pressure-based solver is used to produce an internal field for that specific section of the nozzle (Ansys, 2026). Several assumptions are made for the nozzle simulation as a whole, which affect this validation unit case: the flow is chemically frozen, the cross-sectional area for the subsonic section is constant, and the flow is modeled as 2D axisymmetric. To validate the internal field produced by the pressure solver, a unit case was conducted using zero-pressure-gradient flat-plate simulations in an incompressible (modeled as compressible), subsonic, turbulent flow. The Spalart–Allmaras model, as a one-equation model, is computationally efficient for large mesh sizes, such as those used for CD nozzles. The NASA Turbulence Modeling Resource provides a validation case for my simulation to be compared against, which can be found at (AIAA TMRWG, 2026).

![Flate Plate Mesh](plots/Mesh_N_BCs.png)
*Figure 8: Mesh with a 175 x 90 resolution and its corresponding boundary conditions.*

The baseline mesh used features a 175 by 90 grid size, with the boundary conditions shown in Figure 8. For these turbulence models, it is crucial that the mesh spacing near the wall remains under $y^+ = 1$ to fully resolve the viscous sublayer. Using the calculated y-coordinate value for $y^+ = 1$, I applied a simple expansion ratio to cluster the nodes closely along the y-axis. For the baseline mesh, close cell center to the wall was 1.4343e-05 meters.

### Case using Spalart-Allmaras

For the Spalart-Allmaras model, the inlet values where set to $$\tilde{\nu} = \nu \times 3 = 4.7\times10^{-5} \frac{m^2}{s}$$ and letted $$\nu_t = 0 \frac{m^2}{s}$$ at the inlet float (Spalart and Allmaras 1992). 

### Verification
For verification, the residuals for velocity, pressure, and temperature were recorded to show that they met the convergence criteria, having dropped by at least three orders of magnitude and leveled out. Convergence criteria also dictate that the values of these quantities do not change for a steady-state flow; hence, several probes were placed in key locations, as shown in Table 2.

| Probe Number | Coordinates (x, y) |
| ------------ | ------------------ |
| 1 | 0.5, 0.025 |
| 2 | 0.5, 0.25 |
| 3 | 1, 0.025 |
| 4 | 1, 0.25 |
| 5 | 2, 0.025 |
| 6 | 2, 0.25 |

*Table 2: Probe locations within the domain.*

Probes 1 and 2 are positioned at the leading edge of the plate. This is where the wake forms as the flow hits the plate—a point of potential unsteady behaviour—and is therefore worth monitoring to ensure that the values at those points stabilise. Probes 1, 3, and 5 are set 0.025 m from the wall. This ensures that the quantities are stable at two points within the log-law portion of the boundary layer (in the case of probes 3 and 5), and just above the leading edge to measure how the quantities are affected by the wake (in the case of probe 1). Probes 2, 4, and 6 are located in the freestream above the boundary layer to ensure that no unsteady behaviour is occurring there, and that the freestream values are maintained.

![SA Coarse Velocity Residuals](plots/SA/coarse/ResidualsOfVelocity.png)

*Figure 9: Convergence history of velocity residuals for the SA model on the coarse mesh.*

![SA Coarse Pressure Residuals](plots/SA/coarse/ResidualsOfPressure.png)

*Figure 10: Convergence history of pressure residuals for the SA model on the coarse mesh.*

In figures 9 and 10, it is seen that the residuals have dropped and flatlined at 1e-05 for Ux, 1e-06 for Uy, and 1e-04 for pressure, hence this meets the critica for convergence for the coarse mesh. 

![SA Medium Velocity Residuals](plots/SA/medium/ResidualsOfVelocity.png)

*Figure 11: Convergence history of velocity residuals for the SA model on the medium mesh.*


![SA Medium Pressure Residuals](plots/SA/medium/ResidualsOfPressure.png)

*Figure 12: Convergence history of pressure residuals for the SA model on the medium mesh.*


In figures 11 and 12, it is seen that the residuals have dropped and averaged out at 1e-11 for Ux, 1e-09 for Uy, and 1e-07 for pressure, hence this meets the critica for convergence for the medium mesh. 


![SA Fine Velocity Residuals](plots/SA/fine/ResidualsOfVelocity.png)

*Figure 13: Convergence history of velocity residuals for the SA model on the fine mesh.*


![SA Fine Pressure Residuals](plots/SA/fine/ResidualsOfPressure.png)

*Figure 14: Convergence history of pressure residuals for the SA model on the fine mesh.*

In figures 13 and 14, it is seen that the residuals have dropped to 1e-10 for Ux, 1e-08 for Uy, and 1e-07 for pressure which has averaged out, hence this meets the critica for convergence for the fine mesh. However has the order of maginitude did not flateline, it is important to show the actually values staying considtance over the iterations. Hence, figures 15-17 shows the final values that each of the probes converged on by the end of the simulation plotted against the amount of cells used in the simulation. For all 3 variables, no sigificate change was found as the mesh was refined. 

![SA Fine Pressure Residuals](plots/SA/AllLevelsVelocityProbes.png)

*Figure 15: Velocity Probes.*

![SA Fine Pressure Residuals](plots/SA/AllLevelsPressureProbes.png)

*Figure 16: Pressure Probes.*

![SA Fine Pressure Residuals](plots/SA/AllLevelsTemperatureProbes.png)

*Figure 17: Temperature Probes.*

To understand the contribution of viscosity to the drag, the skin coefficient profile is calcualted across the plate, as seen in Figure 18, and in table 3, at several key points on the plate, using $$Re_{\theta} at that point, the relative and GCI error is calcualted and found to have convergenced at those points, i.e. the GCI and relative error decreases as the meshes is refined.

![SA Skin Friction Coefficient](plots/SA/SkinCoefficient.png)

*Figure 18: Local skin friction coefficient distribution along the surface calculated with the SA model.*

| $$Re_{\theta}$$ | Base Mesh | x1.5 Mesh | x2 Mesh | Base → x1.5 (%) | x1.5 → x2 (%) |  $$GCI_{medium}$$ (%)  | $$GCI_{fine}$$ (%) | 
| ----------- | --------- | --------- | ------- | --------------- | ------------- | --------------- | ------------- |
| 4000        | 0.00295   | 0.00311   | 0.00312 |       5.11      |      0.42  |       0.66      |      0.12  |
| 6000        | 0.00276   | 0.00289   | 0.00291 |       4.49      |      0.45  |       0.72      |      0.15  |
| 8000        | 0.00265   | 0.00275   | 0.00276 |       3.83      |      0.48  |       0.81      |      0.21  |
| 10000       | 0.00256   | 0.00265   | 0.00266 |       3.47      |      0.29  |       0.44      |      0.08  |
| 11500       | 0.00250   | 0.00258   | 0.00259 |       3.28      |      0.25  |       0.20      |      0.07  |

*Table 3: Verification of mesh convergence using the skin-friction coefficient* $$C_f$$ *at selected momentum-thickness Reynolds numbers. Percentage differences are calculated relative to the finer mesh solution.*

Because turbulent boundary layers follow at generalised law at in the viscous sub-layer and the log-law region, it is important to compare the velocity profile with universal laws such as Coles Law of the Wall (AIAA TMBWG, 2026), which will be discussed further in the validation section. In Figure 19, there is a between 5 < $$y^+$$ < 30 that is called the buffer zone, between the viscous sub-laye, $$y^+$$ =< 5, and log-law reigion, $$y^+$$ => 30, where either viscous or turbulent effects domainate. Hence, due to it's compelexity, this study will only validate the viscous sub-layer and log-law $$u^+$$ velocity profile. In Figure 19, between 5 < $$y^+$$ < 30,  the $$u^+ = y^+$$ is used as a placeholder while at $$y^+$$ < 5, $$y^+ = u^+$$ and $$y^+ > 30$$ uses Coles Mean Velocity profile law (AIAA TMBWG, 2026). All finement levels show simiilar profile in Figure 19, and in Table 4, relative and GCI errors measured at several key $$y^+$$ values, thsoe errors decrease with mesh refinement with the error decreasing with ascation into the log-law reigion soon after leaving the buffer zones.

![SA Dimensionless Velocity Profile (u+ vs y+)](plots/SA/u+y+.png)

*Figure 19: Dimensionless boundary layer velocity profile (* $$u^+$$ *vs* $$y^+$$ *) plotted against the theoretical law of the wall using the SA model.*


| $$y^+$$ | Base Mesh | x1.5 Mesh | x2 Mesh | Base → x1.5 (%) | x1.5 → x2 (%) |  $$GCI_{medium}$$ (%)  | $$GCI_{fine}$$ (%) | 
| -----| ------- | ------- | ------- | ---- | ------------- | --------------- | ------------- |
| 35 | 13.9 | 13.6 | 13.6 | 2.44 | 0.573 | 0.832 |0.365 |
| 40 | 13.9 | 13.9 | 13.9 | 0.137 | 0.0689 | 0.148 | 0.119 |
| 50 | 14.4 | 14.5 | 14.6 | 1.21 | 0.472 | 1.56 | 0.961 |
| 100 | 16.2 | 16.3 | 16.3 | 0.283 | 0.0259 | 0.0405 | 8.14e-03 |
| 300 | 18.8 | 18.9 | 18.9 | 0.730 | 0.00415 | 0.0102 | 0.909 |
| 1000 | 22.0 | 21.9 | 21.9 | 0.521 | 0.210 | 0.382 | 0.258 |
| 1500 | 23.0 | 23.0 | 23.0 | 0.199 | 0.0177 | 0.0275 | 5.41e-03 |

*Table 4: Verification of mesh convergence based on the u+ value relative to the y+ values. Percentage differences are calculated relative to the finer mesh solution.*

To characterise the momentum that is lost to the boundary layer, the $$Re_{\theta}$$​ profile is plotted along the plate, as seen in Figure 20, for all mesh refinements. In Figure 20, the $$Re_{\theta}$$​ results show agreement with each other, although they do not converge with the data provided by (AIAA TMRWG, 2026), which will be discussed in the validation section. In Table 5, the relative and GCI errors decrease as the mesh is refined, and all errors are well under 1%. The data from (AIAA TMRWG, 2026) has a maximum error of 4% at the leading edge, which decreases downstream to 3.08% at the end of the plate, as seen in Table 8. At the leading edge, there is a sudden change in the flow's velocity as it transitions from the freestream velocity to zero due to the no-slip boundary condition of the plate. This means that the pressure fluctuations interfere with the viscosity domain close to the leading edge, as evidenced at $$\frac{x}{L_{plate}} = 0.2$$ in Table 6. As shown in the mesh in Figure 8, the grid has been refined slightly towards the leading edge to minimise this error as much as possible by resolving the stagnation point's pressure fluctuations. The data from AIAA TMRWG (2026) represents a typical Reθ​ progression for a turbulent boundary layer that increases similarly to the 0.8 power law; it is expected that turbulence models, such as the Spalart–Allmaras model, will exhibit a noticeable error while following a similar progression.

![SA Momentum Thickness Reynolds Number vs X](plots/SA/ReThetaVsX.png)

*Figure 20: Development of the momentum thickness Reynolds number (* $$Re_{\theta}$$ *) along the streamwise direction $$\frac{x}{L}$$ for the SA model.*


| $$\frac{x}{L_{plate}}$$ | Base Mesh | x1.5 Mesh | x2 Mesh | Base → x1.5 (%) | x1.5 → x2 (%) | $$GCI_{medium}$$ (%)  | $$GCI_{fine}$$ (%) | 
| ----- | --------- | --------- | ------- | --------------- | ------------- | --------------- | ------------- |
| 0.2   | 3780 | 3620 | 3615 | 4.54 | 0.0161 | 0.0210 | 3.89e-04 |
| 0.4   | 6620 | 6490 | 6443 | 2.10 | 0.695 | 1.87 | 1.02 |
| 0.6   | 9320 | 9130 | 9070 | 2.08 | 0.707 | 1.95 | 1.08 |
| 0.8   | 11900 | 11600 | 11600 | 2.89 | 0.268 | 0.418 | 0.0847 |
| 1.0   | 14300 | 14000 | 13900 | 2.53 | 0.381 | 0.670 | 0.196 |

*Table 5: Verification of mesh convergence based on the momentum-thickness Reynolds number* $$Re_{\theta}$$ *at selected streamwise locations. Percentage differences are calculated relative to the finer mesh solution.*

| $$\frac{x}{L_{plate}}$$ | NASA TMR | x2 Mesh | Error (%) |
| ------------- | -------- | ------- | --------- |
| 0.2 | 3766 | 3615 | 4.00 |
| 0.4 | 6680 | 6443 | 3.54 |
| 0.6 | 9428 | 9067 | 3.83 |
| 0.8 | 11961 | 11578 | 3.21 |
| 1.0 | 14384 | 13941 | 3.08 |

*Table 6: Validation of the mesh-independent solution using the reference data from (AIAA TMBWG, 2026). Percentage error is calculated relative to the reference values.*

### Validation
With the results verified, this section shows that the fine mesh results agree with the data from (AIAA TMBWG, 2026).
In Table 7, the relative error between K-S theory and the fine mesh results are under 1%, showing good agreement with skin coefficient between 4000 < $$Re_{\theta}$$ < 12000.

| $$Re_{\theta}$$ | Kármán–Schoenherr | x2 Mesh | Error (%) |
| ---- | ------- | ------- | ----- |
| 4000 | 0.00314 | 0.00312 | 0.785 |
| 6000 | 0.00290 | 0.00291 | 0.165 |
| 8000 | 0.00275 | 0.00276 | 0.671 |
| 10000 | 0.00263 | 0.00266 | 0.854 |
| 11500 | 0.00257 | 0.00259 | 0.772 |

*Table 7: Validation of the mesh-independent solution using the Kármán–Schoenherr skin-friction correlation. Percentage error is calculated relative to the reference correlation.*

As stated in the verification section, for this study, the log-law region and viscous sublayer are validated, while the buffer zone (i.e., 5<$$y^+$$<30) is disregarded. Within the log-law region, the lowest errors occur between 30 < $$y^+$$ < 100, as Coles' law of the wall operates most accurately here. Beyond $$y^+$$ > 300, the error begins to increase as the flow approaches the outer layer and leave the log-law reigion.

| $y^+$ Location | Coles Theory $u^+$ | x2 Mesh $u^+$ | Relative Error (%) |
| :--- | :--- | :--- | :--- |
| 35 | 13.4 | 13.6 | 1.53 |
| 40 | 13.8 | 13.9 | 0.496 |
| 50 | 14.5 | 14.6 | 0.996 |
| 100 | 16.2 | 16.3 | 0.304 |
| 300 | 19.0 | 18.8 | 0.564 |
| 1000 | 22.4 | 21.9 | 1.96 |
| 1500 | 23.9 | 23.0 | 3.56 |

*Table 8: Validation of the mesh-independent solution using the Coles theory. Percentage error is calculated relative to the reference values.*



### Conclusion
Considering the results produced from this simulation using a pressure-based solver for subsonic, incompressible flow, i.e. Ma < 0.3, agrees with the validation data from Nasa TMR Zero Pressure Gradient flate plate case, means that it can produce the initial internal field for subsonic pre-inlet for a CD nozzle. The simulation settings here will be able to capature the subsonic, high temperature, and high pressure at the end of the combustion chamber as it goes into the nozzle.

## References 

Ghia, U., Ghia, K.N. and Shin, C.T., 1982. High-Re solutions for incompressible flow using the Navier-Stokes equations and a multigrid method. *Journal of Computational Physics*, 48(3), pp. 387–411.

AIAA Turbulence Model Benchmarking Working Group (TMBWG), 2026. *Turbulence Modeling Resource: Zero Pressure Gradient Flat Plate Validation Case*. Available at: <https://tmbwg.github.io/turbmodels/flatplate_val.html> [Accessed 9 June 2026].

Ansys (2026) Ansys Fluent Theory Guide. Canonsburg, PA: Ansys, Inc.

Spalart, P.R. and Allmaras, S.R. (1992) 'A one-equation turbulence model for aerodynamic flows', Technical Report AIAA-92-0439. Reno, NV: American Institute of Aeronautics and Astronautics. doi: 10.2514/6.1992-439.
