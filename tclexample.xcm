# This script gives an example of how one might use the power of tcl's
# scripting language in an XSPEC session.  In this example, XSPEC loops
# thru 3 data files (file1, file2 and file3) and fits them each to the
# same model `wabs(po+ga)'.  After the fit the values of 4 parameters
# for each data set is saved to a file.

# Return TCL results for XSPEC commands.
set xs_return_result 1

# Keep going until fit converges.
query yes

# Open the file to put the results in.
set fileid [open fit_result.dat w]

for {set i 1} {$i < 4} {incr i} {

# Get the file.
  data file$i

# Set up the model.
  mo wabs(po+ga)
  /*

# Fit it to the model.
  fit

log tmp.log
show param 
log none
set f [open tmp.log r]
foreach j { 1 2 3 4} {
set reg "^ *$j.* (\[0-9.eE+-\]+) *\$" 
seek $f 0 
while {! [regexp $reg [gets $f] p dp$j]} {}   
}
close $f 

# Print out the result to the file. This writes the current version
# of chi-squared and the values of parameters 1, 2, and 4.
# puts $fileid "$i [new 0] [show param 1] [show param 2] [show param 4]"

  puts $fileid  "$dp1 $dp2 $dp3" 

# Reset the model.
  model none
}

# Close the file.
close $fileid


