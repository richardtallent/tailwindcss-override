# tailwindcss-override

## What does this do?

This provides a new `o:` variant for your Tailwind CSS configuration. Any class using this variant will _override_ other Tailwind CSS classes.

The purpose is to allow you to use Tailwind CSS classes to create components with reasonable default styles, while still allowing the component _user_ to override those styles with Tailwind CSS (adding only the `o:` prefix).

There's no run-time JavaScript involved, this is all pure CSS.

## How do I use it?

### Tailwind CSS 3

#### 1. Install

```sh
npm install tailwindcss-override
```

#### 2. Add this to your `tailwindcss.config` file in the `module.exports` object

```js
plugins: [
	require('tailwindcss-override')(),
],
```

### Tailwind CSS 4

No plugin installation needed! Just add this CSS variant to your stylesheet:

```css
@custom-variant o (&:not([z]));
```

Note: In Tailwind CSS 4, JavaScript plugins are no longer supported. Instead, you add custom variants directly in your CSS using the `@custom-variant` directive.

### Usage Example (Works with both Tailwind 3 & 4)

#### 1. Create a component with a default style

```HTML
<!-- MyButton.vue -->
<template>
	<button class="font-semibold bg-blue-500"><slot/></button>
</template>
```

#### 2. Override as needed for your component instances

```HTML
<MyButton>My Blue Button</MyButton>
<MyButton class="o:bg-red-500">My Red Button</MyButton>
```

## Caveats

1. Only one "level" of override is possible using this approach. You can't override an override.

2. I haven't tested this with more complex classes like media queries.

3. This CSS feature is [not supported](https://caniuse.com/mdn-css_selectors_where) in IE11 or Safari 8.x and below.

4. Like any variant, using this will increase your CSS bundle size, since the `o:` variants are defined as separate rules from the classes they are based on.

## How does it work?

When you use `o:bg-red-400`, a CSS rule is created for the selector `o\:bg-red-400:not([z])` to match it. The `:not([z])` is tacked on to force a higher specificity above a normal class selector (_i.e._, above other Tailwind CSS classes). The `:not([z])` selector was chosen because it (a) matches anything, (b) raises selectivity, and (c) is compact.

## Migrating from Tailwind CSS 3 to 4

If you're upgrading from Tailwind CSS 3 to 4:

1. **Remove the plugin** from your `tailwind.config.js`
2. **Add the CSS variant** to your main CSS file: `@custom-variant o (&:not([z]));`
3. **Your HTML stays the same** - all existing `o:` classes continue to work

## Why did I create this plugin?

I've been making web apps since 1995, and I really enjoy using Tailwind CSS.

But as a component author, there are also a few annoyances with the utility-class approach. One is that I like to create components that have _reasonable default styles_, but that _also_ allow the component user (usually me!) to "tweak" the style later. But with TailwindCSS's "flat" approach (there's not much actual "cascading" going on, just lots of individual classes), this becomes difficult. The usual recommendationa are:

1. **Don't style your components.** I don't like this approach. I like reasonable, beautiful defaults.

2. **Pass overridable styles as component properties.** 1995 called, it wants its `<FONT COLOR>` back!

3. **Use a higher-level selector in your component instance.** Yes, but that means having to go back to semantic classes and deeper selectors, and either not being able to use TailwindCSS classes for those component instances, or having to use them with `@apply` so they can consistently override the default.

4. **CSS-in-JS**. _I.e._, try to rewrite the whole logic of "what overrides what" in JavaScript and apply the "winning" classes/styles dynamically. A few people are working on this, but it feels kludgy, slow, and brittle to me for 99% of use cases.

I [came up with a similar idea](https://github.com/tailwindlabs/tailwindcss/discussions/3523) in early 2021, and even [created a plugin](https://github.com/richardtallent/tailwindcss-def) for it. The idea with _that_ plugin was to let the component _author_ use a special variant to create low-specificity Tailwind selectors using the new-ish `where()` selector. The problem is, the specificity was a bit _too_ low, so Tailwind's "reset" stylesheet made it impossible to use that plugin to style anything that is part of the reset (since those resets operate at the element level).

This plugin works the opposite way, allowing the developer to _raise_ a Tailwind CSS class's selectivity so it overrides some other (normal) Tailwind class definition.

This variant can be used in combination with other variants, such as `hover:`. I have not tried it with media queries, dark mode, or a few other variants, but I'm hopeful it will work with them as well, and I'm open to PRs if it doesn't.

## License

MIT
