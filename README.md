# Data Analyst Profile

Email: dougmalcolm87@gmail.com

LinkedIn:

### Education

University of Maryland, Baltimore County | *May 2024*

Bachelor of Science: Mathematics

### Skills

- SQL, Excel, PowerBI, Tableau, SAS
- Python (Pandas, Numpy, Matplotlib, Scikit-Learn)
- Statistical Testing (Multiple Regression Analysis, ANOVA, Hypothesis Testing, Chi-Square GoF)

---

# Project 1

## Decoding Pinball Wizardry: A Data-Driven Analysis

<img align="right" src="/assets/edited_wizard.jpeg" width="150" height="200">

### Overview

_Tools Used: **Python, Excel, PowerBI, Regression Analysis**_

What makes a pinball player *great*? 

This analysis aims to uncover the key factors that drive the success of top pinball tournament players.

In this project I... 
- Developed a **Python web scraper**, collecting 731,862 data from the IFPA pinball tournament player website
- Performed **data cleaning and preprocessing** using Pandas to prepare the dataset for analysis
- Created **data visualizations** and descriptive statistics in PowerBI and Excel to grasp data distributions
- Applied **machine learning** techniques with scikit-learn to construct linear regression models
- Analyzed regression results to **derive meaningful insights** about pinball tournament success factors 


### Background

Pinball is a mix of skill and luck. Being able to accurately hit the shots you want, regain ball control after wild trajectories, and nudging the machine to affect the ball path are a couple examples of skills that can extend an otherwise short game. At the same time though, one player may immediately drain the ball after missing a shot whereas another player may miss ten shots before the game decides their ball is done. Playing sufficiently many games is where the skill differences shine through though - the bad and good luck balances out and the better player tends to come out on top. This is why players like Jason Zahler and Escher Lefkoff are able to consistently get 1st place at huge 100+ player tournaments; There are enough tournament rounds for their superior pinball skills to shine through the luck. 

If you are curious about how to play pinball skillfully and what that actually looks like, this youtube playlist from Abe Flips is by far the best resource for learning:

<a href="http://www.youtube.com/watch?feature=player_embedded&v=ch7H19SHkXE&list=PL31W94V2HVSW7ksDyZ_183rUBxiSQon75" target="_blank"> <img src="/assets/vid_pic_new.png" width="355" height="200" border="1" /></a>

Personally, I have been playing in tournaments for 7 years now, and I absoultely love both pinball and thrill of competition. So naturally I was curious... what makes a pinball player great? 

### The Data
The International Flipper Pinball Association [(IFPA)](https://www.ifpapinball.com/) is *the* organization that oversees and tracks competitive pinball. It has a webpage for each of its players showing information like name, age, years active, tournament finishes, and rating. Since the IFPA lacks a robust API, I decided to develop my own web scraper in Python to extract all of the relevant player data.

Here is an example player profile:

<img src="/assets/sample_profile.png">

For each tournament player in the IFPA, the following data is collected...

Data to Collect | Description
--- | ---
Name | Player Name
Age | Player Age
Age Started | Age at time of first tournament; found by (Age - Years Active)
Years Active | Number of years playing in tournaments					
Total Events | Total number of tournaments					
Rating | Measure of pinball skill; Glicko Rating System					
Ranking | Measure of pinball skill and dedication; Compares best 15 tournament results

Both Rating and Ranking are measures of pinball skill. Rating will increase from strong tournament finishes and will decrease from weak tournament finishes. Ranking sums your top 15 best tournament results and compares them against everyone else's top 15 best tournament results. As a result, Ranking does not decrease from weak tournament finishes. This ends up inflating the Ranking of players who play in many many tournaments compared to those who player in fewer tournaments. Consequently, I personally subjectively believe that **Rating** is a better measure of pinball skill rather than ranking. Therefore moving forward, we will use Rating as the dependent variable of interest. 

#### Python Web Scraper

```Python
from bs4 import BeautifulSoup
import requests
import lxml

# Retrives html from IFPA website
html_text = requests.get('https://www.ifpapinball.com/player.php?p=' + str(i + 1)).text
soup = BeautifulSoup(html_text, 'lxml')

# Finds keywords I am looking for
total_events = str(soup.find(string="Total Events").findNext("td").text)

# Writes this text into csv file 
with open('Local File Path', 'a') as f:
    f.write(total_events + "\n")
```

This sample code **extracts and enables us to use the IFPA player data** by using BeatifulSoup, Requests, and lxml libraries to access and parse the website's html. After finding the data we want in the html, we record the data into a cvs file for ease of use. Looping this gave **111,888** rows of data, specifically collecting name, age, years active, total events, rating, and ranking for each competitive pinball player.

#### Exploring the Data

To get a quick grasp of our data, we can look at the distributions and descriptive statistics for each variable we extracted.

<img src="/assets/distributions.PNG">

| Variable | Sample Size | Mean | Median | Std. Deviation | Minimum | Maximum |
| --- | --- | --- | --- | --- | --- | --- |
Age | 10,506 | 42.1 | 43 | 11.9 | 6 | 85 |
Age Started | 10,510 | 37.9 | 38 | 11.6 | 5 | 83 |
Years Active | 116,468 | 1.3 | 0 | 2.8 | 0 | 42
Total Events | 116,308 | 12.5 | 2 | 39.6 | 1 | 1113
Rating | 32,538 | 1174 | 1162 | 192 | 437 | 2149
Ranking | 49,112 | 23879 | 23325 | 14326 | 1 | 51013

Takeaways:

- A whopping **95.36%** of all IFPA players have only played in 1 tournament.
- The average pinball tournament player is **42.1** years old, and has played for **1.3** years in **12.5** tournaments.
- 95% of pinball players with 5+ total events have ratings between 790 and 1558, with an average of **1174** representing 'average skill level'.

### The Model

Now let's build a model to determine which factors lead to pinball greatness, and to what degree. 

#### Building the Model

So, which factors have a statistically significant effect on rating? 

Rating is not very accurate with only a couple of tournament performance data points. So when looking at rating analysis going forward, we will filter for only those players with 5+ Total Events played. This should give us a large enough sample size of tournament performances to capture an decently accurate rating for each player.

Let's use the machine learning sklearn library to build the model:

```Python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression

features = ['Age', 'Age Started', 'Years Active', 'Total Events']
target = 'Rating'
X = cleaned_data[features]
y = cleaned_data[target]

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=101)
Model1 = LinearRegression()
Model1.fit(X_train, y_train)
```

#### Results

After ensuring independent residuals that are normal and homoscedastic about 0, removing insignificant explanatory variables, and checking for low multicollinearity through VIF scores, we arrive at the final model.

<img src="/assets/regression.PNG">

---

Final Model:

**Rating = 1250 + 2(Years Active) + 0.953(Total Events) - 1(Age Started)**

---

We can also look directly at how each of these three variables are correlated with rating:

<img src="/assets/total_events.PNG">

<img src="/assets/years_active.PNG">

<img src="/assets/age_started.PNG">

### Conclusion

Firstly, it is important to acknowledge that filtering for only players with an Age and 5+ Total Events reduced the sample size dramatically. While a smaller n value doesn't invalidate the results, it certainly doesn't help the final model. 

The final model lacks predictive value. With an R^2 value of 0.276 and such small effects from each explanatory varible, there is simply too much unexplained variance in our model. Age Started, Total Events, and Years Active do affect rating to a statistically significant, not-by-chance degree. The effects are just simply so small it would be silly to try to accurately predict rating from these three metrics alone.

**While Age Started, Total Events, and Years Active have an undeniable correlation with skill level, they don't come close to capturing the full picture of what makes a great pinball player.** 

I suspect what truly leads to pinball greatness is not so easily quantifiable. So I want to hear from you; What do you think makes a great pinball player? If you have experience playing pinball, please fill out this [Google Forms survey](https://forms.gle/APDzK8DGoZY6wsNf9) and share your opinion. With enough responses I may be able to perform a qualitative analysis.  
