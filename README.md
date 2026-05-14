# madvadi.github.io
Portfolio of Cenk Tekin CFD Projects

Introduction: This is a presentation of a number of CFD projects, in which, I've be using to keep my skills up to date with advance in OpenFOAM 13 and CFD in general.

1.) Lid-cavity Demostration
To verfificate and validate my own CFD skills in setting up, running, and post-processing the results, I had a go at creating a simulation which I can easily validate agaist well estabished results from Ghia, Ghia, and Shin, 1982.
The model is a 2D square cavity, of unit dimensions (1m x 1m), with a Newtonian fluid having a lid being.
The goal is to see the velocity porfile is at half way along the unit cavity with a Reynolds number of 100 with a grid size, 128 x 128,  and vadliate the results agaist my source.

The results are:

![Results](plots/DataVnV.png)

*Figure 1: Comparison of normalized horizontal velocity ($U_x$) along the vertical centerline ($x=0.5$) against Ghia et al. (1982) benchmark data.*

For the sake of verification, 4 probes where placed at locations along the vertical line in the centre of the cavity (x = 0.5 m) to measure the change in velocity and pressure.
Probe 1: y = 0.0 (m), Probe 2: y = 0.25 (m), Probe 3: y = 0.5 (m), Probe 4: y = 0.75 (m).

The data in Figure 2 and 3 shows that the Velocities and Pressure converage in a value. The residuals for both velocity and pressure also drop down several orders of magnitude, supporting the case that the simulation has convergence as can be seen in Figures 4 and 5.

![Velocity X Results](plots/Velocity_Probes.png)

*Figure 2: Velocity X Probes data at 4 probes over 20 iterations.*

![Pressure Results](plots/Pressure_Probes.png)

*Figure 3: Pressure Probes data at 4 probes over 20 iterations.*

![Velocity X Residuals](plots/residuals_for_u.png)

*Figure 4: Velocity X residual over 20 iterations.*

![Pressure Residuals](plots/Residual_of_Pressure.png)

*Figure 5: Pressure residual over 20 iterations.*

Convergance Mesh... to be continue
