[![DOI](https://zenodo.org/badge/353152239.svg)](https://zenodo.org/badge/latestdoi/353152239) ![license](https://img.shields.io/github/license/andr1976/HydDown) ![buil](https://github.com/andr1976/HydDown/actions/workflows/python-app.yml/badge.svg) [![codecov](https://codecov.io/gh/andr1976/HydDown/branch/main/graph/badge.svg)](https://codecov.io/gh/andr1976/HydDown) [![Streamlit App](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://share.streamlit.io/andr1976/hyddown/main/scripts/streamlit_app.py)
[![CodeQL](https://github.com/andr1976/HydDown/actions/workflows/codeql-analysis.yml/badge.svg)](https://github.com/andr1976/HydDown/actions/workflows/codeql-analysis.yml)
[![status](https://joss.theoj.org/papers/0eed2a25a99589ed8dcdc785c890fb25/status.svg)](https://joss.theoj.org/papers/0eed2a25a99589ed8dcdc785c890fb25)
 
# HydDown
Hydrogen (or other pure gas phase species) depressurization calculations 

This code is published under an MIT license.

Install as simple as:

    pip install hyddown
    
In case `pip` links to a v2.7 of python you will get an error. If so try the following:

    python3 -m pip install hyddown

where python3 is the symlink or full path to the python3 executable installed on your system. 

Run the code as simple as: 

    python main.py input.yml

where main.py is the main script and input.yml is the input file in Yaml syntax. 

Consult the [manual](https://github.com/andr1976/HydDown/raw/main/docs/MANUAL.pdf) for a more rigorous explanation of the software, the implemented methods, and its usage. Further, the manual also contains a few validation studies. 

## Demonstration 
The easiest way to explore the capability of HydDown is the [streamlit app](https://share.streamlit.io/andr1976/hyddown/main/scripts/streamlit_app.py). This version allows calculation of:

- filling of vessel with gas (pressurisation)
- discharge of gas (depressurisation)
- various gases (H2, N2, CH4, He, Air)
- variable size pressure cylinder/vessel
- heat transfer between gas and vessel wall can be turned on/off

## Background
This is a small spare time project for calculation of vessel filling/depressurisation behaviour. This is mainly to demonstrate, that although perceived as a very tedious/difficult/complex problem to solve, actually a fairly limited amount of code is necessary if you have a good thermodynamic backend. 

A few choices is made to keep things simple to begin with:

- [Coolprop](http://www.coolprop.org/) is used as thermodynamic backend
- Mainly pure substances are considered (mixtures can be handled - but calculations can be slow)
- Gas phase only 
- No temperture stratification in the gas phase
- No temperture gradient through vessel wall

The code is as simple as possible. The above choices makes the problem a lot more simple to solve, First of all the pure substance Helmholtz energy based equation of state (HEOS) in coolprop offers a lot of convenience in terms of the property pairs/state variables that can be set independently. Using only a single gas phase species also means that component balances is redundant and 2 or 3-phase flash calculations are not required. That being said the principle used for a single component is more or less the same, even for multicomponent mixtures with potentially more than one phase.

## Description
The following methods are implemented:

- Isothermal i.e. constant temperature of the fluid during depressurisation (for a very slow process with a large heat reservoir)
- Isenthalpic/Adiabatic (no heat transfer with surroundings, no work performed by the expanding fluid)
- Isentropic (no heat transfer with surroundings, PV work performed by the expanding fluid)
- Constant internal energy
- Energy balance. This is the most general case and includes the ability to transfer heat with surroundings

Various mass flow equations are enabled: 

- Orifice 
- Control valve 
- Relief valve (discharge only)
- Constant mass flow

A simple (naive) explicit Euler scheme is implemented to integrate the mass balance over time, with the mass rate being calculated from an orifice/valve equation. For each step, the mass relief/ left in the vessel is known. Since the volume is fixed the mass density is directly given. For the simple methods (isentropic,isenthalpic,isenergetic etc), Coolprop allows specifying density and either H,S or U directly - this is very handy and normally only TP, PH, TS property pairs are implemented, and you would need to code a second loop to make it into am UV, VH or SV calculation. Coolprop is very convenient for this, however for a cubic EOS and for multicomponent Helmholtz energy EOS coolprop only supports a subset of state variables to be specified directly (T,P,quality). For this reason single component HEOS is the main target of this small project.  In case the "Energy balance" method is applied, the heat added from convection and work is accounted for. 

## Basic usage
The Yaml input file is edited to reflect the system of interest. For isothermal/isenthalpic/isentropic/isenergetic calculations the minimal input required are:

- Initial conditions (pressure, temperature)
- vessel dimensions (ID/length)
- valve parameters (Cd, diameter, backpressure)
- Calculation setup (time step, end time)
- Type of gas

If heat transfer is to be considered the calculation type "energybalance" is required. A few options are possible:

- Fixed U (U-value required, and ambient temperature)
- Fixed Q (Q to be applied to the fluid is requried)
- Specified h, the external heat transfer coefficient is provided and either the internal is provided or calculated from assumption of natural convection from a vertical cylinder at high Gr number. Ambient temperature is required.
- Detailed 
- Fire with heat load calculated from the Stefan-Boltzmann equation
