# The Playoffs Game

Historical research to support a simple game design for a playoffs betting pool

## updating historical data

Goal is to be able to see what the expected value is of each seed number, given a set of rules.

1. manually update history.csv with latest results

Suggested prompt: 

```
You said:
I am collecting a summary of NBA playoff history in a csv format. My format is:

year,round_num,team_name,seed_num,games_won

where games_won is the total number of games the team won in the playoffs, across all four rounds.

E.g., here are my rows for the 2020 playoffs:

2020,1,MIL,1,4
2020,1,ORL,8,1
2020,1,IND,4,0
2020,1,MIA,5,4
2020,1,BOS,3,4
2020,1,PHI,6,0
2020,1,TOR,2,4
2020,1,BRK,7,0
2020,1,LAL,1,4
2020,1,POR,8,1
2020,1,HOU,4,4
2020,1,OKC,5,3
2020,1,DEN,3,4
2020,1,UTA,6,3
2020,1,LAC,2,4
2020,1,DAL,7,2
2020,2,MIL,1,1
2020,2,MIA,5,4
2020,2,BOS,3,4
2020,2,TOR,2,3
2020,2,LAL,1,4
2020,2,HOU,4,1
2020,2,LAC,2,3
2020,2,DEN,3,4
2020,3,MIA,5,4
2020,3,BOS,3,2
2020,3,LAL,1,4
2020,3,DEN,3,1
2020,4,MIA,5,2
2020,4,LAL,1,4

Does this format make sense?

If I provide you with this 202X summary: (paste content, or screenshot of bracket)
...can you produce the csv rows I need?
```



2. run command to convert history.csv to sums.csv (total score for each team-year):
```bash
sed -e 's/#.*//' -e '/^$/ d' history.csv | awk -F, 'BEGIN {playin_bonus = 20; round1_bonus = 40; round2_bonus = 40; conf_bonus = 40; champ_bonus = 40 } {year = $1; rounds = $2; round = $3; best_of = $4; team = $5; seed_num = $6; wins = $7; adj_wins = wins; if (round > 0) {adj_wins = 7.0 * wins / best_of}; won = 0; if (best_of != "?" && wins > best_of / 2.0) {won = 1}; score = 0; if (won && round == rounds) {score += champ_bonus}; if (won && round == rounds - 1) {score += conf_bonus}; if (won && round == rounds - 2) {score += round2_bonus}; if (won && round == rounds - 3) {score += round1_bonus}; if (round == 1 && rounds < 4) {adj_wins += 4; score += round1_bonus}; if (round == 1 && rounds < 3) {adj_wins += 4; score += round2_bonus}; if (round == 0) {score += playin_bonus * wins}; score += seed_num^2 * adj_wins; team_year = year "," team; seed[team_year] = seed_num; total_wins[team_year] += adj_wins; total_score[team_year] += score; } END {print "# year,team,score,seed,numwins"; for (team_year in total_score){this_score = total_score[team_year]; this_conf_bonus = 0; this_champ_bonus = 0; this_wins = total_wins[team_year];  print team_year "," this_score "," seed[team_year] "," this_wins} }'  | sort -t, -n -k 2 > sums.csv
```
3. run command to summarize the sums.csv team-year scores by seed, and show average score for each seed:
```
cat sums.csv | awk -F, 'BEGIN {cur_year = 2025; year_decay_rate = 0.97} {year = $1; team = $2; score = $3; seed = $4; wins = $5; year_value = year_decay_rate ^ (cur_year - year); total_score[seed] += score * year_value; total_wins[seed] += wins * year_value; instances[seed] += year_value} END {for (i = 1; i <= 10; i++) { if (instances[i]) {print i "," sprintf("%.02f",instances[i]) "," sprintf("%.02f",total_wins[i] / instances[i]) "," sprintf("%.02f",total_score[i] / instances[i]) }}}'
```
 Note the decay rate; set to 1.0 to have it not matter:


