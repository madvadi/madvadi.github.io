# Engineering Case Studies in Computational Fluid Dynamics
# Project Summary
## Objective

Validate a pressure-based compressible OpenFOAM solver for a turbulent flat-plate boundary layer against the NASA Turbulence Modeling Resource (TMR) benchmark. This validation forms the basis for its application to converging-diverging nozzle simulations, where accurate prediction of wall boundary layers and stable subsonic flow solutions are required.

## Engineering Problem

Pressure-based solvers provide improved numerical stability for low-Mach compressible flows but must be verified before being used in larger engineering applications. The objective of this study was to demonstrate that the solver configuration accurately reproduces a well-established benchmark case while exhibiting mesh-independent behaviour and numerical convergence.

## Software and Tools

* OpenFOAM 13
* ParaView
* Python (post-processing)
* Git (version control)
* Markdown documentation

## Physics

* Compressible subsonic flow (Mach 0.2)
* Turbulent boundary layer
* Ideal gas
* Zero-pressure-gradient flat plate
* Spalart–Allmaras turbulence model

## Numerical Methods

* Structured blockMesh grids
* Three-grid refinement study
* Grid Convergence Index (GCI)
* Residual convergence monitoring
* Probe-based solution verification
* Comparison against NASA TMR reference data

## Key Results

* Stable pressure-based solver configuration established.
* Grid-independent solution demonstrated using GCI methodology.
* Residuals reduced by multiple orders of magnitude with stable monitored quantities.
* Solver behaviour shown to agree with published benchmark data.

## Skills Demonstrated

* CFD solver configuration
* OpenFOAM case development
* Turbulence modelling
* Mesh generation and refinement
* Verification and validation (V&V)
* Numerical error estimation
* Engineering judgement
* Technical report writing

## My Contribution

This project was completed independently to develop and demonstrate practical CFD engineering capability using OpenFOAM 13. My responsibilities included:

* Developing the OpenFOAM case structure and simulation setup.
* Selecting and justifying appropriate physical models and numerical methods.
* Designing structured meshes suitable for boundary-layer resolution.
* Performing a systematic three-grid refinement study.
* Implementing Grid Convergence Index (GCI) calculations to quantify numerical uncertainty.
* Defining and validating appropriate inlet, outlet, wall, and freestream boundary conditions.
* Monitoring residuals and internal solution probes to verify convergence.
* Post-processing simulation results and producing engineering figures.
* Comparing numerical predictions against NASA Turbulence Modeling Resource benchmark data.
* Documenting the methodology, assumptions, and engineering conclusions in a professional technical report.

The objective of this work was not simply to produce CFD results, but to demonstrate a complete engineering workflow including verification, validation, and technical communication.


### Error Calculation Methodology

In the following projects, using the methodology from (Roache, 2009), the calculation of the Grid Convergence Index (GCI) error is as follows:

1. **Calculate Representative Grid Sizes ($h$):** Generate three distinct meshes—coarse, medium, and fine—and compute the characteristic grid size ($h_1, h_2, h_3$) for each using the total domain area and total cell count:
   $$h = \sqrt{\frac{A_{\text{total}}}{N}}$$

2. **Determine Grid Refinement Ratios ($r$):** Calculate the refinement ratios between the grid pairs. It is highly recommended to keep these ratios above 1.3 to ensure the grid resolutions are distinct enough to capture discretization changes:
   $$r_{21} = \frac{h_2}{h_1}, \quad r_{32} = \frac{h_3}{h_2}$$

3. **Extract Solution Variables ($f$):** Run the CFD simulations and extract a critical target scalar variable (such as drag coefficient, lift coefficient, or peak velocity) from all three grids, yielding $f_1$ (fine), $f_2$ (medium), and $f_3$ (coarse).

4. **Solve for the Local Order of Accuracy ($p$):** Solve iteratively for the apparent order of accuracy ($p$) using the grid refinement ratios and the differences between the solutions:
   $$p = \frac{1}{\ln(r_{21})} \left| \ln\left| \frac{\epsilon_{32}}{\epsilon_{21}} \right| + q(p) \right|$$ 
   where $q(p) = \ln\left(\frac{r_{21}^p - s}{r_{32}^p - s}\right)$, $\epsilon_{32} = f_3 - f_2$, $\epsilon_{21} = f_2 - f_1$, and $s = 1 \cdot \text{sgn}\left(\frac{\epsilon_{32}}{\epsilon_{21}}\right)$.

5. **Calculate Relative Error ($\epsilon_{32}$ and $\epsilon_{21}$):** Determine the relative error between the two finest grids:
   $$\epsilon_{21} = \left| \frac{f_2 - f_1}{f_1} \right| \quad \text{and} \quad \epsilon_{32} = \left| \frac{f_3 - f_2}{f_2} \right|$$

6. **Compute the GCI Error:** Finally, calculate the fine- and medium-grid GCI error by applying a safety factor ($Fs$), which is typically set to 1.25 for a rigorous three-grid study: 
   $$GCI_{fine} = \frac{F_s \cdot \epsilon_{21}}{r_{21}^p - 1}$$.

> **Note:** The resulting percentage represents your numerical uncertainty band. It quantifies how close your fine-grid solution is to the theoretical, asymptotic "grid-independent" solution.

## Subsonic Turbulent Boundary Layer over a Flat Plate with a Compressible Pressure Solver

In a convergent-divergent (CD) nozzle, a high-temperature, high-pressure subsonic flow is converted into a supersonic flow before being exhausted from the engine. A high-fidelity simulation must remain stable across two flow regimes (i.e., subsonic and supersonic) while capturing the boundary layer at the nozzle wall to determine quantities such as wall temperature. Using a density-based solver in the subsonic section of a CD nozzle makes it difficult to maintain stability. Therefore, a pressure-based solver is used to produce an internal field for that specific section of the nozzle (Ansys, 2026). Several assumptions are made for the nozzle simulation as a whole, which affect this validation unit case: the flow is chemically frozen, the cross-sectional area for the subsonic section is constant, and the flow is modeled as 2D axisymmetric. 

To validate the internal field produced by the pressure solver, a unit case study was conducted using zero-pressure-gradient flat-plate simulations in an incompressible (modeled as compressible), subsonic, turbulent air flow. The Spalart–Allmaras model, as a one-equation model, is computationally efficient for large mesh sizes, such as those used for CD nozzles. The NASA Turbulence Modeling Resource provides a validation case for my simulation to be compared against, which can be found at (AIAA TMRWG, 2026). The working fluid is air modelled as an ideal gas, using a reference temperature of 300 K and an inlet/freestream velocity which is found to be 69.522 m/s, corresponding to a Mach number of 0.2, matching the NASA TMR benchmark conditions. 

### Mesh and Boundary Conditions

![Flat Plate Mesh](plots/Mesh_N_BCs.png)

*Figure 1: Mesh with a 175 x 90 resolution and its corresponding boundary conditions.*

The baseline mesh features a 175 by 90 grid size, with the boundary conditions shown in Figure 1. For these turbulence models, it is crucial that the mesh spacing near the wall remains under $y^+ = 1$ to fully resolve the viscous sublayer. Using the calculated $y$-coordinate value for $y^+ = 1$, I applied a simple expansion ratio to cluster the nodes closely along the $y$-axis. For the baseline mesh, the closest cell center to the wall was $1.4343 \times 10^{-5}$ meters.

For the grid independence study, a medium mesh was generated by multiplying the baseline mesh nodes by 1.5 in each direction, resulting in a 263 × 135 grid. For the fine mesh, the nodes were doubled in each direction relative to the baseline mesh, resulting in a grid size of 350 × 180. This yields refinement ratios of $r_{32} = 1.5$ and $r_{21} = 1.33$, both of which are above the recommended minimum refinement ratio of 1.1 (Roache, 2009). I chose these refinement sizes to be as efficient as possible while also being within the asymptotic range. 

This validation case was selected particularly because it shares similarities with the inlet boundary conditions of a CD nozzle. An inlet boundary for a CD nozzle requires one of the variables to float at the inlet to keep the throat choked (Anderson, 1995). Hence, total temperature and total pressure were chosen as the inlet boundary conditions, allowing the velocity to float. The total pressure and total temperature values were set using the case-specific ratios $p_t/p_{\text{ref}} = 1.02828$ and $T_t/T_{\text{ref}} = 1.008$, with $T_{\text{ref}} = 300\text{ K}$ and $p_{\text{ref}} = 101,325\text{ Pa}$. The outlet enforces a fixed static pressure matching the reference atmospheric value ($p/p_{\text{ref}} = 1$), while allowing velocity and temperature to float via zero-gradient conditions. 

The top patch boundary condition was set to freestream conditions using the reference velocity, pressure, and temperature values that were also applied to the internal field, thereby achieving a zero-pressure gradient along the $y$-axis. On the bottom surface, between the inlet and the plate, a symmetry condition is used to allow the freestream flow to make contact with the flat plate and enable the boundary layer to develop naturally. This is where the case slightly varies from a CD nozzle; in a nozzle, the inlet is located directly on a wall, meaning an initial turbulent boundary layer must be provided at the inlet. However, this validation case is used primarily to ensure that subsonic flows are correctly resolved by the solver setup. Finally, the flat plate has a no-slip velocity boundary condition, while temperature and pressure were set to zero-gradient. 

For the Spalart–Allmaras model, the inlet values were set to $\tilde{\nu} = 3\nu = 4.7 \times 10^{-5}\text{ m}^2/\text{s}$, while the derived turbulent kinematic viscosity ($\nu_t$) naturally updates based on the local $\tilde{\nu}$ transport field (Spalart and Allmaras, 1992). The internal field was initialized with $\tilde{\nu} = 4.7 \times 10^{-5}\text{ m}^2/\text{s}$, and this same value was used for the top freestream boundary condition. At the outlet, an inlet-outlet condition was specified using this value, which appears to avoid numerical instabilities. On the plate, the boundary condition was set to a fixed value of $0\text{ m}^2/\text{s}$. The parameter $\nu_t$ is calculated everywhere because of its dependence on $\tilde{\nu}$, with the exception of the plate surface, where its value is fixed to $0\text{ m}^2/\text{s}$.

### Verification
For verification, the residuals for velocity, pressure, and temperature were recorded to show that they met the convergence criteria, having dropped by at least three orders of magnitude and leveled out. Convergence criteria also dictate that the values of these quantities do not change for a steady-state flow; hence, several probes were placed in key locations, as shown in Table 1.

| Probe Number | Coordinates (x, y) |
| ------------ | ------------------ |
| 1            | 0.5, 0.025         |
| 2            | 0.5, 0.25          |
| 3            | 1, 0.025           |
| 4            | 1, 0.25            |
| 5            | 2, 0.025           |
| 6            | 2, 0.25            |

*Table 1: Probe locations within the domain.*

Probes 1 and 2 are positioned at the leading edge of the plate. This is where the wake forms as the flow hits the plate—a point of potential unsteady behavior—and is therefore worth monitoring to ensure that the values at those points stabilize. Probes 1, 3, and 5 are set 0.025 m from the wall. This ensures that the quantities are stable at two points within the log-law portion of the boundary layer (in the case of probes 3 and 5), and just above the leading edge to measure how the quantities are affected by the wake (in the case of probe 1). Probes 2, 4, and 6 are located in the freestream above the boundary layer to ensure that no unsteady behavior is occurring there and that the freestream values are maintained.

![SA Coarse Velocity Residuals](plots/SA/coarse/ResidualsOfVelocity.png)

*Figure 2: Convergence history of velocity residuals for the SA model on the coarse mesh.*

![SA Coarse Pressure Residuals](plots/SA/coarse/ResidualsOfPressure.png)

*Figure 3: Convergence history of pressure residuals for the SA model on the coarse mesh.*

In Figures 2 and 3, it can be seen that the residuals have dropped and flatlined at 1e-05 for Ux, 1e-06 for Uy, and 1e-04 for pressure; hence, this meets the criteria for convergence for the coarse mesh. 

![SA Medium Velocity Residuals](plots/SA/medium/ResidualsOfVelocity.png)

*Figure 4: Convergence history of velocity residuals for the SA model on the medium mesh.*

![SA Medium Pressure Residuals](plots/SA/medium/ResidualsOfPressure.png)

*Figure 5: Convergence history of pressure residuals for the SA model on the medium mesh.*

In Figures 4 and 5, it is visible that the residuals have dropped and averaged out at 1e-11 for Ux, 1e-09 for Uy, and 1e-07 for pressure; hence, this meets the criteria for convergence for the medium mesh. 

![SA Fine Velocity Residuals](plots/SA/fine/ResidualsOfVelocity.png)

*Figure 6: Convergence history of velocity residuals for the SA model on the fine mesh.*

![SA Fine Pressure Residuals](plots/SA/fine/ResidualsOfPressure.png)

*Figure 7: Convergence history of pressure residuals for the SA model on the fine mesh.*

In Figures 6 and 7, the residuals have dropped to 1e-10 for Ux, 1e-08 for Uy, and 1e-07 for pressure, which have averaged out; hence, this meets the criteria for convergence for the fine mesh. However, as the order of magnitude did not perfectly flatline, it is important to show that the actual values stay consistent over the iterations. Figures 8–10 display the final values that each of the probes converged on by the end of the simulation, plotted against the number of cells used in the simulation. For all three variables, no significant change was found as the mesh was refined. 

![SA Fine Velocity Probes](plots/SA/AllLevelsVelocityProbes.png)

*Figure 8: Velocity Probes.*

![SA Fine Pressure Probes](plots/SA/AllLevelsPressureProbes.png)

*Figure 9: Pressure Probes.*

![SA Fine Temperature Probes](plots/SA/AllLevelsTemperatureProbes.png)

*Figure 10: Temperature Probes.*

To understand the contribution of viscosity to the drag, the skin friction coefficient profile is calculated across the plate, as seen in Figure 11. In Table 2, at several key points on the plate, the relative and GCI errors are calculated using $\text{Re}_{\theta}$ at that point. The solutions are found to have converged at those points, meaning the GCI and relative errors decrease as the meshes are refined.

![SA Skin Friction Coefficient](plots/SA/SkinCoefficient.png)
*Figure 11: Local skin friction coefficient distribution along the surface calculated with the SA model.*

| $\text{Re}_{\theta}$ | Base Mesh | x1.5 Mesh | x2 Mesh | Base → x1.5 (%) | x1.5 → x2 (%) | $\text{GCI}_{\text{medium}}$ (%) | $\text{GCI}_{\text{fine}}$ (%) |
| -------------------- | --------- | --------- | ------- | --------------- | ------------- | -------------------------------- | ------------------------------ |
| 4000                 | 0.00295   | 0.00311   | 0.00312 | 5.11            | 0.42          | 0.66                             | 0.12                           |
| 6000                 | 0.00276   | 0.00289   | 0.00291 | 4.49            | 0.45          | 0.72                             | 0.15                           |
| 8000                 | 0.00265   | 0.00275   | 0.00276 | 3.83            | 0.48          | 0.81                             | 0.21                           |
| 10000                | 0.00256   | 0.00265   | 0.00266 | 3.47            | 0.29          | 0.44                             | 0.08                           |
| 11500                | 0.00250   | 0.00258   | 0.00259 | 3.28            | 0.25          | 0.20                             | 0.07                           |

*Table 2: Verification of mesh convergence using the skin-friction coefficient* $$C_f$$ *at selected momentum-thickness Reynolds numbers. Percentage differences are calculated relative to the finer mesh solution.*

Because analytic profiles like Coles' Law do not cleanly describe the highly non-linear buffer zone ($5 < y^+ < 30$), validation comparison is isolated to the strictly valid algebraic limits: the linear viscous sublayer and the fully developed log-law region. Due to its complexity, this study will only validate the viscous sublayer and log-law $u^+$ velocity profiles. In Figure 12, between $5 < y^+ < 30$, $u^+ = y^+$ is used as a placeholder, while at $y^+ \le 5$, $y^+ = u^+$ is applied, and $y^+ > 30$ uses Coles' Mean Velocity profile law (AIAA TMRWG, 2026). All refinement levels show a similar profile in Figure 12. In Table 3, the relative and GCI errors measured at several key $y^+$ values systematically decrease with mesh refinement. The error notably decreases upon entering the log-law region soon after leaving the buffer zone (Apsley, 2009).

![SA Dimensionless Velocity Profile (u+ vs y+)](plots/SA/u+y+.png)

*Figure 12: Dimensionless boundary layer velocity profile (* $$u^+$$ *vs* $$y^+$$ *) plotted against the theoretical law of the wall using the SA model, at* $$\text{Re}_{\theta} = 10000$$.

| $y^+$ | Base Mesh | x1.5 Mesh | x2 Mesh | Base → x1.5 (%) | x1.5 → x2 (%) | $\text{GCI}_{\text{medium}}$ (%) | $\text{GCI}_{\text{fine}}$ (%) |
| ----- | --------- | --------- | ------- | --------------- | ------------- | -------------------------------- | ------------------------------ |
| 35    | 13.9      | 13.6      | 13.6    | 2.44            | 0.573         | 0.832                            | 0.365                          |
| 40    | 13.9      | 13.9      | 13.9    | 0.137           | 0.0689        | 0.148                            | 0.119                          |
| 50    | 14.4      | 14.5      | 14.6    | 1.21            | 0.472         | 1.56                             | 0.961                          |
| 100   | 16.2      | 16.3      | 16.3    | 0.283           | 0.0259        | 0.0405                           | 8.14e-03                       |
| 300   | 18.8      | 18.9      | 18.9    | 0.730           | 0.00415       | 0.0102                           | 0.909                          |
| 1000  | 22.0      | 21.9      | 21.9    | 0.521           | 0.210         | 0.382                            | 0.258                          |

*Table 3: Verification of mesh convergence based on the* $$u^+$$ *value relative to the* $$y^+$$ *values. Percentage differences are calculated relative to the finer mesh solution.*

To verify that the growth rate of the boundary layer follows the correct trend, the $$\text{Re}_{\theta}$$ profile is plotted along the plate for all mesh refinements (Figure 13). The $$Re_{\theta}$$ results demonstrate excellent grid convergence with one another, as supported by Table 4, where the relative and Grid Convergence Index (GCI) errors systematically decrease to well under 1%. While grid-independent, the profiles maintain a small, stable offset from the reference data provided by the NASA Turbulence Modeling Resource (NASA TMR, 2026). 

As detailed in Table 5, the maximum deviation from the NASA baseline is 4% near the leading edge, which progressively decreases downstream to 3.08% at the end of the plate. At the leading edge, the flow experiences a severe velocity gradient as it transitions from the freestream velocity to zero to satisfy the no-slip condition. In the ultimate CD nozzle configuration, the simulation inlet is located directly at the combustion chamber wall exit plane rather than introducing an upstream freestream stagnation point. Consequently, the downstream nozzle domain will entirely bypass this leading-edge singularity error. This localized stagnation effect introduces strong local pressure and velocity gradients that impact downstream eddy viscosity development, as evidenced at $$\frac{x}{L_{\text{plate}}} = 0.2$$ in Table 5. To mitigate this geometric singularity, the grid was refined toward the leading edge (Figure 1) to resolve these steep near-wall gradients. Because the NASA TMR data represents a model-agnostic, typical $$Re_{\theta}$$ progression, it is expected that specific implementations of the Spalart–Allmaras model will exhibit a minor, highly bounded variation while matching the overall spatial trend. 

![SA Momentum Thickness Reynolds Number vs X](plots/SA/ReThetaVsX.png)

*Figure 13: Development of the momentum thickness Reynolds number (* $$Re_{\theta}$$ *) along the streamwise direction* $$\frac{x}{L}$$ *for the SA model, at* $$Re_{\theta} = 10000$$.

| $\frac{x}{L_{\text{plate}}}$ | Base Mesh | x1.5 Mesh | x2 Mesh | Base → x1.5 (%) | x1.5 → x2 (%) | $\text{GCI}_{\text{medium}}$ (%) | $\text{GCI}_{\text{fine}}$ (%) |
| ---------------------------- | --------- | --------- | ------- | --------------- | ------------- | -------------------------------- | ------------------------------ |
| 0.2                          | 3780      | 3620      | 3615    | 4.54            | 0.0161        | 0.0210                           | 3.89e-04                       |
| 0.4                          | 6620      | 6490      | 6443    | 2.10            | 0.695         | 1.87                             | 1.02                           |
| 0.6                          | 9320      | 9130      | 9070    | 2.08            | 0.707         | 1.95                             | 1.08                           |
| 0.8                          | 11900     | 11600     | 11600   | 2.89            | 0.268         | 0.418                            | 0.0847                         |
| 1.0                          | 14300     | 14000     | 13900   | 2.53            | 0.381         | 0.670                            | 0.196                          |

*Table 4: Verification of mesh convergence based on the momentum-thickness Reynolds number* $$Re_{\theta}$$ *at selected streamwise locations. Percentage differences are calculated relative to the finer mesh solution.*

| $\frac{x}{L_{\text{plate}}}$ | NASA TMR | x2 Mesh | Error (%) |
| ---------------------------- | -------- | ------- | --------- |
| 0.2                          | 3766     | 3615    | 4.00      |
| 0.4                          | 6680     | 6443    | 3.54      |
| 0.6                          | 9428     | 9067    | 3.83      |
| 0.8                          | 11961    | 11578   | 3.21      |
| 1.0                          | 14384    | 13941   | 3.08      |

*Table 5: Validation of the mesh-independent solution using reference data from (AIAA TMRWG, 2026). Percentage error is calculated relative to the reference values.*

### Validation
With the results verified, this section shows that the fine mesh results agree closely with the data from (AIAA TMRWG, 2026). In Table 6, the relative error between Kármán–Schoenherr (K-S) theory and the fine mesh results is under 1%, showing good agreement with the skin friction coefficient between $4000 < \text{Re}_{\theta} < 12000$.

| $\text{Re}_{\theta}$ | Kármán–Schoenherr | x2 Mesh | Error (%) |
| -------------------- | ----------------- | ------- | --------- |
| 4000                 | 0.00314           | 0.00312 | 0.785     |
| 6000                 | 0.00290           | 0.00291 | 0.165     |
| 8000                 | 0.00275           | 0.00276 | 0.671     |
| 10000                | 0.00263           | 0.00266 | 0.854     |
| 11500                | 0.00257           | 0.00259 | 0.772     |

*Table 6: Validation of the mesh-independent solution using the Kármán–Schoenherr skin-friction correlation. Percentage error is calculated relative to the reference correlation.*

As stated in the verification section, the log-law region and viscous sublayer are validated for this study, while the buffer zone (i.e., $5 < y^+ < 30$) is disregarded. At $${Re}_{\theta} = 10000$$, within the log-law region, the lowest errors occur between $30 < y^+ < 100$, as Coles' law of the wall operates most accurately here. Beyond $y^+ = 300$, where $\frac{y}{\delta} = 0.1$, the error begins to increase as the flow approaches the outer layer, leaving the inner region and log-law layer at $y^+ = 860$, where $\frac{y}{\delta} = 0.3$.

| $y^+$ Location | Coles Theory $u^+$ | x2 Mesh $u^+$ | Relative Error (%) |
| :------------- | :----------------- | :------------ | :----------------- |
| 35             | 13.4               | 13.6          | 1.53               |
| 40             | 13.8               | 13.9          | 0.496              |
| 50             | 14.5               | 14.6          | 0.996              |
| 100            | 16.2               | 16.3          | 0.304              |
| 300            | 19.0               | 18.8          | 0.564              |
| 1000           | 22.4               | 21.9          | 1.96               |

*Table 7: Validation of the mesh-independent solution using Coles theory. Percentage error is calculated relative to the reference values.*

### Conclusion
The results produced from this simulation using a pressure-based solver for subsonic, incompressible flow (i.e., $${Ma} < 0.3$$) agree closely with the validation data from the NASA TMR Zero Pressure Gradient flat plate case, with low GCI errors, under 1.08% which occurred in the $\text{Re}_{\theta}$ in Table 4. This demonstrates that the setup can reliably produce the initial internal field for the subsonic pre-inlet section of a CD nozzle. Consequently, the simulation settings validated here will successfully capture the subsonic, high-temperature, and high-pressure conditions present at the combustion chamber exit, where the gas behaves as an ideal gas at subsonic speeds.

## Engineering Outcome

The validated solver configuration provides confidence that the numerical methodology is suitable for more complex internal flow applications, including converging-diverging nozzle simulations where accurate prediction of near-wall flow behaviour is essential. This project strengthened my practical understanding of:

* CFD verification and validation methodologies.
* Mesh independence and numerical uncertainty estimation.
* Appropriate selection of turbulence models.
* Boundary-condition specification for compressible flows.
* Interpretation of convergence behaviour beyond residual monitoring.
* Technical communication of engineering analyses.

## Future Work

Future developments will extend this validated solver setup to axisymmetric converging-diverging nozzle simulations incorporating heat transfer, variable geometry, and more complex flow physics while maintaining the same verification and validation methodology.


## References 

Ghia, U., Ghia, K.N. and Shin, C.T., 1982. High-Re solutions for incompressible flow using the Navier-Stokes equations and a multigrid method. *Journal of Computational Physics*, 48(3), pp. 387–411.

AIAA Turbulence Model Benchmarking Working Group (TMRWG), 2026. *Turbulence Modeling Resource: Zero Pressure Gradient Flat Plate Validation Case*. Available at: <https://tmbwg.github.io/turbmodels/flatplate_val.html> [Accessed 9 June 2026].

Ansys (2026) Ansys Fluent Theory Guide. Canonsburg, PA: Ansys, Inc.

Spalart, P.R. and Allmaras, S.R. (1992) 'A one-equation turbulence model for aerodynamic flows', Technical Report AIAA-92-0439. Reno, NV: American Institute of Aeronautics and Astronautics. doi: 10.2514/6.1992-439.

Apsley, D., 2009. Structure of a Turbulent Boundary Layer. Lecture Notes: Turbulent Boundary Layers. University of Manchester. Available at: https://personalpages.manchester.ac.uk/staff/david.d.apsley/lectures/turbbl/regions.pdf [Accessed 15 June 2026].

Roache, P.J., 2009. *Fundamentals of Verification and Validation*. Albuquerque: Hermosa Publishers.

Anderson, J.D., 1995. *Computational Fluid Dynamics: The Basics with Applications*. New York: McGraw-Hill.
