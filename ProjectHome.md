Recently I had the need to truncate a section of styled HTML text so it would fit inside a container with a specific pixel width. You might have thought this would be something pretty easy to make happen with a little JQuery and some .css() and .width() commands.

You would have been mistaken. I know I was.

Among the tricky aspects was my desire that the text retain its styling, regardless of the width of the container. I also wanted it to truncate at the appropriate complete letter, rather than chop a word off in the middle of a letter. I wanted to be able to specify a string to use to indicate truncation (eg: “…”). I wanted the complete text to be preserved, and added to the element as a title tag so that it would be available to readers who hovered over the truncated text. And of course, I wanted it to work seamlessly across browsers.

What I decided I needed was a temporary DOM object where I could copy the styled text. I needed something that would report its width based on the width of the text it contained. That ruled out both block elements such as divs (which take their width from their container) and inline elements such as spans (which report zero when queried about their width, regardless of their actual contents). What to do?

Then I realized that tables adjust their width to the width of their contents, AND are able to report that width when queried. Eureka! I just needed to add a temporary table to the DOM somewhere that it would be untouched by other styling, and make sure all the elements that made up the table had no padding, margin, or border settings. I didn’t need to worry about the semantics of using a table this way since I was going to remove it from the DOM as soon as I was done with it. You won’t hear me saying things like that very often, but in this case even I was satisfied.

To copy the styled text, I had to take a two-step approach. After creating a temporary table, I copied the contents of the original element I wanted to measure, and then copied each of the original element’s font-relates styles and applied them to the inner td of the table. Firefox immediately rewarded me with an accurate measurement, which I could use in a recursive loop to chop away one letter at a time until I had the width I desired. It looked like I was going to be done with this one quickly.

I replaced the contents of the original element with the truncated text easily. I tossed in a call to cache the full text and apply it as a title attribute if the text needed truncating. I even made a quick swap of the truncation indicator text (”…”) into the styled table cell from the start, so I could account for the pixels that added to the width. It was looking pretty sweet.

Until I tested it in Internet Explorer.

Now the creepy thing was that it even worked in Internet Explorer with the font size set to the default value. But as soon as I tried font sizes that were larger or smaller, I started seeing the width vary in odd ways. Larger fonts would result in a reported width that was wider by an ever-increasing factor, resulting in truncated strings that were too short for the space. Smaller fonts did the opposite, reporting a narrower width than they actually required, resulting in truncated strings too long to fit the desired width. It wasn’t a straight line rate of change; I could see there was some mysterious equation at work here.

I will admit it took me a few hours of fiddling, and I had to dust off that part of my brain that was paying attention during high school algebra. What I discovered (and this is the golden nugget in this chicken soup of a blog posting) was that Microsoft calculates the pixel size of rendered text based on the square root of the desired font size divided by a base font size of 16.

The calculations gave me a result I could work with. You can see them in action in my final set of JQuery plugins: textTruncate() and textWidth().