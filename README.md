# daf-render-lib

A DOM render library for creating Talmud pages on the web - *the library works, but documentation and testing are in progress!*

The goal of this library is to quickly and easily create an HTML object that contains a fully functioning and native Talmud Page. It can also work for any other purposes that require a similar layout. You only need two lines of code to implement it into your own project.

## Installation

### Release File

### NPM

## Usage


```javascript
const renderer = dafRenderer("#dafGoesHere", options);
renderer.render(mainHTML, innerHTML, outerHTML, "b");
```

Note: If you are providing particularily complex inputs (mainHTML, innerHTML, outerHTML) to the renderer straight from the HTML file, you may need to place the renderer in a 10 ms timeout to give the browser time to parse the HTML. Refer to test.html for an example of this.

### Options
The renderer can take into account a number of customizable options. If these options are not included, then it will default to what you find below. If you want to change anything, you simply need to pass in an object to the dafRenderer as outlined below with your desired changes.
There are six options you can change:
1. *contentWidth*: This option controls how wide the page should be. Everything else will be automatically adapted to fit the content within the designated width.
2. *mainWidth*: This option controls the percentage of the *contentWidth* which the main body of text takes up. For example, if you set this to 50%, then the main body will take up .5 x *contentWidth*, leaving 25% of the width to each of the side bodies of text.
3. *padding*: This determines the padding within the rendered object, not outside of it, specifically the padding between the three different bodies of text. Anywhere where two bodies of text are next to one another, this option will control how far apart the two texts are. If there is vertical space, the vertical padding will control how large that space is, and if there is horinzontal space, the horizontal padding will control how large that space is. 
4. *fontFamily*: This option controls the fonts that the renderer uses. You can use any of the standard web fonts, or Rashi and Vilna (which are included). If you would like to use other fonts, simply make sure to include them in your file strucutre and refer to them here. 
5. *fontSize*: This controls the font size of the different bodies of text. For now, you cannot set the inner and outer font sizes or line heights independently; both the inner and outer side texts use the size specified under `side`.
6. *lineHeight*: This option controls the vertical spacing between lines within a single body of text. Be careful with making this value too small as it may introduce rendering problems.

```javascript
{
  contentWidth: "600px",
  mainWidth: "50%",
  padding: {
    vertical: "10px",
    horizontal: "16px",
  },
  fontFamily: {
    inner: "Rashi",
    outer: "Rashi",
    main: "Vilna"
  },
  fontSize: {
    main: "15px",
    side: "10.5px"
  },
  lineHeight: {
    main: "17px",
    side: "14px",
  },
}
```

### Data Sources

#### Sefaria

#### Talmud.dev API

## How it Works

### Spacers and the DOM
The layout of the Talmud is not easily replicated with the box-model of the web. This is because there is no such thing as *middle* for the CSS [float](https://developer.mozilla.org/en-US/docs/Web/CSS/float) property, or any other kind of ability to allow multiple bodies of text to wrap around one another. This limitation was overcome with a new paradigm we call *spacers*. Spacers take advantage of the wrap-around principles of [flow](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Normal_Flow). When a *right-floated* body of text encounters a *left-floated* element, it will wrap around that element instead of overlapping. Thus, we can make complex shapes out of the text by using multiple spacers to force the text into the shape we want:

![A picture showing three boxes with text wrapping around them](https://github.com/Jutanium/daf-render-lib/blob/master/doc-pictures/Spacers.PNG)

If we use three layers stacked on top of each other, we can then recreate the page in its entirety:

![A picture sketching the spacer over the daf](https://github.com/Jutanium/daf-render-lib/blob/master/doc-pictures/Spacers%20Together.PNG)

We can see above that there are many spacers, but we only need to worry about three of them. We will define them as such:
- Inner Spacer
- Outer Spacer
- Bottom Spacer

Once we have this structure, where there are three layers each with their own spacers, the only thing left is to calculate the dimensions of the spacers listed above. Specifically, it is important to know their heights (you can actually set thier widths to zero, and rely on floating them left or right). 


### Algorithm
The algorithim has two stages. 

In order to understand both stages, we must understand that there are three possible layouts that are possible:

- Double-Wrap
- Stairs
- Double-Extend

![A picture depicting all three cases](https://github.com/Jutanium/daf-render-lib/blob/master/doc-pictures/Three%20Cases.PNG)



The first stage of the algorithim determines which layout the current page is. In order to do this there are three-five steps:

1. First we calculate the area that each body of text occupies (in terms of px^2).
2. Second we divide the calculated area by each text's respective width to get an expected height.

![A picture depicting Area](https://github.com/Jutanium/daf-render-lib/blob/master/doc-pictures/Area%20Labels.PNG)

3. Third we compare these expected heights. If the main text has the smallest height then we know that we are dealing with the case of **Double-Wrap**.

![A picture depicting a comparison](https://github.com/Jutanium/daf-render-lib/blob/master/doc-pictures/Comparing%20Areas.PNG)

4. If the main text is not smallest we then add the areas of the two smallest texts and divide that by their added widths to get a new height.

![A picture depicting a comparison](https://github.com/Jutanium/daf-render-lib/blob/master/doc-pictures/Adding%20Areas.PNG)

5. We then compare the new height to the largest expected height. If the new height is smaller than the largest expected height then we are dealing with the case of **Stairs**. If not, we are dealing with the case of **Double-Extend**.

The second stage of the algorithim calculates the spacer heights based on the type of layout the page is.
This stage requires only three things:
- The type of layout
- The calculated area values from before
- The padding values that we can optionally apply

We will divide the respective calculations into three parts corresponding to the three kinds of layouts:


For the case of Double-Wrap:
- Inner Spacer = Main Area / Main Width
- Outer Spacer = Main Area / Main Width
- End Spacer = (Inner or Outer Area - (Main Area / Main Width) * Side Width) / Top Width


For the case of Stairs:
- Inner or Outer Spacer = Stair Area / Side Width
- Outer or Inner Spacer = (Main Area + Stair Area - Padding Area) / Combined Width
  - Padding Area = (Combined Height - Stair Height) * Horizontal Padding
- End Spacer = 0


For the case of Double Extend:
- Inner Spacer = Inner Area / Side Width
- Outer Spacer = Outer Area / Side Width
- End Spacer = 0




## License
  
MIT License

Copyright (c) 2020 Dan Jutan Shaun Regenbaum

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## Development
Local setup, contributions, issues guidelines etc.


