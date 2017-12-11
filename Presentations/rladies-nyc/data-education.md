R in the World of Education
========================================================
author: Erin Grand
date: December 12, 2017
autosize: true
css: custom.css
font-family: 'Franklin Gothic Book'
transition: linear
width: 960
height: 700

<div class="footer" style="top: 85%; left: 1%"><img src="US.png" height="100px" width="300" /></div>



About Me
========================================================
<hr></hr>
- Studied Astronomy and Physics at University of Maryland
- Data Science MS from Columbia University 2011
- Currently Associate Director of Data Analytics at Uncommon Schools

***
<hr></hr>
![](http://datascience.columbia.edu/files/seasdepts/idse/image-files/Erin_Grand_400.jpg)


What is a charter school?
========================================================
<hr></hr> 

_A charter school is an independently run public school granted greater flexibility in its operations, in return for greater accountability for performance. The "charter" establishing each school is a performance contract detailing the school's mission, program, students served, performance goals, and methods of assessment._

Uncommon Schools
========================================================
class: smaller
<hr></hr>
![uncommon-schools](http://www.uncommonschools.org/sites/default/files/datagraphics_063017.jpg)
***
<hr></hr>

- Established 1997 (oldest school opens-North Star Academy in Newark); 2005 - CMO formed
- 83% Free and reduced meal population
- Intensive PD for teachers & leaders; Partnerships with local districts to deliver PD; Summer Teaching Fellows diversity recruitment program; Camp Uncommon


What kind of data do we work with? 
========================================================
<hr></hr>

- **Assessments**: Interim assessments
- **Exams**: Common Core aligned state exams, SAT, PSAT, APs, ...etc
- **Classroom**: Grades, attendance, suspensions, ...etc
- **Teacher**: student - course - teacher linkage
- **Staff Data**: HR and Recruitment

Data Challenges
========================================================
<hr></hr>

- Missing/Incomplete data
- Different data sources without matching IDs (i.e HR to Teacher to Student)
- Movement between schools and courses of students and teachers
- Alignment of data and data processes across all schools and regions
- Changing student IDs (not many)
- Human data reporting error
- Historical data quality

========================================================
class: center-img

<img src = "data_sys.png" height = 650px>


Data Challenges We Can Fix!
========================================================
class: smaller
<hr></hr>

- Messy excel sheets (historical or human entered)
- Column names that don't apply anymore
- Lack of historical documentation
- Finding duplicates tests
- Students that take half or one test and the other half of another
- Vanishing leading zeros

***
<hr></hr>

- Tracking of students IDs that change
- Common definitions (i.e "cohort")
- How to refer to school years or school abbreviations
- Data audits


Looking at Duplicates with Janitor
========================================================
class: center-img
<hr></hr>

<small>*Janitor was built with beginning-to-intermediate R users in mind and is optimized for user-friendliness. Advanced users can already do everything covered here, but they can do it faster with janitor and save their thinking for more fun tasks.* (_Sam Firke_) </small>

<small>If you're experienced with Tidyverse in general, you should be able to do everything inside janitor on your own, but we don't have the time to always clean up data without help. </small>

![](http://media3.giphy.com/media/3oKIPCSX4UHmuS41TG/giphy-downsized.gif)


Benefits to using Janitor over writing your own code
========================================================
<hr></hr>

- Functions are tested
- Generally obeys Hadley's official style guide
- Turn many lines of code into one or two
- Pipe-able functions
- Written for the education data space

Messy Excel Sheets 
========================================================
class: center-img
<hr></hr>

<img src = "https://github.com/sfirke/janitor/raw/master/tools/readme/dirty_data.PNG" >
<small>_Image credit to Sam Firke_</small>


Using Janitor to Clean Excel 
========================================================
<hr></hr>


```r
read_excel(filepath, sheet="Sheet1", col_types = "text") %>%
  clean_names() %>%
  remove_empty_cols() %>%
  remove_empty_rows() %>%
  mutate_at(vars(entrydate, exitdate, student_id, yearsinuncommon), as.numeric) %>%
  mutate_at(vars(entrydate, exitdate), excel_numeric_to_date) 
```





Finding Duplicates
========================================================
<hr></hr>


```r
library(tidyverse)
library(janitor)
library(readxl)

students <- read_excel(filepath, sheet="Sheet1", col_types = "text") %>%
  clean_names() %>%
  remove_empty_cols() %>%
  mutate_at(vars(entrydate, exitdate, student_id, yearsinuncommon), as.numeric) %>%
  mutate_at(vars(entrydate, exitdate), excel_numeric_to_date)
```



```r
students %>% 
  get_dupes(student_id)
```

```
# A tibble: 2 x 6
  student_id dupe_count grade yearsinuncommon  entrydate   exitdate
       <dbl>      <int> <dbl>           <dbl>     <date>     <date>
1    5809913          2     6               1 2017-11-10 2017-12-10
2    5809913          2     7               1 2017-11-10 2017-12-10
```


Now What?
========================================================
<hr></hr>



- Correct the dupes individually with `if_else` or `case_when`


```r
mutate(students, grade = if_else(student_id == `r id`, `r grade`, grade))
```

- Summarize by taking minimum date / grade, if that is causing the problem


```r
group_by(students, student_id) %>% summarize(grade = min(grade))
```

- Output the duplicates and manually choose which version to keep


```r
dupes_correct <- read_csv("dupes_correct.csv")
left_join(students, dupes_correct) %>%
  replace_na(list(keep = 0)) %>%
  assert(not_na, keep) %>%
  filter(keep = 0)
```

Managing Data Changes
========================================================
<hr></hr>

Using `get_dupes` and `verify()` from the **assertr** package is a great way to put in checks in case the data changes (which it will).

```
check <- students %>% 
  get_dupes(student_id) %>% 
  verify(nrow(.) == 0)
```

If a students IDs changes, or new duplicates occur, the code will HALT at this step alerting that something is off.


Introducing R to my Team
========================================================
<hr></hr>

**Learnings Along the Way**

- Choose the packages that are needed every day
- Have someone that is active in R community, so that you can be on the cutting edge.
- The more practice someone has, the faster they'll learn. Pair PD sessions with coding projects.

Models
========================================================
class: sub-section

Automation of State Tests
========================================================
class: center-img
<hr></hr>

Entire state test analyses from raw data to dashboard is done with scripts (push button analysis)

<img src = "process.png" size = 90px>

- Gathering, cleaning and basic processing



```r
files <- list.files("../Input/", pattern = ".xlsx", full.names =  TRUE)
nys <- map_dfr(files, prep_nys_files)
```

- Combine historical and new data from all three states
- Data is outputted into Tableau dashboards

Cut Scores: Predicted Pass Rates
========================================================
<hr></hr>


```r
library(rpart)
cut_score <- rpart(profienct ~ ia_score, data = data, method="class")
# plot
# get accuracy
# get first breaking point
```

***
<hr></hr>
![](cut_score.png)

Other Model Projects
========================================================
<hr></hr>

- SGI (small group learning)
- Recruitment projections: Projected Attrition + Projected Growth
- Teacher Effect : Value added model(s)

========================================================
title: false
type: qa
<div style="position:fixed; top:50%;text-align:center;width:100%; display:block;   font-size: 150px;">
Q & A
</div>
<div class="footer">@astroeringrand</div>



