# Styled Media Queries: media queries for `styled-components`

The `mediaQuery` function provided by this module extends [Styled Components](https://www.styled-components.com), giving it **a helper for managing media queries**.

## How does it work?

Here's a quick example:

1. Start by defining your globally-shared media queries. We'll use a static JavaScript object to do this and we will use variable names of our own choosing.
  ```JS
// sharedFile.js

/* You could also name this object "respondTo", "atMedia", "nomNoms", etc. */
export const media = {
  smallOnly: "",
  medium: "",
  large: "",
  wide: "",
  extraWide: ""
};
```
2. Then import the `styled-media-queries` module and use it define the media query "features" we want for each name.
  ```JS
// sharedFile.js
import { mediaQuery } from 'styled-media-queries';

// Totally optional: define some re-usable breakpoint numbers.
export const breakpoint = {
  medium:    666,
  large:     888,
  wide:      999,
  extraWide: 1111,
};

export const media = {
  smallOnly: mediaQuery`(max-width: ${(breakpoint.medium - 1) / 16}em)`,
  medium:    mediaQuery`(min-width: ${breakpoint.medium / 16}em)`,
  large:     mediaQuery`(min-width: ${breakpoint.large / 16}em)`,
  wide:      mediaQuery`(min-width: ${breakpoint.wide / 16}em)`,
  extraWide: mediaQuery`(min-width: ${breakpoint.extraWide / 16}em)`,
  print:     mediaQuery`print`,
};
```
3. Once we've defined our re-usable media queries, `media.smallOnly` and its siblings are now functions that can handle tagged template literals. Use them in any styled component that needs it.
  ```JS
import styled from 'styled-components';
import { media } from './sharedFile.js';

const Button = styled.button`
  width: 100%;

  ${media.medium`
    width: auto;
  `}

  ${media.extraWide`
    font-size: 1.5rem;
  `}
`;
```

## TLDR; if you already know what a tag function is.

```JavaScript
const mediaQuery = (...queryFeatures) => (...rules) => css`
  @media ${css(...queryFeatures)} {
    ${css(...rules)}
  }`
```

The `mediaQuery` method is a tag function that takes the given media query features in a tagged template literal; it returns a new tag function. This new tag function can be used in `styled-components` to take the CSS in a tagged template literal and returns the same CSS but wrapped in the pre-defined media query.

## Why does this exist?

The `css` method of Styled Components is very powerful and allows you to create media query helpers like this:

```JS
// style-utils.js
import { css } from 'styled-components'

export const media = {
  handheld: (...args) => css`@media (max-width: 420px) { ${ css(...args) } }`,
  tablet: (...args) => css`@media (min-width: 421px) { ${ css(...args) } }`,
}
```

The above file creates two "tag functions", `media.handheld` and `media.tablet`, that can be used to handle tagged template literals like this:

```JS
import { media } from './style-utils';

const Box = styled.div`
  font-size: 16px;
  
  ${media.handheld`
    font-size: 14px;
  `}
`;
```

The CSS generated from this will look something like this:

```CSS
.examp {
  font-size: 16px;
}

@media (max-width: 420px) {
  .examp {
    font-size: 14px;
  }
}
```

Pretty neat. The best part of this method is that it allows the developer to pick whatever name they want when defining the static object that holds all the media query helpers (e.g. `media`, `respondTo`, `atMedia`, …). And, since it is a static object, IDEs will offer auto-complete suggestions when you type the name of the object. I.e. typing `media.` in your IDE will offer `media.handheld`, `media.tablet` and all other media queries defined in your object.

The only problem with the above method is that there is a lot of boilerplate. ``(...args) => css`@media (query goes here) { ${ css(...args) } }`,`` and it is not possible (for mere mortals) to memorize that.

So we wrote a simple helper method to make things easier.

## API
 
`mediaQuery`: A helper method to manage media queries.

The `mediaQuery` method is a tag function that takes the given media query features in a tagged template literal; it returns a new tag function. This new tag function can be used in styled components to take a CSS rulesets in a tagged template literal and returns the same CSS but wrapped in the pre-defined media query.

### Arguments

1. `TaggedTemplateLiteral`: A tagged template literal containing the query features of a media query. e.g. `` `(min-width: 48em)` `` or `` `(min-width: 700px), handheld and (orientation: landscape)` ``

### Returns

A new tag function.

### Example

```JS
// An example mediaQueries.js file.
import { mediaQuery } from 'styled-media-queries';

// Define commonly used media queries and use them in all of your components.
export const media = {
  mobileOnly: mediaQuery`(max-width: ${767 / 16}em)`,
  tablet:     mediaQuery`(min-width: ${768 / 16}em)`,
  print:      mediaQuery`print`,
};
```

```JS
// An example component file.
import styled from 'styled-components';
import { media } from './mediaQueries';

const Button = styled.button`
  width: 100%;

  ${media.tablet`
    width: auto;
  `}

  ${media.print`
    color: black;
  `}
`;
```

This will generate CSS like the following:
```CSS
.examp {
  width: 100%;
}

@media (min-width: 48em) {
  .examp {
    width: auto;
  }
}

@media print {
  .examp {
    color: black;
  }
}
```
