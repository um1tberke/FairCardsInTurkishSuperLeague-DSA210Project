# Fair Cards in Turkish SuperLeague DSA210Project

*Do big clubs or home teams get a card advantage?*

## Motivation
In Türkiye, many fans believe that “big clubs” or home teams are treated more softly by referees. Instead of relying on opinions, this project is going to test that claim with data. If a measurable effect exists, it matters for competitive balance and public trust. 

## Project Overview
I will analyze one complete Süper Lig season (preferably **2023–24**) to test two effects:
- **Big-club effect:** Do labeled “big clubs” receive fewer cards than their opponents?
- **Home-field effect:** Do home teams receive fewer cards than away teams?

To keep comparisons fair, I will enrich match data with **pre-match betting odds from Mackolik** (converted to implied probabilities) to proxy expected strength (favorite vs underdog). This helps separate “they got fewer cards because they were stronger” from “they got fewer cards because of bias.”

## Research Questions
- **RQ1:** Do “big-club” teams receive fewer yellow/red cards than their opponents in a systematic way?
- **RQ2:** Does playing at home create a meaningful advantage in card counts?

## Dataset — Sources
Primary data will be compiled from:
- **Mackolik** — match sheets for Süper Lig (teams, date/time, yellow/red cards, and pre-match betting odds).
- **Opta** (as available via public summaries) — to cross-check card/event counts and definitions.
- **Transfermarkt** — additional match details to standardize names and fill gaps.


## Scope & Assumptions (MVP)
- Start with a **single complete season (2023–24)** for consistency; extend later if time allows.
- **Big-club list (fixed):** Fenerbahçe, Beşiktaş, Galatasaray.
- **Red cards** and **Second yellow red cards** are kept separate; I will also run a sensitivity check that treats **“red = 2 yellows.”** Also **second yellow reds will be counted as +1 yellow** for the team.
- No player-level tactics or lineups in MVP; the goal is a **league-level fairness** check.

## Method (high level)
1. **Assemble** one season from Mackolik/Opta/Transfermarkt; standardize team/referee names and parse dates.  
2. **Engineer features** for fair comparison: big-club tags (fixed list above), home/away indicator, implied probabilities from Mackolik odds.  
3. **Quantify effects** with straightforward comparisons and statistical tests:
   - Big-club vs others: team-level card differences.
   - Home vs away: test whether the distribution of `home_cards − away_cards` centers at 0.
4. **Small validation model (sanity check):** a lightweight classifier (e.g., logistic regression) predicting which side receives more cards using the features above.  
5. **Report** effect sizes (with uncertainty) and honest limitations.

## Expected Outcomes
- A quantitative answer about the **direction** and **magnitude** of any big-club or home-field advantage in card counts.
- Either confirm a measurable effect or provide a clear negative result—both are useful.
- A framework that can be extended to multiple seasons later.

## Limitations & Notes
- Source definitions may differ (e.g., straight red vs second yellow); I will document any harmonization.
- Odds are a **proxy** for strength and may add noise.
- Style of play and match context (derby intensity, stakes) are out of 
- Only **public data** will be used; all sources will be cited.

- ---

## Current status – Data collected

For this milestone, I have manually collected one full Süper Lig season (2023–24) for the three big clubs:

- **Beşiktaş** – 38 league matches  
- **Fenerbahçe** – 38 league matches  
- **Galatasaray** – 38 league matches  

These are stored in three CSV files:

- `bjk23_24.csv`
- `fb23_24.csv`
- `gs23_24.csv`

Each row represents a single match from the big-club perspective and contains:

- `season`, `week`
- `home_away` – `"H"` if the big club is home, `"A"` if it is away
- `opponent` – opposing team name
- `home_odds`, `draw_odds`, `away_odds` – pre-match betting odds from Mackolik
- `team_yellow`, `team_red` – yellow / red cards for the big club
- `opp_yellow`, `opp_red` – yellow / red cards for the opponent
- `team_fouls`, `opp_fouls` – total fouls by each side

In the notebook, I also engineer several derived variables:

- `diff_yellow`, `diff_red`, `diff_fouls` – big club minus opponent
- `card_points = yellow + 2 × red`
- `diff_card_points` – big club minus opponent in card points
- implied probabilities (`home_prob`, `draw_prob`, `away_prob`) computed from the pre match odds
- `big_win_prob`, `opp_win_prob` and an indicator `is_favourite` from the big-club perspective
- `home_yellow`, `away_yellow`, `diff_home_yellow` and the same for card points, from the stadium (home vs away) perspective.

- ## Exploratory data analysis (EDA)

Some key descriptive patterns from the 114 matches:

- Big clubs receive on average **2.25 yellow cards per match**, while their opponents receive **2.68**.  
  The mean yellow-card difference `diff_yellow = team_yellow − opp_yellow` is about **−0.43**, meaning big clubs get roughly 0.4 fewer yellow cards than their opponents on average.
- Using the “card points” metric `card_points = yellow + 2 × red`, the mean difference `diff_card_points` is around **−0.45**, so big clubs also receive fewer total disciplinary points.
- By team, Fenerbahçe and Galatasaray tend to have negative mean `diff_yellow` (fewer cards than opponents), while Beşiktaş is closer to balanced or slightly worse.
- Foul–card relationships are moderate for big clubs but stronger for their opponents.  
  For opponents, the Pearson correlation between `opp_fouls` and `opp_yellow` is about **0.44**, so higher foul counts are more tightly linked to yellow cards compared to big clubs.
- Using implied probabilities from the odds, big clubs are favourites in most matches.  
  When they are **pre-match favourites**, their mean card-point difference is **negative** (they receive fewer card points); when they are **not favourites**, the mean difference becomes **positive**, and they receive more card points than their opponents.

The Jupyter notebook includes histograms and bar charts that visualize `diff_yellow`, team-level differences and favourite vs non-favourite patterns.

## Hypothesis tests

### RQ1 – Big-club effect

To test whether big clubs systematically receive fewer cards than their opponents, I use:

- `diff_yellow = big-club yellows − opponent yellows`
- `diff_card_points = big-club card points − opponent card points`, where card points = yellow + 2 × red

For `diff_yellow`, the sample mean is approximately **−0.43**.  
A one-sample t-test with

- H₀: E[diff_yellow] = 0  (no systematic difference)
- H₁: E[diff_yellow] < 0  (big clubs receive fewer yellow cards)

gives a negative t-statistic with a one-sided p-value around **0.01–0.02**.  
At the 5% significance level, I reject the null hypothesis and conclude that big clubs receive fewer yellow cards than their opponents in this sample.

For `diff_card_points`, the mean is about **−0.45**.  
The corresponding one-sample t-test again produces a one-sided p-value below **0.05**, so the result is robust when treating red cards as worth two yellow cards.

### RQ2 – Home-field effect

To study home-field effects, I rewrote each match from the stadium perspective and defined:

- `home_yellow`, `away_yellow` – yellow cards received by the home and away teams
- `diff_home_yellow = home_yellow − away_yellow`
- the analogous `diff_home_card_points` using the card-points metric

Across 114 matches, the means are:

- `diff_home_yellow` ≈ **−0.24**
- `diff_home_card_points` ≈ **−0.29**

Both suggest that home teams receive slightly fewer cards, but one-sample t-tests against 0 give two-sided p-values around **0.18–0.22**, which are not significant at the 5% level. Statistically, I cannot claim a clear home-field advantage in card counts in this dataset.
