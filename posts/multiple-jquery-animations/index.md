As I was working on a website I am developing for a friend of mine, I realized I had a problem with multiple jQuery animations slowing the browser down. This statement by itself may seem like a no-brainer, but it is not as obvious as you may think. The thing is that these animations were not *simultaneously* occurring. Rather, they were being used to fade a div element in (which serves as a popup) and then out again when the user closes it. The code originally looked something like this:

```javascript
function openPopup(popup) {
    $(popup).show();
    $(popup).animate({opacity: 1});
}

function closePopup(popup) {
    $(popup).animate({opacity: 0}, function() {
       $(popup).hide();
    });
}
```

The problem I was encountering was that after a few times of the popups being opened and closed, the fade-ins and fade-outs began to lag severely and just got exponentially worse the more times they were opened and closed. What confused me at first was the mistaken belief that the animations were somehow finished after the jQuery animate function had completed its opacity change (even the “hide()” method had been triggered in the closePopup() function above). It seems however, they were not ending entirely, causing a build-up of continuous jQuery animation. My first attempt to solve the problem was the following:

```javascript
function openPopup(popup) {
    $(popup).show();
    $(popup).animate({opacity: 1}, function() {
       $(popup).stop();
    });
 }

function closePopup(popup) {
    $(popup).stop().animate({opacity: 0}, function() {
       $(popup).stop();
       $(popup).hide();
    });
 }
```

Essentially, after the animation had completed, I was calling the jQuery stop() function to make sure the animation was completely and entirely finished to make sure that the browser’s memory usage would not skyrocket. This did not solve the problem at all, however. The animations continued to lag after a while, but, oddly enough, there was really never a significant increase in the memory used by the browser like there is if you have too many timers running at once. This confused me, so I took to the internet to see if anyone else had a solution. After scrounging around the internet for a while looking for a solution, I finally stumbled into a simple one which solved the problems I was having:

```javascript
function openPopup(popup) {
    $(popup).show();
    $(popup).stop().animate({opacity: 1});
 }

function closePopup(popup) {
    $(popup).stop().animate({opacity: 0}, function() {
       $(popup).hide();
    });
 }
```

Simply by calling stop() before starting the new animation, it guaranteed that only one animation at a time would be running for that element. I’m not quite sure I understand this entirely because it seems to me that in order for this to work, the animation would still have to be running in the first place. It does work though. I suppose it makes sure that all previous animations are stopped before executing the next one, but what is confusing about that reasoning is that my previous attempt with calling the stop() function directly after the animation had completed (see above) should have worked. Anyone have any ideas as to why this works? I haven’t found anything on the internet other than that it *does* work.