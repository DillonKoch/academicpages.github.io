---
title: "News Web Scraper"
excerpt: "This is a Python program I wrote to automatically email me news I'm interested in every day.
<br/><img src='https://live.staticflickr.com/65535/48076231618_df2667facf_b.jpg'>"
collection: portfolio
---
After working on a project as a Data Science intern at Collins Aerospace in which I scraped the web for product security-related news about the company and its products, I wanted to apply what I learned in another setting. I thought one useful way to use web scraping in Python would be to write a program that automatically emails me a PDF every day with news and other useful information I'm interested in.

### What's Included
- Next events on my Google Calendar
- Weather in my area
- My recent Fitbit activity
- News articles

### Google Calendar Events
To show the next events I have coming up, I used the Google Calendar API in Python to pull data about my next 10 events. I won't describe all the setup steps for this, but essentially I was able to get information about my next events like start/end time, location, and the name of the event. I used that data to insert a column like this in the output PDF:
![Calendar](https://live.staticflickr.com/65535/48667483093_40d93aa157_o.png)

### Local Weather
Finding local weather data was fairly straightforward. I used https://weather.com/ to find the current weather in my area (either Cedar Rapids, IA or Iowa City, IA). Here's how I scraped this information from python:

```python
# this is a method used in my DailyPDF class, initialized with my current location
def get_weather(self):
    link = self.weather_loc_dict[self.location]  # using the link for my current location
    
    sp1 = self.getsp1(link)     # using BeautifulSoup to get the HTML of the website
    
    # temperature
    temp = sp1.find_all('div', attrs={'class':'today_nowcard-temp'})
    temp = temp[0].get_text()
    
    # description
    phrase = sp1.find_all('div', attrs={'class':'today_nowcard-phrase'})
    phrase = phrase[0].get_text()
    
    # feels
    feels = sp1.find_all('span', attrs={'class': 'btn-text'})
    feels = feels[0].get_text()
    
    feels_num = sp1.find_all('span', attrs={'class':'deg-feels', 'classname':'deg-feels'})
    feels_num = feels_num[0].get_text()
    
    # high/low
    hilo = sp1.find_all('span', attrs={'class':'deg-hilo-nowcard'})
    hi = hilo[0].get_text()
    lo = hilo[1].get_text()
    hilo_msg = 'Low: {} / High: {}'.format(lo, hi)
    
    return temp, phrase, feels, feels_num, hilo_msg
```

The result of this method in my final PDF looks like this:

![Weather](https://live.staticflickr.com/65535/48667521168_165ec0e16f_o.png)

### Fitbit Data
I used the Python Fitbit API to access my recent Fitbit data, which also had some setup I won't go into. There were three charts I created with this data to show my step totals, sleep, and heart rate in the past week. These are the three methods of my FitbitData class I use to produce the charts I insert to the output PDF:
##### Step Totals
```python
    def plot_steps(self):
        # step totals the past week, and the days they occurred on
        step_list = [self.client.activities(day)['summary']['steps'] for day in self.past_7_days]
        day_list = [str(day.month) + '-' + str(day.day) for day in self.past_7_days]
            
        df = pd.DataFrame({'steps':step_list,
                           'day':day_list})
        # plotting the data
        ax = sns.relplot(x='day', y='steps', kind='line', data=df)
        y_min = max(min(step_list)-1000, 0)
        y_max = max(11000, max(step_list))
        plt.ylim([y_min, y_max])  
        plt.title('Step Totals in the Past Week', fontsize='x-large')
        plt.xticks(size='large')
        plt.ylabel('Steps', fontsize='x-large')
        plt.xlabel('')
        plt.hlines(10000,day_list[0], day_list[-1], linestyles='dashed', label='Goal', colors='grey')
        plt.annotate('10k Step Goal', (day_list[-1], 9900))
        plt.savefig('steptotals.png', bbox_inches='tight')
        plt.clf()
```
![step plot](https://live.staticflickr.com/65535/48668086257_1f6928c3a7_o.png)

##### Sleep Data
```python
    def plot_sleep(self):
        deepL = []          # empty lists to hold data for each type of sleep
        lightL = []
        remL = []
        wakeL = []
        dayL = []
        # finding data for each day in the past week
        for i in [6, 5, 4, 3, 2, 1, 0]:
            day = date.today() - datetime.timedelta(days=i)
            dayL.append(day)
            stage_dict = self.client.sleep(day)['summary']['stages']
            deepL.append(stage_dict['deep'])
            lightL.append(stage_dict['light'])
            remL.append(stage_dict['rem'])
            wakeL.append(stage_dict['wake'])
       
        # Creating the Chart
        plt.rcParams['axes.spines.right'] = False
        plt.rcParams['axes.spines.top'] = False
        r = list(range(7))
        
        p1 = plt.bar(r, deepL, label='Deep', color=(0/255, 0/255, 204/255, 1))
        p2 = plt.bar(r, remL, bottom=deepL, label='REM', color=(30/255, 144/255, 255/255, 1))
        two = np.add(deepL, remL).tolist()
        p3 = plt.bar(r, lightL, bottom=two, label='Light', color=(135/255, 206/255, 250/255, 1))
        three = np.add(two, lightL).tolist()
        p4 = plt.bar(r, wakeL, bottom=three, label='Awake', color=(240/255, 128/255, 128/255, 1))
        
        plt.legend([p4, p3, p2, p1], ['Awake', 'Light', 'REM', 'Deep'], loc='center left',
                    bbox_to_anchor=(1, 0.5))
        plt.annotate('8-hour sleep goal', (6.6, 470))
        plt.hlines(480, -0.5, 6.5, linestyle='dashed', colors='grey')
        plt.yticks([0, 60, 120, 180, 240, 300, 360, 420, 480], [0, 1, 2, 3, 4, 5, 6, 7, 8])
        plt.ylabel('Hours of Sleep', fontsize='large')
        plt.title("Last Week's Sleep Totals", fontsize='x-large')
        
        day_labels = [str(d.month) + '-' + str(d.day) for d in self.past_7_days]
        plt.xticks(list(range(7)), day_labels)
        
        plt.savefig('sleeptotals.png', bbox_inches='tight')
        plt.show()
        plt.clf()
```
![sleep plot](https://live.staticflickr.com/65535/48668139212_20e8826f7a_o.png)

##### Heart Rate
```python
    def plot_HR(self):
        # getting heart rates from the Fitbit API
        rates = [self.client.activities(day)['summary']['restingHeartRate'] for day in self.past_7_days]
        # axis tick marks
        yticks = list(range(min(rates), max(rates)+1))
        xticks = [str(day.month) + '-' + str(day.day) for day in self.past_7_days]
        # plotting
        plt.plot(rates)
        plt.yticks(yticks, yticks)
        plt.xticks(list(range(7)), xticks)
        plt.title('Resting Heart Rate Last Week', fontsize='large')
        plt.ylabel('Beats per Minute')
        plt.savefig('restingHR.png', bbox_inches='tight')
        plt.clf()
```
![heart rate plot](https://live.staticflickr.com/65535/48668013106_8386283737_o.png)
## News Articles

To find news articles I'm interested in, I wrote functions for each source I wanted to get news from. Each function inspects the site's HTML code to extract headlines and links, then puts them in a brief message that will be emailed to me. For example, here's the function I used to get news from TechCrunch:

**Import packages:**
```python
# import packages 
from bs4 import BeautifulSoup as soup
from googlesearch import search
import pandas as pd
import urllib.request
import textwrap
import smtplib, ssl
from datetime import date
```
**Then I began writing the function for TechCrunch**

This code will use the TechCrunch URL to extract the site's HTML, encode it properly, and store it in the *sp* variable:
```python
# Scraping news from TechCrunch:
def TCheadlines(limit=3, keywords=[]):
  fp = urllib.request.urlopen('https://techcrunch.com/')
  mybytes = fp.read()
  mystr = mybytes.decode("utf8")
  fp.close()
  sp = soup(mystr, 'html.parser')
```
After that, I found all the headlines by searching the HTML for the 'a' tag and the "post-block__title__link" class. This is just the pattern TechCrunch uses to post headlines on their website, so I can easily find them by looking for that particular tag-class combination. The way I found this was just by right-clicking on the site and pressing 'inspect' to look through the site's HTML (you can do this for any website).

```python
  # finding headlines
  headlines = sp.find_all('a', attrs={'class':'post-block__title__link'}, href=True)
```
The next step was to create a list that includes all the headlines and links. To do this, I'll use the *limit* argument this function takes, which specifies how many articles you'd like to be returned. The for loop below uses this to extract the headline text, the article link, and insert them as a tuple into *bothList*. 
```python
  # adding headlines and links to bothList
  bothList = []
  for i in range(limit+1):     # since indexing starts at 0, I'll add 1 to limit
    newH = headlines[i].get_text()
    newH = newH.replace('  ', ' ').replace('\t', '').replace('\\', ' ').replace('\n', '')
    newL = str(headlines[i]).split('"')[3]
    # now, newH has the text headline for each story, newL has the link to the story
    bothList.append((newH, newL))
```
Once we have all the headlines and links, it's time to put them in a message that will later be inserted into an email. To do this, I begin by creating an emtpy string named *message*. After that, articles can be inserted in one of two ways:
1. If a list of keywords was given to the second argument to this function, *keywords*, then only the articles whose headlines have one of the words/phrases from the *keywords* list will be included.
2. Otherwise, the function will just print out the headline and link for the number of articles you ask for.
```python
  # putting all the headlines and links into a final message
  message = ''
  if len(keywords) == 0:    
    for pair in bothList:
      message += str(pair[0]) + '\n' + str(pair[1]) + '\n\n'
      #print(pair, '\n')
  else:
    for word in keywords:
      for pair in bothList:
        if word in pair[0]:
          message += str(pair[0]) + '\n' + str(pair[1]) + '\n\n'
          #print(pair)
      
  return message
  ```
Here's an example of this function being used:
```python
print(TCheadlines(limit=10, keywords = ['Tesla', 'Silicon Valley']))
```
![TechCrunch 3](https://live.staticflickr.com/65535/48076231758_cd544d6523_b.jpg)
Since there were only three articles that included the keywords I asked for in their headlines, only three were returned. Had I not included any keywords, 10 articles would be returned.

### Functions for The Verge and The Ringer
Two other sites I like to read are The Verge and The Ringer, so I included those sites too. The functions for those sites are slightly different in that these sites have multiple categories of news I'd like to monitor as well as the main page.

This difference appears at the beginning of the functions for these sites. Let's look at my function for The Verge:
```python
def vergeHeadlines(content=None, limit=5, keywords=[]):
  link = 'https://www.theverge.com'
  
  if content == 'AI':
    link += '/ai-artificial-intelligence'
  elif content == 'Facebook':
    link += '/facebook'
  elif content == 'Amazon':
    link += '/amazon'
  elif content == 'Google':
    link += '/google'
  elif content == 'Tesla':
    link += '/tesla'
  elif content == 'tech':
    link += '/tech'
```
Instead of only taking the *limit* and *keywords* arguments, this function also takes one called *content*. This allows me to specify which type of content from the site I'd like to be returned. The options I included are:
- Artificial Intelligence
- Facebook
- Amazon
- Google
- Tesla
- Tech (in general)

If I specify one of these types, then the function will add on to the base URL with that section of the website and extract articles there. If not, then the program will get articles from the main page. The rest of the function operates the same as the TechCrunch function, only modified for the HTML differences between the sites.

Here's an example of this function being used:
```python
print(vergeHeadlines(content='AI', limit=5, keywords=[]))
```
![AI 5](https://live.staticflickr.com/65535/48076231618_df2667facf_b.jpg)
My function for the Ringer works the same way as The Verge, inspecting the NFL, NBA, Tech, and Politics sections of that website.

## End Result
Finally, I use a function that combines these news-related functions together to create one big message that I send in an email to myself. The email is automatically sent via Python code from a gmail account I created for this purpose to my other email account. Running the program takes about 10 seconds in total, and is a very convenient way for me to get the news I want every day!

If you'd like to be sent one of these emails just to see what they look like (or if you want them daily) let me know at
dillon-koch@uiowa.edu and I'll send you one!
