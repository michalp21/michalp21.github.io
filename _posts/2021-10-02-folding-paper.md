---
layout: post
title: "Folding Paper"
categories: article
tags: exponential-growth
---

> I argue that the so-called folding paper puzzle fails to say anything about our ability to comprehend exponential growth.

<!--more-->

## Setup
A common illustration of exponential growth begins with a piece of paper:

> Q1: If you fold a piece of paper 50 times, how tall would it be?

You might guess a few feet, but if you do the math, the result is pretty surprising. Assuming a sheet of paper is .1 mm thick, then $$.0000001*2^{50} = 112589990.684 \text{km}$$ which is about three-quarters the distance from the earth to the sun. Wow!

Is it really true that humans are incapable of thinking about exponents? Are we forever doomed to linear thinking? Here's my thoughts as to why it's a bit overblown.

## Folding Bad
First, semantics. 

Assuming a US Letter size sheet, aka "printer paper," the dimensions are standardized at 8.5 x 11 in, or 215.9 x 279.4 mm. The only reputable source I could find for thickness was this random [Epson support page](https://files.support.epson.com/docid/cpd4/cpd43251/source/printers/source/specifications/reference/scp800/spex_paper_printer_scp800.html), claiming roughly .1 mm. People have experience folding a piece of paper in half, let's say at most five times, and assuming a typical folding pattern[^fold], the final dimensions of the folded paper are 53.975 x 34.925 x 3.2 mm. While the thickness is noticeably larger than before, we can see the width and length are still over ten times as large as the thickness.

The truth is, the paper is already difficult to fold. After a couple more folds, the paper cannot be folded without tearing at the crease. At eight folds, the thickness has already surpassed both width and height (13.49375 x 17.4625 x 25.6 mm). We started by creasing a flat object, but soon we're slicing a paper tower on the vertical axis and stacking the halves on top of each other.[^decay]

Some potential problems:
- The concept of folding only works for thin objects! If we were to fold something with any thickness, then we would rarely think to fold it along its longest dimension.
- It might not occur to someone that they are *doubling* the thickness every fold, which is key to understanding the problem, even without reference to exponents.

It's just a bad example. Here's a better question:

> Q2: What is the height of a stack of paper after doubling it 50 times?

Turns out, people will also give a hopelessly incorrect answer for stacking. What gives?

## Linear Case
Maybe it would help to look at a question with the same answer, but removing any exponentiation.

> Q3: How high would a stack of one trillion papers be?

In general, for any exponential question where the stack is doubled $$n$$ times, we can ask a linear question about a stack of $$2^n$$ papers.

Some of the people I asked this question gave a wide range of estimates, ranging from the height of a skyscraper or Mt. Everest – still massive underestimates. These people can imagine the thickness of a single sheet, but seem to be using their gut, instead of running any kind of calculations, to extend this knowledge to the height of a large stack.

Some clever people will make much better guesses for Q3 than Q2. Let's call a **ballpark answer** one that's the around the correct order of magnitude. The trick is, a ballpark answer doesn't require hardly any calculation besides keeping track of zeros (powers of 10). A trillion has 12 zeros, and the thickness of paper has -1 zero in millimeters. Just add the zeros. In contrast, most people cannot exponentiate in their head without repeated multiplication, and there doesn't seem to be a quick way to get the number of digits in the final answer (i.e. get a ballpark answer), without using the log function (which just takes us back to square one, since we lack a way to quickly calculate log).

So with a little thinking, we can easily solve a linear problem, but not the exponential problem. Humans are linear monkeys confirmed?

## Ban Arithmetic
Not so fast! The ease with which we can mentally calculate the correct number of digits in Q3 relies on reasonably-sized inputs. Let's say we had a different linear question where we needed to multiply a number with 8962 digits by another number with 3899 digits. Summing the zeros is no longer trivial (not with my mental math abilities anyway). It would probably still be harder to mentally exponentiate enough times to arrive at the same answer. Soon we start considering efficient exponentiation algorithms, encodings etc...

All that goes against the spirit of the question. It would be a bit disappointing if the phrase "humans are not exponential, but linear, thinkers" boils down to "humans can't exponentiate in their heads, but they can add zeros, sometimes." I don't think anyone can argue that exponentiation is a more complex operation than multiplication or addition. The question is supposed to reveal some structure in our intuition, not elementary observations about math, otherwise we wouldn't be talking about the heights of paper towers at all.

I took it upon myself to come up with a new version of Q3 that discourages mental arithmetic:

> Q4: In TWO SECONDS tell me how high a stack of one trillion papers would be.

## Inverse Case
Fine, now we have linear and exponential versions of the paper-stacking riddle we are about equally bad at answering. Now what?

The previous questions, and especially the first two, were too focused on dumbfounding you with the final answer, warping your perfectly normal inability to perform fast, accurate mental math into some shocking moral about our poor linear minds. There's no reason to construct the question in such a way that the final paper stack reaches three quarters of the distance from the Earth to the sun, other than to a) make absolutely certain you will get the answer incorrect so that b) whichever smart guy asks it can take pleasure in correcting you.

I present a much more modest version (disclaimer 1: it's not mine; disclaimer 2: there's no paper in this one):

> Q5: Imagine a lake, with a lily pad population which doubles every day. Say it takes 50 days for the lily pad to cover a lake. On what day is half the lake covered?

Some might guess around 30 days. In fact, no guessing or math is necessary. The lake is half-covered on the 49th day. Duh.

Q5 avoids asking us to project fifty iterations into the future, so it's not setting us up for failure.[^simplest] Instead of showing us that we *don't* tend to think exponentially and leaving it at that, it gives us a simple, alternative perspective so we *can*: **most of the gains are stuffed at the end**! In the case of daily doubling, *half* the value of *any* given day was obtained within one day.

## Conclusion
You might say, "that's a lot of words just to admit that the puzzle is useful after all." Listen, I didn't say it was useless, just overblown. It isn't necessarilly stated that we *can't* think exponentially, but it's always said with a shrug and a sigh, like some eternal truth. Even worse, we arrive at that conclusion through some paper-folding puzzle, when in reality there's several things going on:
- Exponentiation is inherently more complicated than multiplication and addition.
- We can't intuit even linear approximations if they're literally astronomical.
- We are perfectly capable of thinking exponentially. It just takes some practice.

Alright I'm done criticizing a silly puzzle. Off to more productive things.

## Appendix
Easy Python code to test out the folds yourself:

```python
paper = [215.9, 279.4, .1] # US Letter dimensions in mm
folds = 50                 # change to desired number of folds

for i in range(folds):
    if paper[0] < paper[1]:
        paper[1] /= 2
    else:
        paper[0] /= 2
    paper[2] *= 2

print(paper)
```

[^fold]: I assume folds parallel to the shortest of width $$w$$ and height $$h$$. If $$w<h<2w$$ like US Letter sheets, then folds will always alternate along width and height.
[^decay]: And just for fun, the width and height experience exponential decay: $$215.9*.5^{25}=.00000643\text{mm}$$ and $$279.4*.5^{25}=.00000833\text{mm}$$, both ending up close to the diameter of a DNA strand.
[^simplest]: It's the simplest too, and can technically be solved without even understanding exponentiation. In most cases though, people will visualize the growth of the lillies, leading them to think in some way about it.
