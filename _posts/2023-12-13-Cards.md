---
title: 'The Unintuitive Math Behind "Play Your Cards Right"'
date: 2023-12-13 12:13:14
featured_image: '/images/cards/pub.png'
featured_image_h: 50
featured_image_w: 50
excerpt: A quick post on estimating a probability of winning a rather simple card game.
---


# The Unintuitive Math Behind 'Play Your Cards Right'

## Let's get quizzical
It was a typical evening at a London pub, filled with the hum of conversations and the clinking of glasses. I was there for a pub quiz, confident in my knowledge of random trivia. However, as the night unfolded, I discovered two surprising things: first, my trivia knowledge was not as vast as I thought, and second, an intriguing card game caught my attention.

After the quiz was over, they invited a handful of people to play a game they called "Play Your Cards Right". The rules were straightforward; guess whether the next card in the deck is higher or lower. Do it 12 times in a row and you win a few dozen pounds.

Despite such seemingly simple rules, it sounded almost impossible to guess this correctly 12 times in a row. And, indeed, no one won that evening. But the small size of the pot suggested it should be a relatively likely occurrence. Each play was essentially 1 pound and the unwon pot was rolled over to the next quiz night. If the chance of winning the game was as low as I expected, the pot would need to be significantly larger than £80. So what actually is the chance of winning this game?

## Initial Impressions: Simplifying the Odds
Let's start by massively oversimplifying and say that each guess would be a coin flip with a 50:50 chance. The probability of getting 12 coin flips right is a mere 1 in 4096, or 0.024%.

However, if we look at the card ranks, we can start seeing that 50:50 is basically the worst case scenario that happens very rarely.

    Card ranks:
    A 2 3 4 5 6 7 8 9 10 J Q K

You only have a coinflip situation when you get a 7. If you get a King or an Ace (ace has a value of 1 in this version of the game), you have almost a 100% chance of getting it right. (One little detail about the rules: If the next card is equal to yours, tough luck, you just lose). Also, note the symmetry. The probability of winning when drawing a 3 and guessing higher is exactly the same as drawing a J and guessing lower.

So making use of the symmetry, let's estimate the expected case as drawing a 3 or 4 (or 10 or J as per symmetry) and then guesstimate the chance of getting 1 round right as ~75%. So to do that 12 times in a row, the probability is: 0.75^12 = ~3%. So roughly 1:33 chance.

## Diving Deeper
3% chance of a win seems a lot closer to what the pot size suggested that night. But I am not confident about this result with all the simplifications. So let's add a bit more reasoning and math and look at the probability of guessing correctly for the very first card, conditioned on the rank of that card.

    Guess higher:
    A => (51 - 0*4 - 3) / 51 = 0.9411764706 (the 3 remaining aces make you lose)
    2 => (51 - 1*4 - 3) / 51 = 0.862745098 (the 4 remaining aces + 3 remaining 2s make you lose)
    3 => (51 - 2*4 - 3) / 51 = 0.7843137255 (you get the gist...)
    4 => (51 - 3*4 - 3) / 51 = 0.7058823529
    5 => (51 - 4*4 - 3) / 51 = 0.6274509804
    6 => (51 - 5*4 - 3) / 51 = 0.5490196078

    Doesn't matter:
    7 => (51 - 6*4 - 3) / 51 = 0.4705882353

    Guess lower:
    8 => (51 - 5*4 - 3) / 51 = 0.5490196078
    9 => (51 - 4*4 - 3) / 51 = 0.6274509804
    10 => (51 - 3*4 - 3) / 51 = 0.7058823529
    J => (51 - 2*4 - 3) / 51 = 0.7843137255
    Q => (51 - 1*4 - 3) / 51 = 0.862745098
    K => (51 - 0*4 - 3) / 51 = 0.9411764706

    Expected: 0.7239819004
    ^12 = 0.0207363592 => 2.07%

If we just reset the deck every turn, the chance to win would be ~2%. But removing the cards can have a non-negligible effect. If you draw a 3, guess higher, then draw an 8, your chance of getting a lower card now will be 8 => (50 - 5*4 - 3 - 1) / 50 = 0.52, as we've also removed the lower card that you already guessed. The fact that the first card was a 3 however has no effect if the next card you draw is a 5. But missing both the 3 and 5 will have a rather negative effect if the next card is 8.

## Let's Simulate
You can see that when considering the cards removed from the deck the tree of possibilities for 12 rounds becomes immense. Someone good with math and a lot more patient can probably calculate the exact probabilities, but not me. So I resort to code.

To estimate the probability of a win, I'm going to simulate a large number of these games played. I'll simulate the player always making the optimal decision (i.e. picking the more likely outcome based on the cards still left in the deck). And I'll look at how many of the games end up winning after 12 rounds. This kind of probabilistic simulation is a great way to estimate probabilities of events that might be difficult to calculate mathematically.

For those interested in the messy code, see [here](https://github.com/laky/play-your-cards-right/blob/main/simulate.py).

Afterwards, I ran the the game 1,000,000,000 times because that seems like a nice big round number. About 15 hours of productive waiting later, we have the answer: we managed to win 13,475,343 out of 1,000,000,000 games, giving us the **probability of winning as ~1.35%**!

That is very different to the 0.024% I initially used as an estimate. What was your initial guess? Did you have a better intuition about this than me from the start?

## Let's Switch It Up a Bit
Now, there's a tiny further rule. It said that you can discard the first card if you choose to do so. How much does the fact you can discard the initial card help? And what is the optimal strategy for discarding it? I honestly have no intuition in how much effect this rule might have. So let's carry on.

We first need to figure out what cards should be discarded to play optimally. Let's look at the chance of winning conditioned on the first card's rank and on whether we discard the card or keep it. I.e., we compute the chance to win when we get A as a first card and discard vs when we get A as the first card and don't discard. Then we make the algorithm discard all first cards where the probability of success is lower when playing with that card rather than discarding it.

I simulated some quick games in each scenario.

    First card is A, no discard: Won 17339 out of 1000000
    First card is A, discard: Won 13132 out of 1000000
    First card is 2, no discard: Won 15533 out of 1000000
    First card is 2, discard: Won 13364 out of 1000000
    First card is 3, no discard: Won 13986 out of 1000000
    First card is 3, discard: Won 13463 out of 1000000
    First card is 4, no discard: Won 12341 out of 1000000
    First card is 4, discard: Won 13543 out of 1000000
    First card is 5, no discard: Won 11157 out of 1000000
    First card is 5, discard: Won 13510 out of 1000000
    First card is 6, no discard: Won 9993 out of 1000000
    First card is 6, discard: Won 13602 out of 1000000
    First card is 7, no discard: Won 9149 out of 1000000
    First card is 7, discard: Won 13547 out of 1000000
    First card is 8, no discard: Won 10050 out of 1000000
    First card is 8, discard: Won 13548 out of 1000000
    First card is 9, no discard: Won 11315 out of 1000000
    First card is 9, discard: Won 13356 out of 1000000
    First card is 10, no discard: Won 12626 out of 1000000
    First card is 10, discard: Won 13221 out of 1000000
    First card is J, no discard: Won 14025 out of 1000000
    First card is J, discard: Won 13349 out of 1000000
    First card is Q, no discard: Won 15364 out of 1000000
    First card is Q, discard: Won 13429 out of 1000000
    First card is K, no discard: Won 17409 out of 1000000
    First card is K, discard: Won 13312 out of 1000000

So from here, we can see we should discard the first card any time its rank is between 4 and 10. Now, we just implement the rule and play 1,000,000,000 games again to see how it shifts.

A quick 15 hours later, we have our answer. This rule improves the chances of winning a bit, but not that much. We won 14,702,256 out of 1,000,000,000 games => **1.47% chance of win**!

This journey from a casual pub game to a mathematical exploration was unexpected and entertaining for sure. It challenged my assumptions and showcased the power of analytical thinking and computational simulations. Have you had a moment when your intuition and the actual probabilities disagreed?
