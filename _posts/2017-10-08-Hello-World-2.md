---
title: "Crime and elevated trains"
author: "Christopher Eshleman"
date: "Oct 8, 2017"
output: html_document
subtitle: First steps - snagging some data
---

##Taking advantage of New York City's commitment to public accessibility re: data

Is crime beneath/immediately surrounding elevated trains different from (worse than) crime elsewhere? I had a tour guide (Robert Brenner, fantastic guy) yesterday tell us it was so. He was talking about the old Allen Street elevated train and the fact that the street beneath the track was a hotbed for Jewish prostitution. Apparently this was an issue in New York City at one point. So I’ll do this full analysis later and, for now, I’ll just focus on getting one of the datasets I need. 


--- 


We dive into a slice of 311 data specific to street tree damage reported immediately following major storms. The City of New York makes 311 data available on its [Open Data Portal](https://nycopendata.socrata.com/), and we will investigate it using the software $R$. We will wrangle the data with spatial packages like $sp$, $rgdal$ and $maptools$. Then, we will plot the data using shapefiles from the City's planning department and $R$’s famous $ggplot2$ package. Finally, we will combine 311 with data from the US Census Bureau and the City's tree census to construct a statistical model using the package $RStan$.

Identifying which residents report storm damage has a variety of applications. For example, we could spread awareness of the 311 system to neighborhoods where, for whatever reason, residents report fewer inceidences. Or, we can better understand spikes of 311 activity and create policies that more fairly distribute resources after a major storm. For this project, however, our choice of storms is of purely academic interest. We choose to look at tree damage because storms are well-defined, observable events. After each storm, we can safely assume that the tree damage being reported by a 311 user is damage generated from that storm. Since we can safely assume that service requests coming right after a storm will be due only to the storm, we can take advantage of spatial and temporal variations in the call rate to identify who uses 311.  

However, the natural experiment provided to us by storms is a luxury not available to users of 311 data in general. In most cases, 311 use rates partially reflect agency specific strategies for addressing service requests. And these strategies can greatly complicate the interpretation of the data. For example, 311 rat sightings are heavily determined by neighborhood specific baiting campaigns by the Department of Health and Mental Hygiene which are themselves set according to reported 311 rat sightings. In situations such as these, it is not possible to adjust for the spatial and temporal changes due to agency policy.

This project began as a collaboration between Columbia University's School of International and Public Affairs and the New York City Mayor's Office and Department of Parks and Recreation. For more information on the data, partnership or preliminary findings, see Eshleman et al, 2013 [1].

##Getting Started
###Exploring 311 - where do requests come from?
The 311 data used here can be found on New York City’s  [Open Data Portal](https://nycopendata.socrata.com/). We've already downloaded the data into a directory called "data". The goal of this section is to explore 311 requests and link it to census tract-level socio-demographic and tree data, which will help us understand who is (and isn’t) asking the City for help following major storms. We start by plotting the data over time. First, set up a working drive and load packages. If you have never used these packages, they will need to be "installed" first.

```{r setup, echo=FALSE}
setwd(paste("/Users/sigmamonstr/Github/tutorial_311_trees_p1",sep=""))
```

First, let's check to see if we have all the right packages to do this work. We'll first borrow the pkgTest script introduced in the Zillow tutorial, then loop through through a vector of relevant packages to check if a given package is present or needs to be installed:

- General Wrangling: *plyr*, *reshape2*
- Spatial Wrangling: *rgeos*, *rgdal*, *maptools*
- Plotting: *ggplot2*, *ggmap*

```{r, message=FALSE, warning=FALSE}

## This function will check if a package is installed, and if not, install it
  pkgTest <- function(x) {
      if (!require(x, character.only = TRUE)) {
          print(paste("installing",x))
          install.packages(x, dep = TRUE)
          if (!require(x, character.only = TRUE)) 
              stop("Package not found")
      } else{
        print(paste(x,"found"))
      }
  }

## These lines load the required packages
  packages <- c("rgeos", "rgdal", "maptools", "plyr", "reshape2", "ggplot2", "ggmap", "rstan","StanHeaders")

##Apply the pkgTest using lapply()
  lapply(packages, pkgTest)
```

Next, we'll need to read-in 311 reports. There are two files in the /data directory: one for before 2010, and another for 2010 and after. The original files were obtained from [here for pre-2010](https://data.cityofnewyork.us/Social-Services/311-Service-Requests-for-2009/3rfa-3xsf) and [here for post-2010](https://nycopendata.socrata.com/Social-Services/311-Service-Requests-from-2010-to-Present/erm2-nwe9).

```{r part1a,message=FALSE,warning=FALSE}

#311 Reports are divided into two datasets, each is provided in zipped form
#We read both datasets into R separately and then combine them together
  temp = tempfile()
  unzip("data/311_Service_Requests_for_2009.zip",exdir=getwd())
  reports09 <- read.csv("311_Service_Requests_for_2009.csv")
  unlink(temp)
  
  temp = tempfile()
  unzip("data/311_Service_Requests_from_2010_to_Present.zip",exdir=getwd())
  reports10plus <- read.csv("311_Service_Requests_from_2010_to_Present.csv")
  unlink(temp)
  reports10plus$Resolution.Description <- NULL 
  
#Append 09 with 10+ together
  reports = rbind(reports09,reports10plus) 
  rm(reports09,reports10plus)

#To facilitate our analysis, we convert the report created date into a date object
  reports$Date <- as.Date(reports$Created.Date,format="%m/%d/%Y")

#Here we provide the dates for 8 major storms
  storm.dates <- as.Date(c("2009-08-18", #BILL
                           "2009-10-07",
                           "2010-03-13",
                           "2010-05-08",
                           "2010-09-16", #TORNADO
                           "2011-08-30", #IRENE
                           "2011-10-29",
                           "2012-10-29"), #SANDY
                         format = "%Y-%m-%d")

#We plot the log of 311 requests by day over study period and denote each storm by a vertical line. Daily 311 reports span several orders of magnitude, and the log transformation helps us to visualize these different orders on the same plot.

  ggplot() + 
    theme_bw() + 
    geom_point(aes(x,log(freq,base=10),alpha=log(freq,base=10)),data=count(reports$Date)) +
    geom_vline(aes(xintercept = as.numeric(storm.dates)),color="red",alpha=.7,linetype=2,
               data = data.frame(storm.dates)) +
    xlim(as.Date("2009-01-01",format="%Y-%m-%d"),as.Date("2013-01-01",format="%Y-%m-%d")) +
    theme(legend.position="none") + 
    labs(x="day",y=expression(log[10](311)))

#You can save any plot with the ggsave command
#ggsave(best_plot_ever.png,dpi = 500)
```

In this temporal plot, the points correspond to the number of daily 311 reports, and the dotted red lines demark the dates of major storms. This plot reveals, not surprisingly, that 311 requests for tree service between 2009 and 2013 spiked during these major storms. Note that these requests are on the log scale – in reality, the spikes jump by orders of magnitude above request rates experienced during normal days. New York City residents will remember the first spike as 2009’s Hurricane Bill and the last as 2012's Superstorm Sandy. We can use the amazing "facet_grid" feature of $ggplot2$ to quickly stratify this plot by type of request.

```{r part1b,message=FALSE,warning=FALSE}
ggplot(count(reports,c("Date","Descriptor"))) + 
  aes(Date,log(freq,base=10),alpha=log(freq,base=10)) +
  theme_bw() + 
  geom_point() +
  geom_vline(aes(xintercept = as.numeric(storm.dates)),color="red",alpha=.7,linetype=2,
             data = data.frame(storm.dates)) +
  xlim(as.Date("2009-01-01",format="%Y-%m-%d"),as.Date("2013-01-01",format="%Y-%m-%d")) +
  theme(legend.position="none") + 
  labs(x="day",y=expression(log[10](311))) +
  facet_grid(Descriptor~.,switch="y")
```

For this project, we will limit our analysis to damage reported within two days of each major storm (the day the storm hits plus the next full day.) These storms are large enough so that nearly every street tree along the storm path was at risk of being damaged. In addition, the time intervals are short enough that requests will not be a function of the strategy the Department of Parks and Recreation took to remove the tree damage in response to 311 reports.

```{r part1c,message=FALSE,warning=FALSE}
#We remove all reports not within two days of the storm. This is best done by first creating a function that tests if a report is in a storm intervals and then "applying" it across all storms.
  reports.dates <- cbind(storm.dates,storm.dates+1)
  interval.index <- function(dates,interval) which(dates >= interval[1] & dates <= interval[2])
  reports <- reports[unlist(apply(reports.dates,1,interval.index,dates=reports$Date)),]

#We also remove reports with no location. These happen to be infrequent.
  reports <- reports[!is.na(reports$X.Coordinate..State.Plane.),]
```

###Spatial data
To plot the data spatially, we'll need to read in some shapefiles. We will use two shapefiles: one for [census tracts](http://www1.nyc.gov/assets/planning/download/zip/data-maps/open-data/nyct2010wi16b.zip) and one for [public parks]. Note that the census tract file used in this tutorial is derived from the [cartographic boundary shapefiles](https://www.census.gov/geo/maps-data/data/cbf/cbf_tracts.html) available through the US Census Bureau.

Generally when we map city data, we include the parks shapefile as a reference point. Frankly, we do it because New Yorkers are terrible at locating their residence on a map. It could be due in some part to New York's subway maps, which are heavily distorted to facilitate the labeling of each stop. However, New York City residents can generally locate their residence in relation to the nearest park. The parks shapefile is the perfect way to reorient New Yorkers for this discussion. 

###Census tract shapefile
```{r part1d,message=FALSE,warning=FALSE}
#Read in shapefile from source
    temp = tempfile()
    url="http://www1.nyc.gov/assets/planning/download/zip/data-maps/open-data/nyct201016b.zip"
    download.file(url, temp) ##download the URL taret to the temp file
    unzip(temp,exdir=getwd()) ##unzip that file
    tracts <- readOGR("nyct2010_16b/nyct2010.shp","nyct2010")
    
#We convert the shapefile to a data.frame for plotting
    tracts@data$id <- rownames(tracts@data)
    tracts.points <- fortify(tracts, region="id")
    tracts.df <- join(tracts.points, tracts@data, by="id")

#We plot the tract data using ggplot2
#For other color palette options, see the ggplot documentation at http://docs.ggplot2.org/current/scale_brewer.html
    ggplot(tracts.df) +
      aes(long,lat,group=group,fill=factor(BoroCode)) + 
      geom_polygon() +
      coord_equal() +
      scale_fill_brewer("NYC Borough",palette="Set2") 
```

###NYC Parks Shapefile
```{r, message=FALSE, warning=FALSE}
  #Read in shapefile from source
    temp = tempfile()
    url="https://data.cityofnewyork.us/api/geospatial/rjaj-zgq7?method=export&format=Original"
    download.file(url, temp) ##download the URL taret to the temp file
    unzip(temp,exdir=getwd()) ##unzip that file
    parks <- readOGR("DPR_ParksProperties_001/DPR_ParksProperties_001.shp",
                     "DPR_ParksProperties_001")
    
  #We convert the shapefile to a data.frame for plotting
    parks@data$id <- rownames(parks@data)
    parks.points <- fortify(parks, region="id")
    parks.df <- join(parks.points, parks@data, by="id")

  #We now plot parks over tracts
    ggplot() +
      theme_nothing() +
      geom_polygon(aes(long,lat,group=group,fill=factor(BoroCode)),data=tracts.df) +
      geom_polygon(aes(long,lat,group=group),data=parks.df,fill="white") +
      coord_equal() +
      scale_fill_brewer("NYC Borough",palette="Set2") 
```

We would like to count the number of 311 reports per tract. However, the 311 data does not contain a tract id. We need to use the latitude-longitude information of each report to assign it to a census tract. This requires a "point in polygon" test where the reports are the points and the shapefile is the polygon.

```{r part1e,message=FALSE,warning=FALSE}
#We convert each report to a spatial points (sp) object.
  reports.sp <- SpatialPoints(coord=reports[, c("X.Coordinate..State.Plane.","Y.Coordinate..State.Plane.")],
                              proj4string=tracts@proj4string)
    
#We use the over function to find the tract for each report
  reports <- cbind(reports,over(x=reports.sp,y=tracts))
  
#We remove points with no assinged Borough
  reports <- reports[!is.na(reports$BoroName),]

#To facilitate plotting, we will also create some informative variables
#mapvalues  allows us to relabel factor levels with different lengths
  reports$Storm <- mapvalues(factor(reports$Date), from = levels(factor(reports$Date)), to = paste("Storm",sort(rep(1:8,2)),sep=""))
  df <- as.data.frame(table(reports[,c("Storm","BoroCT2010")]))
  colnames(df) <- c("Storm","BoroCT2010","Reports")

#We use the table function to count each report by tract (BoroCT2010), Storm and report type
  report_descrp <- function(des) as.data.frame(table(reports[reports$Descriptor==des,c("Storm","BoroCT2010")]))$Freq
  temp <- matrix(unlist(lapply(levels(reports$Descriptor),report_descrp)),
                 ncol=length(levels(reports$Descriptor)))
  colnames(temp) <- c("Hanging.Limb","Limb.Down","Tree.Down","Tree.Poor","Tree.Uprooted","Split.Tree")
  df <- cbind(df,data.frame(temp))
```

Now we can plot the results. Looking at the spatial plots, we can confirm that the reports do in fact correspond to the major storms that occurred during those time intervals. For example, Storm 5 corresponds to a set of tornados that ran through Brooklyn, Queens and a bit of Staten Island in the fall of 2010.

```{r part1f,message=FALSE,warning=FALSE}
#We plot the spatial distribution of all reports for a specific storm
  btracts.df <- join(tracts.df,df,by="BoroCT2010")
  ggplot(btracts.df[btracts.df$Storm=="Storm5",])+
    aes(x=long,y=lat,group=group,fill=log(Reports+1,base=10))+
    geom_polygon(color="grey",size=.1)+
    scale_fill_continuous(low="white",high="black",na.value="grey")+
    theme_nothing(legend=TRUE )+
    theme(legend.position="bottom",
          strip.background = element_rect(fill="white"))+
    geom_polygon(aes(x=long,y=lat,group=group),data=parks.df,fill="dark green",
                 color="white",size=.1,alpha=.5) +
    labs(fill=expression(log[10](311))) +
    coord_equal()

#We plot the spatial distribution of each storm
  ggplot(btracts.df)+
    aes(x=long,y=lat,group=group,fill=log(Reports+1,base=10))+
    geom_polygon(color="grey",size=.1)+
    scale_fill_continuous(low="white",high="black",na.value="grey")+
    theme_nothing(legend=TRUE )+
    theme(legend.position="bottom",
          strip.background = element_rect(fill="white"))+
    geom_polygon(aes(x=long,y=lat,group=group),data=parks.df,fill="dark green",
                 color="white",size=.1,alpha=.5) +
    labs(fill=expression(log[10](311))) +
    coord_equal() +
    facet_wrap(~Storm,ncol=4)

#We plot each damage type for a specific storm. Storm 8 is Sandy
  btracts.df <- join(tracts.df,df[df$Storm=="Storm8",],by="BoroCT2010")
  btracts.df$AllReports <- btracts.df$Reports
#This requires us to "melt" the data.frame by damage type for faceting
  colnames(btracts.df)
  mtracts.df <- melt(btracts.df[,c(1:7,21:26)],id=colnames(btracts.df)[1:7])
  mtracts.df$variable <-   
    factor(mtracts.df$variable,labels=gsub("\\.","",levels(mtracts.df$variable)))

ggplot(mtracts.df)+
  aes(x=long,y=lat,group=group,fill=log(value+1,base=10))+
  geom_polygon(color="grey",size=.1)+
  scale_fill_continuous(low="white",high="black",na.value="grey",
                        labels = function(x) format(x,digits = 1))+
  theme_nothing(legend=TRUE )+
  theme(legend.position="bottom",
        strip.background = element_rect(fill="white"))+
  geom_polygon(aes(x=long,y=lat,group=group),data=parks.df,fill="dark green",
               color="white",size=.1,alpha=.5) +
  facet_wrap(~variable,scales="free") +
  labs(fill=expression(log[10]("311"))) +
  coord_equal()
```

##Who is making the requests?
So far we have looked at where and when requests are made but not at who is generating them. For that information we turn to the US Census Bureau’s "Planning Database" ([PDB](http://www.census.gov/research/data/planning_database/)). This data set combines the decennial census with the American Community Survey ([2014 PDB -  Tract-level](http://www.census.gov/research/data/planning_database/2014/docs/PDB_Tract_2014-11-20.zip) ). We will also add data on the City’s decennial tree census to adjust for tracts with higher damage rates. We have already downloaded the planning database and the NYC tree census to our data folder. We'll need to combine both of these datasets with our work from the previous section.

####Planning Database: Loading + Transformation
To start, we'll download and read in the PDB data: [http://www.census.gov/research/data/planning_database/2014/docs/PDB_Tract_2014-11-20.zip](http://www.census.gov/research/data/planning_database/2014/docs/PDB_Tract_2014-11-20.zip)
```{r part2a,message=FALSE,warning=FALSE}

#create temp file, save zip file, unzip, extract main csv file
#We read the "people"" census. Remove all non-NYC entries
  temp <- tempfile()
  url= "http://www.census.gov/research/data/planning_database/2014/docs/PDB_Tract_2014-11-20.zip"
  download.file(url,temp)
  people_census <- read.csv(unz(temp, "pdb2014trv9_us.csv"))
  unlink(temp)

#Remove all non-NYC entries
  people_census <- people_census[people_census$State_name == "New York",]
  people_census <- people_census[,-sort(unique(c(grep("ACSMOE",colnames(people_census)),
                                                 grep("pct",colnames(people_census)),
                                                 grep("CEN",colnames(people_census)))))]
  people_census <- people_census[people_census$County_name == "New York County" |
                                 people_census$County_name == "Queens County" | 
                                 people_census$County_name == "Richmond County"|
                                 people_census$County_name == "Kings County" |
                                 people_census$County_name == "Bronx County",]
  
#We create a tract-specific BoroCT2010 variable so census can be merged with previous work
  people_census$BCode <- as.numeric(as.character(factor(people_census$County_name,labels=c(2,3,1,4,5))))
  temp <- paste("00000",as.character(people_census$Tract),sep="")
  temp <- substr(temp,nchar(temp)-5,nchar(temp))
  people_census$BoroCT2010 <- factor(paste(people_census$BCode,temp,sep=""))
  
#We remove unimportant variables (like measurement error and variables that are not counts)
  people_census <- people_census[,c(11:127,131)]
  people_census <- people_census[,-c(101,102,116,117)] #remove income variables since not counts
  people_census <- people_census[,-c(45)] #remove Hmong since it has zero variance
```

####Tree Census: Loading + Transformation
https://data.cityofnewyork.us/Environment/2005-Street-Tree-Census/29bw-z7pj
Download and transform the Tree Census data into census tract form.

```{r ,message=FALSE,warning=FALSE}
#Read in Tree Census data (may take a few minutes)
  tree_census <- read.csv("https://data.cityofnewyork.us/api/views/29bw-z7pj/rows.csv?accessType=DOWNLOAD")
  tree_census = data.frame(table(tree_census$boro_ct))
  colnames(tree_census) <- c("BoroCT2010","treecount")
  
#We merge the two census datasets together
  census <- na.omit(join(people_census,tree_census,by="BoroCT2010"))
  df <- join(census,df,by="BoroCT2010")
  btracts.df <- join(tracts.df,df,by="BoroCT2010")

#We add one to avoid dividing by zero
  btracts.df$HHDReports <-  (btracts.df$Reports+1)/(btracts.df$Tot_Housing_Units_ACS_08_12+1)
  btracts.df$TreeReports <-  (btracts.df$Reports+1)/(btracts.df$treecount+1)

#We set as NA tracts with fewer than 100 households
  btracts.df$HHDReports[btracts.df$Tot_Housing_Units_ACS_08_12<100] <- NA
  btracts.df$TreeReports[btracts.df$treecount<10] <- NA

#We plot requests per household and requests per tree
ggplot(btracts.df[btracts.df$Storm=="Storm8",])+
  aes(x=long,y=lat,group=group,fill=log(HHDReports,base=10))+
  geom_polygon(color="grey",size=.1)+
  scale_fill_continuous(expression(log[10]("311/HHD")),
                        low="white",high="black",na.value="grey",
                        breaks = c(-3,-2,-1,0))+
  theme_nothing(legend=TRUE )+
  theme(legend.position="bottom",
        strip.background = element_rect(fill="white"))+
  geom_polygon(aes(x=long,y=lat,group=group),data=parks.df,fill="dark green",
               color="white",size=.1,alpha=.5)+
  coord_equal()

ggplot(btracts.df[btracts.df$Storm=="Storm8",])+
  aes(x=long,y=lat,group=group,fill=log(TreeReports,base=10))+
  geom_polygon(color="grey",size=.1)+
  scale_fill_continuous(expression(log[10]("311/Tree")),
                        low="white",high="black",na.value="grey",
                        breaks = c(-4,-2,-1,0))+
  theme_nothing(legend=TRUE )+
  theme(legend.position="bottom",
        strip.background = element_rect(fill="white"))+
  geom_polygon(aes(x=long,y=lat,group=group),data=parks.df,fill="dark green",
               color="white",size=.1,alpha=.5)+
  coord_equal()
```

We begin by considering the two most obvious explanations for the variation in 311 reports: the number of people and trees in each tract. We do this by creating two plots of damage after Superstorm Sandy (Storm 8), although any storm could be plotted by changing "Storm8" to, say, "Storm3". The first plot shows the (log) amount of damage per household, and the second shows the (log) amount of damage per tree. Compare this to the storm map plotted earlier. In Storm 8, damage is concentrated to the south (Staten Island) and the east (Queens). These plots show less concentration. That's because, by dividng by the number of households or trees, we "account" for the spatial variation of reports due to these variables.  

Dividing by household accounts for less variation than number of trees. The outerborough (suburban) areas of New York City still have a higher report per household rate. The report per tree rate is much more evenly distributed throughout New York City. This suggests the following "no-brainer" theory: that residents on the outerskirts of New York City are more likely to report damage because they have more trees and likely experienced more damage. But stick with us, it's going to get a lot more interesting.

In fact, there's still a lot more variation to explain, and we can get a lot more specific in how residents use 311 because we have over 120 census variables to look at in addition to the number of households and the treecount. Unfortunately, it would be impossible to consider the importance of all of these variables in a principled manner without constructing a statistical model. That's because all of these variables are highly correlated, and there is simply no easy way to visualize such complex relationships with maps.

We've come pretty far, and we've already done much of the heavy lifting. We have explored the data and taken a few extra steps toward a better understanding of who uses 311 to ask government for help -- and, just as important, toward an understanding of which groups might be using the system less than they should be. 

Next, we will investigate when and where 311 reports come from. We will then be able to take advantage of this spatial and temporal variation to estimate who uses 311 - see our follow-up post, "Who uses 311? (Part 2).

[1] Eshleman, C, H Gursoy and G Ng. 2013. Major storms and NYC trees: an analysis of damage and tree characteristics. https://sipa.columbia.edu/sites/default/files/AY13_NYCMayorsOps_FinalReport_0.pdf. 
