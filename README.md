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

## Customizing Tailwind

This approach works, but is the most basic way of including Tailwind. Usually you'll want to customize the configuration to suit your project.

This is explained in detail in the [Tailwind Configuration guide](https://tailwindcss.com/docs/configuration).

### Generate configuration file

The first step is to generate a configuration file.

For Ember, it makes sense to include this file in the `/config` directory and in this example we are going to include the full configuration (in reality, [you should start as minimal as possible](https://tailwindcss.com/docs/configuration#creating-your-configuration-file) but this is useful for this example).

```
npx tailwind init config/tailwind.js --full
```

Which should result in something like this

```
✅ Created Tailwind config file: config/tailwind.js
```

### Updating path to configuration in build pipeline

As per the [configuration guides](https://tailwindcss.com/docs/configuration#using-a-different-file-name) update the plugin to include this new path.

```js
// ember-cli-build.js
'use strict';

const EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function(defaults) {
  let app = new EmberApp(defaults, {
    postcssOptions: {
      compile: {
        plugins: [
          require('tailwindcss')('./config/tailwind.js')
        ]
      }
    }
  });
  return app.toTree();
};
```

Now adding a custom color to the configuration and including it in template will work as expected.

This is a commit showing this working.

## Adding components and utilities

The addon `ember-cli-tailwind` allows for the easy creation of Tailwind [utilities](https://tailwindcss.com/docs/adding-new-utilities) and [components](https://tailwindcss.com/docs/extracting-components).

Both of which are integral to working with Tailwind.

While with our current solution it is possible to add them directly to the `app.css` file pretty easily (int he right place) splitting them up into separate files would make a cleaner and more modular approach like `ember-cli-tailwind` does.

### Installing PostCSS

To do that we need to now include `postcss-import` which is a plugin to inline `@import` rules content.

```
yarn add postcss-import -D
```

### Updating configuration

After installing we need to update the `app.css` and `ember-cli-build.js` to reflect this change.

```js
// ember-cli-build.js
'use strict';

const EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function(defaults) {
  let app = new EmberApp(defaults, {
    postcssOptions: {
      compile: {
        plugins: [
          require('postcss-import'),
          require('tailwindcss')('./config/tailwind.js')
        ]
      }
    }
  });
  return app.toTree();
};
```

It may be worth pointing out that although this worked for me [there has been discussion](https://github.com/jeffjewiss/ember-cli-postcss/issues/371#issuecomment-464084011) about also the need to include the `node_modules` path in the configuration.

### Updating CSS importing

For the `app.css` file some changes are needed to the way the tailwind files are imported and after creating two new files `components.css` and `utilities.css` they can be imported (in the right order) too.

This approach of switching from `@tailwind` directives to `@import` for `postcss-import` is explained in the [Tailwind documentation on using CSS](https://tailwindcss.com/docs/adding-new-utilities#using-css).

```
@import "tailwindcss/base";

@import "tailwindcss/components";
@import "components.css";

@import "tailwindcss/utilities";
@import "utilities.css";
```

Making these changes now means that utilities and components can be easily added and used in your application.

This commit shows a working example of a custom utility and component being used.

## Purging unused CSS

Now that Tailwind is set up and working correctly the final step is to remove all the unused CSS selectors from the outputted CSS to reduce unnecessary filesize.

To do this there is another library [PurgeCSS](https://www.purgecss.com/) which we can use.

### Installing PurgeCSS

```
yarn add purgecss -D
```
