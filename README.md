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

## Dataset (planned) — Sources
Primary data will be compiled from:
- **Mackolik** — match sheets for Süper Lig (teams, date/time, yellow/red cards, and pre-match betting odds).
- **Opta** (as available via public summaries) — to cross-check card/event counts and definitions.
- **Transfermarkt** — additional match details to standardize names and fill gaps.


## Scope & Assumptions (MVP)
- Start with a **single complete season (2023–24)** for consistency; extend later if time allows.
- **Big-club list (fixed):** Fenerbahçe, Beşiktaş, Galatasaray, Trabzonspor.
- **Red cards** are kept separate; I will also run a sensitivity check that treats **“red = 2 yellows.”**
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
