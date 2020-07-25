---
layout: post
title:      "Multilinear Regrets"
date:       2020-07-25 23:51:51 +0000
permalink:  multilinear_regrets
---


Despite the title of this post, I actually enjoyed the our project on multilinear regression.  It was both fascinating and frustrating at the same time.  That said, mistakes were made that I'll quickly address that probably would have changed the outcome drastically.  Learn from my mistakes, friends. 


## Regret No.1: Don't be nice to Outliers##

Don't be nice to outliers, guys.  What have they ever done for you? Nothing! They just cause confusion and pain.  We all know they're toxic, so why am I bringing this up?  Well, I was too nice to them.  

At first I was merciless.  I used the IQR approach by defining the first and third quartiles, as well as the interquartile range, then establishing the upper and lower whiskers of the column's box-plot using the following code: 

```
Q1=df[<col>].quantile(0.25)
Q3=df[<col>].quantile(0.75)
IQR=Q3-Q1

Lower_Whisker = Q1–1.5*IQR
Upper_Whisker = Q3+1.5*IQR
```
Then I applied these conditions as follows:
```
df = df[(df[<col>] < Upper_Whisker) & (df[<col>] > Lower_Whisker)]
```
Note that you don't really have to remove the points outside of the lower whisker, I just placed it there.  In fact, in practice, I only eliminated all of the outliers greater than ```Upper_Whisker.``` The result was that I removed a great deal of the outliers, but I went from a dataset of 21,000 to a set of 17,000 (Mind you that even with this much data lost, there were still outliers remaining!).  Losing this much data made me feel uncomfortable, though.  I really wanted to keep my dataset intact as much as possible.  So, instead, I opted for a more gentle approach.

On attempt two, I used the z-score method where you eliminate all data that falls outside of three standard deviations of the mean this would result in eliminating the biggest outliers and still leave me with 99% of my data (a perfect compromise, I thought).  So I employed the following code: 

```from scipy import stats

z = np.abs(stats.zscore(df[<col>]))

df1 = df[(z < 3)]```

What were my results? 

![Imgur](https://i.imgur.com/3uAGJUo.png)

This is just one example of the outliers being "removed".  This should have been a big red flag, but I was young and naive.  "This data IS within three standard deviations, afterall," I said, trying to justify myself.  "Maybe it will give me some insights that I wouldn't have gotten otherwise!"

![Imgur](https://i.imgur.com/C6rOqEH.png)
QQ-plot

![Imgur](https://i.imgur.com/4y9z94Q.png)
A Heteroscedastic Nightmare

This is what I get for being nice.  The lesson here is that outliers don't care about your model or your feelings.  Just get rid of them and don't feel bad about it.

## Regret No. 2: More Method, Less Madness

I chose to try the OSEMN approach to data science (or at least attempted) because it seemed more streamlined and organic to me.  Don't get me wrong, I'm not blaming my short comings on the OSEMN method, but rather that I, as a person who is new to the magical world of Data Science, would have benefitted more from a more structured approach like CRISP-DM, or even KDD.  I just feel that halfway through the project, I got the sinking feeling that I had deviated from the process that I had chosen and that my model was suffering as a result.  

## Regret No. 3: Transform, then Normalize

Again, this is obvious, but I was a dum-dum last week. 

![Imgur](https://i.imgur.com/aaOtbah.gifv)

In my haste to rectify my model, I mixed up the steps in the workflow and normalized my continuous variables before transforming them.  This resulted in the log transformation achieving very little, which, in turn, made me misinterpret the results and think that the transformation was useless.  We've all been there.  Don't judge me.  

## Conclusion:

You can't say "outliers" without saying "liars".  They lie and confuse your model.  Get rid of them and don't feel bad about it.  Your model will turn out better and you'll be happier.

Don't get so caught up in your own thing that you fail to adhere to the scientific processes.  You'll save time (and money).

Lastly, keep calm.  I mentioned that I mixed up the workflow because I was panicking.  So when things aren't looking right, just keep calm and code on.



