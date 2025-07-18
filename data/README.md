# data

Data files called 2009-2021_smoked_formatted.csv and Total_drug_weight.csv is stored within this Data folder. 

## 2009-2021_smoked_formatted.csv

- `Year`: Refers to the year when the survey took place 
- `Gender`: Refers to the gender of the person whose response is stored. Only possible options are Female, Male and Total, which represents Female and Male answers together
- `Category`: Given the format of community-provided data, this variable represents specific subsection of population we are looking at. Possible options are Age, Grade (in scool), Race/Ethnicity of a specific person answering the questionnare. 
- `Subcategory`: Refers to more specific subsection of population within chosen category. Possible answers are specific age such as "14 or younger" (Age), "12"" (grade in school), "Hispanic" (Race/Ethnicity)
- `X.`: Refers to the percentage of students who answered at least one day to the posed question. In this specific case, the question was: "During the past 30 days, on how many days did you smoke cigarettes?" 
- `Confidence.Interval`: Refers to the 95% Confidence Interval, which is the values within which the true smoking rate in the chosenpopulation is likely to fall.
- `Number.of.Students`: Refers to the number of students who answered at least 1 day to the posed question (weighted).

Below is a quick overview of the data using `dim()` and `glimpse()`:

Rows: 205 Columns: 7

glimpse()

$ Year                <int> 2009, 2009, 2009, 2009, 2009, 2009, 2009, 2009, 2009, 2009, 2009, 2…

$ Gender              <chr> "Female", "Male", "Female", "Male", "Female", "Male", "Female", "Ma…

$ Category            <chr> "Total", "Total", "Age", "Age", "Age", "Age", "Age", "Age", "Age", …

$ Subcategory         <chr> "-", "-", "14 or younger", "14 or younger", "15", "15", "16", "16",…

$ X.                  <chr> "16.20%", "13.30%", "10.80%", "9.20%", "11.20%", "9.20%", "14.80%",…

$ Confidence.Interval <chr> "(12.9%-19.6%)", "(10.8%-15.8%)", "(8.8%-12.8%)", "(6.7%-11.8%)", "…

$ Number.of.Students  <chr> "792", "306", "62", "25", "135", "52", "184", "79", "260", "93", "1…


## Total_drug_weight.csv 

- `Year`: Refers to the year when the data was collected

- `April`: Represents total lbs. of medications collected during Free Drug Take Back Events	in April

- `October`: Represents total lbs. of medications collected during Free Drug Take Back Events	in October

- `Total`: Total lbs. of medications collected during Free Drug Take Back Events		

Below is a quick overview of the data using `dim()` and `glimpse()`:

Rows: 16
Columns: 4

$ Year    <chr> "2011", "2012", "2013", "2014", "2015", "2016", "2017", "2018", "2019", "2019…

$ April   <chr> "1110", "1890", "1527", "2301", "1695", "1044", "1108", "1043", "1020", "0", …

$ October <chr> "1547", "1437", "1465", "991", "1500", "0", "1300", "841", "3064", "3064", "3…

$ Total   <chr> "2657", "3327", "2992", "3292", "3195", "1044", "2408", "1884", "4084", "3064…