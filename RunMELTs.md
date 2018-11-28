Modeling Off-Axis Seamounts; Expediting insight into near-ridge mantle heterogeneity
================

Introduction and Problem Statement
----------------------------------

In an age where seeing the unseen is commonplace, scientists still have surprisingly few constraints on mantle dynamics beneath our feet. Studies performed at mid-ocean ridges attempt to address this gap in knowledge by examining chemistry of basalts produced at ridge axes where distance to the mantle is minimal (often less than 1 km). Ideally, those basalts are as close to representing the mantle as a geochemist can access. However, at fast-spreading ridges a continuous thermal barrier allows for efficient mixing of mantle material prior to eruption, so materials collected on-axis (the most common site for sampling) only represent an average composition. To probe any heterogeneities in the mantle, ~300 basalt samples were collected from a long seamount chain perpendicular to the fast-spreading East Pacific Rise in an attempt to by-pass homogenization mixing and see into the mantle.

To assess heterogeneity, fractional crystallization, source mixing, and mantle melting models are compared to the 300 basalt samples. However, the stages from inputing parameters into the melting program (alphaMELTs) to producing visualizations of the results are arduous and time-consuming. This results in week-long modeling sessions that are frequently inadequate and need adjusted and re-run. To expedite the melt model results, this script will incorporate the computing power of R to:

a)run alphaMELTs given chosen parameters

b)read the alphaMELTs output results

c)process the data

d)feed it into a script which plots the results in real-time without all of the intermediate, time-consuming steps inherent to running alphaMELTs manually.

This script will effectively reduce the time required to visualize the results (now &lt;10 minutes to run and visualize results) so users can immediately decide which parameters to adjust, quickly adjust those, and automate getting updated results.

Methods and Datasets
--------------------

###### Install R Markdown

``` r
#install.packages("rmarkdown") (Commented out to not re-install each time)
#This script is published using a template from installation of R Markdown : https://rmarkdown.rstudio.com/index.html
```

###### Upload seamount dataset csv as a table with a header

``` r
Elements <- read.table(file = "C:/Users/Molly/Documents/EPR/Code/MajorTrace.csv", sep = ',', header = T)
#way too large a table to print, but syntax is "print(Elements)"
```

###### Adjust parameters for the model

-   Environment file

``` r
file.edit(file = "C:/Users/Molly/Documents/windows_alphamelts_1-8/windows_alphamelts_1-8/isentropic_melt_env.txt", title = "Environment")
# edit parameters accepted by the program such as min and max pressures and temperatures, increments of melting, type of melting which for this environment is isentropic (decompression melting). Ensure output is un-commented
```

-   Batch file

``` r
file.edit(file = "C:/Users/Molly/Documents/bin/batch.txt", title = "Batch File")
#edit the in-program prompts such as which starting mantle composition to use, specific temperatures for different stages of melt calculations, change water content, starting pressure and temperature, and name the output files
```

-   Pause script until prompted (after editing the files)

``` r
readline(prompt = "Press [Enter] to continue")
```

    ## Press [Enter] to continue

    ## [1] ""

``` r
#this pause provides the user an opportunity to change the environment and batch controls before executing the program. Must press enter to continue and the program will say that 
```

###### Execute MELTs

``` r
#This is what the test.bat file looks like and what it is doing: Execute and apply flags 
#C:\Users\Molly\Documents\bin\run_alphamelts.command ## execute the alphaMELTs program ##
#-f C:\Users\Molly\Documents\windows_alphamelts_1-8\windows_alphamelts_1-8\isentropic_melt_env.txt  ## environment file ##
#-b C:\Users\Molly\Documents\bin\batch.txt ## automate running the full program using batch.txt ##
#-p C:\Users\Molly\Documents\bin ## set location for output (-p)
shell.exec("C:/Users/Molly/Documents/bin/test.bat") #use the R shell to execute external program through R
print('done') #output text lets user know MELTs is done with its calculations and has successfully output files
```

    ## [1] "done"

Results
-------

###### Upload results from MELTs and format

MELTs output is a space-delimited text file (unseparated into columns) which needs to be uploaded as a table

``` r
output <- read.table("C:/Users/Molly/Documents/bin/models.traceint", header = TRUE, skip = 7) #skip 7 pre-header lines, keep actual header (row 8)
#print(output) uncomment this to check that the model output seems reasonable
```

###### Calculate important trace element ratios

Create new columns for the model results so the ratios can be plotted. For my purposes those are La/SmN and Nb/La since La/SmN changes dramatically with melting whereas Nb/La is primarily controlled by source variability

``` r
output$LaN <- output[,"La_1D"]/0.687 #Normalize the La values to primitive mantle and put in new column
output$SmN <- output[,"Sm_1D"]/0.444 #Normalize the Sm values to primitive mantle and put in new column
output$LaSmN <- output[,"LaN"]/output[,"SmN"] #calculate ratio of normalized values for normalized ratio and put in new column
output$NbLa <- output[,"Nb_1D"]/output[,"La_1D"] #calculate ratio of Nb and La and make new column for it
```

###### Plot models

Plot the model data against the seamount data in La/SmN and Nb/La space. La/SmN is a proxy for varying extents of melting in the mantle whereas Nb/La is a proxy for source variability without the effects of melting. Plotting the models with data provides a constraint on how much melting (extent of run) of which source (each run) contributes to the compositions on the seamounts. If the models do not model the data well, adjusting parameters (above) and re-running can show how parameters shift the model.

``` r
plot(output$LaSmN, output$NbLa, col = 'black', pch = 20, xlab = "La/SmN", ylab = "Nb/La", xlim = c(0.5,4), ylim = c(0,3)) #create a plot of the model points, set axis limits and labels
legend("topright", col = c('black', 'blue'), pch = c(20,17), legend = c('Increments of Melting', 'Seamount Data')) #add a legend to the plot
lines(output$LaSmN, output$NbLa, col = 'black', lty = 1) #make a line out of the melting data to show continuity
points(Elements$La.SmN, Elements$Nb.La, col = 'blue', pch = 17) #plot the seamount data points for comparison
```

![](RunMELTs_files/figure-markdown_github/plot-1.png) \#\# Next Steps \#\#\#\#\#\#Write several batch files to loop through and run the program multiple times in a row. Then loop through the outputs and plot them all together \#\#\#\#\#\#Use outputs to inform which parameters work best, and adjust accordingly and run again \#\#\#\#\#\#Use the models to constrain extents of melting and source contribution contributing to the seamount heterogeneity by plotting various trace element ratios of the models
