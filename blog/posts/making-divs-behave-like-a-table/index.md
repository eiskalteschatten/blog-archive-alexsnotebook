A couple of days ago whilst working I was trying to figure out how to make a series of DIV tags work like a table. After doing a bit of research online, I [stumbled upon the solution](https://snook.ca/archives/html_and_css/getting_your_di).

Essentially, you need three layers of DIV tags just as you would need at least three layers of TABLE/TR/TD tags. Each DIV tag represents the TABLE, TR, and TD tags, respectively. So, for example:

```html
<div class="table">
  <div class="row">
    <div class="cell"></div>
    <div class="cell"></div>
    <div class="cell"></div>
  </div>
  <div class="row">
    <div class="cell"></div>
    <div class="cell"></div>
    <div class="cell"></div>
  </div>
</div>
```

plus the CSS classes:

```css
.table {
  display: table;
}

.row {
  display: table-row;
}

.cell {
  display: table-cell;
}
```

would create the equivalent of:

```html
<table>
  <tr>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>
```

One thing to be aware of with this method, however, is that it is not compatible with older browsers which don’t support CSS3. Internet Explorer 7 and 8 will, in that case, probably be the primary concern. Older versions of other browsers won’t be as troublesome since users tend to update them more often.

You are probably asking why on earth you would do such a thing with DIVs rather than just use a normal table. The answer to that is mostly a matter of preference, but DIV tags do give you more flexibility when it comes to dynamic webpages. If you want to, for example, dynamically rearrange the page or swap items within the grid, DIVs offer more capability than the traditional table tags and you won’t risk breaking everything since you can simply remove the class from the DIV via jQuery or JavaScript.