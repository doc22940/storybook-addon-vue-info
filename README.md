## Deprecated

This addon is deprecated due to [the retirement of official addon-info](https://github.com/storybookjs/deprecated-addons).

Storybook now has an alternative addon, called [Docs addon(`addon-docs`)](https://github.com/storybookjs/storybook/tree/next/addons/docs),
which comes with native Vue.js support and automatically props/events/slots documentation powered by vue-docgen-api.

There will be no feature addition nor bug fixes to this repo. Please use Docs addon instead.

### Migration to Docs

As described above, Docs addon uses `vue-docgen-api` via `vue-docgen-loader`.
They are also the tools `storybook-addon-vue-info` uses internally.
So the migration steps is quite simple.

#### docgen tools are no longer `peerDependencies`

Since the Docs addon specifies `vue-docgen-api` and `vue-docgen-loader` as direct dependency,
you don't have to list them in your `package.json`.

```diff
 "dependencies": {
-  "vue-docgen-api": "x.x.x",
-  "vue-docgen-loader": "x.x.x"
 }
```

Of course, you can keep them to controll which exact versions to use.

#### Explicitly specify which component to documentate

You need to set `component` field in your story metadata.

```js
// foo.stories.js
import MyComponent from './my-component.vue'

export default {
  title: 'Components/MyComponent',
  component: MyComponent
}

export const story = () => ({
  components: { MyComponent },
  template: '<my-component/>'
})
```

#### Move `summary` inside a JSDoc comment or MDX

`summary` option equivalent in Docs addon is component comments or MDX.
Docs addon reads a component comment and displays it as a description for the component.

```js
// legacy.stories.js
export const myStory = () => ({
  /* ... */
})

myStory.story = {
  info: {
    summary: 'foo bar'
  }
}
```

```html
<!-- component -->
<script>
  /**
   * foo bar
   */
  export default {
    /* ... */
  }
</script>
```

Or you can use [MDX](https://github.com/storybookjs/storybook/tree/next/addons/docs#mdx) for more complicated usage.

---

<div align="center">
  
  <img src="./assets/logo.png" width="104" alt="logo">
  <br/>

[![Build Status](https://travis-ci.com/pocka/storybook-addon-vue-info.svg?branch=master)](https://travis-ci.com/pocka/storybook-addon-vue-info)
[![npm version](https://badge.fury.io/js/storybook-addon-vue-info.svg)](https://badge.fury.io/js/storybook-addon-vue-info)
[![Monthly download](https://img.shields.io/npm/dm/storybook-addon-vue-info.svg)](https://www.npmjs.com/package/storybook-addon-vue-info)
[![GitHub license](https://img.shields.io/github/license/pocka/storybook-addon-vue-info.svg)](https://github.com/pocka/storybook-addon-vue-info/blob/master/LICENSE)
[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg)](https://github.com/prettier/prettier)

</div>

<hr/>

## storybook-addon-vue-info

A Storybook addon that shows Vue component's information.

- [Demo][live examples]

## Requirements

- `@storybook/vue>=4.0.0`

## Getting started

First, install the addon.

```sh
npm install --save-dev storybook-addon-vue-info

# yarn add -D storybook-addon-vue-info
```

Then register in `addons.js`.

```js
// .storybook/addons.js

// Don't forget "/lib/" !!
import 'storybook-addon-vue-info/lib/register'
```

And setup a webpack loader in order to extract component information with [vue-docgen-api](https://github.com/vue-styleguidist/vue-docgen-api).

```sh
npm install --save-dev vue-docgen-loader vue-docgen-api

# yarn add -D vue-docgen-loader vue-docgen-api
```

```js
// .storybook/webpack.config.js

// This example uses "Full control mode + default".
// If you are using other mode, add payload of `config.module.rules.push` to rules list.
module.exports = ({ config }) => {
  config.module.rules.push({
    test: /\.vue$/,
    loader: 'vue-docgen-loader',
    enforce: 'post'
  })

  return config
}
```

## Usage

Add `withInfo` decorator then set `info` options to the story.

NOTE: `info` option is required for the addon. If you omit it, the addon does nothing.

```js
import { storiesOf } from '@storybook/vue'

import { withInfo } from 'storybook-addon-vue-info'

storiesOf('MyComponent', module)
  .addDecorator(withInfo)
  .add(
    'foo',
    () => ({
      components: { MyAwesomeComponent },
      template: '<my-awesome-component/>'
    }),
    {
      info: {
        summary: 'Summary for MyComponent'
      }
    }
  )
```

You can set the addon as global decorator.

```js
// config.js
import { addDecorator } from '@storybook/vue'

import { withInfo } from 'storybook-addon-vue-info'

addDecorator(withInfo)
```

To set default options, use `setDefaults`.

```js
// .storybook/config.js
import { setDefaults } from 'storybook-addon-vue-info'

setDefaults({
  header: false
})
```

For more details, see [live examples].

## Options

| Name               | Data type                                     | Default value                                       | Description                                                                        |
| ------------------ | --------------------------------------------- | --------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `header`           | `boolean`                                     | `true`                                              | Whether to show header or not.                                                     |
| `source`           | `boolean`                                     | `true`                                              | Whether to show source(usage) or not.                                              |
| `wrapperComponent` | `Component`                                   | [default wrapper](src/components/Wrapper/index.vue) | Override inline docs component.                                                    |
| `previewClassName` | `string`                                      | `undefined`                                         | Class name passed down to preview container.                                       |
| `previewStyle`     | Style object                                  | `undefined`                                         | Style passed down to preview container.                                            |
| `summary`          | `string`                                      | `''`                                                | Summary for the story. Accepts Markdown.                                           |
| `components`       | `{ [name: string]: Component }\|null`         | `null`                                              | Display info for these components. Same type as component's `components` property. |
| `docsInPanel`      | `boolean`                                     | `true`                                              | Whether to show docs in addon panel.                                               |
| `useDocgen`        | `boolean`                                     | `true`                                              | Whether to use result of vue-docgen-api.                                           |
| `casing`           | `"kebab" \| "camel" \| "pascal" \| undefined` | `undefined`                                         | Which case to use. For detailed usage, see below.                                  |

### Valid `casing` options

```js
{
  // Don't convert names
  casing: undefined
}

{
  // Show names in kebab-case
  casing 'kebab'
}

{
  // Show prop names in camelCase and
  // Show component names in PascalCase
  casing: 'camel' // or 'pascal'
}

{
  // Show prop names in camelCase and
  // Show component names in kebab-case
  casing: {
    props: 'camel',
    component: 'kebab'
  }
}
```

In addition to addon options, we have a component option.

### Set descriptions manually

With vue-docgen-api, the addon automatically shows descriptions and types extracted by docgen ([see example in vue-docgen-api README](https://github.com/vue-styleguidist/vue-styleguidist/tree/dev/packages/vue-docgen-api#example)).
However, if you want to explicitly specify desciprion for component props, events or slots, add `description` option for your story component.

Assume `<my-awesome-component>` have props `label` and `visible`.

```js
storiesOf('MyComponent', module)
  .addDecorator(withInfo)
  .add(
    'foo',
    () => ({
      components: { MyAwesomeComponent },
      template: '<my-awesome-component/>',
      description: {
        MyAwesomeComponent: {
          props: {
            // These description will appear in `description` column in props table
            label: 'A label for my awesome component',
            visible: 'Whether component is visible or not'
          },
          events: {
            click: 'Event for user clicking the component'
          },
          slots: {
            default: 'Place text or icon here'
          }
        }
      }
    }),
    {
      info: true
    }
  )
```

For more detail, please take a look at [live example](https://storybook-addon-vue-info.netlify.com/?path=/story/examples-advance-usage--set-descriptions-manually).

## Example

For real example, see `example` directory.

[live examples]: https://storybook-addon-vue-info.netlify.com/?path=/story/examples-basic-usage--simple-example
