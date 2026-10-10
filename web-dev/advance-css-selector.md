# CSS Selectors and Filters

## jQuery-only selector extensions

These are **not** valid CSS, they only work inside jQuery selectors (e.g. `$("li:even")`), and most of them are deprecated since jQuery 3.4 in favor of methods like `.first()`, `.last()`, `.even()`, `.odd()`, `.eq()`.

```css
/* only get the first matched element of css-selector */
css-selector:first

/* only get the last matched element */
css-selector:last

/* only apply to even index elements (0-based index, so 1st, 3rd, 5th...) */
css-selector:even

/* only apply to odd index elements (0-based index, so 2nd, 4th, 6th...) */
css-selector:odd

/* select elements that have at least one child node (including text) */
css-selector:parent

/* select elements whose text contains the given text */
p:contains('specific word')
```

## Standard CSS pseudo-classes

```css
/* apply to css-selector elements that are the first child of their parent */
css-selector:first-child

/* apply to css-selector elements that are the last element of their type (tag name) among their siblings */
css-selector:last-of-type

/* CSS equivalents of :even / :odd (1-based!) */
css-selector:nth-child(even)
css-selector:nth-child(odd)
```