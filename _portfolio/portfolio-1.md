---
title: "Lock It In Sports Betting Analysis"
excerpt: "Data I collected and analyzed about the TV show Lock It In (not the pic below)

<br/><img src='https://live.staticflickr.com/65535/48440110437_e93350f32b_o.jpg'>"
collection: portfolio
---

Lock It In is a TV show on Fox Sports 1 focused on sports betting. The show is hosted by Rachel Bonetta and also includes analysts Todd Fuhrman, Clay Travis, and Cousin Sal. Each week the three analysts are given $1,000 to bet throughout the week, and they usually make 4 bets per day (20/week). Whoever has the most money at the end of the week wins.

I became interested in sports betting after it was federally legalized in May 2018, and especially now since it's legal in Iowa as of August 2019. What's not to love about it? It's a great mix of sports and data, it makes watching games more interesting, and if you do well you can make money! 

I also find it much more interesting than fantasy sports because sports betting looks at the games through mostly the same lens as the players and coaches. For example, Tom Brady and Bill Belichick would be glad to win a game 13-3, as would people who bet on the Patriots. On the other hand, someone who had Tom Brady in fantasy football would probably be disappointed that he didn't score more touchdowns.

### Data Collection
I had a week off between the end of the spring semester at the University of Iowa and my summer internship at Collins Aerospace, so I watched Lock It In on TV and got the idea to begin collecting and analyzing data from the show. Every bet made on the show is displayed like image below, which made this process consistent and straightforward.

(pic)

There are four types of bets that I'll keep track of:
* __Spread Bets:__ bets made on the outcome of a game plus or minus the spread. If the spread is Warriors -3.5, then the Warriors need to win the game by at least 4 points for this bet to cash
* __Money Line Bets:__ bets made purely on the outcome of the game. Betting the favorite will result in a lower payout than a spread bet if they win, while betting an underdog will pay out more money than a spread bet.
* __Prop Bets:__ bets made on aspects of the game other than the final outcome. This can be a wide range of bets - a certain player's point total, whether a team scores over/under a number of points, a player to win MVP, etc.
* __Parlay Bets:__ A parlay bet is a bet that includes multiple bets from the first three categories. Every bet in the parlay must win for the bettor to win the parlay. If you bet a parlay made up of 5 individual bets, you won't make any money unless all 5 bets cash.

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
