---
title: Governing Equations in SU2
permalink: /docs/Theory/
---

`TK:: Are all information that seem to be identical between FVM and FEM solver actually identical. E.g. Sutherlands law for viscosity or non-ideal fluids are available for the FEM solver, Turbulence models. Somewhere a disclaimer could be written s.th. like Unless you know exactly intend to use the nodal DG solver... Use the FVM solvers. I dont know how Edwin sees this guide. Because I might be an idea to leave the DG solver out of this completely. There is also no tutorial. If I am well informed, mesh genertion is not that straight forward either.  `

`TK:: Btw I am not looking at correct punctuation at the end of the formulas as there is none ;)`

`TK:: `

**This guide is for version 7 only**

This page contains a very brief summary of the different governing equation sets that are treated in each of the solvers within SU2. The reader will be referred to other references `TK:: Good idea... no reference given though :) Sorry turbulence is given. Do you plan to include publications as well. That would be especially fitting for your incompressible paper.` for the full detail of the numerical implementations, but we will also describe the approaches at a high level here.

---

## Content ##
- [Compressible Navier-Stokes](#compressible-rans)
- [Compressible Euler](#compressible-euler)
- [Incompressible Navier-Stokes](#incompressible-rans)
- [Incompressible Euler](#incompressible-euler)
- [Turbulence Modeling](#turbulence-modeling) `TK:: section was missing`
- [Elasticity](#elasticity)
- [Heat Conduction](#heat-conduction)
  
---

# Compressible Navier-Stokes #

| Solver | Version | 
| --- | --- |
| `NAVIER_STOKES`, `RANS`, `FEM_NAVIER_STOKES` | 7.0.0 |


SU2 solves the compressible Navier-Stokes equations expressed in differential form as

$$ \mathcal{R}(U) = \frac{\partial U}{\partial t} + \nabla \cdot \bar{F}^{c}(U) - \nabla \cdot \bar{F}^{v}(U,\nabla U)  - S = 0 $$

where the conservative variables are the working variables and given by 

$$U = \left \{  \rho, \rho \bar{v},  \rho E \right \}^\mathsf{T}$$ 

$$S$$ is a generic source term, and the convective and viscous fluxes are

$$\bar{F}^{c}   = \left \{ \begin{array}{c} \rho \bar{v}  \\ \rho \bar{v} \otimes  \bar{v} + \bar{\bar{I}} p \\ \rho E \bar{v} + p \bar{v}   \end{array} \right \}$$

and 

$$\bar{F}^{v} = \left \{ \begin{array}{c} \cdot \\ \bar{\bar{\tau}} \\ \bar{\bar{\tau}} \cdot \bar{v} + \kappa \nabla T  \end{array} \right  \}$$

`TK:: tau dot v i.e. matrix dot vector ... what is that?`

where $$\rho$$ `TK:: I am a huge fan of giving units, maybe introducing a fixed nomenclature could be a good way to enforce consistency across domains (is it even possible to avoid double definition of a symbol? If not first come first serve principle).` is the fluid density [kg/s], $$\bar{v}=\left\lbrace u, v, w \right\rbrace^\mathsf{T}$$ $$\in$$ $$\mathbb{R}^3$$ is the flow speed in Cartesian system of reference [m/s], $$E$$ is the total energy per unit mass [J/kg], $$p$$ is the static pressure [Pa], $$\bar{\bar{\tau}}$$ is the viscous stress tensor [Pa], $$T$$ is the temperature [K], $$\kappa$$ is the thermal conductivity [W/(m*K)], and $$\mu$$ is the viscosity [Pa*s]. The viscous stress tensor can be expressed in vector notation as

$$\bar{\bar{\tau}}= \mu \left ( \nabla \bar{v} + \nabla \bar{v}^{T} \right ) - \mu \frac{2}{3} \bar{\bar I} \left ( \nabla \cdot \bar{v} \right )$$

Assuming a perfect gas `TK:: Are there other options? If yes ` with a ratio of specific heats $$\gamma$$ [-] and specific gas constant $$R$$ [J/(K*mol)], one can close the system by determining pressure from $$p = (\gamma-1) \rho \left [ E - 0.5(\bar{v} \cdot \bar{v} ) \right ]$$ and temperature from the ideal gas equation of state $$T = p/(\rho R)$$. Conductivity can be a constant, or we assume a constant Prandtl number $$Pr$$ [-] such that the conductivity varies with viscosity as $$\kappa = \mu c_p / Pr$$. 

It is also possible to model non-ideal fluids within SU2 using more advanced fluid models that are available, but this is not discussed here. Please see the tutorial on the topic.

For laminar flows, $$\mu$$ is simply the dynamic viscosity $$\mu_{d}$$, which can be constant or assumed to satisfy Sutherland's law as a function of temperature alone, and $$Pr$$ is the dynamic Prandtl number $$Pr_d$$. For turbulent flows, we solve the Reynolds-averaged Navier-Stokes (RANS) equations. In accord with the standard approach to turbulence modeling based upon the Boussinesq hypothesis, which states that the effect of turbulence can be represented as an increased viscosity, the viscosity is divided into dynamic and turbulent components, or  $$\mu_{d}$$ and $$\mu_{t}$$, respectively. Therefore, the effective viscosity in becomes `TK:: I do like the litte explanation on how turbulent effects are accounted for. Maybe mention SA and SST is available. Explanation of Turbulence equations necessary (but that can follow in a later step.)` `TK:: For more detailed information see the seperate section` [Turbulence Modeling](#turbulence-modeling)

$$\mu =\mu_{d}+\mu_{t}$$

Similarly, the thermal conductivity in the energy equation becomes an effective thermal conductivity written as

$$\kappa =\frac{\mu_{d} \, c_p}{Pr_{d}}+\frac{\mu_{t} \, c_p}{Pr_{t}}$$

where we have introduced a turbulent Prandtl number $$Pr_t$$. The turbulent viscosity $$\mu_{t}$$ is obtained from a suitable turbulence model involving the mean flow state $$U$$ and a set of new variables for the turbulence. 

Within the `NAVIER_STOKES` and `RANS` solvers, we discretize the equations in space using a finite volume method (FVM) with a standard edge-based data structure on a dual grid with vertex-based schemes. The convective and viscous fluxes are evaluated at the midpoint of an edge. In the `FEM_NAVIER_STOKES` solver, we discretize the equations in space with a nodal Discontinuous Galerkin (DG) finite element method (FEM) with high-order (> 2nd-order) capability.

---

# Compressible Euler #

| Solver | Version | 
| --- | --- |
| `EULER`, `FEM_EULER` | 7.0.0 |

SU2 solves the compressible Euler equations, which can be obtained as a simplification of the compressible Navier-Stokes equations in the absence of viscosty and thermal conductivity. They can be expressed in differential form as

 $$ \mathcal{R}(U) = \frac{\partial U}{\partial t} + \nabla \cdot \bar{F}^{c}(U) - S = 0 $$

where the conservative variables are the working variables and are given by 

$$U = \left \{  \rho, \rho \bar{v},  \rho E \right \}^\mathsf{T}$$ 

$$S$$ is a generic source term, and the convective flux is

$$\bar{F}^{c}   = \left \{ \begin{array}{c} \rho \bar{v}  \\ \rho \bar{v} \otimes  \bar{v} + \bar{\bar{I}} p \\ \rho E \bar{v} + p \bar{v}   \end{array} \right \}$$

where $$\rho$$ is the fluid density, $$\bar{v}=\left\lbrace u, v, w \right\rbrace^\mathsf{T}$$ $$\in$$ $$\mathbb{R}^3$$ is the flow speed in Cartesian system of reference, $$E$$ is the total energy per unit mass, $$p$$ is the static pressure, and $$T$$ is the temperature. Assuming a perfect gas with a ratio of specific heats $$\gamma$$ and gas constant $$R$$, one can close the system by determining pressure from $$p = (\gamma-1) \rho \left [ E - 0.5(\bar{v} \cdot \bar{v} ) \right ]$$ and temperature from the ideal gas equation of state $$T = p/(\rho R)$$.

Within the `EULER` solvers, we discretize the equations in space using a finite volume method (FVM) with a standard edge-based data structure on a dual grid with vertex-based schemes. The convective and viscous fluxes are evaluated at the midpoint of an edge. In the `FEM_EULER` solver, we discretize the equations in space with a nodal Discontinuous Galerkin (DG) finite element method (FEM) with high-order (> 2nd-order) capability.

---

# Incompressible Navier-Stokes #

| Solver | Version | 
| --- | --- |
| `INC_NAVIER_STOKES`, `INC_RANS` | 7.0.0 |


SU2 solves the incompressible Navier-Stokes equations in a general form allowing for variable density due to heat transfer through the low-Mach approximation (or incompressible ideal gas formulation). The equations can be expressed in differential form as

$$ \mathcal{R}(V) = \frac{\partial V}{\partial t} + \nabla \cdot \bar{F}^{c}(V) - \nabla \cdot \bar{F}^{v}(V,\nabla V)  - S = 0 $$

where the conservative variables are given by 

$$U=\left\lbrace \rho, \rho\bar{v},\rho c_{p} T \right\rbrace ^\mathsf{T}$$

but the working variables within the solver are the primitives given by

$$V = \left \{  p, \bar{v}, T \right \}^\mathsf{T}$$ 

$$S$$ is a generic source term, and the convective and viscous fluxes are

$$\bar{F}^{c}(V) = \left\{\begin{array}{c} \rho \bar{v} \\ \rho \bar{v} \otimes \bar{v} + \bar{\bar{I}} p \\ \rho c_{p} \, T \bar{v} \end{array} \right\}$$

$$\bar{F}^{v}(V,\nabla V) = \left\{\begin{array}{c} \cdot \\ \bar{\bar{\tau}} \\  \kappa \nabla T \end{array} \right\} $$

where $$\rho$$ is the fluid density, $$\bar{v}=\left\lbrace u, v, w \right\rbrace^\mathsf{T}$$ $$\in$$ $$\mathbb{R}^3$$ is the flow speed in Cartesian system of reference, $$p$$ is the pressure, $$\bar{\bar{\tau}}$$ is the viscous stress tensor, $$T$$ is the temperature, $$\kappa$$ is the thermal conductivity, and $$\mu$$ is the viscosity. The viscous stress tensor can be expressed in vector notation as

$$\bar{\bar{\tau}}= \mu \left ( \nabla \bar{v} + \nabla \bar{v}^{T} \right ) - \mu \frac{2}{3} \bar{\bar I} \left ( \nabla \cdot \bar{v} \right )$$

In the low-Mach form of the equations, the pressure is decomposed into thermodynamic and dynamic components. $$p$$ is interpreted as the dynamic pressure in the governing equations, and $$p_o$$ is the thermodynamic (operating) pressure, which is constant in space. The system is now closed with an equation of state for the density that is a function of temperature alone $$\rho  = \rho(T)$$. Assuming an ideal gas with a specific gas constant $$R$$, one can determine the density from $$\rho = \frac{p_o}{R T}$$. `TK:: In the case of constant density the energy equation is effectively decoupled from the continuity and momentum equation and can therefore the energy equation is optional.`

Conductivity can be a constant, or we assume a constant Prandtl number $$Pr$$ such that the conductivity varies with viscosity as $$\kappa = \mu c_p / Pr$$. For laminar flows, $$\mu$$ is simply the dynamic viscosity $$\mu_{d}$$, which can be constant or assumed to satisfy Sutherland's law as a function of temperature alone, and $$Pr$$ is the dynamic Prandtl number $$Pr_d$$. For turbulent flows, we solve the incompressible Reynolds-averaged Navier-Stokes (RANS) equations. In accord with the standard approach to turbulence modeling based upon the Boussinesq hypothesis, which states that the effect of turbulence can be represented as an increased viscosity, the viscosity is divided into dynamic and turbulent components, or  $$\mu_{d}$$ and $$\mu_{t}$$, respectively. Therefore, the effective viscosity in becomes

$$\mu =\mu_{d}+\mu_{t}$$

Similarly, the thermal conductivity in the energy equation becomes an effective thermal conductivity written as

$$\kappa =\frac{\mu_{d} \, c_p}{Pr_{d}}+\frac{\mu_{t} \, c_p}{Pr_{t}}$$

where we have introduced a turbulent Prandtl number $$Pr_t$$. The turbulent viscosity $$\mu_{t}$$ is obtained from a suitable turbulence model involving the mean flow state $$U$$ and a set of new variables for the turbulence. 

The governing equation set in the general form above is very flexible for handling a number of variations in the modeling assumptions, from constant density inviscid flows up to variable density turbulent flows with a two-way coupled energy equation and temperature-dependent transport coefficients. Natural convection and 2D axisymmetric problems can be treated in a straightforward manner with the addition of source terms.

Within the `INC_NAVIER_STOKES` and `INC_RANS` solvers, we discretize the equations in space using a finite volume method (FVM) with a standard edge-based data structure on a dual grid with vertex-based schemes. The convective and viscous fluxes are evaluated at the midpoint of an edge. We apply a density-based scheme that is a generalization of artificial compressibility in order to achieve pressure-velocity coupling and solve the incompressible equations in a fully coupled manner. `TK:: Especially here it would be awesome to link the different tutorials that you made! I guess cross-referencing between tutorials, theory and other sections is a task for itself...`

The interested reader is pointed towards a [this paper](https://economon.github.io/docs/AIAA-2018-3111.pdf) which contains more details.

---

# Incompressible Euler #

| Solver | Version | 
| --- | --- |
| `INC_EULER` | 7.0.0 |

SU2 solves the incompressible Euler equations as a simplification of the low-Mach formulation above in the absence of viscosity and thermal conductivity (no energy equation is required). The equations can be expressed in differential form as

$$ \mathcal{R}(V) = \frac{\partial V}{\partial t} + \nabla \cdot \bar{F}^{c}(V) - S = 0 $$

where the conservative variables are given by 

$$U=\left\lbrace \rho, \rho\bar{v} \right\rbrace ^\mathsf{T}$$

but the working variables within the solver are the primitives given by

$$V = \left \{  p, \bar{v} \right \}^\mathsf{T}$$ 

$$S$$ is a generic source term, and the convective flux is

$$\bar{F}^{c}(V) = \left\{\begin{array}{c} \rho \bar{v} \\ \rho \bar{v} \otimes \bar{v} + \bar{\bar{I}} p \end{array} \right\}$$

where $$\rho$$ is a fluid density (constant), $$\bar{v}=\left\lbrace u, v, w \right\rbrace^\mathsf{T}$$ $$\in$$ $$\mathbb{R}^3$$ is the flow speed in Cartesian system of reference, and $$p$$ is the dynamic pressure.

Within the `INC_EULER` solver, we discretize the equations in space using a finite volume method (FVM) with a standard edge-based data structure on a dual grid with vertex-based schemes. The convective and viscous fluxes are evaluated at the midpoint of an edge. We apply a density-based scheme that is a generalization of artificial compressibility in order to achieve pressure-velocity coupling and solve the incompressible equations in a fully coupled manner.

---

# Turbulence Modeling #

| Solver | Version | 
| --- | --- |
| `RANS`, `INC_RANS` | 7.0.0 |

The Shear Stress Transport (SST) model of Menter and the Spalart-Allmaras (S-A) model are two of the most common and widely used turbulence models. The S-A and SST standard models, along with several variants, are implemented in SU2. The reader is referred to the [NASA Turbulence Modeling Resource](https://turbmodels.larc.nasa.gov/index.html) (TMR) for the details of each specific model, as the versions in SU2 are implemented according to the well-described formulations found there.

Within the turbulence solvers, we discretize the equations in space using a finite volume method (FVM) with a standard edge-based data structure on a dual grid with vertex-based schemes. The convective and viscous fluxes are evaluated at the midpoint of an edge.

---

# Elasticity #

| Solver | Version | 
| --- | --- |
| `ELASTICITY` | 7.0.0 |

For structural analysis of solids in SU2, we solve the elasticity equations in a form allowing for geometric non-linearities expressed as

$$ \rho_s \frac{\partial^2 \mathbf{u}}{\partial t^2} = \nabla (\mathbf{F} \cdot \mathbf{S}) + \rho_s \mathbf{f} $$

where $$\rho_s$$ is the structural density, $$\mathbf{u}$$ are the displacements of the solid, $$\mathbf{F}$$ is the material deformation gradient, $$\mathbf{S}$$ is the second Piola-Kirchhoff stress tensor, and $$\mathbf{f}$$ is the volume forces on the structural domain. 

In the `ELASTICITY` solver, we discretize the equations in space with a nodal finite element method (FEM).

---

# Heat Conduction #

| Solver | Version | 
| --- | --- |
| `HEAT_EQUATION_FVM` | 7.0.0 |

The governing equation for heat conduction through a solid material can be expressed in differential form as the following:

$$ R(U) = \frac{\partial U}{\partial t} - \nabla \cdot \bar{F}^{v}(U,\nabla U) - S = 0 $$

where the conservative variable is $$U=\left\lbrace \rho_s c_{p_s} T\right\rbrace$$, $$\rho_s$$ is the solid density, $$c_{p_s}$$ is the specific heat of the solid, and $$T$$ is the temperature. The viscous flux can be written as 

$$ \bar{F}^{v}(U,\nabla U) = \kappa_s \nabla T $$

where $$\kappa_s$$ is the thermal conductivity of the solid. The material properties of the solid are considered constant.

Within the `HEAT_EQUATION_FVM` solver, we discretize the equations in space using a finite volume method (FVM) with a standard edge-based data structure on a dual grid with vertex-based schemes. The viscous flux is evaluated at the midpoint of an edge.


`TK:: Alright, the dual time source term for unsteady simulations is currently missing. All that LES/DES stuff from Eduardo. And prob a bit more. As mentioned already cross-referencing to meaningful tutorials and linking papers for further reading would be really cool and helpful -> "Hey this is feature X and here is more detailed information in a paper and there is a tutorial which contains this feature." 
But all in all it is a pretty good start. In an ideal world one would have a seperate numerics section in the future that explains (or links to meaningful pages) further the FVM approach followed by SU2 and all the other good stuff ROE-scheme, gradient computation, boundary implementations etc.`