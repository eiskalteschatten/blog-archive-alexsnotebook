<figure><img loading="lazy" decoding="async" src="caspar-camille-rubin-89xuP-XmyrA-unsplash.jpg" alt=""></figure>

I recently wrote a small script using TypeScript that can be used to determine the size of the view port using the breakpoints as defined by Bootstrap. The script was written with the intention of using it with React in order to hide or show elements without using CSS so that the element never even shows up in the DOM.

```javascript
// View port widths from Bootstrap
export const xs = 0;
export const sm = 576;
export const md = 768;
export const lg = 992;
export const xl = 1200;
export const vw = Math.max(document.documentElement.clientWidth || 0, window.innerWidth || 0);

export const isSmDown = (): boolean => vw <= sm;
export const isMdDown = (): boolean => vw <= md;
export const isLgDown = (): boolean => vw <= lg;

export const isSmUp = (): boolean => vw >= sm;
export const isMdUp = (): boolean => vw >= md;
export const isLgUp = (): boolean => vw >= lg;

export const isXs = (): boolean => vw <= xs;
export const isSm = (): boolean => vw >= sm && vw < md;
export const isMd = (): boolean => vw >= md && vw < lg;
export const isLg = (): boolean => vw >= lg && vw < xl;
export const isXl = (): boolean => vw >= xl;
```

I don’t know why WordPress decided to escape the “&&” on lines 18-20 when I specifically chose a code block, but whatever.

The script is also available as a [GitHub Gist](https://gist.github.com/eiskalteschatten/7e22105d6cd9e6ea37c05b0c80aadd6d).