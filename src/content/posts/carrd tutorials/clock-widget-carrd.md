---
title: Clock Widget Carrd Tutorial
published: 2025-03-05
description: 'Create a nice clock widget to show your local time/the viewers time'
image: ''
tags: ["tutorial"]
category: 'Carrd'
draft: false 
lang: ''
---

![](src/assets/images/carrd_tutorials/clockEx.PNG)

I recently have been revamping my freelancing website and wanted to implement a simple clock widget to show potential clients what my current time is. I thought it'd be useful tip for others so here's how I did it and how you can adjust it.

To implement this widget, when adding to your carrd site, select embed code. You'll then copy and paste the code below, making changes as you see fit. It's onvly visible on the published site unfortunately, so you'll need to publish the site to see the final product.

:::tip
If your site is already live but you want to test it first, you can always create a new carrd site and give it a temp `carrd.co` name just for testing purposes. It can always be deleted afterwards or held onto for future testing endeavors.
:::

---

First, here's the entire script:

```javascript
<div id="clock-widget" style="background: #FFFFFF; border: 4px solid black; padding: 10px; color: black; font-family: VT323, monospace; text-align: center;">
<span id="clock"></span>
</div>

<script>
function updateClock() {
const clock = document.getElementById('clock');
const now = new Date();
const options = { timeZone: 'America/New_York', hour: '2-digit', minute: '2-digit', second: '2-digit' };
clock.textContent = new Intl.DateTimeFormat('en-US', options).format(now);
}

setInterval(updateClock, 1000);
updateClock(); // Initialize immediately
</script>
```
Now let's go over how can you change this to your liking.

First, let's take a look at this line of code:

`style="background: #FFFFFF; border: 4px solid black; padding: 10px; color: black; font-family: VT323, monospace; text-align: center;"`

Let's break it down:
1. `background` sets the color of the background, here I have it set to white
2. `border` is what adds the black border you see around it
    1. `4px` is the size of the border
    2. `solid` means the border itself is solid, you can try other things like `dash` to have it appear as dashes instead
    3. `black` is the color
3. `padding` is the room between the text and the borders (gotta give it room to breath!)
4. `color` is the color of the font
5. `font-family` is the font used. My site uses `VT323` font from carrd, but I have `monospace` set as a backup font in case it fails for some reason
6. `text-align` is what makes the text align at a certain point, in this case in the center, but you can also use `left` or `right` alginment

Now let's look at how I set the timezone.

FIrstly, if you want it to just show the users timezone, then remove:

```javascript
const options = { timeZone: 'America/New_York', hour: '2-digit', minute: '2-digit', second: '2-digit' };
clock.textContent = new Intl.DateTimeFormat('en-US', options).format(now);
```

and replace it with:

```javascript
clock.textContent = now.toLocaleTimeString();
```

Now if you want to show your current timezone, you just need to change the `timeZone` variable. You can ask `ChatGPT` what your timezone tag would be. The `2-digit` parts is jut what makes it appear as just the first two digits (useful for displaying seconds) and then I use `en-US` as the format for the clock. Again, you can ask `ChatGPT` what to use if you want it to appear differently.

And that's it! Pretty neat huh? Hopefully you understand enough of what's happening to be able to adjust it to your liking :D