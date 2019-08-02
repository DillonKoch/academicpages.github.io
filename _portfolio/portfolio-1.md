---
title: "Lock It In Sports Betting Analysis"
excerpt: "Data I collected and analyzed about the TV show Lock It In (not the pic below)
<br/><img src='https://live.staticflickr.com/65535/48117188003_5fd758e4cc_o.png'>"
collection: portfolio
---

Lock It In is a TV show on Fox Sports 1 focused on sports betting. The show is hosted by Rachel Bonetta and also includes analysts Todd Fuhrman, Clay Travis, and Cousin Sal. Each week the three analysts are given $1,000 to bet throughout the week, and they usually make 4 bets per day (20/week). Whoever has the most money at the end of the week wins.

I became interested in sports betting after it was federally legalized in May 2018, and especially now since it's legal in Iowa as of August 2019. What's not to love about it? It's a great mix of sports and data, it makes watching games more interesting, and if you do well you can make money! 

I also find it much more interesting than fantasy sports because sports betting looks at the games through mostly the same lens as the players and coaches. For example, Tom Brady and Bill Belichick would be glad to win a game 13-3, as would people who bet on the Patriots. On the other hand, someone who had Tom Brady in fantasy football would probably be disappointed that he didn't score more touchdowns.

### Data Collection
I had a week off between the spring semester at Iowa and my summer internship at Collins Aerospace, so I watched Lock It In and got the idea to begin collecting and analyzing data from the show. Every bet made on the show is displayed like image below, which made this process consistent and straightforward.

(pic)

To collect this data, I created a Python class that would create a dataframe to store data about each analyst's bets. The class is created with the bettor's name and job, and will create empty dataframes to hold bet data:
```python
class Bettor:
  def __init__(self, name, job):
    self.name = name
    self.job = job
    self.numBets = 0
    self.spreadBets = pd.DataFrame(columns=['ID', 'Bet Date', 'Gameday', 'Bet',
                                            'To Win', 'Home Team', 'Away Team', 'Pick',
                                            'Spread', 'League', 'Outcome'])
    self.moneyLineBets = pd.DataFrame(columns = ['ID', 'Bet Date', 'Gameday', 'Bet',
                                                 'To Win', 'Home Team', 'Away Team', 'Pick',
                                                 'Money Line', 'League', 'Outcome'])
    self.parlayBets = pd.DataFrame(columns = ['ID', 'Bet No.', 'Bet Type', 'Bet Date', 'Gameday',
                                              'Bet', 'To Win', 'Home Team', 'Away Team',
                                              'Pick', 'Spread', 'Money Line', 'League', 'Outcome'])
    self.propBets = pd.DataFrame(columns = ['ID', 'Bet Date', 'Gameday', 'Bet',
                                            'To Win', 'Pick',
                                            'Money Line', 'League', 'Outcome'])
    self.allBets = pd.DataFrame(columns = ['ID', 'Bet No.', 'Bet Type', 'Bet Date', 'Gameday',
                                           'Bet', 'To Win', 'Home Team', 'Away Team',
                                           'Pick', 'Spread', 'Money Line', 'League', 'Outcome'])
```
After creating the constructor, I also wrote methods to populate those dataframes based on the type of bet being made. There is one method for each type of bet (spread, money line, prop, parlay) since they each include slightly different data. 

Here's the method to add a spread bet (e.g. Warriors -3 vs Raptors):
```python
  def betSpread(self, amount, toWin, home, away, pick, spread, league, betDate, gameDate):
    ID = self.numBets + 1
    newRow = [ID, betDate, gameDate, amount, toWin, home, away, pick, spread, league, np.nan]
    
    # add the new row to dataframes including spread bets and all bets
    self.spreadBets.loc[len(self.spreadBets)] = newRow
    self.allBets.loc[len(self.allBets)] = [ID, np.nan, 'Spread', betDate, gameDate, amount, toWin, 
                                           home, away, pick, spread, np.nan, league, np.nan]
    self.numBets += 1
```

