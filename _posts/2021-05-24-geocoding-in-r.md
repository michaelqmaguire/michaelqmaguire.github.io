---
layout: post
title: Geocoding in R
tags: [R, Geocoding, Mapping, Tidyverse]
linktormd: true
thumbnail-img: /assets/img/geocoding-in-R.gif
always_allow_html: yes
comments: true
---

#### It's your usual meeting time on Zoom, and maybe you're taking a sip of coffee. 

All is well, and then your professor/supervisor tells you that they have a list of addresses that they need geocoded.

There are a couple camps here: 

  1. You feel comfortable working with GIS software.
  2. You're not familiar with GIS software, but you know how to use the [Census Geocoder](https://geocoding.geo.census.gov/geocoder/locations/addressbatch?form).
  3. You have no clue what to do.
  
If you're in the first camp, you *might* be fine. Assuming you have access to the software you've used, you can get these addresses geocoded.

But what if you **don't** have access to that software? 

What if you know how to use the Census Geocoder, but you always get frustrated because it doesn't always return a great match rate?

What if you have no idea what you're doing?

#### Enter R!

Now, if you let me talk long enough, I'll talk your ear off about how you can use R for everything (not everything, but most things). 

There are so many benefits:

  1. It's free to download -- no licensing or software purchase necessary
  2. It has a couple dialects, with recent ones--the tidyverse--being particularly modern, powerful, and understandable.
  3. Your results can be reproducible, and you can re-run your analyses if your professor/supervisor adds additional records! No tedious backtracking, getting on the Census website, etc.
  4. **Did I mention it's free?**
  
#### Let's look at an example.

I went ahead and grabbed some local medical institutions that presumably have prescribing capabilities. 

I figured that something like this is relatable considering we live in the area.

First things first, though -- you need to load the libraries.

Before that, though, you need to have them installed! Only run the following two `install.packages` commands if you do **not** have packages installed.

We will be working with two packages:

  1. `tidyverse`
  2. `tidygeocoder`
  
If you do not have these packages installed, run the following in your console:

```r
install.packages("tidyverse")
install.packages("tidygeocoder")
```

*Now* we can load the libraries.

```r
library(tidyverse)
library(tidygeocoder)
```

Cool! We're locked and loaded!

I'm now going to load the addresses of those pharmacy locations around Gainesville, FL.

I am going to use the `tribble` function from the `tibble` package. `tibble` is loaded with the `tidyverse` package.

Here's what you need to know: `tribble` stands for *transposed* `tibble`, and a tibble is basically a data set/data frame. No need to get lost in the details here -- it's going to spit out exactly what we need, and it is what your data will probably look like anyway.

I'm going to store the addresses for the pharmacies into an object called 'pharmacies'. Go ahead and copy this and paste it into your R session!

```r
pharmacies <- 
  tribble(
    ~ Name, ~ Address, ~ CityStateZip, ~ Phone,
    "Malcom Randall VA Medical Ctr",	 "1601 SW Archer Rd",	      "Gainesville,FL 32608",	"(352)376-1611",
    "Shands Hospital-University FL",	 "1600 SW Archer Rd",	      "Gainesville,FL 32610",	"(352)265-0111",
    "Tacachale",	                     "1621 NE Waldo Rd",	      "Gainesville,FL 32609",	"(352)955-5000",
    "North Florida Evaluation/Trtmnt", "1200 NE 55th Blvd",	      "Gainesville,FL 32641",	"(352)375-8484",
    "Shands Rehabilitation Hospital",	 "4101 NW 89th Blvd",	      "Gainesville,FL 32606",	"(352)265-5491",
    "Shands Vista",	                   "4101 NW 89th Blvd",	      "Gainesville,FL 32606",	"(352)265-5497",
    "N Florida Reg Med Ctr",	         "6500 W Newberry Rd",	    "Gainesville,FL 32605",	"(352)333-4000",
    "Tri County Hospital Williston",	 "125 SW 7th St",	          "Williston,FL 32696",	  "(352)528-2801",
    "Reception & Medical Center",	     "7765 S County Road 231",	"Lake Butler,FL 32054",	"(386)496-6000",
    "Shands Starke Medical Center",	   "922 E Call St",	          "Starke,FL 32091",	    "(904)368-2300",
    "Lake Butler Hospital",	           "850 E Main St",	          "Lake Butler,FL 32054",	"(386)496-2323",
    "Ocala Regional Medical Center",	 "1431 SW 1st Ave",	        "Ocala,FL 34471",	      "(352)401-1000",
    "Munroe Regional Medical Center",	 "1500 SW 1st Ave",	        "Ocala,FL 34471",	      "(352)351-7200"
  )

pharmacies
```
```
## # A tibble: 13 x 4
##    Name                       Address             CityStateZip       Phone      
##    <chr>                      <chr>               <chr>              <chr>      
##  1 Malcom Randall VA Medical~ 1601 SW Archer Rd   Gainesville,FL 32~ (352)376-1~
##  2 Shands Hospital-Universit~ 1600 SW Archer Rd   Gainesville,FL 32~ (352)265-0~
##  3 Tacachale                  1621 NE Waldo Rd    Gainesville,FL 32~ (352)955-5~
##  4 North Florida Evaluation/~ 1200 NE 55th Blvd   Gainesville,FL 32~ (352)375-8~
##  5 Shands Rehabilitation Hos~ 4101 NW 89th Blvd   Gainesville,FL 32~ (352)265-5~
##  6 Shands Vista               4101 NW 89th Blvd   Gainesville,FL 32~ (352)265-5~
##  7 N Florida Reg Med Ctr      6500 W Newberry Rd  Gainesville,FL 32~ (352)333-4~
##  8 Tri County Hospital Willi~ 125 SW 7th St       Williston,FL 32696 (352)528-2~
##  9 Reception & Medical Center 7765 S County Road~ Lake Butler,FL 32~ (386)496-6~
## 10 Shands Starke Medical Cen~ 922 E Call St       Starke,FL 32091    (904)368-2~
## 11 Lake Butler Hospital       850 E Main St       Lake Butler,FL 32~ (386)496-2~
## 12 Ocala Regional Medical Ce~ 1431 SW 1st Ave     Ocala,FL 34471     (352)401-1~
## 13 Munroe Regional Medical C~ 1500 SW 1st Ave     Ocala,FL 34471     (352)351-7~
```

Cool! Our data is in the 'pharamcies' object, and we're ready to go. So what's next?

Well, if you've never dealt with address data before, you need to know one thing: **ADDRESS INFORMATION CAN BE VERY, VERY DIRTY AND DIFFICULT TO CLEAN!**

What I've put up here is relatively clean, but I figured a little bit of data management would be prudent.

Looking at the 'CityStateZip' field, we can see that everything is jammed together when it probably shouldn't be.

Let's parse it using the `tidyr` package which is housed in the `tidyverse`.

Now, one feature of the tidyverse is its use of the pipe, or the `%>%` symbol. Pipes practically say *take this object on the left and feed it to what I have after it*.

I like to eat, so maybe think of the pipe as you taking your favorite food (the object) and feeding it into your body (and whatever follows).

So here, we're doing a couple things:

  1. We're first storing our results in the object called 'pharmaciesCSZSeparate'.
  2. Then, we're taking the 'pharmacies' object from above, and feeding it into the `tidyr::separate` function. 
    - `tidyr::separate` separates columns based on a delimiter or a given characteristic.
  3. We're setting our data argument to `.`, which is telling the `separate` function that our dataset is what we're feeding it, which is the 'pharmacies' object.
  4. We're wanting to separate the 'CityStateZip' field.
  5. We want to separate the 'CityStateZip' field into two new fields: (1) 'City' and (2) 'StateZip'
  6. We're wanting to split the 'CityStateZip' field where a comma occurs. If you're not familiar with regular expressions, just know that you have to escape whatever you want to break on with double backslashes in R.
  7. Let's keep that feeding going -- we're taking that chewed-up pharmacies object from #2-#6 and feeding it into another `separate` function
  8. Again, we've already declared the data element to be the 'pharmacies' object through the pipe, so fill this with another `.`.
  9. We now want to split the newly created 'StateZip' field.
  10. And we're splitting it into two new fields, 'State' and 'Zip'.
  11. And we're splitting on a space! `s` stands for space in regular expressions.

Again, feel free to copy and paste this code and put it into your R session!

```r
pharmaciesCSZSeparate <-
  pharmacies %>% # 1
    separate(    # 2
      data = .,  # 3
      col = CityStateZip, # 4
      into = c("City", "StateZip"), # 5
      sep = "\\,", # 6
  ) %>% # 7
    separate(
      data = ., # 8 
      col = StateZip, # 9
      into = c("State", "Zip"), # 10
      sep = "\\s" # 11
    )

pharmaciesCSZSeparate
```
```
## # A tibble: 13 x 6
##    Name                     Address            City       State Zip   Phone     
##    <chr>                    <chr>              <chr>      <chr> <chr> <chr>     
##  1 Malcom Randall VA Medic~ 1601 SW Archer Rd  Gainesvil~ FL    32608 (352)376-~
##  2 Shands Hospital-Univers~ 1600 SW Archer Rd  Gainesvil~ FL    32610 (352)265-~
##  3 Tacachale                1621 NE Waldo Rd   Gainesvil~ FL    32609 (352)955-~
##  4 North Florida Evaluatio~ 1200 NE 55th Blvd  Gainesvil~ FL    32641 (352)375-~
##  5 Shands Rehabilitation H~ 4101 NW 89th Blvd  Gainesvil~ FL    32606 (352)265-~
##  6 Shands Vista             4101 NW 89th Blvd  Gainesvil~ FL    32606 (352)265-~
##  7 N Florida Reg Med Ctr    6500 W Newberry Rd Gainesvil~ FL    32605 (352)333-~
##  8 Tri County Hospital Wil~ 125 SW 7th St      Williston  FL    32696 (352)528-~
##  9 Reception & Medical Cen~ 7765 S County Roa~ Lake Butl~ FL    32054 (386)496-~
## 10 Shands Starke Medical C~ 922 E Call St      Starke     FL    32091 (904)368-~
## 11 Lake Butler Hospital     850 E Main St      Lake Butl~ FL    32054 (386)496-~
## 12 Ocala Regional Medical ~ 1431 SW 1st Ave    Ocala      FL    34471 (352)401-~
## 13 Munroe Regional Medical~ 1500 SW 1st Ave    Ocala      FL    34471 (352)351-~
```

We're moving now! Everything is separated, and it's looking clean.

Let me iterate, *cleaning address data will rarely be this easy*.

I digress -- now how the heck do we get that latitude and longitude?

#### Enter `tidygeocoder`!

Okay, okay, I'm looking at the wall of text above and I'm feeling a bit bad -- I don't like writing a ton, and I like providing efficient solutions. 

Nonetheless, we're going to use the `geocode` function in the `tidygeocoder` package to do the work. Interestingly, you have a couple options with their `method =` argument, and I encourage you to check out their [documentation](https://jessecambon.github.io/tidygeocoder/index.html) instead of me explaining every single facet of what it does.

Let's go through this step by step again:

  1. We're taking that 'pharmaciesCSZSeparate' object and *feeding* it into the `geocode` function.
  2. We're setting the `.tbl = ` argument to `.` because we've already fed the `geocode` function the necessary object (pharmaciesCSZSeparate).
  3. Geocode Argument 1: `street` should be set to the `Address` field in our dataset. It refers to the number-direction-street portion of the address.
  4. Geocode Argument 2: `city` should be set to the `City` field in our dataset.
  5. Geocode Argument 3: `state` should be set to the `State` field in our dataset.
  6. Geocode Argument 4: `postalcode` should be set to the `Zip` field in our dataset.
  7. This is where things get fun, you specify a method. I chose 'census' for those of us familiar with the census geocoder.

Here I am again -- copy and paste this into your R session!

```r
pharamciesCensusMethod <- 
  pharmaciesCSZSeparate %>% # 1
    geocode(
      .tbl = ., # 2
      street = Address, # 3
      city = City, # 4
      state = State, # 5
      postalcode = Zip, # 6
      method = "census" # 7
    )

pharamciesCensusMethod
```
```
## # A tibble: 13 x 8
##    Name                Address        City     State Zip   Phone       lat  long
##    <chr>               <chr>          <chr>    <chr> <chr> <chr>     <dbl> <dbl>
##  1 Malcom Randall VA ~ 1601 SW Arche~ Gainesv~ FL    32608 (352)376~  NA    NA  
##  2 Shands Hospital-Un~ 1600 SW Arche~ Gainesv~ FL    32610 (352)265~  29.6 -82.3
##  3 Tacachale           1621 NE Waldo~ Gainesv~ FL    32609 (352)955~  NA    NA  
##  4 North Florida Eval~ 1200 NE 55th ~ Gainesv~ FL    32641 (352)375~  29.7 -82.2
##  5 Shands Rehabilitat~ 4101 NW 89th ~ Gainesv~ FL    32606 (352)265~  29.7 -82.4
##  6 Shands Vista        4101 NW 89th ~ Gainesv~ FL    32606 (352)265~  29.7 -82.4
##  7 N Florida Reg Med ~ 6500 W Newber~ Gainesv~ FL    32605 (352)333~  29.7 -82.4
##  8 Tri County Hospita~ 125 SW 7th St  Willist~ FL    32696 (352)528~  29.4 -82.5
##  9 Reception & Medica~ 7765 S County~ Lake Bu~ FL    32054 (386)496~  30.0 -82.3
## 10 Shands Starke Medi~ 922 E Call St  Starke   FL    32091 (904)368~  29.9 -82.1
## 11 Lake Butler Hospit~ 850 E Main St  Lake Bu~ FL    32054 (386)496~  30.0 -82.3
## 12 Ocala Regional Med~ 1431 SW 1st A~ Ocala    FL    34471 (352)401~  29.2 -82.1
## 13 Munroe Regional Me~ 1500 SW 1st A~ Ocala    FL    34471 (352)351~  29.2 -82.1
```

Alright, we're getting somewhere, but notice those `NA` fields. Ugh, just like the Census Geocoder, some addresses don't end up getting geocoded. How frustrating. Time to go to google and manually copy the latitude and longitude, right?

Think again! Just change that method to `cascade`. What this does is it uses a combination of the `census` method **AND** the `osm` method. `osm` stands for open street map, and you can read more about that [here](https://wiki.openstreetmap.org/wiki/About_OpenStreetMap).

You guessed it -- copy and paste this into your R session!

```r
pharmaciesCascadeMethod <- 
  pharmaciesCSZSeparate %>% # 1
    geocode(
      .tbl = ., # 2
      street = Address, # 3
      city = City, # 4
      state = State, # 5
      postalcode = Zip, # 6
      method = "cascade" # 7
    )

pharmaciesCascadeMethod
```
```
## # A tibble: 13 x 9
##    Name           Address      City   State Zip   Phone     lat  long geo_method
##    <chr>          <chr>        <chr>  <chr> <chr> <chr>   <dbl> <dbl> <chr>     
##  1 Malcom Randal~ 1601 SW Arc~ Gaine~ FL    32608 (352)3~  29.6 -82.4 osm       
##  2 Shands Hospit~ 1600 SW Arc~ Gaine~ FL    32610 (352)2~  29.6 -82.3 census    
##  3 Tacachale      1621 NE Wal~ Gaine~ FL    32609 (352)9~  29.7 -82.3 osm       
##  4 North Florida~ 1200 NE 55t~ Gaine~ FL    32641 (352)3~  29.7 -82.2 census    
##  5 Shands Rehabi~ 4101 NW 89t~ Gaine~ FL    32606 (352)2~  29.7 -82.4 census    
##  6 Shands Vista   4101 NW 89t~ Gaine~ FL    32606 (352)2~  29.7 -82.4 census    
##  7 N Florida Reg~ 6500 W Newb~ Gaine~ FL    32605 (352)3~  29.7 -82.4 census    
##  8 Tri County Ho~ 125 SW 7th ~ Willi~ FL    32696 (352)5~  29.4 -82.5 osm       
##  9 Reception & M~ 7765 S Coun~ Lake ~ FL    32054 (386)4~  30.0 -82.3 census    
## 10 Shands Starke~ 922 E Call ~ Starke FL    32091 (904)3~  29.9 -82.1 census    
## 11 Lake Butler H~ 850 E Main ~ Lake ~ FL    32054 (386)4~  30.0 -82.3 census    
## 12 Ocala Regiona~ 1431 SW 1st~ Ocala  FL    34471 (352)4~  29.2 -82.1 census    
## 13 Munroe Region~ 1500 SW 1st~ Ocala  FL    34471 (352)3~  29.2 -82.1 census
```

Wow! Each address is geocoded at this point! How exciting!

We can then write out the file using the `readr` function called `write_csv`.

```r
write_csv(x = pharmaciesCascadeMethod, file = "save-to-my-directory")
```

## Hold up: I don't want to do all this, can I just do all this in Excel and pull in the file?

Sure! I'm a big advocate of reproducibility and having a paper-trail, so I think everything should be written in code.

But I know that's not ideal for many of you, especially if the time frame is slim.

Here is what I would suggest for you to do:

  1. Create/arrange the (1) Address, (2) City, (3) State, and (4) Zip fields in Excel for each address that you need.
  2. Boot up RStudio.
  3. Load/install `tidyverse`, `tidygeocoder`, and `readxl` packages. You do not need to reinstall if you've already installed one.
  4. Read in the dataset into R using:
      - `read_excel` from the `readxl` package for Excel files. **AGAIN, YOU MUST INSTALL AND LOAD!**
      - `read_csv` from the `readr` package (automatically loaded with `tidyverse`). If you save it from Excel as a .csv, this function will read in the data.
      - `read.csv` using the base-R `utils` package (you do not need to load this - it is already loaded when you start R). Again, if you save it from Excel as a .csv, this function will read in the data appropriately, but you have to be wary of the `stringsAsFactors` argument mentioned below.
  5. Feed your imported dataset to `tidygeocoder`.
  6. Output the file.
  
Another tutorial:

```r
library(readxl)

# If you saved your file as an Excel file, use this.

to_geocode <-
  read_excel(
    path = "path-to-your-file" 
  )

# If you saved your file as a csv, you can use readr::read_csv or you can use the base-R read.csv

# readr approach

to_geocode <-
  read_csv(
    file = "path-to-your-file"
  )

# Base-R Approach

to_geocode <-
  read.csv(
    file = "path-to-your-file" # You must use backslashes in your directory, or you can double forward slash (//),
    stringsAsFactors = FALSE # default is to read in character variables as factors.
  )
```

Now, we throw it into the geocoder!

```r
geocoded <- 
  to_geocode %>%
    geocode(
      .tbl = to_geocode,
      street = Address,
      city = City,
      state = State,
      postalcode = Zip,
      method = "cascade"
    )
```

We can then write it out to our destination directory.

```r
write.csv(x = geocoded, file = "save-to-my-directory") # base-R approach
write_csv(x = geocode, file = "save-to-my-directory") # readr approach
```

I'm going to save plotting for another time, because there's so much that can be done within R.

### Information about R if you've never touched it before:

I recommend:
  
  1. Downloading R from [CRAN](https://cran.r-project.org/).
  2. Then downloading [RStudio](https://www.rstudio.com/products/rstudio/download/). RStudio is a graphic user interface. It isn't R itself -- it's like the body of the car while R from CRAN is the engine. 
  3. When both have been installed open up the RStudio app and begin coding! 
  
If you need any help navigating the graphic user interface (GUI) of RStudio, feel free to reach out to me. I'm available in person most days, and I can certainly meet over Zoom.

Until next time! Have a great week!
