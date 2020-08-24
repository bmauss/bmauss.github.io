---
layout: post
title:      "A Tale of Two Models"
date:       2020-08-24 01:12:10 +0000
permalink:  a_tale_of_two_models
---


T'was the best of models.  T'was the worst of models.  Before we get ahead of ourselves, let's start from the beginning.

It kinda started like most heist movies.  I see a project proposal.  The objective seemed straightforward enough:  Use the Seattle Terry Stops data set to build a model that can predict if an arrest will occur during a "Terry Stop".  Sweet and simple (conceptually), just the way I like it.  Then (like in most heist movies) it all hit the fan when I looked at the dataset...

![Imgur](https://i.imgur.com/BuXvbTS.png)

Do you see the placeholders???  Would you believe me if I told you that between 60-70% of the rows in this data set have either a placeholder or NaN value?  You should, because it's true.  Cleaning that data changed me, man.  You can't just drop all of that data!  And if it wasn't the placeholders, it was the vague descriptions of each of the columns and their values.  In addition to some feature engineering, something had to be done to clear up some of the confusing and redundant data.  For example, how could and an encounter with a police officer have a "Stop Resolution" of "**Arrest**" but under "Arrested" you see "**No**"?  Well, it's because there are two types of arrests: **Non-custodial** and **Custodial**! Non-custodial arrest is when you get a ticket and a Custodial (or physical arrest) is when they put you in cuffs.  At least that one I could fix.  All of the values that said "Arrest" and weren't **flagged** as *arrested* were relabeled as "non-custodial" and then I went on my merry way.  Sadly, the other vague Stop Resolutions (like Field Contact, which ranges from community outreach to breaking up a fight and frisking the suspects) couldn't be fixed as neatly.

Then it came time to modeling and four out of six of the models were amazing!  I finally decided that Random Forest was the best one for the dataset and just look at these scores!

![Imgur](https://i.imgur.com/VLRFAo6.png)

Considering the dataset I had to work with, this is probably too good to be true, right?  That's what I thought.  So I printed out the confusion matrix for the test data.

![Imgur](https://i.imgur.com/Y91fmtj.png)

Only a single false negative!  Crazy!  Makes you wonder about the Feature Importance, right?

![Imgur](https://i.imgur.com/HonSLV6.png)

Huh... Seems to really be relying on the "Stop Resolution".  I compared this with the feature importance of the other models and all of the high performing ones ONLY relied on "Stop Resolution".  That's when I realized what happened.  By relabeling all of those tickets as "non-custodial", everything labeled "Arrest" then had a 100% correlation to the target variable.  That's why it was performing so well.  

This got me thinking, what if I removed that feature entirely and reran the model?

![Imgur](https://i.imgur.com/gNCCJT3.png)

Boom!  But where is that accuracy coming from?

![Imgur](https://i.imgur.com/fOO0FDv.png)

So we went from a model with that let 1 guilty man go free, to a model that arrests everyone and let's the courts sort them out.  How did the Feature Importance turn out?

![Imgur](https://i.imgur.com/KST1A5D.png)

Despite it's flaws, it's actually a little more realistic.  If I were to be placing bets on if someone was gonna get arrested, my money would also be on the repeated offender.  And as we've seen with 2020's recent spike in violent crimes, some years are better than others.

Is there a moral to this story? I dunno. Just think it's interesting how the tiniest change can make the biggest difference.
