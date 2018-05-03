---
layout: post
title:  (Old) An analysis on Gwent's ladder
author: Chee Yew Lim
keywords: Gwent, ladder, mmr, rank point, rank, analysis 
date:   2018-05-01 10:14:32 -0700
summary: "How to calculate rank points and what affect it in Gwent"
categories: blog
---

> 01/05/2018 - I did this analysis about a year ago and posted it on Reddit. The developers have since changed the ladder system multiple times, so some of the following information might be incorrect.  

So I analyzed around 130 games to investigate how the MMR/Elo rating system in Gwent functions. This allows players to have a better understanding of the system and utilize it while climbing the ladder.  

# Intro 
Gwent is an online two-player card game. In this game, there is a ranked system that is used to determine who is a better player (just like FIDE chess rating). Each player has a MMR/rating points (think of it as a number), winning will increase your points while losing will have the opposite effect.
The more points you have the better. Besides that, when you reach a certain rating points, you will obtain a rank (i.e. rank 13 requires 2250 points). There are a total of 15 ranks, with rank 15 (requires 3900 points) being the highest. 
With that out of the way, let discussed what I found :)    

# What I found:
Score, cards remaining, or the number of rounds won’t affect the MMR/points you gain or lose. The major factor that determines the MMR you gain or lose is the MMR difference between you and your opponent. 
In addition, your rank will also affect the MMR but only when you are losing (see explanation below).   

---
    
# Winning  
Winning is easy to calculate because your rank doesn’t matter. Generally, it works how we expected it to be. The higher the MMR you have than your opponent, the less MMR you gain when you win, vice versa (look at the graph below to have a better idea how much). 
If you want to calculate how much points you will gain, just use the formula in the graph below. 

![Graph of mmr when winning]({{ site.url }}/assets/img/gwent_rank_win.png)

---

# Losing
Losing is much harder to calculate because your rank matters. The higher the rank you have, the more point you will lose. I have two graphs here showing the difference. 
Even though these two graphs have a similar pattern, we can see that when facing the same MMR opponent, we lose around 33 points at rank 6, but 43 points at rank 14. Much more data at different rank will be needed to verify how it really works, but it is too time-consuming for me so I stopped after collecting around 130 games at these two different rank.  

![Graph of mmr when losing at rank6]({{ site.url }}/assets/img/gwent_rank_loss_6.png)
![Graph of mmr when losing at rank14]({{ site.url }}/assets/img/gwent_rank_loss_14.png)

---

# Draw
Draw (or a disconnect that led to a draw), depends on whether you have higher or lower MMR than your opponent. If you have higher MMR, you will lose points. 
However, the points you lose will be much lower when compared to winning/losing. Vice versa, you gain points when you have lower MMR. 

[*Raw data is here if you needed it.*][raw-data] 

---

# Conclusion 
- One major factor that affects the MMR/points you gain/lose is the MMR difference between you and your opponent.
- Calculating the points when you win is easy because it is the same at all 15 ranks.
- Losing is different, you lose more point when you are at a higher rank, which mean climbing at lower rank is easier.
- Win rate to climb at around rank 6: ~40.5%
- Win rate to climb at around rank 14: ~46.7%

# Extra 
I climbed to top 300 and rank 15 (highest rank during beta) in two weeks >.<

![Top 300]({{ site.url }}/assets/img/gwent_top_300.png)


[raw-data]: https://docs.google.com/spreadsheets/d/1Q7Wi6Go1ahuDGGp2q5DeuFGslv2eIEoNWq_X62edY5M/edit#gid=0