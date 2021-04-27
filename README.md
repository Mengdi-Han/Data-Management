# Data-Management
Using Rstudio to convert semi-structure data

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r}

# install.packages("readxl")
# install.packages("writexl")
library(tidyverse)
library(readxl)
library(writexl)

```

# 1.reading file paths
```{r}
# read paths
getwd()
all.files <- list.files("table/")     
dir <- paste("./table/",all.files,sep="")    
n <- length(dir)     

# new dataframe for storing 
new.table <- data.frame()
onetable <-data.frame()

```
# 2.read excels
```{r}
#loop countries for reading
for (i in 1:n)   
{
 (files <- list.files(dir[i]))
 (subdir <- paste(dir[i],"/",files,sep=""))   
 m <- length(subdir)       
 
 #loop specific country's excels for reading
 for (j in 1:m) {        
  
   (tempo1 <- subdir[j])
   
   #read data of "country" and "flow"
   try({
      tempo2 <-  read_xlsx(tempo1, skip =1, n_max = 3)
      tempo2
      tempo2 <- tempo2[,-2]  
   }, silent=TRUE)
 
   
   #read data of "product" and "year"
    try({
      data.p.y <- read_xlsx(tempo1, skip =5) 
      data.p.y <- data.p.y[-1,-2]    
      data.p.y <- data.p.y[1:(nrow(data.p.y)-3), ]
      data.p.y <- gather(data.p.y,product,value,-Product)
      colnames(data.p.y)[1]="year"
      
      #organize data.p.y and data.c.f to onetable
      onetable <- data.p.y 
      onetable <- add_column(onetable, flow = tempo2[1,2], .after = "year")
      onetable <- add_column(onetable, country = tempo2[3,2], .before = "year")
      } ,silent=TRUE)
   
   #store data
   new.table <- rbind(new.table, onetable)
    
 }
  
}
 
#clear duplicated data
new.table <- distinct(new.table)

```

# 3. total record number is 573950; each product is of 8830 items
```{r}
#summarise record number

(total.number <- nrow(new.table))
new.table$product <- as.factor(new.table$product)
new.table <-group_by(new.table,product)
summarise(new.table, n()) 

```
