Project memo
================
Liz, Charlie, Noku - Healthy Androscoggin - Smoke Trends Analysis

This document contains a detailed account of the data clean up for our
data and the design choices we are making for the plots.

## Data Clean Up Steps for Overall Data

### Step 1: Transforming PDFs documents into Excel tables

Our community partner data was presented as a collection of PDF
documents. Thus, our first step of data cleaning process involved
transforimg presented data into Excel files. To achieve that, we used
PDF to Excel Converter online
(<https://acrobat.adobe.com/link/acrobat/pdf-to-excel?x_api_client_id=adobe_com&x_api_client_location=pdf_to_excel>).

### Step 2: Data Cleaning within Excel

Next, we proceeded to clear formatting of the tables we saved from PDF
documents. After that, we conducted some simple data preparation,
including removal of empty rows and columns within each year’s dataframe
and change of column names to be consistent across years.

### Step 3: Downloading Excel data as CSV files

During this step we downloaded all CSV files representing smoke data for
year 2009, 2013, 2015, 2017, 2019, and 2021.

### Step 4: Data Cleaning with R

Initially, we tried to clean each `smoke[year].csv` data file
separately, which involved a lot of copy-pasting of the same code. Thus,
we decided to write a function to achieve better efficiency.

In this function, we clean and reshape survey data from a CSV file into
a tidy format. We begin by reading the file and renaming the columns for
clarity. Then, we assign a group_type (such as Age, Grade, or
Race/Ethnicity) based on patterns in the data and remove header rows.
Next, we create three separate data frames for total, female, and male
responses, converting percentages to numeric values and cleaning
confidence intervals. We then standardize the format across these data
frames, combine them into a single long-format dataset, and add the
survey year.

``` r
clean_data <- function(file_path, year) {
  df <- read.csv(file_path, stringsAsFactors = FALSE)

  # Rename columns
  colnames(df) <- c(
    "category", "total_percent", "total_ci", "total_number",
    "female_percent", "female_ci", "female_number",
    "male_percent", "male_ci", "male_number"
  )

  # Add group_type
  df_clean <- df %>%
    mutate(group_type = case_when(
      str_detect(category, "----Age----") ~ "Age",
      str_detect(category, "----Grade----") ~ "Grade",
      str_detect(category, "----Race/Ethnicity----") ~ "Race/Ethnicity",
      TRUE ~ NA_character_
    )) %>%
    fill(group_type, .direction = "down") %>%
    filter(!str_detect(category, "----"))

  # Build long format using mutate + select
  total_df <- df_clean %>%
    mutate(
      gender = "total",
      percent = as.numeric(str_remove(total_percent, "%")),
      ci = ifelse(total_ci %in% c("^", "*"), NA, total_ci),
      number = as.numeric(total_number)
    ) %>%
    select(subgroup = category, group_type, gender, percent, ci, number)

  female_df <- df_clean %>%
    mutate(
      gender = "female",
      percent = as.numeric(str_remove(female_percent, "%")),
      ci = ifelse(female_ci %in% c("^", "*"), NA, female_ci),
      number = as.numeric(female_number)
    ) %>%
    select(subgroup = category, group_type, gender, percent, ci, number)

  male_df <- df_clean %>%
    mutate(
      gender = "male",
      percent = as.numeric(str_remove(male_percent, "%")),
      ci = ifelse(male_ci %in% c("^", "*"), NA, male_ci),
      number = as.numeric(male_number)
    ) %>%
    select(subgroup = category, group_type, gender, percent, ci, number)

  # Combine and add year
  df_long <- bind_rows(total_df, female_df, male_df) %>%
    mutate(year = year)

  return(df_long)
}
```

### Step 5: Call Function on All CSV Files

In this step, we use the function outlined above to clean each
`smoke[year].csv` file separately.

``` r
df_2009 <- clean_data("/cloud/project/data/smoke2009.csv", 2009)
df_2013 <- clean_data("/cloud/project/data/smoke2013.csv", 2013)
df_2015 <- clean_data("/cloud/project/data/smoke2015.csv", 2015)
df_2017 <- clean_data("/cloud/project/data/smoke2017.csv", 2017)
df_2019 <- clean_data("/cloud/project/data/smoke2019.csv", 2019)
df_2021 <- clean_data("/cloud/project/data/smoke2021.csv", 2021)
```

### Step 6: Combine Each Year’s Dataframe into a Single Dataframe

Lastly, we combine all dataframes we have recieved through the process
of cleaning into a single dataframe called `smoke_data`.

``` r
# Combine all dataframes together

smoke_data <- bind_rows(df_2009, df_2013, df_2015, df_2017, df_2019, df_2021)
```

## Plots

The color scheme was selected with accessibilty in mind. Specifically,
we chose to use Brewer R library.

### Plot 1: Percentage of teens who smoked at least once in the past 30 days by Age

#### Data cleanup steps specific to plot 1

Since our data is hard to work with as a whole, we filter it
specifically to include only Totals for all age groups, meaning data
points that are not separated by gender.

``` r
total_age_df <- smoke_data |>
  filter(group_type == "Age", gender == "total")
```

#### Final Plot 1

``` r
ggplot(total_age_df, aes(x = year, y = percent, color = subgroup, group = subgroup)) +
  geom_line(size = 1) +
  geom_point(size = 2) +
  scale_color_brewer(palette = 3, 
                     type = "qual",
                     guide = guide_legend(reverse = TRUE)) +
  scale_x_continuous(breaks = c(2009, 2011, 2013, 2015, 2017, 2019, 2021)) +
  geom_vline(xintercept = 2014, color = "darkgray", linetype = "dashed") +
  annotate("text", x = 2014, y = Inf, label = "E-cigarettes surpass \nconventional cigarettes \namong U.S. youth", vjust = 2, hjust = 0.5, size = 3.3) +
  labs(
    title = "Percentage of teens who smoked at least once in the past 30 days",
    subtitle = "Based on self-reported responses across 2009-2021 surveys",
    x = "Year",
    y = "Percent",
    color = "Age Subgroup"
  ) + 
    theme_minimal()
```

    ## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
    ## ℹ Please use `linewidth` instead.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

    ## Warning: Removed 1 row containing missing values or values outside the scale range
    ## (`geom_line()`).

    ## Warning: Removed 1 row containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="memo_files/figure-gfm/total-smoke-plot-by-age-1.png" alt="Line graph showing age-based trends in youth smoking in Androscoggin County from 2009 to 2021. The x-axis represents survey years, and the y-axis shows the percentage of teens who reported smoking at least once in the past 30 days. The graph includes five age subgroups: 14 or younger, 15, 16, 17, and 18 or older. The chart is included to illustrate how smoking rates vary by age and have changed over time. Older teens (18+) consistently had the highest smoking rates, peaking at 26% in 2009 and declining to about 10% in 2021, with a slight increase in 2017. All age groups show a general decline in smoking, with the youngest group reporting the lowest rates throughout. A vertical line in 2014 marks when e-cigarette use surpassed traditional cigarettes among U.S. youth, contextualizing a key shift in nicotine use behavior."  />

``` r
ggsave("plot1.png", bg = "white")
```

    ## Saving 7 x 5 in image

    ## Warning: Removed 1 row containing missing values or values outside the scale range
    ## (`geom_line()`).
    ## Removed 1 row containing missing values or values outside the scale range
    ## (`geom_point()`).

### Plot 2: Percentage of teens who smoked at least once in the past 30 days by Race/Ethnicity

#### Data cleanup steps specific to plot 2

Since our data is hard to work with as a whole, we filter it
specifically to include only Totals for all age groups, meaning data
points that are not separated by gender. Then, we filter it by Race and
Etnnicity.

``` r
total_race_df <- smoke_data |>
  filter(group_type == "Race/Ethnicity", gender == "total")
```

#### Final Plot 2

``` r
cleaned_df <- total_race_df %>%
  mutate(subgroup = str_replace_all(subgroup, fixed("**"), "")) |>
  filter(subgroup %in% c("American Indian or Alaskan Native", "White", "Hispanic", "Black or African American", "Asian"))

ggplot(cleaned_df, aes(x = year, y = percent, color = subgroup, group = subgroup)) +
  geom_line(size = 1) +
  geom_point(size = 2) +
  scale_color_brewer(palette = 3, 
                     type = "qual",
                     guide = guide_legend(reverse = TRUE)) +
  scale_x_continuous(breaks = c(2009, 2011, 2013, 2015, 2017, 2019, 2021)) +
  labs(
    title = "Percentage of teens who smoked at least once in the past 30 days",
    subtitle = "Based on self-reported responses across 2009-2021 surveys",
    x = "Year",
    y = "Percent",
    color = "Race/Ethnicity"
  ) + 
    theme_minimal()
```

    ## Warning: Removed 2 rows containing missing values or values outside the scale range
    ## (`geom_line()`).

    ## Warning: Removed 4 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="memo_files/figure-gfm/total-smoked-plot-race-1.png" alt="Line graph displaying trends in youth smoking rates by race and ethnicity in Androscoggin County from 2009 to 2021. The x-axis represents survey years, while the y-axis shows the percentage of teens who reported smoking at least once in the past 30 days. The chart includes separate lines for five racial and ethnic groups: White, Hispanic, Black or African American, Asian, and American Indian or Alaskan Native. The purpose of this chart is to highlight disparities in smoking behavior among different racial groups. While overall smoking rates declined over time, Hispanic and American Indian or Alaskan Native youth consistently reported higher smoking rates than their peers, signaling the need for more targeted public health interventions."  />

``` r
ggsave("plot2.png", bg = "white")
```

    ## Saving 7 x 5 in image

    ## Warning: Removed 2 rows containing missing values or values outside the scale range
    ## (`geom_line()`).
    ## Removed 4 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

### Plot 3: Youth Smoking Rates in 2009 vs 2021 by Gender and Age

#### Data cleanup steps specific to plot 3

Since our data is hard to work with as a whole, we filter it
specifically to include only Female and Male data points for all age
groups. Then, we subset it to include only years 2009 and 2021 for
easier comparison.

``` r
male_female_df <- smoke_data |>
  filter(group_type == "Age", gender != "total")

df_filtered <- male_female_df %>%
  filter(year %in% c(2009, 2021))
```

#### Final Plot 3

``` r
ggplot(df_filtered, aes(x = subgroup, y = percent, fill = factor(year))) +
  geom_col(position = position_dodge(width = 0.9), width = 0.8) +
  facet_wrap(~ gender) +
  geom_text(aes(label = paste0(percent, "%")),
            position = position_dodge(width = 0.9),
            vjust = -0.5,
            color = "black",
            size = 3.5) +
  scale_fill_manual(values = c("2009" = "#C77CFF", "2021" = "#00BFC4")) +
  labs(
    title = "Youth Smoking Rates in 2009 vs 2021 by Gender and Age",
    x = "Age Group",
    y = "Percent Who Reported Smoking",
    fill = "Year"
  ) +
  theme_minimal() +
  theme(
  legend.position = c(0.075, 0.83),
  legend.background = element_rect(fill = "lightgray", color = NA),
)
```

    ## Warning: A numeric `legend.position` argument in `theme()` was deprecated in ggplot2
    ## 3.5.0.
    ## ℹ Please use the `legend.position.inside` argument of `theme()` instead.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

    ## Warning: Removed 2 rows containing missing values or values outside the scale range
    ## (`geom_col()`).

    ## Warning: Removed 2 rows containing missing values or values outside the scale range
    ## (`geom_text()`).

<img src="memo_files/figure-gfm/smore-rate-gender-plot-1.png" alt="rouped bar chart comparing youth smoking rates in 2009 and 2021 across different age groups, separated by gender. The x-axis represents age groups (14 or younger, 15, 16, 17, and 18 or older), and the y-axis shows the percentage of teens who reported smoking at least once in the past 30 days. Purple bars represent 2009 data, and teal bars represent 2021 data. The chart is divided into two panels: female on the left and male on the right. The purpose of this visualization is to show both the overall decline in smoking over time and the persistent age and gender differences. In both 2009 and 2021, older teens reported the highest rates, with 18+ males reaching nearly 29% in 2009. By 2021, all groups saw major declines, though smoking remained more common among older male teens than younger or female groups."  />

``` r
ggsave("plot3.png", bg = "white")
```

    ## Saving 7 x 5 in image

    ## Warning: Removed 2 rows containing missing values or values outside the scale range
    ## (`geom_col()`).
    ## Removed 2 rows containing missing values or values outside the scale range
    ## (`geom_text()`).
