# 2020-11-04：Purge/Removing unused tailwind css in preactjs

keyword: `Purge`, `tailwind`, `tailwindcss`, `preactjs`, eliminate tailwind css bundle size

## My project info
```json
{
  "devDependencies": {
    "preact-cli": "^3.0.3",
    "sirv-cli": "1.0.3"
  },
  "dependencies": {
    "preact": "^10.3.2",
    "preact-render-to-string": "^5.1.4",
    "preact-router": "^3.2.1",
    "tailwindcss": "^1.9.6"
  }
}
```


## ref: (tailwindcss document)
- https://tailwindcss.com/docs/controlling-file-size#removing-unused-css

----------------------------------------------

## Step1 install "@fullhuman/postcss-purgecss"

```
# Using npm
npm install @fullhuman/postcss-purgecss --save-dev

# Using yarn
yarn add @fullhuman/postcss-purgecss -D
```

## Step2 `preact.config.js`

```js
// tailwindcss purgecss
const purgecss = require('@fullhuman/postcss-purgecss')({
  // Specify the paths to all of the template files in your project
  content: [
    './src/**/*.html',
    './src/**/*.jsx',
    './src/**/*.js',
  ],

  // This is the function used to extract class names from your templates
  defaultExtractor: content => {
    // Capture as liberally as possible, including things like `h-(screen-1.5)`
    const broadMatches = content.match(/[^<>"'`\s]*[^<>"'`\s:]/g) || []

    // Capture classes within other delimiters like .block(class="w-1/2") in Pug
    const innerMatches = content.match(/[^<>"'`\s.()]*[^<>"'`\s.():]/g) || []

    return broadMatches.concat(innerMatches)
  }
})


export default (config, env, helpers) => {

  const postCSSLoader = helpers.getLoadersByName(config, 'postcss-loader')[1]
  postCSSLoader.loader.options.plugins = [
    require("tailwindcss")("./tailwind.config.js"),
    require("autoprefixer"),
    ...process.env.NODE_ENV === 'production'  // <-- Purge only when production
      ? [purgecss]
      : []
  ].concat(postCSSLoader.loader.options.plugins);
};
```

## Step3 `package.json`, add NODE_ENV=production

```json
{
  "scripts": {
    "build": "NODE_ENV=production preact build"
  }
}
```
