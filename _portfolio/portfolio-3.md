---
title: "Alliant Energy Data Visualizations"
excerpt: "I created two data visualizations to describe the effect a department's training events had on the company's preparation level.
<br/><img src='https://live.staticflickr.com/65535/48117195073_1b196bca09_b.jpg'>"
collection: portfolio
---
As an IT Data Warehouse Business Intelligence Intern at Alliant Energy during summer 2018, I was asked by another department to create data visualizations for them. The Continuity of Operations Department had collected data about the company's preparation level over the course of multiple training exercises and wanted me to display the data for other people in the company to see. After working with the department to get a clear understanding of what they wanted, I created two visualizations using Python.

## Visualization #1
The first visualization described the company's performance in four areas across different training events:

![pic1](https://live.staticflickr.com/65535/48117195073_1b196bca09_b.jpg)

### The code I used to create this visualization:
```python
# import packages
import matplotlib.pyplot as plt

# creating lists of performance scores for each area
op_com = [0, 1, 2.5, 3]
ppm = [0, 2, 3, 3]
op_coord = [0, 1.5, 2.5, 3]
planning = [1, 1, 2, 3]

# variable to use for the x-axis of the plot
x_axis = [1, 2, 3, 4]

# creating the red, yellow, and green shaded regions on the plot
fig, ax = plt.subplots()
ax.axhspan(0, 1, alpha = 0.2, color='red')
ax.axhspan(1, 2, alpha = 0.2, color='yellow')
ax.axhspan(2, 3, alpha = 0.2, color='green')

# plotting the four lines and adding labels for the legend
plt.plot(x_axis, op_com, label='Operational Communications')
plt.plot(x_axis, ppm, label='Physical Protective Measures')
plt.plot(x_axis, op_coord, label='Operational Coordination')
plt.plot(x_axis, planning, label = 'Planning')

# creating the legend and placing it in the bottom right of the plot
plt.legend(loc='bottom right')

# editing the x and y tick marks to make more sense
plt.xticks([1, 2, 3, 4], ['Prior to GridEx III\n(2015)', 'GridEx III\n(2016)', 'GridEx IV\n(2017)', 'Dark Sky\n(2018)'])
plt.yticks([0.5, 1.5, 2.5], ['Bad', 'Average', 'Good'])

# adding x and y labels to the plot and changing the size of the x label
plt.xlabel("Training Event", size = 13)
plt.ylabel("Preparation Level")

# adding a title and making it bigger
plt.title("Alliant Energy Preparedness Over Time", size = 14)

# getting rid of the lines on top and right of the plot - they were unnecessary
ax.spines['right'].set_visible(False)
ax.spines['top'].set_visible(False)

# display the plot
plt.show() 

# tried to save it this way, but it didn't work - took a screenshot instead
plt.savefig('Alliant Energy Perparedness Plot 1.png')
```
## Visualization #2
The second chart describes how the company performed in eight specific criteria across the same four training events.

![pic2](https://live.staticflickr.com/65535/48117209143_7a694284e9_b.jpg)

#### The code I used to create this visualization:

```python
# import packages
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

# create the data and put it into a dataframe
raw_data = {'Exercise_objective': ['Provide Voice Communication', 'Examine Coordination Issues', 'Validate Security Guard Contract', 'Observe Drone Support', 'Perform WICAMS Test', 'Introduce CONOPS', 'National Incident Management System', 'Validate Notification Process'],
            'GridEx_III': [1, 1, 0.2, 0.2, 0.2, 0.2, 1, 0.2],
            'Columbia': [3, 2, 2, 2, 1, 3, 3, 2],
            'GridEx_IV': [3, 2, 2, 3, 0.2, 0.2, 3, 2],
            'Dark_Sky': [3, 3, 1, 3, 3, 2, 2, 3]}
df = pd.DataFrame(raw_data, columns = ['Exercise_objective', 'GridEx_III', 'Columbia', 'GridEx_IV', 'Dark_Sky'])


pos = list(range(len(df['GridEx_III'])))
width = 0.15
fig, ax = plt.subplots(figsize=(10, 5))

# create the bars that go in the barplot
plt.bar(pos, df['GridEx_III'], width, label='GridEx_III')
plt.bar([p + width for p in pos], df['Columbia'], width, label='Columbia')
plt.bar([p + width*2 for p in pos], df['GridEx_IV'], width, label='GridEx_IV')
plt.bar([p + width*3 for p in pos], df['Dark_Sky'], width, label='Dark_Sky')

# set the labels
ax.set_ylabel('Performance', size=13)
ax.set_xlabel('Exercise Objective', size=13)
ax.set_title('Exercise Ratings Over Time', size=14)

# set the tick values
plt.xticks([0.08, 1.15, 2.3, 3.3, 4.25, 5.25, 6.25, 7.3], ['Provide\nVoice\nCommunication', 'Examine\nCoordination\nIssues', 'Validate\nSecurity Guard\nContract', 'Observe\nDrone\nSupport', 'Perform\nWICAMS Test', 'Introduce\nCONOPS', 'National\nIncident\nManagement\nSystem', 'Validate\nNotification\nProcess'])
plt.yticks([0.2, 1, 2, 3], ['Not Applicable', 'Bad', 'Average', 'Good'], size=12)

# add a legend
#plt.legend(['GridEx III', 'Columbia', 'GridEx IV', 'Dark Sky'], loc='right')

# getting rid of the lines on top and right of the plot - they were unnecessary
ax.spines['right'].set_visible(False)
ax.spines['top'].set_visible(False)
```
### Conclusion
This project was a good opportunity for me to implement the Python skills I was beginning to learn at the time in a real work setting. It was also a nice way to help another department explain the effect their work had on the company.
