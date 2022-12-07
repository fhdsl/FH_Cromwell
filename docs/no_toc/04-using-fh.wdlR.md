

# Using the `fh.wdlR` Package

Chapter 3 showed you how to use the Hutch Shiny app to submit workflows to running Cromwell servers, and to monitor their progress. The Shiny app is built using an R package (fh.wdlR) available via GitHub.  

https://github.com/FredHutch/fh.wdlR

You can also use this R package through R/RStudio on your local machine (on VPN or on campus) to directly submit workflows to your Cromwell server from the R command line, and to track calls and workflow execution status directly. 


## Install `fh.wdlR` from GitHub
You will need the following packages installed first:

```r
install.packages(pkgs = c("httr", "jsonlite", "magrittr",
                "dplyr", "ssh", "purrr", "paws", "tidyr"))

# Not required for the package but certainly handy and used in our demo here:
install.packages("tidyverse")
```

Then you can install the most recent version of `fh.wdlR` by:

```r
require(remotes)
remotes::install_github('FredHutch/fh.wdlR')
```

Install a specific release version (in this case v2.0.2) by:
```r
require(remotes)
remotes::install_github('FredHutch/fh.wdlR@v2.0.2')
```


## Example workflow process


```r
## Load packages
library(fh.wdlR); library(tidyverse);
```


Tell your R session how to find your Cromwell server (note you'll need to be on campus or on VPN).

```r
## Set your Cromwell URL
setCromwellURL(nodeAndPort = "gizmoXXX:20202")
```

### Validate your workflow formatting

```r
list.files(pattern = "*.wdl")
valid <- cromwellValidate(WDL = "myworkflow.wdl"); valid[["errors"]]
```

Go fix your issues (if there are any), now send your workflow to Cromwell.

### Submit Workflows

```r
thisJob <- cromwellSubmitBatch(WDL = "myworkflow.wdl",
                    Params = "myworkflow-parameters.json",
                    Batch = "myworkflow-batch.json",
                    Options = "workflow-options.json")

# thisJob$id is now the unique Cromwell ID for your entire workflow - you can use that to request all sorts of metadata!!!
thisOne<- thisJob$id; thisOne
```
Now get all your metadata and track the workflow!!

### Track Workflows

```r
# Returns a data frame of all jobs run in the past number of days (uses your database)
jobs <- cromwellJobs(days = 2)

# Returns a data frame (one line if you only submit one workflow id) containing workflow level metadata
w <- cromwellWorkflow(thisOne)

# This is handy to print the current status of the workflow(s) is(are)
w$status

# Returns a data frame containing all call level metadata
c <- cromwellCall(thisOne)

# Handy set of dplyr commands to tell you about how the various calls are doing
c %>% group_by(callName, executionStatus) %>% summarize(status = n()) %>% arrange(executionStatus)

# Returns a data frame containing call level call caching  metadata
ca <- cromwellCache(thisOne)

# Handy set of dplyr commands to tell you about what sort of call caching is happening
ca %>% group_by(callCaching.hit, callName) %>% summarize(hits = n())

# Opens up a popup in your browser with a timing diagram in it.
cromwellTiming(thisOne)

# Returns a data frame containing call level failure metadata
f <- cromwellFailures(thisOne)

# Will tell Cromwell to abort the current workflow - note this cannot be undone and it will take a while to stop all the jobs.  
abort <- cromwellAbort(thisOne)

# When a workflow is done, request information about the workflow outputs.
out <- cromwellOutputs(thisOne)
```

## Look Under the Hood

When all else fails, pick through the ugly metadata yourself to see what's happening. 

```r
# Ugly list of raw metadata should you need it for workflow troubleshooting
WTF <- cromwellGlob(thisOne); WTF[["failures"]]
```
