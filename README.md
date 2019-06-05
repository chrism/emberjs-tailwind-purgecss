#Ember.js, Tailwind 1.0 and PurgeCSS working example

## Background

An excellent Ember addon `ember-cli-tailwind` already exists which wraps Tailwind CSS and provides easy integration in an Ember application.

However, although Tailwind itself has [recently released v1.0 with some key differences and updates](https://tailwindcss.com/docs/upgrading-to-v1) the addon currently includes a specific pre v1.0 version of Tailwind.

Additionally, [PurgeCSS](https://www.purgecss.com/)—which is designed to remove unused CSS automatically and very useful considering the Tailwind approach to CSS—is not straightforward to implement when using the addon.

For these reasons it may be useful to include Tailwind and PurgeCSS without the use of an addon.

This repo provides a working example and guide to adding it to a project.

## Creating a new Ember project

Assuming you already have `yarn` and `ember-cli` installed.

```
ember new emberjs-tailwind-purgecss --yarn
```

## Installing PostCSS

In the [Tailwind installation guide](https://tailwindcss.com/docs/installation#using-tailwind-with-postcss) it explains

> For most projects, you'll want to add Tailwind as a PostCSS plugin in your build chain.

Luckily, a [PostCSS](https://postcss.org/) addon already exists for Ember, [Ember CLI Postcss](https://jeffjewiss.github.io/ember-cli-postcss/)

It can be installed with

```
ember install ember-cli-postcss
```

On it's own this doesn't do much but allows PostCSS plugins to be included in the build pipeline.

## Installing Tailwind

### Installing package

In our case we want to start with Tailwind, so following the [Tailwind installation guide](https://tailwindcss.com/docs/installation#1-install-tailwind-via-npm) first we install the package from npm (using `yarn`).

```
yarn add tailwindcss --dev
```

### Adding directives

Then add the directives to `styles/app.css`.

```
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Including in the build pipeline

The final step is to include Tailwind in the list of PostCSS plugins in `ember-cli-build.js`

```js
// ember-cli-build.js
'use strict';

const EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function(defaults) {
  let app = new EmberApp(defaults, {
    postcssOptions: {
      compile: {
        plugins: [
          require('tailwindcss')
        ]
      }
    }
  });
  return app.toTree();
};
```

This is even [helpfully documented in the PostCSS installation guide](https://tailwindcss.com/docs/installation#ember-js).

After updating `templates/application.hbs` to include some Tailwind classes it shows that it is working correctly.

```hbs
<section class="container mx-auto mt-4">
  <h1 class="text-2xl text-red-500">Example title using Tailwind</h1>
  <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.</p>
</section>

{{outlet}}
```

You can see this basic working example in this commit.
