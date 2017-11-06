# xspec
xspec scripts

This page gives XSPEC (and shell) scripts for performing Monte Carlo tests within XSPEC. 

* source.pha - The "real" dataset for the worked example
* xmmpn.rmf - The response matrix for the data
* xmmpn.arf - The ancillary response file for the data
* sim-rand.xcm - XSPEC script to produce simulated spectra
* groupall - Bash script to run GRPPHA over all simulated spectra
* fit.xcm - XSPEC script to fit all (grouped) simulated spectra

Read the [mc.pdf](mc.pdf) file for my (circa 2006) explanation.

## Warning

These scripts were written 2005-2006, tested using XSPEC v12.2.1ao running under Scientific Linux 4.2. They have not been maintained since then. I offer no guarantee they will work on other systems.

## Referencing the scripts

If you make use of this code in your work, please do cite the following paper,
which first used the package, and for which sour was originally developed.

