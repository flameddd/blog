## Github Actions deploy Preactjs to Github Pages
  A step by step tutorial to teach how to use `Github Actions` to deploy a `preactjs Web App` to `Github pages` well (also `installable(PWA)`)

### github issue
I post same article on github
- https://github.com/preactjs/preact/issues/2178

### keywords
- `Github Actions`, [preactjs](https://github.com/preactjs/preact), `Github Pages` 

## demo repo
- https://github.com/flameddd/preact-github-actions-to-github-pages
- github page: https://flameddd.github.io/preact-github-actions-to-github-pages/

## about this note (demo)
- `"preact"`: `"^10.0.1"`
- `"preact-router"`: `"^3.0.0"`
- use `master` branch to trigger `Github Actions`
- use `package.lock.json`
  - If don't want to use `master` or `package.lock.json` ->  step 4, 5, 6

### problem
We deploy project is under a route NOT a domain. 
- under `https://Useranme.github.io/repro_name`
- **NOT** under `https://Useranme.github.io/`

So, we need some trick to make this work.

### steps
1. add repo public/private keys: follows https://github.com/peaceiris/actions-gh-pages#1-add-ssh-deploy-key
- Go to `Deploy Keys` and add your public key with the `Allow write access`
- Go to `Secrets` and add your private key as `ACTIONS_DEPLOY_KEY` <--!!!
2. add `dot-json`: `npm i --save-dev dot-json`
3. add npm script: `"build:gh": "GITHUB_PAGES=REPO_NAME preact build && dot-json ./build/manifest.json start_url \"/REPO_NAME/\""`
  - replace `2 REPO_NAME` with your repo name !!!
4. update `.gitignore`, remove `package-lock.json`
5. add `.github` folder
  - add `workflows` folder under `.github` folder
    - add `gh-pages.yml` file under `workflows` folder
    - so, it would be `.github/workflows/gh-pages.yml`
6. edit `gh-pages.yml`
```yaml
name: github pages

on:
  push:
    branches:
    - master

jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
    - uses: actions/checkout@v1

    - name: Setup Node
      uses: actions/setup-node@v1
      with:
        node-version: '10.x'

    - name: Cache dependencies
      uses: actions/cache@v1
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - run: npm ci

    - run: npm run build:gh
 
    - name: Deploy
      uses: peaceiris/actions-gh-pages@v2
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        PUBLISH_BRANCH: gh-pages
        PUBLISH_DIR: ./build
```
7. eidt `src/manifest.json`, update icons src from `/assets/...` to `./assets/...`

```json
  "icons": [
    {
      "src": "./assets/icons/android-chrome-192x192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "./assets/icons/android-chrome-512x512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ]
```
8. create `preact.config.js` file  
```js
export default {
  webpack(config, env, helpers, options) {

    const publicPath = process.env.GITHUB_PAGES
      ? `/${process.env.GITHUB_PAGES}/`
      : '/';
    const ghEnv = process.env.GITHUB_PAGES
      && JSON.stringify(`${process.env.GITHUB_PAGES}`);

    config.output.publicPath = publicPath;
    const { plugin } = helpers.getPluginsByName(config, 'DefinePlugin')[0];
    Object.assign(
      plugin.definitions,
      { ['process.env.GITHUB_PAGES']: ghEnv }
    );

  },
};
```
9. add `/src/baseroute.js` file
```js
let basename = '';

if (process.env.GITHUB_PAGES) {
  basename = `/${process.env.GITHUB_PAGES}`;
}

export default basename;
```
10. edit `src/components/app.js`. import and `add baseroute to path`  
```js
// import Header from './header';
import baseroute from '../baseroute'; // <-- make sure path correctly

// export default class App extends Component {
  // ...
  // render() {
	// 	return (
	// 		<div id="app">
	// 			<Header />
	// 			<Router onChange={this.handleRoute}>
          <Home path={`${baseroute}/`} />  // <-----
          <Profile path={`${baseroute}/profile/`} user="me" />  // <-----
          <Profile path={`${baseroute}/profile/:user`} />  // <-----
	// 			</Router>
	// 		</div>
	// 	);
	// }
// }
```

11. edit `src/components/header/index.js` import and `add baseroute to href`  
```js
// import style from './style.css';
import baseroute from '../../baseroute'; // <-- make sure path correctly

// const Header = () => (
	// <header class={style.header}>
		// <h1>Preact App</h1>
		// <nav>
      <Link activeClassName={style.active} href={`${baseroute}/`}>Home</Link>
      <Link activeClassName={style.active} href={`${baseroute}/profile`}>Me</Link>
      <Link activeClassName={style.active} href={`${baseroute}/profile/john`}>John</Link>
		// </nav>
	// </header>
// );
```

12. push to `master` branch, and wait `actions` done
13. go to repo's `settings`, `GitHub Pages` sections and **switch** `Source` to `gh-pages` branch.
  - I do it manually, but I also had exp this setting switch automatically
  - as My exp, it may need wait 1 ~ 3 min github page update