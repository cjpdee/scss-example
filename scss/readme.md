# SCSS Style Guide

## Formatting

### Prettier

For the sake of avoiding arguments about formatting, Prettier should be used with the 'format on save' functionality in modern IDEs. Prettier is a widespread, opinionated code formatter that takes the decision away from developers. The goal here is that if a machine is doing the formatting, there's nobody to waste time arguing with about code style. A predefined set of rules is enforced, and that's the end of it.

### Naming

SCSS classes should be entirely lowercase, using a kebab-case-formatting-style. The only exception to this is for cases where reactive JS frameworks are being used, in which case developers should follow a component-driven design pattern and name the $block variable (as well as filename) the same as the reactive component.

```
// MyComponent.jsx
...JS component lives here...
```

```
// MyComponent.scss
$block: 'MyComponent';

.#{$block} {
  // kebab-case lowercase naming continues inside the block
  &__element {}
}
```

### Units

Pixel and percentage units should be used primarily, along with viewport units when required. REM units and EM units should generally be avoided as these can lead to complications and less control over styling.

If a number is 0, omit the unit:

```
width: 100px;
height: 0;
```

## Methodology

### BEM

Synergist uses the CSS methodology BEM, which aims to improve modularity, reusability and overall CSS structure. It also helps to avoid complications caused by CSS nesting.

https://getbem.com/introduction/
https://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/
https://www.phase2technology.com/blog/used-and-abused-css

#### Block

The top-level name for the component. This represents the outermost HTML element for the component. Avoid using .block-wrapper style naming:

```
// do this
.block {
  &__inner {}
}

// not this
.block-wrapper {
  &__block
}
```

To take this further, we also want to assign the block name to a variable. This is helpful for a number of reasons, but the main benefit is that if the file needs to be refactored it's very easy to change the block name. The block can also be referenced when nesting.

```
$block: 'component-name';

.#{$block} {
  &--modifier {}

  &__element {
    .#{$block}--modifier & {
      // ...
    }
  }
}
```

#### Element

Any children of the parent block are elements. Separate the element from the block with the block\_\_element naming convention. If the element uses multiple words, kebab-case can be used.

```
.block {
  &__element {}
  &__other-element {}
}
```

#### Modifier

Modifiers are alternative styles that can be applied to the base class. These should be used in addition to the base classes:

```
// do this
<button class="button button--primary"></button>

// not this
<button class="button--primary"></button>
```

Where possible, modifiers should be applied at the block level. This means that modifiers can be applied in one place to affect its children, since multiple children could be affected by the modifier.

```
// do this
<button class="button button--primary">
  <div class="button__icon"></div>
</button>

// not this
<button class="button button--primary">
  <div class="button__icon button__icon--primary"></div>
</button>
```

The SCSS for this button might look something like this:

```
$block: 'button';

.#{$block} {
  // base styles
  &--positive {
    background-color: $c-positive;
    color: white;
  }

  &__icon {
    width: 24px;
    height: 24px;

    .#{$block}--primary & {
      fill: $c-positive;
      stroke: white;
    }
  }
}
```

### Utility Classes

If BEM is tasked with handling modular components, utility classes should fill in the gaps between those components. This idea is taken from the Tailwind framework and similar concepts.

https://tailwindcss.com

```
<div class="flex items-start justify-end">
  <button class="button button--primary mr-4">Reload</button>
  <button class="button button--primary mr-2">Prev</button>
  <button class="button button--primary">Next</button>
</div>
```

This means our component styles can be self-contained, but still be reusable in a different context.
It also gives us a way to create generic containers or layouts without needing to create a new BEM-style component.
Our utility classes should be limited in scope to prevent misuse. Compared to Tailwind, we should only be using the following:

- Margins
- Flex layouts

Utility classes can also use media queries using the same Tailwind syntax:

```
<button class="button button--primary mb-2 laptop:mb-4">Hello World</button>
```

## General Rules

### Nesting

Nesting should generally be kept to a minimum. SCSS should have 4 levels of nested scope at most, except for unusal cases where this is required.

```
.component {
  // 1
  &__nested-component {
    // 2

    .component.is-active & {
      // 3

      &:before {
        // 4
      }
    }
  }
}
```

### Media Queries

Queries should be pointed upwards, meaning the base styles for a class should cover mobile, tablets etc. and the styles inside a media query should cover laptop and above.

```
// target screen size under 1024px
@include bp(laptop) {
  // targets screen size of over 1024px
}
```

### !important

The important rule should almost never be used. The only acceptable case is to override vendor stylesheets e.g. plugins that use !important

### Unknown numbers

Here is an example of an unknown number:

```
width: calc(100% - 200px);
```

Where does 200px come from? While it may seem trivial, future developers may not understand why this number in particular is required.

Naming the number with a variable can help to clear this up.

```
$sidebar-width: 200px;

width: calc(100% - $sidebar-width);
```

### Document flow

When styling elements, aim to rely on the natural flow of the DOM to use the minimum amount of CSS. A clear understanding of the CSS box model can help with this: https://www.simplilearn.com/tutorials/css-tutorial/css-box-model

Along the same lines, take care when using properties that alter the document flow. Consider `position: absolute`: While a useful tool, overuse can create layouts that do not interact well on different devices. The developer may need to spend extra time in positioning these elements because they do not interact with other elements on the page.
https://soulandwolf.com.au/blog/what-is-document-flow/

### is-[adjective], has-[noun]

To clear up confusion about what a class is doing, we can use `.is-[adjective]` or `.has-[noun]` classes to indicate a change in state. This is different to BEM modifiers which are generally not dependent on state.

Take this example of a nav:

```
.nav {
  // ...
  display: none;

  &--blue {
    background-color: blue;
  }

  &.is-open {
    display: flex;
  }
}
```

We could also use the `.has-[noun]` syntax in the same way.

### Animations

When creating an animation, we should almost always prefer CSS-based transitions over animating with JS.
