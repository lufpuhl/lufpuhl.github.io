---
layout: post
title:  "Value Mask"
date:   2019-01-24
excerpt: "This is an informal article about a technique that can be used to evaluate the visual presences of the elements of your game."
tag:
- technique
- composition
comments: false
---

## Introduction

This is an informal article about a technique that can be used to evaluate the presence of the visual elements in your game. There are plenty of things that change this visual presence like motion, color contrast, line composition, outlines and so on. Value contrast is one of them.

## Definitions

In this article, every time the word Value is written with capital ‘V’, it is referencing the third component from HSV[^wikiHSV] color representation, V or Value.

## Context

On video games, visual readability is something that a lot of people do not take into account when evaluating how good a game is. However, this subject is as important as how good or responsive a controller is.

Games with great visual composition are very comfortable to play, helping you understand what is happening on the screen. One great example, for me, is TowerFall Ascension[^TFA]: a fast-paced game on which the colors were so well chosen, that, even with tons of arrows flying around, you can still keep track of your character and your opponents' positions.

Another game that has changed a lot in order to improve its readability is League of Legends. There is a beautiful technical presentation about how they create and direct their VFX creation[^vfxLOL].

## Color Value

One of the techniques you can use in order to evaluate the visual presence of elements in your game it is the Value contrast analysis. Depending on the compositions, the elements with higher Value would bring more focus to them, especially when put side by side with other low Value elements. An example is having a  special attack (high Value) flying above the background (low Value) getting more focus because of the Value contrast.

The Value of a color is obtained by the highest number among its Red (R), Blue (B) and Green (G) color channels[^wikiHSV]. The following line of code shows how to obtain it from a variable called **Color**.

```glsl
Value = max(Color.R, Color.G, Color.B)
```

The image below shows a visual example of this line in action.

<figure>
	<center><img src="/assets/img/value_mask/color_value_calculus.png"></center>
	<center><figcaption>Example of how to obtain the color Value.</figcaption></center>
</figure>


## Post-processing Effects

In order to verify the Value distribution among the elements of our game, i.e. to see the Value of each color on the screen, a Value Mask can be used. However, before explaining how the mask works, a brief introduction about what a post-processing effect may be necessary.

In games, they are effects that are added after having the entire information about how the rendered scene should be, but before actually showing it on the screen. Great examples are bloom, anti-aliasing, fog, chroma aberration, motion blur, lens flare.

<figure class="half">
    <img src="/assets/img/value_mask/post-processing.png">
    <center><figcaption>Left image: How the sphere should look on display. Right image: how it is shown after the bloom post-processing effect being applied.</figcaption></center>
</figure>

## Technique: Value Mask

Therefore, the Value Mask is a post-processing effect. To apply it, after having the complete scene rendered, you can take the every pixel color and swap it to its own pixel color Value before it goes to the screen. Pseudocode for the effect:

What we have:
- The color of each pixel that would have on the screen. As we process one pixel at a time, the variable that contains the current pixel color will be called **PixelColor** in RGB;
- A function that is called for each pixel before it goes to the screen called **ProcessPixelBeforeGoingToDisplay**.

``` glsl
//Called for each pixel on the screen
//Returns the pixel color that will be on the screen
Color ProcessPixelBeforeGoingToDisplay(Color PixelColor)
{
    Color outputPixelColor;

    pixelColorValue = max(PixelColor.R, PixelColor.G, PixelColor.B);

    outputPixelColor.R = pixelColorValue;
    outputPixelColor.G = pixelColorValue;
    outputPixelColor.B = pixelColorValue;

    return outputPixelColor;
}
```

So, the new pixel color will range from black to white based on its color value and the result will be the following one.

<figure class="half">
    <center><img src="/assets/img/value_mask/tower_fall_00.png"></center>
    <center><img src="/assets/img/value_mask/towerfall_00_value_mask.png"></center>
    <center><figcaption>Normal image (top). Image with Value mask effect applied (bottom).</figcaption></center>
</figure>

This image above is from the game mentioned before, Towerfall Ascension [^TFA].

## Improvement: Value Heat Map

One problem that the value mask does not solve well, it that is does not help to identify the numbers behind the Value. Thus, to deal with this problem, instead of taking the pixel color Value and showing it directly, it is possible to use the pixel color Value to sample a color ramp and obtain a Value Heat Map Mask.

<figure>
	 <center><img src="/assets/img/value_mask/color_ramp.png"></center>
	 <center><figcaption>Color ramp versus Value range.</figcaption></center>
</figure>

``` glsl
//Called for each pixel on the screen
//Returns the pixel color that will be on the screen
Color ProcessPixelBeforeGoingToDisplay(Color PixelColor)
{
    Color outputPixelColor;

    pixelColorValue = max(PixelColor.R, PixelColor.G, PixelColor.B);

    //Sampling the color ramp with the current pixel color Value
    outputPixelColor = sampleRamp(ColorRamp, pixelColorValue);

    return outputPixelColor;
}
```

Let's apply these color ramp on the TowerFall[^TFA] image we had before:

<figure class="half">
    <center><img src="/assets/img/value_mask/towerfall_00_value_mask.png"></center>
    <center><img src="/assets/img/value_mask/towerfall_00_value_hm_mask.png"></center>
    <center><figcaption>Value Mask without color ramp (top). Value mask with color ramp - heat map (bottom).</figcaption></center>
</figure>

With the color ramp, it is easier to see that the color Values of some elements of the game:
* Arrows have a reddish color on the heat map -> Color values around 250
* Walkable tiles have a greenish color on the heat map -> Color values around 130
* Background has a bluish color on the heat map -> Color values around 30

Moreover, if necessary, you can have multiple color ramps, in order to observe specific color value ranges. If by your game design rules, you should only use the value range from 200 to 250 for UI elements, you could use a gray ramp from 0 to 200 and only add colors from 200 to 250 in order to check if any other visual element that is not respecting this rule.

## Final considerations

As the Value Mask is a post-processing effect, it can be applied to a game in real-time, i.e. you can apply it while you are playing the game or even on your engine editor (e.g. Unity). This makes easier to verify the Value composition of a game project because you can see and tweak values without the need for building or exporting images for further analysis.

Be creative with your value heat maps. They may help a lot to both analyse specific value ranges or its full spectrum of values.

Finally, as said before, there are a lot of techniques that balance the visual presence of games and Value contrast is only one of them. Feel free to use the stuff I wrote above in your projects or as an inspiration for creating more visual analysis techniques :).


## References
[^wikiHSV]: [https://en.wikipedia.org/wiki/HSL_and_HSV](https://en.wikipedia.org/wiki/HSL_and_HSV)

[^TFA]: [http://www.towerfall-game.com/](http://www.towerfall-game.com/)

[^vfxLOL]: [https://nexus.leagueoflegends.com/en-us/2017/10/dev-leagues-vfx-style-guide/]( https://nexus.leagueoflegends.com/en-us/2017/10/dev-leagues-vfx-style-guide/)
