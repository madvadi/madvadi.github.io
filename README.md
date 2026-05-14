# madvadi.github.io
Portfolio of Cenk Tekin CFD Projects

Introduction: This is a presentation of a number of CFD projects, in which, I've be using to keep my skills up to date with advance in OpenFOAM 13 and CFD in general.

1.) Lid-cavity Demostration
To verfificate and validate my own CFD skills in setting up, running, and post-processing the results, I had a go at creating a simulation which I can easily validate agaist well estabished results from Ghia, Ghia, and Shin, 1982.
The model is a 2D square cavity, of unit dimensions (1m x 1m), with a Newtonian fluid having a lid being.
The goal is to see the velocity porfile is at half way along the unit cavity with a Reynolds number of 100 with a base line mesh of grid size, 128 x 128, and vadliate the results agaist my source.
With setting the reference pressure to 0 at the bottom left concer of the domain, I've set the boundary conditions as:

![MeshAndBC](plots/MeshAndBC.png)

*Figure 1: The baseline mesh with the boundary conditions highlighted.*

With the initial field for pressure being set to a uniform 0, for velocity, a uniform 0 m/s in all direction, as seen in Figure 1.

The results compared with the vadlidation data in Ghia et al,1982 are presented in Figure 2:

![Results](plots/DataVnV.png)

*Figure 2: Comparison of normalized horizontal velocity (Ux) along the vertical centerline (x=0.5) against Ghia et al. (1982) benchmark data.*

Verification of Results

For the sake of verification, 4 probes where placed at locations along the vertical line in the centre of the cavity (x = 0.5 m) to measure the change in velocity and pressure.
Probe 1: y = 0.0 (m), Probe 2: y = 0.25 (m), Probe 3: y = 0.5 (m), Probe 4: y = 0.75 (m).

The data in Figure 3 and 4 shows that the Velocities and Pressure converage in a value. The residuals for both velocity and pressure also drop down several orders of magnitude, supporting the case that the simulation has convergence as can be seen in Figures 5 and 6.

![Velocity X Results](plots/Velocity_Probes.png)

*Figure 3: Velocity X Probes data at 4 probes over 20 iterations.*

![Pressure Results](plots/Pressure_Probes.png)

*Figure 4: Pressure Probes data at 4 probes over 20 iterations.*

![Velocity X Residuals](plots/residuals_for_u.png)

*Figure 5: Velocity X residual over 20 iterations.*

![Pressure Residuals](plots/Residual_of_Pressure.png)

*Figure 6: Pressure residual over 20 iterations.*

![Convergance Mesh](plots/ConvMesh.png)

*Figure 7: Convergance Mesh comparing 129 x129, 258 x 258, and 516 x 516 grid sizes.*

In Figure 7,shows that the results from all levels of refinement do not change significantly at all and with the error tabled in table 1.

| Mesh Resolution | Total Cells | Max $U_x$ | % Change |
| :--- | :--- | :--- | :--- |
| 129 x 129 | 16,641 | -0.216195 | — |
| 258 x 258 | 66,564 | -0.217195 | 0.463% |
| 516 x 516 | 266,256 | -0.215817 | 0.175% |

*Table 1, Convergance Mesh error when refined results are compared with the 129 x 129 mesh results.*

Conclusion 

From the results, we can see that the results from the Lid-Cavity simulation have been verificated and validated agaist Ghia et al. 1982 paper for a Reynolds number of 100.
