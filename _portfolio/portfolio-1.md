---
title: "Portfolio item number 2"
excerpt: "Short description of portfolio item number 2 <br/><img src='/images/500x300.png'>"
collection: portfolio
---

Part of my job as a Data Management and Analysis Student Assistant at the Academic Support & Retention department at the University of Iowa deals with the Excelling@Iowa survey. This survey is given each semester to freshmen and new transfer students. It asks them 89 questions about their experience at the University of Iowa including their academics, study habits, social experiences, financial situation, future plans, and more. 

People and departments throughout the university often want to see how certain groups of students' responses to the survey differ. For example, the college of business may ask for a report explaining business students' responses to the survey and how they differ from non-business students. To create this report, I used to use Excel to create pivot tables and charts by hand:
![break](https://live.staticflickr.com/65535/47943619652_d7b19ded95_o.png)
![pic 1](https://live.staticflickr.com/65535/47943425576_40fe968ff1_b.jpg)
![pic 2](https://live.staticflickr.com/65535/47943457771_80e884fbcd_o.png)
![pic 3](https://live.staticflickr.com/65535/47943431182_dd66b09873_o.png)
![break](https://live.staticflickr.com/65535/47943619652_d7b19ded95_o.png)
![pic 4](https://live.staticflickr.com/65535/47943474541_25cf540244_b.jpg)
![pic 5](https://live.staticflickr.com/65535/47943489693_a1ed863571_b.jpg)
![break](https://live.staticflickr.com/65535/47943619652_d7b19ded95_o.png)
## Code Sample:
#### Continue reading if you're interested in some of the Python code that I used in this function for each of the steps above.
**Step 1: Import Survey Data**

```python
# import packages
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import math

# import data
All = pd.read_csv('ALL SURVEYS Fall 16 - Spring 19.csv')
```
Now that all the survey data from Fall 2016 through Spring 2019 is imported, I'll separate the data into the two populations being compared in the report...

**Step 2: Separate Data into Two Populations**
As an example, I'll compare the Fall 2017 survey results to the Fall 2018 results.

```python
# df17 incldues all survey data from Fall 2017
df17 = All[All['Survey'] == 'Fall 2017']

# df18 includes all data from Fall 2018
df18 = All[All['Survey'] == 'Fall 2018']
```

**Steps 3 - 4: Create charts for each group's average responses**
To calculate the average response to each question, I created a Python class named 'Chart'. 
```python
class Chart:
  """Creating Charts for the annual Excelling@Iowa Survey"""
  
  def __init__(self, Pop1, Pop2, Pop1_name, Pop2_name):
    self.Pop1 = Pop1
    self.Pop2 = Pop2
    self.Pop1_name = Pop1_name
    self.Pop2_name = Pop2_name
```
Each instance of the method is created with its name, self, the two datasets including the students' survey responses, Pop1 and Pop2 (df17 and df18 in this example), and two strings representing the name for each group in the final charts.

The class also includes 23 methods. Methods 1-22 each create and display one chart visualizing responses from the survey, and the final method displays all charts at once.

#### Here's how the code for the first method works:

Configuring the plot parameters:
```python
  def plot1(self):
    # setting the parameters for the plot
    dim = (10, 5)
    fig, ax = plt.subplots(figsize=dim)
```
Inserting the chart title and questions:
```python
    # Question labels and title
    Title1 = 'Belonging and Fit'
    Q0010 = 'I spend time with people\nthat I like'
    Q0020 = 'I spend time with\npeople that have similar\ninterests as me'
    Q0030 = 'I feel like I belong at\nthe University of Iowa'
    Q0040 = 'I am satisfied with my\nsocial life at the\nUniversity of Iowa'
```
Creating two smaller dataframes that only include the survey questions for this chart, and combining them into one:
```python
    # this will create Pop1_df, the data from Fall 17
    x = self.Pop1.iloc[:,17:].mean(axis=0, skipna = True)
    Pop1_df = pd.DataFrame(x)
    Pop1_df.columns = ['Average Response']
    Pop1_df['Pop'] = [self.Pop1_name]*89
    
    # this will create Pop2_df, the data from Fall 18
    y = self.Pop2.iloc[:,17:].mean(axis=0, skipna = True)
    Pop2_df = pd.DataFrame(y)
    Pop2_df.columns = ['Average Response']
    Pop2_df['Pop'] = [self.Pop2_name]*89
    
    # combine the two datasets into one:
    data = pd.concat([Pop1_df, Pop2_df], axis=0)
    part1 = data.iloc[0:4,:]
    part2 = data.iloc[89:93,:]
    final = pd.concat([part1, part2])
    final['Question'] = [Q0010, Q0020, Q0030, Q0040]*2
    final = final.round(decimals=2)
```
Using the dataframe I just created to plot the results:
```python
    # create the plot itself:
    ax1 = sns.barplot(x="Question", y="Average Response",
                      hue="Pop", data = final)
    plt.legend(bbox_to_anchor=(1.05, 0.5), loc=2, borderaxespad=0.)
    ax1.set_ylim([0, 7])
    for p in ax1.patches:
        ax1.text(p.get_x() + p.get_width()/2., p.get_height(), p.get_height(), 
                fontsize=11, color='black', ha='center', va='bottom', weight='bold')
    plt.title(Title1, fontsize=20, weight='bold')
    plt.xlabel(' ')
    plt.ylabel('Average Response')
```
And finally, here's the output of calling this method:
```python
# create the instance:
Compare_17_18 = Chart(df17, df18, "2017 Freshmen", "2018 Freshmen")

# call the .plot1() method:
Compare_17_18.plot1()
```
![plot1](https://live.staticflickr.com/65535/48026562768_ffa570f8f4_o.png)

The other methods are all very similar, they just use different questions to create the charts.

### Conclusion
This was a cool project for me to implement some of the Python skills I've been learning and save myself a lot of time at work. It used to take about 5-6 hours of monotonous work to create those 22 charts by hand, but now it takes at most 10 minutes to use this program from start to finish!

