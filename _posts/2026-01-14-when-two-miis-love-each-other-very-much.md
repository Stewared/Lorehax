---
layout: post
title: "When Two Miis Love Each Other Very Much..."
article_title: "You See Kids, When Two Miis Love Each Other Very Much..."
date: 2026-01-14
thumbnail: assets\/images\/Tomodachi_Love.png
flags: [miis, decompilation]
author: Kestron, WKoA
---
*An analysis, explanation, and recreation of child Mii generation in Tomodachi Life*

## Basic Summary of the Idea
Howdy folks, for those just reading and who don't know about this mildly niche game, let me do a quick summary.
<img src="\assets\images\TomodachiLife.png" alt="The Tomodachi Life title screen." style="width: 50%;float:right;margin:1rem;">


Tomodachi Life is a sequel to the Japan exclusive Tomodachi Collection. In Collection, you follow the lives of Miis as they do things on this island, and it culminates in marriage. Fun fact! Miis were actually originally developed for this game, but ended up being brought over and integrated on a system level into the Wii before it came out. In Tomodachi Life, it goes a step further - married Miis can have babies that will eventually grow up into a fully autonomous resident on the island. Because the game is tied to real-world time (like Animal Crossing), the whole process takes a little over a week, both in-game and in real life. Three days after one of the two Miis ask if they can have a child, one will be born as a newborn. Then each day following it will progress one stage of life further, culminating in "moving out" on day six.

This process was always quite fascinating, as they seem to inherit traits from both parents and be distinctly both parents at once, while also being an entirely individual Mii themselves. As the developer of a library to interact with Miis, I thought it might be a fun thing to include as one of the features you can do with it. And so I started observing the Mii children. Traits were directly copied from one parent in many instances, but the hairstyles seemed to be much more interesting. The hairstyles were always unique, yet similar to one of the parents. I did a lot of research online off and on for a very long time, and never got much further than a potential mapping of female hairstyles by age group by the mother's hairstyle, suggesting that each stage of life came with a progression of groups hairstyles could be pulled from.

As interesting as this table was, I couldn't verify its authenticity, and it had no equivalent data for male Miis. Additionally, one hairstyle is listed as Unknown entirely. This was where progress on understanding the process stagnated for three years.

Eventually I revisited the idea. There were essentially two options. One was to use save file editing to try and generate statistics and observed information repeatedly, and the other was to decompile the game's code.

I tried the former first. After a couple days of work, the way I was doing it was taking many hours for one singular pass, most of those hours just carefully clicking through the same prompts over, and over, and over, and over, and over again. This was then going to require a script being written to extract information from a save file format neither I nor anyone I can find online comprehensively reverse engineered. This wasn't going to work well.

And then came the latter. With the help of WKoA, after several days, we believe we have a working model that generates children the exact way the game does. Let me break down what we found. Disclaimer, I may have missed things, or misinterpreted bits, but I am somewhat confident this is accurate.

## How Baby Miis Are Formed

Let's start off with a disclaimer. The code we examined was very messy, this is partly compilation optimizations and decompiler confusion, but it's also just straight up messy. Things happen in loops that don't need to, code is over engineered, dead code blocks that are never run are in at least two places. And we only looked at a small portion. We're not going on a line by line of the exact way the code works. I am going to roughly explain some basics from the slightly (or drastically) more efficient flow that is, to our knowledge, 1:1. So the output should be exactly the same on the surface, but the algorithm being followed is cleaned up to be more efficient, make more sense, and flow better, despite having the same output. We are describing this process here, the more efficient one.
<img src="\assets\images\GuessWhatWeHadABaby.png" alt="The two default Miis saying 'Guess what? We had a baby!'" style="width: 50%;float:left;margin:1rem;">
To start, three things are set. The main parent, the matching parent, and a `randomBytes` array. The `randomBytes` array is basically a collection of eight randomly decided 0s and 1s. You might ask why this is eight bytes and not eight bits, I know I did, but that's just how the original does it. These are going to decide on a few of the fields and which parent provides which of them. The matching parent is simply the parent with the same gender as the child, via our current understanding. Then there's the main parent. The main parent is decided by whether `randomBytes` first value is `0` or `1`, and will contribute the larger portion of the facial features. For my replication we copy so much from the `mainParent` that I just make the baby Mii a clone of that parent, and then change the fields that aren't supposed to copy this as necessary. We then clear out several fields that children Miis will never possess. These being beards, facial features such as wrinkles, and makeup. Glasses are a surprising addition to this list but I found no code to apply glasses and I can find no evidence of children Miis wearing glasses online or in my own playthrough. Interestingly, a child Mii's hair is seemingly never flipped. The birthday is, of course, the day this code is being run.

The gender and name can be overridden by the player, but can also be randomly selected. There is a list of potential names by gender, with a subset of names straddling both.

Observationally one, including myself, might assume the skin colors are ordered by how similar one is to the next in a gradient pattern, and an average is selected from between the two parents. This is, however, not the case. There is an entire table deciding which skin colors are valid or invalid based on the parents' skin colors to select from randomly here. I do wonder if the game's process here is lossy, but the table logic used in the recreation should in theory match the original logic.

Now several things are decided based off of `randomBytes`. The third value controls which parent provides the hair color. The eyebrow color will always be the same as the hair color decided here. The fourth decides which parent provides the eye type. The fifth, eye color. Sixth, nose type. Seventh, mouth type. Eighth, mouth color. It's interesting to me that mouth color doesn't just copy the parent providing the mouth type, as  all of the mouths affected by recoloring are the more lipstick like mouths. The first value was used to decide the parent providing more of the facial traits used, but the second value is never used. Whether or not the mole is present is decided by matching to a random parent, and could have been decided by this value.

Two values here are decided exclusively by which parent is the same gender as the resulting child. You would expect these values to be the eye type and the mouth type, due to eyelashes and lipstick, but neither of these values are included. Instead, the two values are... hair style and eyebrow type. Hair style makes sense to me, but eyebrow type? Okay. This is probably related to the same logic that always matches the color of the eyebrows to the selected color for the hair.

Now height and weight get into a bit of wacky math algorithms, but it works seemingly well so.
Height takes the height of a randomly decided parent, and then divides it by eight. It then multiplies it by `1.4`... five times in a row. It then ensures the final height is between `0` and `127`.
Weight takes the weight of a randomly decided parent, and then... adds `1`. This value is divided by `4`, and `48` is added to the total. It then takes this total, and adds the (total minus sixty four) and multiplied by `0.2` to itself... five times. It then adjusts it to ensure it's within the boundaries of `0`-`127`.

We now have two steps left. Hairstyle generation and offsets throughout the younger years. Anything else that has gone unmentioned is because it is cloned from the `mainParent`. This is things like the `position`s, `size`s, `rotation`s, `squash` values, and so on. The types of these features is set randomly, but the positions of those features are set by the parent of the same gender. 

At this point, we generate the offsets for the newborn face. The nose becomes the minimum size. The eyes and eyebrows, are moved up on the face by two, the mouth is moved down by two. All values are double checked to ensure they're within the minimum and maximum values these can be for a Mii's face. We then make four more stages in between, adjusting these values to be spread equally between.

### Hairstyle Generation
Nearly finally, we'll explain how hairstyles work. These are somewhat more complicated and so come with their own section and infographic. First we take the hairstyle of the parent of the same gender, and find it in a giant table mapping which hairstyles go to which groups. Each entry includes two paths to go down depending on which gender the hairstyle came from.
We're now sorted into one of ten different hairstyle groups. Each hairstyle group includes four subgroups. We select one hairstyle randomly from each subgroup.
The first stage's hairstyle selected is from the first subgroup, and the second and third stage's hairstyle is selected from the second subgroup.

However, for the fourth/fifth, and sixth, stages, there is a 33% chance for boys and a 25% chance for girls of not advancing the subgroup they're selecting from. So the fourth/fifth stage of life will pull from the third subgroup, but there is a chance it will pull from the second subgroup instead. When the sixth subgroup comes around, it will pull from the fourth subgroup if none of the chances are hit, the third subgroup if the chance was hit for the fourth/fifth subgroup, and the second subgroup if it was hit for both the fourth/fifth subgroup *and* the sixth subgroup.
<img src="\assets\images\hairstyles.png" alt="A chart of which Mii hairstyles lead to which baby hairstyles." style="width: 100%;">

### Extra Niceties
Lastly, I in my own recreation just did a couple of extra things. Since I was not and had no interest in tracking personality things, I instead had the Mii select a random favorite color based off of the possible clothing colors associated with the different personalities that Mii could've been choosing from. I also made the height value being stored begin at `0` for the newborn stage, and slowly slide up to the full size Mii as it grows up over time. The original just uses custom models and therefore never had to bother with any smaller heights anywhere. Seemingly, the game changes the face type when rendering younger ages, and so for all but the last two stages the face shape is overridden with a smaller rounder face that more closely matches the look of the original.

## Conclusion

And so there you go, that's how baby Miis are generated in Tomodachi Life. You can see this applied to Node.js [here](https://github.com/Stewared/MiiJS/blob/4c30ea9e94ffaebbb77c01dad2bbbedc831702dd/index.js#L1569), and you can see the tables I was referencing converted to JSON [here](https://github.com/Stewared/MiiJS/blob/4c30ea9e94ffaebbb77c01dad2bbbedc831702dd/data.json#L1939). I've built a fully working recreation into our [MiiJS](https://github.com/Stewared/MiiJS) library, which allows you to manipulate Miis in every way I can think of from Node.js.

This was an interesting rabbit hole to fall down. Hairstyle generation in particular is quite thought out, and it's fascinating to me. If you read all the way through, I hope you enjoyed this deep dive as much as I did.

Here's an example child generated from this function we've been recreating. Guest B and E are two of the generic Miis that have been around since the Wii, the resulting "Stella" was fully generated using our brand new recreation function and rendered using a reverse engineered FFL library. Or in other words, Stella has never been on Nintendo hardware, not even for rendering or QR code generation. She was fully generated and rendered by MiiJS code.
<img src="\assets\images\CustomChildGenerationOutput.png" alt="An image of Stella as she grows up over six stages of life, and the parents that she was generated from." style="width: 100%;">