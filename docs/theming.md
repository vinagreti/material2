# Theming your Angular Material app


### What is a theme?
A **theme** is the set of colors that will be applied to the Angular Material components. The
library's approach to theming is based on the guidance from the [Material Design spec][1]. 

In Angular Material, a theme is created by composing multiple palettes. In particular, 
a theme consists of:
* A primary palette: colors most widely used across all screens and components.
* An accent palette: colors used for the floating action button and interactive elements.
* A warn palette: colors used to convey error state.
* A foreground palette: colors for text and icons.
* A background palette: colors used for element backgrounds.

In Angular Material 2, all theme styles are generated _statically_ at build-time so that your
app doesn't have to spend cycles generating theme styles on startup.

[1]: https://material.google.com/style/color.html#color-color-palette

### Using a pre-built theme
Angular Material comes prepackaged with several pre-built theme css files. These theme files also
include all of the styles for core (styles common to all components), so you only have to include a
single css file for Angular Material in your app. 

You can include a theme file directly into your application from 
`@angular/material/core/theming/prebuilt`

If you're using Angular CLI, this is as simple as including one line
in your `style.css`  file:
```css
@import '~@angular/material/core/theming/prebuilt/deeppurple-amber.css';
```

Alternatively, you can just reference the file directly. This would look something like
```html
<link href="node_modules/@angular/material/core/theming/prebuilt/indigo-pink.css" rel="stylesheet">
``` 
The actual path will depend on your server setup. 

You can also concatenate the file with the rest of your application's css.

### Defining a custom theme
When you want more customization than a pre-built theme offers, you can create your own theme file.

A theme file is a simple Sass file that defines your palettes and passes them to mixins that output
the corresponding styles. A typical theme file will look something like this:
```scss
@import '~@angular/material/core/theming/all-theme';
// Plus imports for other components in your app.

// Include the base styles for Angular Material core. We include this here so that you only
// have to load a single css file for Angular Material in your app.
@include md-core();

// Define the palettes for your theme using the Material Design palettes available in palette.scss
// (imported above). For each palette, you can optionally specify a default, lighter, and darker
// hue.
$primary: md-palette($md-indigo);
$accent:  md-palette($md-pink, A200, A100, A400);

// The warn palette is optional (defaults to red).
$warn:    md-palette($md-red);

// Create the theme object (a Sass map containing all of the palettes).
$theme: md-light-theme($primary, $accent, $warn);

// Include theme styles for core and each component used in your app.
// Alternatively, you can import and @include the theme mixins for each component
// that you are using.
@include angular-material-theme($theme);
```

You only need this single Sass file; you do not need to use Sass to style the rest of your app.

If you are using the Angular CLI, support for compiling Sass to css is built-in; you only have to 
add a new entry to the `"styles"` list in `angular-cli.json` pointing to the theme 
file (e.g., `unicorn-app-theme.scss`).

If you're not using the Angular CLI, you can use any existing Sass tooling to build the file (such
as gulp-sass or grunt-sass). The simplest approach is to use the `node-sass` CLI; you simply run:
```
node-sass src/unicorn-app-theme.scss dist/unicorn-app-theme.css
```
and then include the output file in your application.

The theme file can be concatenated and minified with the rest of the application's css.

#### Multiple themes
You can extend the example above to define a second (or third or fourth) theme that is gated by 
some selector. For example, we could append the following to the example above to define a 
secondary dark theme:
```scss
.unicorn-dark-theme {
  $dark-primary: md-palette($md-blue-grey);
  $dark-accent:  md-palette($md-amber, A200, A100, A400);
  $dark-warn:    md-palette($md-deep-orange);

  $dark-theme: md-dark-theme($dark-primary, $dark-accent, $dark-warn);
  
@include angular-material-theme($dark-theme);   
}
```

With this, any element inside of a parent with the `unicorn-dark-theme` class will use this
dark theme.

### Styling your own components
In order to style your own components with our tooling, the component's styles must be defined 
with Sass. 

You can consume the theming functions and variables from the `@angular/material/core/theming`.
You can use the `map-get` function to extract the theming variables and `md-color` function to extract a specific color from a palette. For example:

custom-input.component.scss

```scss
@import '~@angular/material/core/theming/theming';

@mixin custom-input-theme($theme) {
  $primary: map-get($theme, primary);
  $accent: map-get($theme, accent);
  $warn: map-get($theme, warn);
  $background: map-get($theme, background);
  $foreground: map-get($theme, foreground);
  
  // Placeholder colors. Required is used for the `*` star shown in the placeholder.
  $input-placeholder-color: md-color($foreground, hint-text);
  $input-floating-placeholder-color: md-color($primary);
  $input-required-placeholder-color: md-color($accent);
  
  // Underline colors.
  $input-underline-color: md-color($foreground, hint-text);
  $input-underline-color-accent: md-color($accent);
  $input-underline-color-warn: md-color($warn);
  $input-underline-disabled-color: md-color($foreground, hint-text);
  $input-underline-focused-color: md-color($primary);

  .md-input-placeholder {
    color: $input-placeholder-color;

    // :focus is applied to the input, but we apply md-focused to the other elements
    // that need to listen to it.
    &.md-focused {
      color: $input-floating-placeholder-color;

      &.md-accent {
        color: $input-underline-color-accent;
      }
      &.md-warn {
        color: $input-underline-color-warn;
      }
    }
  }

  // See md-input-placeholder-floating mixin in input.scss
  md-input input:-webkit-autofill + .md-input-placeholder,
  .md-input-placeholder.md-float:not(.md-empty), .md-input-placeholder.md-float.md-focused {

    .md-placeholder-required {
      color: $input-required-placeholder-color;
    }
  }

  .md-input-underline {
    border-color: $input-underline-color;

    .md-input-ripple {
      background-color: $input-underline-focused-color;

      &.md-accent {
        background-color: $input-underline-color-accent;
      }
      &.md-warn {
        background-color: $input-underline-color-warn;
      }
    }
  }
}
```

### Future work
* Once CSS variables (custom properties) are available in all the browsers we support,
  we will explore how to take advantage of them to make theming even simpler.
