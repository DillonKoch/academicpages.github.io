---
title: "News Web Scraper"
excerpt: "This is a Python program I wrote to automatically email me news I'm interested in every day. <br/><img src='/images/500x300.png'>"
collection: portfolio
---
After working on a project as a Data Science intern at Collins Aerospace in which I scraped the web for security-related news about the company and its products, I wanted to apply what I learned in another setting. I thought one useful way to use web scraping in Python would be to write a program that automatically emails me with news I'm interested in. 

To do this, I wrote functions for each source I wanted to get news from. Each function inspects the site's HTML code to extract headlines and links, then puts them in a brief message that will be emailed to me. For example, here's the function I used to get news from TechCrunch:

**First I needed to import some packages:**
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

This code will use the TechCrunch URL to extract the site's HTML, encode it properly, and store it in the *_sp_* variable:
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
The next step was to create a list that includes all the headlines and links. To do this, I'll use the _*limit*_ argument this function takes, which specifies how many articles you'd like to be returned. The for loop below uses this to extract the headline text, the article link, and insert them as a tuple into _*bothList*_. 
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
Once we have all the headlines and links, it's time to put them in a message that will later be inserted into an email. To do this, I begin by creating an emtpy string named *_message_*. After that, articles can be inserted in one of two ways:
1. If a list of keywords was given to the second argument to this function, _*keywords*_, then only the articles whose headlines have one of the words/phrases from the *_keywords_* list will be included.
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
Instead of only taking the _*limit*_ and _*keywords*_ arguments, this function also takes one called _*content*_. This allows me to specify which type of content from the site I'd like to be returned. The options I included are:
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
Finally, I use a function that combines these news-related functions together to create one big message that I send in an email to myself. The email is automatically sent from a gmail account I created for this purpose to my other email account. Running the program takes about 10 seconds in total, and is a very convenient way for me to get the news I want every day!

If you'd like to be sent one of these emails just to see what they look like (or if you want them daily) let me know at

dillon-koch@uiowa.edu and I'll send you one!
