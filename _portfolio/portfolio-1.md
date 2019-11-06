---
title: "Lock It In Sports Betting Analysis - Season 1 (IN PROGRESS)"
excerpt: "Data I collected and analyzed about the TV show Lock It In
<br/><img src='https://live.staticflickr.com/65535/48440110437_e93350f32b_o.jpg'>"
collection: portfolio
---

Lock It In is a TV show on Fox Sports 1 focused on sports betting. The show is hosted by Rachel Bonetta and also includes analysts Todd Fuhrman, Clay Travis, and Cousin Sal. Each week the three analysts are given $1,000 to bet throughout the week, and they usually make 4 bets per day (20 per week). Whoever has the most money at the end of the week wins.

Every bet made on the show is displayed like image below, which made the data collection process consistent and straightforward.

![todd bet](https://live.staticflickr.com/65535/48795549558_5e995bb680_b.jpg)

### Terminology
* __Spread Bets:__ bets made on the outcome of a game plus or minus the spread. If the spread is Warriors -3.5, then the Warriors need to win the game by at least 4 points to win the bet.
* __Moneyline Bets:__ bets made purely on the outcome of the game. Betting the favorite will result in a lower payout than a spread bet if they win, while betting an underdog will pay out more money than a spread bet.
* __Moneyline Notation:__ Most bets are usually given with a moneyline indicating how likely the bet is to cash. Positive moneylines describe how much money you would win if you bet $100, and negative moneylines tell you how much you need to bet to win $100. For instance, betting $100 on a moneyline of +200 would pay out $200 if you win. Betting $300 on a moneyline of -300 would win you $100.
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
After collecting 381 bets from the show in season 1, I created visualizations using matplotlib and seaborn to understand the data better. To create these plots, I used one dataframe for each of the four bettors' data (these are named sal, clay, todd, and jason) and one dataframe with everyone's data (df).

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
all_bet_types = pd.Series(list(bet_types) + [parlay_count], index=list(bet_types.index)+['Parlay'])

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
Something else I wanted to analyze was the amount wagered per bet compared to the potential winnings of those bets. Remember that each bettor gets $1,000 to make about 20 bets per week, but they can choose how much to wager on each bet. 

Here's the code I used to plot the amount bet and potential payout distributions:
```python
# Plotting the Amount Bet Distributions
sns.distplot(sal['Bet'], hist=False, label='Cousin Sal')
sns.distplot(clay['Bet'], hist=False, label='Clay')
sns.distplot(todd['Bet'], hist=False, label='Todd')
plt.xlabel('Amount Bet')
plt.title('Amount Bet Distributions', fontsize='x-large')
plt.xticks([0, 200, 400, 600, 800], ['$0', '$200', '$400', '$600', '$800'])
```
```python
# Plotting the Potential Payout Distributions
sns.distplot(sal['To Win'], hist=False, label='Cousin Sal')
sns.distplot(clay['To Win'], hist=False, label='Clay')
sns.distplot(todd['To Win'], hist=False, label='Todd')
plt.xlabel('Amount Bet')
plt.title('Distributions of Potential Payouts', fontsize='x-large')
plt.xlim((-250, 3000))
plt.xticks([0, 500, 1000, 1500, 2000, 2500, 3000], 
           ['$0', '$500', '$1000', '$1500', '$2000', '$2500', '$3000'])
```
![Amount bet vs Potential Payout](https://live.staticflickr.com/65535/48587444766_793448d6db_b.jpg)

**(I didn't plot Jason's distribution because he had much fewer bets)*

Let's also take a look at the bets individually using two scatterplots. The one on the left shows all bets, while the plot on the right zooms in to view the majority of bets in more detail.
```python
# Plotting all bet amounts and potential payouts
g = sns.relplot(x='Bet', y='To Win', hue='Bettor', data=df)
plt.xlabel('Amount Bet', fontsize='large')
plt.ylabel('Potential Payout', fontsize='large')
plt.title('Money Bet vs Potential Payout')
# these two lines were used in the plot on the right to zoom in:
plt.ylim([0, 500]) 
plt.xlim([0, 500])
```
![Amount Bet vs Potential Payouts](https://live.staticflickr.com/65535/48587761121_bf968ce5aa_b.jpg)

Looking at the scatterplot on the left, it's clear that Clay made the bets with the highest potential payout. Having watched the show, I know that this was partially due to him losing in the season 1 show standings and trying to win a huge bet to catch up to Sal and Todd. It'll be interesting to if he still makes bets with large potential payouts in season 2 of the show.

When you look at the scatterplot on the right with symmetrical axes, there appears to be two main trends of bets. The first is a trend of bets in which the potential payout increases as the amount bet increases (data points moving from the bottom left towards the top right of the plot). This is about what you'd expect - the more you bet, the more you could win. 

The other trend I saw was that there are many bets with over $100 of potential payout and less than $100 bet, but only a few bets with under $100 payout and over $100 bet. This makes it very clear that the bettors preferred bets with a small chance of succeeding that could win big over bets that have a high likelihood of winning, but odds that require a large amount bet to win anything noteworthy. I think this is more a reflection of the show's structure than an optimal sports betting strategy. These bettors want to maximize their returns on $1,000 of bets every week, so placing $400 on a bet with -800 odds to win $50 aren't nearly as attractive as bets that could pay out equal or more money than the amount bet.


I also wanted to look into how the bettors chose to bet home and away teams, and how successful those teams were. To do this I created two bar charts describing the number of bets on home and away teams, and how much those teams won.
```python
ha_df = df[df['Bet Type'] != 'Prop']

home = list(ha_df['Home Team'])
away = list(ha_df['Away Team'])
bet = list(ha_df['Pick'])
outcome = list(ha_df.Outcome)

home_bet = 0
away_bet = 0
home_wins = 0
home_losses = 0
away_wins = 0
away_losses = 0

for home, away, bet, outcome in zip(home, away, bet, outcome):
    if bet == home:
        home_bet += 1
        if outcome == 'Win':
            home_wins += 1
            away_losses += 1
        else:
            home_losses += 1
            away_wins += 1
    elif bet == away:
        away_bet += 1
        if outcome == 'Win':
            away_wins += 1
            home_losses += 1
        else:
            away_losses += 1
            home_wins += 1

# creating the visualizations

# bets placed code
ha_betdf = pd.DataFrame({'Number of Bets':[home_bet, away_bet],
                          'Team Status':['Home', 'Away']})

sns.catplot(x='Team Status', y='Number of Bets', data=ha_betdf, kind='bar')
plt.ylabel('Number of Bets', fontsize='large')
plt.xlabel('Team Status', fontsize='x-large')
plt.title('Bets Placed on Home and Away Teams', fontsize='x-large')

# home and away wins code
ha_winsdf = pd.DataFrame({'Number of Wins':[home_wins, away_wins],
                          'Team Status':['Home', 'Away']})
ha_winsdf

sns.catplot(x='Team Status', y='Number of Wins', kind='bar', data=ha_winsdf)
plt.ylabel('Number of Bets', fontsize='large')
plt.xlabel('Team Status', fontsize='x-large')
plt.title('Home and Away Team Wins', fontsize='x-large')
```

![pic](https://live.staticflickr.com/65535/48796199687_1f68d56f21_o.png)
![pic](https://live.staticflickr.com/65535/48796059161_d25aa6129f_o.png)

As expected, home teams won slightly more games than away teams, but there was a bigger difference in the amount of games bet on home teams than away teams. Judging by these charts, the bettors could benefit by betting on away teams more often.

I was also interested in learning how many bets were placed on each league. Keeping in mind that the data I collected was from May to August, we can see that the NBA and MLB were bet on most frequently:
```python
df['League'].value_counts()[0:10].plot(kind='barh')
plt.title('Bets Made per League', fontsize='x-large')
plt.xlabel('Number of Bets', fontsize='x-large')
plt.yticks(fontsize='large')
```
![pic](https://live.staticflickr.com/65535/48796199757_a1aa5f0ceb_o.png)


To go one level deeper, I also decided to find out how successful each bettor was in each league:
```python
# Sample code for creating Todd's plot, all others are nearly identical
sns.catplot(x='Earnings', y='League', kind='bar', data=byLeague(todd).sort_values(by=['Earnings']),
           palette=sns.diverging_palette(10, 133, n=18), height=5, aspect=2)
plt.xticks([-200, -100, 0, 100, 200, 300], ['-$200', '-$100', '$0', '$100', '$200', '$300'])
plt.ylabel('')
plt.title("Todd's Winnings by League", fontsize='xx-large')
```
![pic](https://live.staticflickr.com/65535/48795702908_cd8a6f9d36_o.png)
![pic](https://live.staticflickr.com/65535/48796199717_6846cf0d5a_o.png)
![pic](https://live.staticflickr.com/65535/48795702943_bb6279c69c_o.png)
![pic](https://live.staticflickr.com/65535/48796199642_da440ec188_o.png)

I also thought it would be interesting to look at the individual teams that were bet on most frequently, and which leagues those teams belonged to:
```python
all_teams = df['Home Team'].append(df['Away Team'], ignore_index=True)

x = pd.DataFrame(all_teams.value_counts()[0:10])
x['League'] = pd.Series(['NBA','NBA','Soccer','NHL','NBA','NBA','NHL','Soccer',"MLB",'MLB'],index=x.index)
x.columns = ['Bets', 'League']
x['Team'] = x.index

sns.catplot(x='Bets', y='Team', kind='bar', hue='League', dodge=False, data=x)
plt.title('Teams Bet Most Often', fontsize='large')
plt.ylabel('')
plt.xlabel('Number of Bets', fontsize='large')
```
![pic](https://live.staticflickr.com/65535/48796059086_cc26488887_o.png)

Another idea I had was to calculate which bettor had the best and worst individual day throughout the summer. 

![pic](https://live.staticflickr.com/65535/48795703053_d4073b4474_o.png)

Cousin Sal had by far the best individual day, while Todd had the single worst day.

Finally, I wanted to show how successful each bettor was overall. These two plots show bettors' win-loss records and their total earnings over time:
```python
# Creating Series objects with W-L records for each person
sal_wl = sal.Outcome.value_counts()
clay_wl = clay.Outcome.value_counts()
todd_wl = todd.Outcome.value_counts()
jason_wl = jason.Outcome.value_counts()

# Create DataFrame with W-L records
wl_df = pd.DataFrame([sal_wl, todd_wl, clay_wl, jason_wl], index=['Sal', 'Todd', 'Clay', 'Jason'])

# create the plot
wl_df.plot(kind='bar', color=['#F08080', '#32CD32'])
plt.xticks(rotation=0, fontsize='large')
plt.ylabel('Number of Bets', fontsize='x-large')
plt.title("Win-Loss Records", fontsize='xx-large')
plt.legend(fontsize='large')
```
![pic](https://live.staticflickr.com/65535/48795703018_0ea4378b7c_o.png)

```python
sns.relplot(x='Bet Date', y='Earnings', kind='line', ci=None, hue='Bettor', data=df)
plt.xticks([0, 11, 20], ['May 13', 'June 3', 'June 19'])
plt.xlabel('Bet Date', fontsize='large')
plt.ylabel('Cumulative Earnings', fontsize='large')
plt.title('Total Earnings per Person', fontsize='x-large')
plt.yticks([-500, 0, 500, 1000, 1500, 2000, 2500, 3000],
           ['-$500', '$0', '$500', '$1000', '$1500', '$2000', '$2500', '$3000'])
```
![pic](https://live.staticflickr.com/65535/48796059201_7d04a4b69c_o.png)

We can see from the plot above that Clay led the group in the middle of the summer, but Cousin Sal finished much higher than everyone else.

### Predicting Future Bets
Finally, I have also begun using this data to predict the outcome of new bets that each member on the show makes. 

### Future Ideas
I'm currently collecting data for season 2 of the show, so I'll implement some of my ideas with the new data, including:
* Using web scraping to collect data about each game (final score, home and away team names, etc)
* Collecting images of each bet from the show (like the one at the top of this page) and training an AI agent to detect data about the bet and collect it for me.
* 




