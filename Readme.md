Simplified Metadata
================
Michelle M. Fink, using code from Chris Tracey & Tim Howard

#### Last revised: 2019-04-23

#### Built on R 3.4.4

### 1\. Required to run the code:

``` r
library(knitr)
library(sf)
library(raster)
library(xtable)
library(tinytex)
library(stringr)
library(tables)
library(readxl)
```

(Not sure all of these ^ packages are still used with the modified
code.)

  - SimplifiedMetadata\_knitr.rnw
  - metadata\_inputs.xlsx
  - MikTex installed, see
<http://miktex.org/>

### 2\. The spreadsheet contains metadata info that we want to knit into an abbreviated version of the MoBI metadata

Spreadsheet contains 1 tab with:

  - Sciname  
  - CommName
  - Code1 (species code)
  - Code2 (Element ID)  
  - Grank (rounded)
  - Inputs  
  - InputSource  
  - Filename  
  - ContactName  
  - ContactOrg  
  - ContactEmail  
  - Notes (basic methods)  
  - Reviewed  
  - Final

### 3\. Set the variables:

(you will want to change these)

``` r
working.dir <- "C:/Michelle/Metadata/test"
baselayers.dir <- "D:/GIS/Base/USA"
models.dir <- "M:/GIS_Projects/MOBI"
xls <- "metadata_inputs.xlsx"

ranks <- list("Critically Imperiled", "Imperiled", "Vulnerable", "Apparently Secure", "Secure")
names(ranks) <- c("G1", "G2", "G3", "G4", "G5")
bibtable <- read.delim(paste(working.dir,"bibtable.tab", sep = "/"), stringsAsFactors=FALSE)
states <- paste(baselayers.dir, "STATES.shp", sep = "/")
counties <- paste(baselayers.dir, "COUNTIES.shp", sep = "/")
sfstates <- st_read(states, quiet = T)
sfcounties <- st_read(counties, quiet = T)
```

### 4\. Read each record in the spreadsheet and pass to knitr

``` r
setwd(working.dir)
in.meta <- read_xlsx(xls, col_types = "text")

for (i in 1:nrow(in.meta)){
  sdm.sciname <- in.meta$SciName[i]
  sdm.comname <- in.meta$CommName[i]
  results_shape <- raster(paste(models.dir, in.meta$Filename[i], sep = "/"))
  rasext <- extent(results_shape)
  x1 <- rasext[1] - (rasext[1] * 0.01)
  x2 <- rasext[2] + (rasext[2] * 0.01)
  y1 <- rasext[3] - (rasext[3] * 0.01)
  y2 <- rasext[4] + (rasext[4] * 0.01)
  NSurl <- paste("http://explorer.natureserve.org/servlet/NatureServe?searchName=",gsub(" ", "+", sdm.sciname, fixed=TRUE), sep="")
  n <- nchar(in.meta$Filename[i])
  model_run_name <- substr(in.meta$Filename[i], 1, n-4)
  sdm.code1 <- in.meta$Code1[i]
  sdm.code2 <- in.meta$Code2[i]
  sdm.grank <- in.meta$Grank[i]
  sdm.modeler <- paste(in.meta$ContactName[i], in.meta$ContactEmail[i], in.meta$ContactOrg[i], sep = ", ")
  sdm.program <- in.meta$ContactOrg[i]
  sdm.inputs <- in.meta$Inputs[i]
  sdm.source <- in.meta$InputSource[i]
  sdm.notes <- in.meta$Notes[i]
  sdm.review <- in.meta$Reviewed[i]
  sdm.final <- in.meta$Final[i]
  knit(paste(working.dir,"SimplifiedMetadata_knitr.rnw",sep="/"), output=paste(model_run_name, ".tex",sep=""))
  call <- paste0("pdflatex -interaction=nonstopmode ", model_run_name, ".tex")
  system(call)
  system(call) # 2nd run to apply citation numbers
}
```

**Note:** the maps need a lot of work.
