This is a project I initiated for fun. It is inspired by one of the projects I undertook during my PhD [https://journals.asm.org/doi/full/10.1128/mSystems.00181-18]. 

In that project I used the genomic content of microbes to predict the outcome of a pair-wise interaction.

Here, I extend that logic to Magic the Gathering limited deck construction. I use data made available by [https://www.17lands.com/public_datasets]

I used the SNC PremierDraft data (it's a large file, so you'll have to download it yourself if you want to replicate this.)

The first part of the script parses the SNC data to produce a data matrix that looks like this:
<img width="935" alt="Screen Shot 2022-07-05 at 11 31 53 PM" src="https://user-images.githubusercontent.com/36259203/177462875-aca06c75-e639-43ec-ac19-524e0532a919.png">
 
Each row corresponds to a unique deck that was built and played in Premier Draft on MTG Arena.
Each column, except the last, is the name of one of the cards in the SNC set. Each value is the average number of copies the player chose to play
in their deck during their run.
The last column is the win rate of the deck at the end of it's run. This is the response variable and what I'm trying to predict in this project.

In the file I downloaded there were 45,399 records available. 

The underlying assumption I make is that if an average player could play a given deck an infinite number of times, they would converge on a "true" winrate for the deck.

Since a premier draft run ends when a deck either accumulates 7 wins or 3 losses, we won't obtain a large enough sample size for any single deck to determine what it's "true" win rate is.
As a result, a regression model can be expected to have very large residuals and poor correlation. However, by comparing a deck's composition against historical observations we may be able to predict the "true" win rate of the deck after all.
The idea is that even though our model will generate a point estimate for the win rate of a deck that is potentially quite far from the observed win rate for any given deck, if we aggregate the results of all decks predicted to be in the same range (e.g. predicted to win 63% +/-1% of the time), if the model is correct the observed mean win rate will be quite close to the mean predicted win rate.

Such a model could be used to guide card selection during the drafting process.

I originally went straight to random forest, as it's my go to model and very good when significant interactions lurk in the data.
After revisiting this project, I decided I should try linear regression - since that really should be the starting point anyway. 

To start I defined a training set and a test set using 20% (9,079 records)of the available data to train and holding the remaing 80% to test the predictions on.
The figure below is the result:
![LR_mtg](https://user-images.githubusercontent.com/36259203/177468935-2e201547-36f6-40d3-80e5-06acc11c1a57.png)

On the X-axis are the predicted win rates generate by the model. On the Y-axis are the observed win rates. As I expected the variance is very high, and the R^2 is .07 (displayed on top of the figure.)

What happens if we aggregate prediction bins? That is, if we collect each sample that the model predicted would be between 40-42% win rate, what is the average win rate inside that bin? Some bins won't have any obsrvations falling inside so I'll omit those. Among those bins that had 5 or more observations inside, here is what happens:


![lr_test](https://user-images.githubusercontent.com/36259203/177469676-e50fa7ce-c31d-43ec-9803-336d4aa3d209.png)

Suddenly the predictions are really, really good! What if I use 80% of the available data to predict the remaining 20%?

![points](https://user-images.githubusercontent.com/36259203/177469906-b33e856c-759d-42f4-b961-fd8e5457a679.png)
The point estimates are still terrible.


![population](https://user-images.githubusercontent.com/36259203/177470142-9af50106-c8aa-4c7d-8d00-f712b9cf5552.png)

And the population level estimates are basically the same.

At some point I'd like to clean this code up so that it's more readable, and implement a learning curve so we can see how much data we really need before we can start generating useful models. A learning curve will also help identify diminishing returns.

Other interesting questions to address:
How valid are these models as time goes on? It takes time for players to figure out which strategies are the best, so willa model that is trained on data from the initial release of a set still be good on the last week before rotation?
How soon can statistical models be used to solve a given format?
