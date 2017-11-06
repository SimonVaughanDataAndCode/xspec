# xspec
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
which first used the package, and for which these scripts were originally developed.

[Hurkett, C., Vaughan S., et al. 2008, ApJ, v679, p587](http://adsabs.harvard.edu/abs/2008ApJ...679..587H)

## Explanation

### Monte Carlo methods

Monte Carlo methods use random (or quasi-random) data to solve
problems. In the context of hypothesis testing, one 
generates randomised data based on the null hypothesis model,
taking care to make the fake data as realistic as possible, 
and uses them as a 'control' sample with which to calibrate the
test statistic. In the source detection example one would simulate
a large ensemble of images, by randomising the background level,
and measure the flux at the source position in each one. 
The frequency distribution (histogram) of the fluxes produced from
the fake data 
is a Monte Carlo estimate of the reference PDF.
As well as making sure the data are simulated as accurately as
possible one must also simulate a large number of datasets 
so that the histogram of the test statistics from the simulations
converges on the true PDF. Even if there is 
no analytical expression for the reference distribution, it is
always possible to find it using the Monte Carlo method so long
as one can simulate a sufficient number of (realistic) fake data.

Monte Carlo methods are extremely powerful and conceptually simple.
The drawback is that they may require a large amount of computer
processing time to generate and analyse a large quantity of simulated
data. 

### Application to the F-test

Maybe you are trying to test for an emission line in a spectrum;
adding the line to the model improves the fit a bit, but you don't
know whether the improvement should be considered significant or
not. 
You could use the F-test but one of the
assumptions behind the F-test is not valid in this case [1]. Normally you
would measure the F-statistic and compare this with 
a reference distribution
-- this tells you how unexpected your value of
F is. (In this case the reference distribution for the F-test is the
Fisher-Snedecor distribution.) 
The reference distribution gives you a probability, or p-value, for
the given value of F. 
If the probability is small (let's say
p=0.001) then you conclude this result is unlikely to have occurred by
chance, so it must be a significant detection (people often quote this
by inverting the false alarm probability: 100*[1-p] = 99.9 per
cent confidence). But, as we just said, the case of adding a line 
violates one of the fundamental assumptions behind the F-test and so
you cannot use the textbook reference 
distribution to go from an F-value (what you measure from the data) to
a p-value (how significant it is). But we can solve the problem
using a Monte Carlo approach.

### An overview of the method

The general idea is as follows. First you need to define exactly what
it is you want to test. If you want a clear answer you need a clear
question! In the case of line detection, perhaps you are comparing
a power law to a power law plus an emission line. 
The null
hypothesis is that the simpler of these two models is true --
in this example the
null hypothesis is that the spectrum is just a power law. The
alternative hypothesis is that the spectrum is a power law plus an
emission line. The way you go about making a hypothesis test is to
measure some test statistic from the data. Maybe you used the F-test
and measured an F-value. (The F-value comes from the decrease in
chi-square when you add a line to the model, and the number of
degrees of freedom.) But you don't know the reference distribution to
turn this into a probability. What you need to do is make a large
batch of fake data for which the null hypothesis is true, and measure
the same test statistic for each of the fake datasets. So you make a
fake dataset, measure the test statistic, make another one, measure
the test statistic, etc. etc. If you keep doing this you will build up
the distribution of the test statistic assuming the null hypothesis is
true (because all your fake data are produced using the null
hypothesis). In the 'power law vs. power law plus line' example what
we do is simulate a spectrum of a power law, then fit the data using a
power law with and without a line, and then measure the F-value. Over
and over again. As you perform more simulations you build up a clearer
picture of the distribution of F-values (if the null hypothesis is
true). 

You can then
ask the question: how many simulations show a larger test
statistic (e.g. F-value) than the one I got for my real data? Did
the value I got 
for my test statistic appear in many of the simulations or only
very rarely? Maybe the F-value was 6.71 from the real data. And when
we ran the 1,000 simulations we found only 3 out of the 1,000 had a
value bigger than this.
We could conclude there is a 3/1000 chance of getting an F-value like
the one observed if the null hypothesis is true. 
This is the false alarm probability, and since it is quite small 
we may interpret it as indicating the
null hypothesis is false, and we therefore favour the
alternative hypothesis. 
In other words we could say the line is
detected at 99.7 per cent confidence (because 997/1000 simulations
showed a smaller F-value). 

### Outline of a simple MC method

A simple Monte Carlo significance test works along the following lines:

1. Define the null and alternative hypotheses 

2. Choose a test statistic: call it T

3. Measure the test statistic of the real data: call it T_0

4. Definite loop. For each i=1,2,...,N:

   a. Produce a simulated data set: D_i
   
   b. Measure the test statistic from the simulated data: T_i

5. Calculate where T_0 falls in the distribution of T_i

The p-value is fraction of the T_i values that exceed the measured
T_0 value: p = n'[ T_i >= T_0 ]'/N. Inverting this the significance
is 1-p = n'[ T_0 > T_i ]'/N. Ensure
N is large or this will not be a very accurate representation (the
error on the p value is sqrt'{p(1-p)/N}', which comes from the
binomial formula).

It is vital that you make a *fair* measurement of the
test statistic from the simulated data -- you must be careful not to
bias this measurement 
based on your prior experience of the real data. 
What ever you did to the real data, you must also do to the 
simulated ('control') data, otherwise you are not performing 
a fair like-for-like test.

### Script files for XSPEC

That's the theory dealt with. Now for a worked example of
running a Monte Carlo test using XSPEC.

XSPEC will allow you to run a sequence of commands from a script. This
means you can automate the process of generating and fitting a
large sequence of fake data. If you are trying to examine 1,000 spectral
simulations this really is the only way to do it. Basically all you do
is write the appropriate commands into an XSPEC script file '\*.xcm'
and then you can run it from within XSPEC. XSPEC uses
TCL (Tool Command Language) to control its operation, so you can also
add basic 
control structures (like loops) and input/output commands to your
script. In this way an XSPEC script with a few TCL commands is
quite a powerful tool.

A slight technical problem is that you may need more than just
XSPEC. If your real data were grouped have have 20 counts per bin,
then you must do this to your fake data too. But this needs
to be done by GRPPHA -- outside of XSPEC. What I do is break down the
process into a set of basic tasks, each of which has its own script. 

1. I use one
XSPEC script to produce N fake datasets. 
2. Then I use a shell script
(from the UNIX command line, outside of XSPEC) to run GRPPHA on each
of the fake data files. 
3. Then I have another XSPEC script to load each
of the (now grouped!) fake data files in turn, fit them, and save the
results to a text file. 
4. Then I use an R script, or a Fortran program
(or whatever) to examine the results and see how the simulated
distribution of the test statistic compares to the 'real' number.  

There are a number of ways to run an XSPEC script like this. One is
to simply use the '@' symbol, like you would a normal XSPEC '\*.xcm' file.
```
xspec>  @script.xcm 
```
A better way is to use the following from the UNIX command line
```
unix>  xspec - script.xcm 
```
This will start XSPEC and run the script. If you end the script 
with an *exit* command it will then return you to the UNIX command line.



[1]: There are two conditions that must be satisfied for the F-test to
follow its expected theoretical reference distribution. These are that
the two models being compared are *nested*, and that the null
values of the additional parameters are not on the boundary of
possible parameter space. This second condition is violated when
testing for a line (or any other additive component) because the null
value of one of the new parameters (normalisation) is zero, which is
the boundary of the parameter space. 
You should also have enough counts per bin to be able to use chi-square
properly as well. (Or use direct maximum likelihood fitting.)
See [Protassov et al. (2002)](http://adsabs.harvard.edu/abs/2002ApJ...571..545P).
