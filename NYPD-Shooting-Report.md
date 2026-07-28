NYPD Shooting Incident Data Project
================
W. Jencyowski

## Data Import

``` r
# NYPD Shooting Incident Data (Historic) from www.catalog.data.gov
nypd_url <- "https://data.cityofnewyork.us/api/v3/views/833y-fsy8/export.csv?accessType=DOWNLOAD"
nypd_raw <- read_csv(nypd_url)
```

    ## Rows: 29744 Columns: 21
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (12): OCCUR_DATE, BORO, LOC_OF_OCCUR_DESC, LOC_CLASSFCTN_DESC, LOCATION...
    ## dbl   (5): INCIDENT_KEY, PRECINCT, JURISDICTION_CODE, Latitude, Longitude
    ## num   (2): X_COORD_CD, Y_COORD_CD
    ## lgl   (1): STATISTICAL_MURDER_FLAG
    ## time  (1): OCCUR_TIME
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## Renaming

### Renaming each column for clarity and ease of reading

``` r
nypd <- nypd_raw |> 
    rename(
        Key = INCIDENT_KEY,
        Date = OCCUR_DATE,
        Time = OCCUR_TIME,
        Boro = BORO,
        Inside_outside = LOC_OF_OCCUR_DESC,
        Precinct = PRECINCT,
        Jurisdiction = JURISDICTION_CODE,
        Location = LOC_CLASSFCTN_DESC,
        Location_description = LOCATION_DESC,
        Murder = STATISTICAL_MURDER_FLAG,
        Perp_age = PERP_AGE_GROUP,
        Perp_sex = PERP_SEX,
        Perp_race = PERP_RACE,
        Vic_age = VIC_AGE_GROUP,
        Vic_race = VIC_RACE,
        Vic_sex = VIC_SEX,
        )
```

## Refactoring

### Refactoring column type based off the dataset footnotes

``` r
nypd$Key <- as.integer(nypd$Key)
nypd$Date <- as.Date(nypd$Date, "%m/%d/%Y")
# OCCUR_TIME is already is already hr/min/sec
nypd$Boro <- as.factor(nypd$Boro)
nypd$Inside_outside <- as.factor(nypd$Inside_outside)
nypd$Precinct <- as.integer(nypd$Precinct)
nypd$Jurisdiction <- as.integer(nypd$Jurisdiction)
nypd$Location <- as.factor(nypd$Location)
```

## Cleaning perp and victim demographics

### This next section cleans the age, sex, and race columns using the naniar library, specifically the `replace_with_na` function.

#### Finding perp demo outliers

``` r
# Using the `unique()` command  to find each demegraphic that doesn't fit into the established convention
unique(nypd$Perp_age)
```

    ##  [1] "25-44"   "(null)"  "45-64"   "18-24"   "<18"     "65+"     "2021"   
    ##  [8] "1028"    NA        "UNKNOWN" "1020"    "940"     "224"

``` r
unique(nypd$Perp_sex)
```

    ## [1] "M"      "(null)" "F"      "U"      NA

``` r
unique(nypd$Perp_race)
```

    ## [1] "BLACK"                          "(null)"                        
    ## [3] "BLACK HISPANIC"                 "WHITE HISPANIC"                
    ## [5] "ASIAN / PACIFIC ISLANDER"       "UNKNOWN"                       
    ## [7] "WHITE"                          NA                              
    ## [9] "AMERICAN INDIAN/ALASKAN NATIVE"

#### Replacing outlier values with NA

``` r
# entries in each column that don't match convention. Unknowns are changed to NA for uniformity across the data set
perp_age_filter <- c("(null)", "2021", "1028", "UNKNOWN", "1020", "940", "224")
perp_sex_filter <- c("(null)", "U")
perp_race_filter <- c("(null)", "UNKNOWN")

# replacing those values with NA
nypd <- nypd |> 
    replace_with_na(replace = list(Perp_age = perp_age_filter,
                                   Perp_sex = perp_sex_filter,
                                   Perp_race = perp_race_filter)) 
```

#### Finding vic demo outliers

``` r
# the same process as above is done for the victims
unique(nypd$Vic_age)
```

    ## [1] "25-44"   "18-24"   "<18"     "45-64"   "65+"     "UNKNOWN" "1022"

``` r
unique(nypd$Vic_sex)
```

    ## [1] "M" "F" "U"

``` r
unique(nypd$Vic_race)
```

    ## [1] "BLACK"                          "WHITE HISPANIC"                
    ## [3] "WHITE"                          "BLACK HISPANIC"                
    ## [5] "ASIAN / PACIFIC ISLANDER"       "UNKNOWN"                       
    ## [7] "AMERICAN INDIAN/ALASKAN NATIVE"

#### Replacing vic demo outlier values with NA

``` r
# creating a list of strings from each column that doesn't match convention
# unknowns are changed to NA for uniformity across the data set
vic_age_filter <- c("UNKNOWN", "1022")
vic_sex_filter <- c("U")
vic_race_filter <- c("UNKNOWN")

# replacing those values with NA
nypd <- nypd |> 
    replace_with_na(replace = list(Vic_age = vic_age_filter,
                                   Vic_sex = vic_sex_filter,
                                   Vic_race = vic_race_filter)) 
```
