# madvadi.github.io
Portfolio of Cenk Tekin CFD Projects

Introduction: This is a presentation of a number of CFD projects, in which, I've be using to keep my skills up to date with advance in OpenFOAM 13 and CFD in general.

1.) Lid-cavity Demostration
To verfificate and validate my own CFD skills in setting up, running, and post-processing the results, I had a go at creating a simulation which I can easily validate agaist well estabished results from Ghia, Ghia, and Shin, 1982.
The model is a 2D square cavity, of unit dimensions (1m x 1m), with a Newtonian fluid having a lid being.
The goal is to see the velocity porfile is at half way along the unit cavity with a Reynolds number of 100 with a base line mesh of grid size, 129 x 129, and validate the results agaist my source.
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

*Figure 3: Velocity X Probes data at 4 probes over 4000 iterations.*

![Pressure Results](plots/Pressure_Probes.png)

*Figure 4: Pressure Probes data at 4 probes over 4000 iterations.*

![Velocity X Residuals](plots/residuals_for_u.png)

*Figure 5: Velocity X residual over 4000 iterations.*

![Pressure Residuals](plots/Residual_of_Pressure.png)

*Figure 6: Pressure residual over 4000 iterations.*

![Convergance Mesh](plots/ConvMesh.png)

*Figure 7: Convergance Mesh comparing 129 x129, 258 x 258, and 516 x 516 grid sizes.*

In Figure 7, shows that the results from all levels of refinement do not change significantly at all and with the error tabled in Table 1.

| Mesh Resolution | Total Cells | Max $U_x$ | % Change |
| :--- | :--- | :--- | :--- |
| 129 x 129 | 16,641 | -0.216 | — |
| 258 x 258 | 66,564 | -0.217 | 0.463% |
| 516 x 516 | 266,256 | -0.216 | 0.175% |

*Table 1, Convergance Mesh error when refined results are compared with the 129 x 129 mesh results.*

Conclusion 

From the results, we can see that the results from the Lid-Cavity simulation have been verificated and validated agaist Ghia et al. 1982 paper for a Reynolds number of 100.

Subsonic Turbulent Boundary Layer over a Flate Plate with a Compressible Pressure Solver

I produced 2 zero pressure gradient flate plate simulations in compressible, subsonic, turbulent flow using the k-omega SST and Spalart-Allmaras.
Nasa Turbulent Modelling Resource provides a validation case for my simulation to be compared agaist, which can be found at https://tmbwg.github.io/turbmodels/flatplate_val.html.

The baseline mesh that is used is a 175 by 90 grid size with the boundary conditions, as seen in Figure 8.

![Flate Plate Mesh](plots/Mesh_N_BCs.png)
*Figure 8: Mesh with 175 x 90 and the boundary conditions.*

For these turbulents models, it is important that the mesh near the wall is under y+ = 1 in order to resolve the visous sub-layer. Using the calcualted value of y coordiante for y+ = 1, I used a simple gradient ratio for bunching up the nodes along the y axis.

For the Spalart-Allmaras model, I used set the inlet for nuTilda = 3*nu, nuTilda = 4.7e-05 meters^2 per seconds and nut = 0. 


![SA Coarse Velocity Residuals](plots/SA/coarse/ResidualsOfVelocity.png)

*Figure 9: *


![SA Coarse Pressure Residuals](plots/SA/coarse/ResidualsOfPressure.png)

*Figure 10: *


![SA Coarse Velocity Probability](plots/SA/coarse/ProbVelocity.png)

*Figure 11: *


![SA Coarse Pressure Probability](plots/SA/coarse/ProbPressure.png)

*Figure 12: *


![SA Coarse Temperature Probability](plots/SA/coarse/ProbTemperature.png)

*Figure 13: *


#### Medium Mesh
![SA Medium Velocity Residuals](plots/SA/medium/ResidualsOfVelocity.png)

*Figure 14: *


![SA Medium Pressure Residuals](plots/SA/medium/ResidualsOfPressure.png)

*Figure 15: *


![SA Medium Velocity Probability](plots/SA/medium/ProbeVelocity.png)

*Figure 16: *


![SA Medium Pressure Probability](plots/SA/medium/ProbPressure.png)

*Figure 17: *


![SA Medium Temperature Probability](plots/SA/medium/ProbTemperature.png)

*Figure 18: *


#### Fine Mesh
![SA Fine Velocity Residuals](plots/SA/fine/ResidualsOfVelocity.png)

*Figure 19: *


![SA Fine Pressure Residuals](plots/SA/fine/ResidualsOfPressure.png)

*Figure 20: *


![SA Fine Velocity Probability](plots/SA/fine/ProbeVelocity.png)

*Figure 21: *


![SA Fine Pressure Probability](plots/SA/fine/ProbePressure.png)

*Figure 22: *


![SA Fine Temperature Probability](plots/SA/fine/ProbeTemperature.png)

*Figure 23: *


### Boundary Layer & Aerodynamic Coefficients
![SA Skin Friction Coefficient](plots/SA/SkinCoefficient.png)

*Figure 24: *


![SA Dimensionless Velocity Profile (u+ vs y+)](plots/SA/u+y+.png)

*Figure 25: *


![SA Momentum Thickness Reynolds Number vs X](plots/SA/ReThetaVsX.png)

*Figure 26: *

The GCI error from all 3 meshes is calculated to be 1.7%.



![SA Coarse Velocity Residuals](plots/k_omegaSST/coarse/ResidualsOfVelocity.png)

*Figure 27: *


![SA Coarse Pressure Residuals](plots/k_omegaSST/coarse/ResidualsOfPressure.png)

*Figure 28: *


![SA Coarse Velocity Probability](plots/k_omegaSST/coarse/ProbVelocity.png)

*Figure 29: *


![SA Coarse Pressure Probability](plots/k_omegaSST/coarse/ProbPressure.png)

*Figure 30: *


![SA Coarse Temperature Probability](plots/k_omegaSST/coarse/ProbTemperature.png)

*Figure 31: *


#### Medium Mesh
![SA Medium Velocity Residuals](plots/k_omegaSST/medium/ResidualsOfVelocity.png)

*Figure 32: *


![SA Medium Pressure Residuals](plots/k_omegaSST/medium/ResidualsOfPressure.png)

*Figure 33: *


![SA Medium Velocity Probability](plots/k_omegaSST/medium/ProbVelocity.png)

*Figure 34: *


![SA Medium Pressure Probability](plots/k_omegaSST/medium/ProbPressure.png)

*Figure 35: *


![SA Medium Temperature Probability](plots/k_omegaSST/medium/ProbTemperature.png)

*Figure 36: *

#### Fine Mesh
![SA Fine Velocity Residuals](plots/k_omegaSST/fine/ResidualsOfVelocity.png)

*Figure 37: *


![SA Fine Pressure Residuals](plots/k_omegaSST/fine/ResidualsOfPressure.png)

*Figure 38: *


![SA Fine Velocity Probability](plots/k_omegaSST/fine/ProbVelocity.png)

*Figure 39: *


![SA Fine Pressure Probability](plots/k_omegaSST/fine/ProbPressure.png)

*Figure 40: *


![SA Fine Temperature Probability](plots/k_omegaSST/fine/ProbTemperature.png)

*Figure 41: *


### Boundary Layer & Aerodynamic Coefficients
![SA Skin Friction Coefficient](plots/k_omegaSST/SkinCoefficient.png)

*Figure 42: *


![SA Dimensionless Velocity Profile (u+ vs y+)](plots/k_omegaSST/u+y+.png)

*Figure 43: *


![SA Momentum Thickness Reynolds Number vs X](plots/k_omegaSST/ReThetaVsX.png)

*Figure 44: *


For the k-$\Omega$ SST model, .... The GCI error from all 3 meshes is calculated to be ...
