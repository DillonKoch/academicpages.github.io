---
title: "Lock It In Sports Betting Analysis - Season 1 (IN PROGRESS)"
excerpt: "Data I collected and analyzed about the TV show Lock It In
<br/><img src='https://live.staticflickr.com/65535/48440110437_e93350f32b_o.jpg'>"
collection: portfolio
---

Lock It In is a TV show on Fox Sports 1 focused on sports betting. The show is hosted by Rachel Bonetta and also includes analysts Todd Fuhrman, Clay Travis, and Cousin Sal. Each week the three analysts are given $1,000 to bet throughout the week, and they usually make 4 bets per day (20 per week). Whoever has the most money at the end of the week wins.

I became interested in sports betting after it became federally legal in May 2018, and especially now since it's legal in Iowa as of August 2019. It's a great mix of sports and data, it makes watching games more interesting, and if you do well you can make money! 

I also find it more interesting than fantasy sports because sports betting looks at the games through mostly the same lens as the players and coaches. For example, Tom Brady and Bill Belichick would be glad to win a game 13-3, as would people who bet on the Patriots. On the other hand, someone who had Tom Brady in fantasy football would probably be disappointed that he didn't score more touchdowns.

This project began when I had a week off between school and my summer internship, so I watched Lock It In on TV and got the idea to begin collecting and analyzing data from the show. Every bet made on the show is displayed like image below, which made this process consistent and straightforward.

(pic)

### Terminology
There are four types of bets that I'll keep track of:
* __Spread Bets:__ bets made on the outcome of a game plus or minus the spread. If the spread is Warriors -3.5, then the Warriors need to win the game by at least 4 points to win the bet.
* __Moneyline Bets:__ bets made purely on the outcome of the game. Betting the favorite will result in a lower payout than a spread bet if they win, while betting an underdog will pay out more money than a spread bet.
* __Moneyline Notation:__ Non-spread bets are usually given with a moneyline indicating how likely the bet is to cash. Positive moneylines describe how much money you would win if you bet $100, and negative moneylines tell you how much you need to bet to win $100. For instance, betting $100 on a moneyline of +200 would pay out $200 if you win. Betting $300 on a moneyline of -300 would win you $100.
* __Prop Bets:__ bets made on aspects of the game other than the final outcome. This can be a wide range of bets - a player's point total, whether a team scores over/under a number of points, a player to win MVP, and so on.
* __Parlay Bets:__ A parlay bet is a bet that includes multiple bets from the first three categories. Every bet in the parlay must win for the bettor to win the parlay. If you bet a parlay made up of 5 individual bets, you won't make any money unless all 5 bets are successful.

### Data Collection
To collect this data, I created a Python class that would create a dataframe to store data about each analyst's bets. The class is created with the bettor's name and job, and will create empty dataframes to hold bet data:
```python
class Bettor:
  def __init__(self, name, job):
      self.name = name
      self.job = job
      self.numBets = 0
  
      self.allBets = pd.DataFrame(columns = ['ID', 'Bet No.', 'Bet Type', 'Bet Date', 'Gameday',
                                             'Bet', 'To Win', 'Home Team', 'Away Team',
                                             'Pick', 'Spread', 'Money Line', 'League', 'Outcome'])
```
After creating the constructor, I also wrote methods to populate the dataframe based on the type of bet being made. There is one method for each type of bet (spread, money line, prop, parlay) since they each include slightly different data. Every type of bet is also added to the "all bets" dataframe as well.

For example, here's the method I used to add a spread bet (e.g. Warriors -3 vs Raptors):
```python
  def betSpread(self, amount, toWin, home, away, pick, spread, league, betDate, gameDate):
      ID = self.numBets + 1
      newRow = [ID, np.nan, 'Spread', betDate, gameDate, amount, toWin, home, away, pick, 
                spread, np.nan, league, np.nan]
      self.allBets.loc[len(self.allBets)] = newRow
      self.numBets += 1
```
After using this spread method and other similar ones, the data will look like this:
![pic](https://live.staticflickr.com/65535/48440711922_b93871136b_b.jpg)
You may notice some missing values since I'm displaying the "all bets" dataframe above. This is because some bets have different features than others. For example, only spread bets have a value in the spread column, not all prop bets have a home and away team, only parlay bets have a "Bet No." to keep track of all the bets in the parlay, and so on. Using the bet type-specific dataframes will show only the relevant data to that particular bet type.

To display one type of bet at a time, I also created methods with properties like this:
```python
    @property
    def spreadBets(self):
        df = self.allBets[self.allBets['Bet No.'].isnull()]
        df = df[df['Bet Type'] == 'Spread']
        df = df.iloc[:,[0,2,3,4,5,6,7,8,9,10,12,13]]
        return df
        
    @property
    def moneyLineBets(self):
        df = self.allBets[self.allBets['Bet No.'].isnull()]
        df = df[df['Bet Type'] == 'Money Line']
        df = df.iloc[:,[0,2,3,4,5,6,7,8,9,11,12,13]]
        return df
    
    @property
    def propBets(self):
        df = self.allBets[self.allBets['Bet No.'].isnull()]
        df = df[df['Bet Type'] == 'Prop']
        df = df.iloc[:,[0,2,3,4,5,6,7,8,9,11,12,13]]
        return df
    
    @property
    def parlayBets(self):
        df = self.allBets[self.allBets['Bet No.'].notnull()]
        return df
```
These methods will query each type of bet from the "all bets" dataframe and return a new dataframe with only the relevant features for that bet type.
#### Sample Usage
To use this class, I first created an instance for each analyst on the show:
```python
Sal = Bettor('Cousin Sal', 'Writer, Comedian, Podcast Host')
Todd = Bettor('Todd Fuhrman', 'Oddsmaker, TV Analyst, Podcast Host')
Clay = Bettor('Clay Travis', 'Writer, Radio/Podcast Host')
``` 
After that all I need to do is begin adding bets each person makes using the appropriate method:
```python
Clay.betProp(50, 600, "Rory McIlory wins PGA Championship", 1200, "PGA", d(5, 13), d(5, 19))

Todd.betParlay(spread(25, 43, "Sharks", "Blues", "Blues", 1.5, 'NHL', d(5, 13), d(5, 13)), 
             moneyLine(25, 43, "Hurricanes", "Bruins", "Hurricanes", -110, 'NHL', d(5, 13), d(5, 13)))
             
Sal.betSpread(50, 130, 'Raptors', 'Bucks', 'Bucks', -9.5, 'NBA', d(5, 21), d(5, 21))

```
After collecting hundreds of bets like this from season 1 of the show, I was able to view all bets each bettor had made with the "allBets" member variable (e.g. Todd.allBets). Then I saved each bettor's data and started analyzing it.
### Feature Engineering
Before using the data, I wanted to create a feature that represented the total earnings for each bettor over time. Each bettor's earnings would begin at $0, then increase or decrease over time based on their bet outcomes. That way I can see how each bettor won or lost money over time, and see how much money everyone ended with.

To create this feature, I wrote a function that could be applied to each bettor's data:
```python
def get_earnings(df):
    """Calculates the cumulative earnings over time for one bettor"""
    bet_amounts = list(df.Bet)
    towin_amounts = list(df['To Win'])
    outcomes = list(df.Outcome)
    bet_no = list(df['Bet No.'])

    new_earnings = []        # list to track cumulative earnings over time
    for i in range(len(df.Bet)):
        if i == 0:
            if outcomes[0] == 'Win':
                new_earnings.append(towin_amounts[0])
            else:
                new_earnings.append(-bet_amounts[0])
        else:
            if type(bet_no[i]) != str or 'Bet 1' in bet_no[i]: 
                if outcomes[i] == 'Win':
                    new_earnings.append(new_earnings[i-1] + towin_amounts[i])
                else:
                    new_earnings.append(new_earnings[i-1] - bet_amounts[i])
            else:
                new_earnings.append(new_earnings[-1])

    return pd.Series(new_earnings, index=df.index)
```

The function began by adding the amount won/lost of the first bet to the new_earnings list, then added or subtracted money for the rest of the bets based on the outcome.

Since the function returns the cumulative earnings in a pandas Series, adding the feature to the data was easy:
```python
todd['Earnings'] = get_earnings(todd)
clay['Earnings'] = get_earnings(clay)
sal['Earnings'] = get_earnings(sal)
jason['Earnings'] = get_earnings(jason)
```
**Jason McIntyre filled in while Clay Travis was on vacation, so Jason had a week of bets as well*
### Exploratory Data Analysis
After collecting 381 bets from the show, I created visualizations using matplotlib and seaborn to understand the data better. To create these plots, I used one dataframe for each of the four bettors' data (these are named sal, clay, todd, and jason) and one dataframe with everyone's data (df).

First, I wanted to look at which types of bets were made most often by the analysts. To do this I had to calculate how many parlay bets were placed:
```python
parlays = df[df['Bet No.'].notnull()]   # Null values indicate the bet was not a parlay

parlay_count = 0
for item in parlays['Bet No.']:
    if 'Bet 1' in item:             # only counting each parlay once
        parlay_count += 1  
```
Then I compared the frequencies of each bet type:

```python
# finding the original counts of bets
bet_types = df['Bet Type'].value_counts()

# adding in the parlays
all_bet_types = pd.Series(list(bet_types) + [parlay_count], index=list(bet_types.index) + ['Parlay'])

# creating the plot
all_bet_types.plot(kind='barh', fontsize='x-large')
plt.xlabel('Amount of Bets', fontsize='x-large')
plt.xticks([0, 50, 100, 150, 200])
plt.title('Total Bets Made by Type', fontsize='x-large')
```
![Total Bets Made by Type](https://live.staticflickr.com/65535/48586431532_f5771fc7b8_b.jpg)
Prop bets were by far the most popular type of bet from May to July. 

I also wanted to analyze these bet type frequencies for each individual bettor. To do this, I created a new dataframe containing the value counts for each person and each bet type:
```python
# value counts of prop, moneyline, spread:
bet_type_df = pd.DataFrame({'Clay':clay['Bet Type'].value_counts(),
                            'Sal':sal['Bet Type'].value_counts(),
                            'Todd':todd['Bet Type'].value_counts(),
                            'Jason':jason['Bet Type'].value_counts()})
bet_type_df.fillna(0, inplace=True)                 # fill NA's
bet_type_df['Type'] = bet_type_df.index             # added column to use for seaborn

# add in parlay data:
bet_type_df = bet_type_df.append({'Clay':clay_parlays,
                    'Sal':sal_parlays,
                    'Todd':todd_parlays,
                   'Jason':jason_parlays,
                   'Type':'Parlay'}, ignore_index=True)
```
With this dataframe, I was able to plot each bettor's bet type frequencies. The code for Clay's plot is below (code for the other three plots were nearly identical).
```python
# Clay
sns.catplot(x='Type', y='Clay', data=bet_type_df, kind='bar')
plt.title("Clay's Bet Types", fontsize='x-large')
plt.ylabel('Number of Bets', fontsize='large')
plt.xlabel('Bet Type', fontsize='large')
```
![Clay and Sal Charts](https://live.staticflickr.com/65535/48586586806_bc7f16f7d6_b.jpg)
![Todd and Jason Charts](https://live.staticflickr.com/65535/48586685956_d26f20e5b7_b.jpg)
As expected, prop bets were the most common for all four bettors. The distribution for all bet types was also fairly similar for all four people. We can also tell some of the different preferences they each have. Cousin Sal made much more moneyline bets and parlays than anyone else, Todd made much more prop bets than any other type, and Jason never made a parlay or spread bet.

While we're looking at the types of bets each person made, let's take a look at the four bettors' earnings by bet type over time. 

To do this, I'll make a new dataframe using the get_earnings( ) function I defined earlier. This dataframe will include all cumulative earnings over time for each individual type of bet. As an example, this is how I created the data for prop bets:
```python 
# prop
prop_df = df[df['Bet Type'] == 'Prop']
prop_df = prop_df[prop_df['Bet No.'].isnull()]              # getting rid of parlays
prop_plot_df = prop_df.loc[:,['Bet Type', 'Bet Date']]      # only relevant columns
prop_plot_df.sort_values(by=['Bet Date'], inplace=True)     # in order by date
prop_plot_df['Earnings'] = get_earnings(prop_df)            # add earnings
```
After repeating similar code for the other 3 bet types, I put them all together and produced a line plot:
```python
# all together
bet_type_earnings_df = pd.concat([spread_plot_df, moneyLine_plot_df, prop_plot_df, parlay_plot_df])
bet_type_earnings_df.drop_duplicates(subset=['Bet Date', 'Bet Type'], keep='last', inplace=True)
bet_type_earnings_df.sort_values(by=['Bet Date'], inplace=True)
bet_type_earnings_df
```
```python
sns.relplot(x="Bet Date", y="Earnings", hue="Bet Type", kind='line', data=bet_type_earnings_df)
plt.title('Earnings by Bet Type', fontsize='large')
plt.ylabel('Cumulative Earnings', fontsize='large')
plt.xlabel('')
plt.xticks([0, 8, 16, 24], ['May 13', 'May 27', 'June 11', 'July 4'])
plt.yticks([-1000, 0, 1000, 2000, 3000], ['-$1000', '$0', '$1000', '$2000', '$3000'])
```
![Total Earnings by Bet Type](https://live.staticflickr.com/65535/48586806456_5cc108d0e1_b.jpg)
