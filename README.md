## Handing Missing Values with Claude

### 1. Getting started with new data

Download both the season_stats.csv and actual.csv.
https://drive.google.com/drive/folders/10lQNIYeLfM3LLpyBq7DCiGdHAEYBv-ZG?usp=sharing

For this exercise you will use ONLY the season_stats.csv.  The actual.csv is saved until the very end when you want to verify your work.


You'll be using Claude in VSC to help explore a new dataset and try out a few different methods of dealing with missing values.
You'd like to fill missing values in with approximate values instead of deleting the rows
or leaving them null.

Remember, start by telling Claude you want to discuss the process before any code is written.  Also, tell Claude you would like short responses - usually a paragraph is enough.  If you like you can add a stipulation at the end of every response where Claude asks, "Would you like more information?" - totally up to you.  Also, review every change Claude plans to make in your notebook.

First, provide Claude with the season_stats.csv file.  

Prompt Claude to inform you about any nulls in the data.  You can follow up
by asking if the rows containing nulls have anything in common.


### 2. Bring the data into python

 - Read season_stats.csv into python and name is 'stats'
 - Make a stats2, stats3, and stats4 df using stats.copy().

### 3. We want to replace the null values with a prediction

You may know some ways to work with null values from your previous work. The
simplest, and most common, ways to deal with null values is either to eliminate
those rows from analysis or replace the null values with an average from the
column.

We are going to ask Claude to help us replace the values in increasingly complex
ways.  While the simplest way can be done in only a few lines of code, the more complex methods
are time consuming even you are experienced working with them.

**Note:** It is import to understand how each of these methods work.  That will let you
understand when some may or may not work.  Make sure that you ask Claude the pros and cons of each approach.
In addition, it is important to understand how each method works so that you can investigate
if the output is not quite what you expect.

### 4. A simple solution - stats2

- Prompt Claude to give you pandas code to replace the
null values in the rush_yds column with the average of the non-null values in
that column.

- Alter your prompt to maintain the int datatype for the column.

- Use the code Claude provides, with minor adjustments as needed, to replace
 your null values in the stats2 dataframe with the average.

- Examine your dataframe to check that your null values have been replace by
reasonable values.

### 5. Slightly more complex - stats3

- Promt Claude to give you a python script to will allow you to replace the
null values in the rush_yds column with the team average.  (The team is in
the 'name' column). Again, be sure you prompt ensures the datatype of the
rush_yds column remains int.

- Use the code Claude provides, with minor adjustments as needed, to replace
your null values with the team average.  Do this in your stats3 dataframe.

- Examine your dataframe to check that your null values have been replaced by
reasonable values.

- Again, ask Claude the pros and cons of this approach.

### 6. Most complex - stats4

- Prompt Claude to give you a code to replace the
null values in the rush_yds column with values predicted by using the values
in the 'name', 'tm_score', 'opp_score', 'pass_yds' columns.  

- This approach will be the most complex and will likely use the scikit-learn library.

- Use the code Claude provides, with adjustments as needed, to replace your
null values with predicted values based on other values within the row.

- Examine your dataframe to check that your null values have been replaced by
reasonable values.

- Again, ask the pros and cons of this approach.

### 7. Reading in the actual data

- Read in the actual.csv and save it as 'actual' dataframe.

### 8. Testing your results

- We want to test our different methods to see which was most effective.  We
will do this by using two common tools - 'mean absolute error' and 'mean squared
error'.  
  - Mean absolute error - The average absolute error among predictions
  - Mean squared error - The error of each prediction squared (Punishes larger
  errors more)

- Prompt ChatGPT to provide you with a code that allows you get the mean
absolute error comparing the 'rush_yds' and 'actual_rush_yds' columns in
only the rows where you replaced null values.

- Use this code to get the mean absolute error value for stats2, stats3, and
stats4 dataframes.  Which method did best?  Which method was the worst?

- Now prompt Claude to provide you with code to find the mean squared error.

- Apply this code to find the mean squared error to stats2, stat3, and stats4
dataframes.  Which perfomed the best?  Which was the worst?

### 9. Reflection - Deliverable - This is to be completed and Emailed to me by next class.

- Do you think the more complex null replacement methods
always/usually/sometimes/rarely perform better than the less complex ones?

- In the most complex method we chose some columns to include and some to
exclude.  Why did we do this?  

- Would Claude be good at differentiating which columns to include?

- Would understanding relationships among columns
help in the process of choosing which to include?  Do you think there is ever
a down-side to including too many columns or do you think more data should mean
more accuracy?

- If we didn't have the actual values to test our results which ways could we
use Claude to test the accuracy?
