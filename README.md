# A Data-Driven Exploration of Pokémon

DSA 210 – Introduction to Data Science (Spring 2026)

## Motivation

Pokémon as a franchise has been a part of my life since childhood. When I was younger, I didn’t properly understand how the games systems worked or why certain game design decisions were made. As I grew older, I found myself pondering about the game design decisions behind not only Pokémon but anything I play. I also found myself interested in the effectiveness of computer algorithms in learning the same way players do after investing time in a game. Would a computer algorithm value the same qualities or make the same decisions as a good player? Is it possible to algorithmically learn the most effective tactic available (META) of a game the same way a player base does? This project gave me the opportunity to explore these questions through the application of data science practices on past 30 years of Pokémon game data.

## Data Sources

1. **PokéAPI**
* **Source:** PokéAPI (RESTful open-source database)
* **Collection Method:** Automated script collecting 6 raw JSON files (`abilities_raw`, `generations_raw`, `move_damage_classes_raw`, `pokemon_raw`, `species_raw`, `types_raw`)
* **Preprocessing:** Stripped localized names and descriptions; removed non-canon assets (e.g., *Pokémon XD*); removed unique gimmicks like Cosplay Pikachu; manually corrected inaccurate generation indicators for unique alternate forms like Mega Evolutions. Refer to data_processing.ipynb for more details.

2. **Smogon API**
* **Source:** Smogon API 
* **Collection Method:** Manual extraction of 8 comprehensive format usage JSON files
* **Preprocessing:** Filtered formats using Smogon’s ~4.52% weighted usage membership rule; established the non-usage-based "Ubers" tier using competitive banlists and manual exclusions; resolved string discrepancies between naming systems (e.g., `ogerpon-wellspring` vs. `ogerpon-wellspring-mask`). Refer to data_prep_and_feature_engineering.ipynb for more details.

## Data Analysis

### 1. Data Preparation

* Standardized text inputs and handled multi-source structural string mismatches.
* Ordinally mapped the 7 usage-based competitive tiers from 0 (highest) to 6 (lowest).
* Split dataset into an 80/20 train/test split, applying stratification to account for class distribution imbalances (e.g., the lowest tier, ZU, heavily dominates the dataset).

### 2. Exploratory Data Analysis (EDA)

EDA focused on evaluating long-term generational design paradigm shifts and structural traits:

* Analyzed move damage frequencies, discovering physical moves outnumber status and special moves.
* Visualized generation-specific type distributions using horizontal bar plots to test rarity over time (confirming Dragon-type inflation in Gen 6 was heavily driven by Mega Evolutions).
* Explored generation representation inside the Overused (OU) tier using scatter plots, revealing dominance of newer generations.

All generated visualizations can be found in the `/visualizations` directory.

## Hypothesis Testing

### Hypothesis 1: Are Water types bulkier, or is Suicune gaslighting us?
---
**Motivation**

Historically, bulky Water types have performed exceptionally well in competitive Pokémon; specifically, **Suicune** was a mainstay in multiple generations. Because of this, there is a common perception of Water as a bulky type. This however, is an assumption made based on the performance of a few. Thus I wanted to test this assumption through the application of statistical methods.

**Approach**

Bulk was quantified as the percentage of a Pokémon's **Base Stat Total (BST)** allocated to defensive stats.

The formula for **Bulk** is defined as:

$$Bulk = \frac{HP + Defense + Sp. Defense}{BST}$$

**Question:** Is the bulk distribution of Water-type Pokémon significantly different from the bulk distribution of Pokémon of other types?

**Independent Samples T-Test**

The independent samples t-test was chosen because the means of two normally distributed populations are compared.

**Variance Check & Test Version Selection**
Before performing the test, the variance of both distributions was checked to determine the appropriate method (Standard vs. Welch's T-Test).

| Population | Variance |
| :--- | :--- |
| **Water Types** | 0.0053 |
| **Non-Water Types** | 0.0059 |

The variances are nearly equal and the hypothesis claims that Water types have *greater* bulk. As such, a **One Sided Standard Independent Samples T-Test** is used.

**Hypotheses**

**Null Hypothesis ($H_0$):** The mean bulk of Water types is equal to the mean Bulk % of other Pokémon types.
$$H_0: \mu_{water} = \mu_{other}$$

**Alternative Hypothesis ($H_1$):** The mean bulk of Water types is greater than the mean bulk of other Pokémon types.
$$H_1: \mu_{water} > \mu_{other}$$

**Results**

Turns out Suicune does indeed warp my perception. The calculated p-value is **0.3412** with a significance level of **0.05** we fail to reject the null hypothesis. Thus it can be concluded, Water types do not have a statistically significant bulk advantage over other types.

### Hypothesis 2: Did the Physical/Special split create a change in the way Pokémon are designed?
---
**Some Background and Motivation**

There are two types of damaging moves in Pokémon: **physical** and **special**. They are calculated using the attack and special attack stats, respectively. Until generation 4 a moves damage type depended on the moves type, for example all dark type moves were **special** while all fighting type moves were **physical**. Because of this Pokémon from the first 3 generations in general tend to have stat distributions considered weird by modern standards i.e. they do not specialize in one attacking stat instead they have mediocre stats in both. This is to ensure they can use attacks from their own type(s) decently well and to expand their options for moves from other types.

In generation 4 moves were split into physical and special instead of depending on their type, now every type had moves from both damage types. This I believe influenced the way Pokémon are designed as it allowed them to specialize in one attacking stat regardless of their type(s).

**Approach**

For this hypothesis I quantified the impact of the **physical/special** split as follows:

$${Offensive Gap} = |{Attack} - {Sp. Attack}|$$

**Question:** Do Pokémon from generation 4 onwards have a higher offensive gap in general?

**Independent Samples T-Test**

The independent samples t-test was chosen because the means of two independent populations are compared. The data is right skewed instead of normally distributed, since t-test assumes normal distribution a non-parametric test will also be used to verify the results of this test.

**Variance Check & Test Version Selection**
Before performing the test, the variance of both distributions was checked to determine the appropriate method (Standard vs. Welch's T-Test).

| Population | Variance |
| :--- | :--- |
| **Pre-Gen 4** | 437.7588 |
| **Post-Gen 4** | 702.5510 |

There is a significant difference between the variances and the alternative hypothesis claims pre-gen 4 offensive gap is *lower*. As such, a **One Sided Independent Samples Welch's T-Test** is used.

**Hypotheses**

**Null Hypothesis ($H_0$):** The mean offensive gap of pre-gen 4 Pokémon is equal to the mean offensive gap of post-gen 4 Pokémon.
$$H_0: \mu_{pre} = \mu_{post}$$

**Alternative Hypothesis ($H_1$):** The mean offensive gap of pre-gen 4 Pokémon is lower than the mean offensive gap of post-gen 4 Pokémon.
$$H_1: \mu_{pre} < \mu_{post}$$

**Results**

The calculated p-value is **0.000000000008** with a significance level of **0.05** we safely reject the null hypothesis. According to Welch's t-test there is significant evidence to back up my intuition. Due to the skewed distribution a Mann-Whitney U test is performed to be sure.

**Mann-Whitney U Test**

Mann-Whitney U test was chosen as the non-parametric test as it is analogous to t-test.

**Hypotheses**

**Null Hypothesis ($H_0$):** The offensive gap distribution of pre-gen 4 Pokémon is the same as the offensive gap distribution of post-gen 4 Pokémon.

**Alternative Hypothesis ($H_1$):** The offensive gap distribution of pre-gen 4 Pokémon is stochastically less than the offensive gap distribution of post-gen 4 Pokémon.

**Results**

The calculated p-value is **0.000000003544** with a significance level of **0.05** we safely reject the null hypothesis. The results of the Mann-Whitney U test supports results of the Welch's t-test. It can be concluded that pre-gen 4 Pokémon have a lower offensive gap thus proving the **physical/special** split as a turning point in Pokémon design paradigms.  

### Hypothesis 3: Does generation 9 have significant powercreep?
---
**Motivation**

Powercreep: (collectible games, video games, roleplaying games) The situation where successive updates or expansions to a game introduce more powerful units or abilities, leaving the older ones underpowered.
Definition from https://en.wiktionary.org/wiki/power_creep

As of the time of this project the latest generation of Pokémon is generation 9 which introduced a lot of very powerful Pokémon that dominate the competitive scene such as Great Tusk, King Gambit and Chien Pao. Based on this the existence of powercreep seems obvious however I did not play the generation 9 games yet as such I have only been exposed to the competitively well performing Pokémon of this generation through the internet. This created a perception of powercreep that may or may not be true depending on the rest of the Pokémon from this generation. As such I wanted to see if data agreed with my perception.

**Approach**

For this hypothesis a Pokémon's **Base Stat Total (BST)** will be used as an indicator of its strenght.

The formula for **BST** is defined as:

$$\text{BST} = \sum (\text{Base Stats})$$

where Base Stats = (HP, Attack, Defense, Special Attack, Special Defense, Speed)

**Question:** Is the BST distribution of generation 9 Pokémon significantly higher than that of Pokémon from previous generations?

**Mann-Whitney U Test**

As the data is non-normally distributed, Mann-Whitney U test was chosen because it makes no assumptions on the distribution of the data.

**Hypotheses**

**Null Hypothesis ($H_0$):** The BST distribution of generation 9 Pokémon is the same as the BST distribution of previous generation Pokémon.

**Alternative Hypothesis ($H_1$):** The BST distribution of generation 9 Pokémon is stochastically higher than the BST distribution of previous generation Pokémon.

**Results**

The calculated p-value is **0.0006** with a significance level of **0.05** we safely reject the null hypothesis. The results of the Mann-Whitney U test shows significant evidence in favor of my claim. It can be concluded that generation 9 Pokémon have a BST lead over Pokémon of previous generations thus proving the existence of powercreep.  

## Feature Engineering

The goal during ML phase of the project was to teach an ML algorithm what makes a Pokémon competitively viable. The task was framed as a 7-way classification task where a Pokémon is fed into the model, and the model is asked to identify which tier the Pokémon belongs to. If the usage based statistics from the Smogon API was used as features it would lead to data leakage. A very simple example is the Smogon API provides a list of Pokémon that counters a given Pokémon based on how many knockouts (KOs) or switch-outs they have achieved against the Pokemon in question. Based on this information, the model can easily infer if a Pokémon’s counter is in its training set, then this Pokémon must also be in the same tier. To avoid data leakage only the inherent qualities of a Pokémon were considered as features alongside some potential role identification based on which qualities were valuable historically.

* **Stat Derived Features:** `PhysBulk` and `SpeBulk` (aggregating HP and defensive splits); `AtkToSpaRatio` (quantifying mixed vs. hyper-specialized attacking orientations); and `WastedStatRatio` (evaluating stat points allocated to an unused attacking class).
* **Speed Tiers:** Binned continuous speed attributes into 4 discrete ordinal competitive `SpeedTier` brackets.
* **Typing Scores:** Generated algebraic `TypeOffensiveScore` and `TypeDefensiveScore` based on the defensive and offensive profile of the typing.
* **Role Encoding:** Multi-hot encoded categorical variables mapping movepools to key battle roles (*Hazard Setter, Pivot, Setup, Recovery, Utility, Priority*).
* **Dynamic Ability Encoding:** Programmed internal cross-validation encoding loops to multi-hot encode a curated subset of 13 high-impact competitive abilities (*Intimidate, Regenerator, Prankster, etc.*) only if present in the active training fold.

## Machine Learning Models

Two primary classification models were developed to predict a Pokémon’s Smogon competitive tier:

### Model Setup

* **Training Approach:** 5-fold Stratified Cross-Validation on the training pool.
* **Optimization Goal:** Maximize Macro $F_1$-Score to combat heavy class imbalances.
* **Hyperparameter Tuning:** Automated 35-trial searches based on ranges defined by me for relevant hyperparameters via Optuna.

### Results

| Model | Cross-Val Macro $F_1$ | Holdout Macro $F_1$ | Target Evaluation Insight |
| --- | --- | --- | --- |
| **Dummy Classifier (Most Frequent)** | 0.0840 | — | Scoring 41.67% accuracy by predicting the largest class (ZU) but terrible F1-score. |
| **XGBoost Classifier** | 0.3015 | 0.2782 | Strong at isolating the lowest tier (ZU), but heavily confused by middle brackets and upper-tier boundary lines (OU vs. Ubers). |
| **Support Vector Machine (SVM)** | 0.3096 | 0.2704 | Similar performance to XGBoost, provided more distributed predictions across middling tiers. |

While both models massively outperformed naive guessing, their macro $F_1$-scores underscore the inherent limits of using flat, static features to capture a changing strategic environment.

## Key Findings

* **The "Bulky Water" Myth Exposed:** Structurally, Water-types are not statistically bulkier than other types. Community perception is a clear case of selection bias driven by remembering standouts like Suicune.
* **The Paradigm Shift of Gen 4:** The physical/special split was an absolute turning point in Pokémon design paradigms, permanently shifting Pokémon away from balanced, jack-of-all-trades stat lines into hyper focused specialists.
* **Verifiable Powercreep:** Generation 9's dominant presence in the competitive scene isn't just recency bias; its fully-evolved roster holds a mathematically undeniable stat advantage over older generations.
* **The Context Dilemma of the META:** The ML attempts prove the context dependent nature of the task which is backed up by my personal experience with games. Often it takes years for players to discover what is strong and META and when they look back on what they thought was strong when the game first released, they notice that they did not understand how the game worked back then. Viability is fundamentally relational and context-dependent; a Pokémon's tier is dictated by how it pairs with or counters transient threats, not just its own values.

## Limitations and Future Work

### Limitations and Future Work

My current models look at each Pokémon as an isolated row in a spreadsheet. Since a Pokémon's viability is defined by its relationships with the rest of the META. The next step would be to develop methods to find ways to integrate this context wihout introducing data leakage. Reframing the task could be helpful in accomplishing this. Instead of asking a model, “Is this Pokémon strong?” I could ask, “How does this Pokémon disrupt the existing dominant threats?” This would shift the problem from a classification task to an ecosystem simulation, which more accurately represents how a community solves a metagame.


Instead of feature engineering an embedding approach could be considered to capture more subtle relantionships that make certain tools more powerful when certain Pokémon have access to them. For example, pivot moves are often more powerful when used by fast Pokémon as it allows them to deal damage while avoiding retaliation. 


In this study I focused on the current snapshot of the generation 9 metagames, but Pokémon is a 30 year old franchise. Instead of focusing on a snapshot a temporal appraoch could be used to teach a model the evolution of different metagames throughout the franchises lifespan in hopes of identifying trends within the progression of metagames. 
