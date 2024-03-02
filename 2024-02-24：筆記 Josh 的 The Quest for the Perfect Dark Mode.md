# 2024-02-24：筆記 Josh 的 The Quest for the Perfect Dark Mode.md
## Josh Comeau, 2020/04/22
### https://www.joshwcomeau.com/react/dark-mode/

----------------

以前就看過這篇，很有印象，但沒有實作過。最近想想，過陣子說不定會用到這方面的細節概念，所以來好好重讀一次這篇。

作者有把 code example 分享出來
- https://github.com/joshwcomeau/dark-mode-minimal  

(作者用 Gatsby 來做 blog，所以文章中有些步驟是用 Gatsby 做舉例說明的)

----------------

## 1. Perfect Dark Mode 的要求
- User 能夠切換  Light/Dark mode
- 保存 User 偏好，未來訪問時使用正確的 color theme
- 根據 User OS 的 preferred color scheme 為首選方案。若沒有設置，應該 default 為 Light
- App 首次載入不能 flicker，即使 User 選擇了 non-default color theme
- 永遠不該顯示錯誤的 toggle state


根據上面這些要求，initial color theme 應該是什麼？  
1. 最優先根據 `localStorage` 存的值做顯示 (light/dark)
2. 如果 `localStorage` 是 `undefined`，接著判斷 `preferred-color-scheme`
3. 如果 `preferred-color-scheme` 也是 `undefined`，那就預設為 light


----------------

## 2. 第一步

首先先來個 function (依據上面的條件)來做更新 color theme

```js
function getInitialColorMode() {
  const persistedColorPreference = window.localStorage.getItem('color-mode');
  const hasPersistedPreference = typeof persistedColorPreference === 'string';
  
  // 1. User 有曾經選擇過，那就依據這個選擇來顯示
  if (hasPersistedPreference) { 
    return persistedColorPreference;
  }

  // 2. User 沒選擇過，那接著來確認 來 prefers-color-scheme
  const mql = window.matchMedia('(prefers-color-scheme: dark)');
  const hasMediaQueryPreference = typeof mql.matches === 'boolean';
  if (hasMediaQueryPreference) {
    return mql.matches ? 'dark' : 'light';
  }

  // 3. If they are using a browser/OS that doesn't support
  // color themes, let's default to 'light'.
  return 'light';
}
```

接著 client 需要 manage state:

```js
function getInitialColorMode() { /* ... */ }

export const ThemeContext = React.createContext();
export const ThemeProvider = ({ children }) => {
  const [colorMode, rawSetColorMode] = React.useState(getInitialColorMode);
  const setColorMode = (value) => {
    rawSetColorMode(value); 
    
    // Persist value into localStorage
    window.localStorage.setItem('color-mode', value);
  };

  return (
    <ThemeContext.Provider value={{ colorMode, setColorMode }}>
      {children}
    </ThemeContext.Provider>
  );
};
```

上面的 code
- `useState` 執行 `getInitialColorMode` 來取得 initial value
- 另外建立 `setColorMode` function，會把 value persist 到 localStorage 裏，下次開 App 時就能用

----------------

## 3. 上面的 code 會有問題

`getInitialColorMode` function 是在 on mount 時立即執行，來確定初始值是什麼
- 但 first reader 不會發生在 client 上
  - 所以當 React first renders時，我們無法知道 User 的顏色偏好是什麼，因為該 render 發生在 server


如果先假設 User 想要 light mode，然後 client rehydrated 後將其換掉
- 這樣就會 **flicker**


<p align="center">
  <video width="80%" muted autoplay loop loading="lazy" src="https://www.joshwcomeau.com/videos/dark-mode/light-flash.mp4" ></video>
</p>  

----------------

## 4. A workable solution

解法的大方向:
- 所有相關的 style 都採用 CSS variables
- 在 compile (build time) 時，在 HTML inject `<script>` 放在 page content 前面
- 這個 `<script>` 會弄清楚 User 的 color preference
- 用 JavaScript 來 update CSS variables
在該腳本標籤中，計算出用戶的顏色偏好是什麼
使用 JavaScript 更新 CSS 變量



這是初始 HTML 的樣子(在 JS bundle 執行之前):

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Awesome Website</title>
  </head>
  <body>
    <script>
      /*
        - Check localStorage
        - Check the media query
        - Update our CSS variables depending
          on those values.
      /*
    </script>
    <div>
      <h1>My Awesome Website<h1>
      <p>Content here.</p>
    </div>
```

### Blocking HTML
阻止 HTML
上面 inject 的 `<script>` 在 `body` 裡面的最上面
- 這很重要，因為 `script` 是 blocking 的
- 所以在這段 JS 執行前，不會 paint 任何東西在 screen 上面


看看這個例子
- `<h1>` 要等到 `<script>` 執行完才會出現
```html
<body>
  <script>
    alert('No UI for you!');
  </script>
  <h1>Page Title</h1>
</body>

```

<p align="center">
  <video width="80%" muted autoplay loop loading="lazy" src="https://www.joshwcomeau.com/videos/dark-mode/alert.mp4" ></video>
</p> 


### CSS variables
CSS variables 最棒的是，它們是 reactive，Variable 更新，HTML 立刻 updates
- 所以策略是靠 CSS variables 來 styling，然後 update CSS variables
- 我們可以在所有 component 中用 CSS variables，如:

```js
const Button = styled.button`
  background: var(--color-primary);
`;
```

----------------

## 5. 在 Gatsby 中更新 HTML
當 Gatsby build 時
- 它會為網站每個頁面生成一個 HTML
- 我們的目標是注入一個 `<script>`
- Gatsby build process 的每一步都能讓我們加一些自定義行為

所以這邊要在 root folder 建立 `gatsby-ssr.js`
- Gatsby build time 時就會執行它

```js
const MagicScriptTag = () => {
  const codeToRunOnClient = `
(function() {
  alert("Hi!");
})()
  `;
  // eslint-disable-next-line react/no-danger
  return <script dangerouslySetInnerHTML={{ __html: codeToRunOnClient }} />;
};
export const onRenderBody = ({ setPreBodyComponents }) => {
  setPreBodyComponents(<MagicScriptTag />);
};
```

解釋上面的 code
- `onRenderBody` 是 Gatsby exposes 的 lifecycle method
    - 在 build HTML 時，會執行這個 function     
- `setPreBodyComponents` 會插入 React element `<body>` 裡面的最上面
- `MagicScriptTag` 就單純是個 React component，它 render <`script>` 
- 這裡採用 IIFE，避免 global namespace pollution

(這邊 `dangerouslySetInnerHTML` 的值是在 build 傳的，不太可能有人能在這裡 inject 惡意程式)


----------------

## 6.  Crossing the chasm

在這樣建構後，code 就會在 client 端執行
- 必須要這樣做，因為我們還沒有正確的資訊。我們不知道 User 的 localStorage 中有什麼，也不知道 OS preferred color scheme
- 反之亦然！當 code 執行前，它將無法訪問任何我們的 JS bundle code。它甚至會在 bundle 下載之前執行！
  - 這意味著它不會自動知道我們的 design tokens 是什麼


產生我們需要的 script:
```js
const MagicScriptTag = () => {
  let codeToRunOnClient = `
(function() {
  function getInitialColorMode() {
    /* 最上面的 code sample */
  }
  const colorMode = getInitialColorMode();
  const root = document.documentElement;
  root.style.setProperty(
    '--color-text',
    colorMode === 'light'
      ? '${COLORS.light.text}'
      : '${COLORS.dark.text}'
  );
  root.style.setProperty(
    '--color-background',
    colorMode === 'light'
      ? '${COLORS.light.background}'
      : '${COLORS.dark.background}'
  );
  root.style.setProperty(
    '--color-primary',
    colorMode === 'light'
      ? '${COLORS.light.primary}'
      : '${COLORS.dark.primary}'
  );
  root.style.setProperty('--initial-color-mode', colorMode);
})()`;
  // eslint-disable-next-line react/no-danger
  return <script dangerouslySetInnerHTML={{ __html: codeToRunOnClient }} />;
};
```

現在 `getInitialColorMode` 在 client 執行，我們能拿到正確的 initial color
- 我們就可以開始設定 CSS variables 了

```js
root.style.setProperty(
  '--color-text',
  colorMode === 'light'
    ? 'black' // resolves from ${COLORS.light.text}
    : 'white' // resolves from ${COLORS.dark.text}
);
``` 

這邊為了簡單呈現，所以用  `root.style.setProperty` function
- 但如果有超過 10 種以上的顏色，那 code 就很冗長了，最後面有改良 code 的版本
- 這裡還設了一個 `--initial-color-mode` 屬性，到時候 React 決定初始狀態時，可以讀這個值



### State management
接著要有 state 來管理狀態，讓 User 能自由切換 light/dark  


這裡可以看到，如何從 `--initial-color-mode` 取得 initial value，然後 setState:
- 初始狀態 `undefined`，因為 first render (compile time)，沒法訪問 window
-  React rehydrates 後，立即檢查 `--initial-color-mode`
```js
export const ThemeContext = React.createContext();
export const ThemeProvider = ({ children }) => {
  const [colorMode, rawSetColorMode] = React.useState(undefined);
  React.useEffect(() => {
    const root = window.document.documentElement;
    const initialColorValue = root.style.getPropertyValue(
      '--initial-color-mode'
    );
    rawSetColorMode(initialColorValue);
  }, []);
  const setColorMode = (value) => {
    /* TODO */
  };

  return (
    <ThemeContext.Provider value={{ colorMode, setColorMode }}>
      {children}
    </ThemeContext.Provider>
  );
};
```

----------------

## 7. 整個過程的順序

1. Compile-time: (這段在 server 上執行)
    1.1 React renders  
    1.2 `<script>` is injected
2. Run-time: (這段在 client 上執行)
    2.1 `<script>` executes  
    2.2 HTML is shown  
    2.3 React rehydrates  
    2.4 React re-renders  

----------------

## 8. Adding a toggle
有 state 管理狀態了，接著實作 toggle 功能:
1. update React state
2. update localStorage
3. update 所有 CSS variables


(code 很冗長繁瑣，部分原因是為了給讀者清楚邏輯，最後面有改良的 code)
```js
function setColorMode(newValue) {
  const root = window.document.documentElement;
  // 1. Update React color-mode state
  rawSetColorMode(newValue);
  // 2. Update localStorage
  localStorage.setItem('color-mode', newValue);
  // 3. Update each color
  root.style.setProperty(
    '--color-text',
    newValue === 'light' ? COLORS.light.text : COLORS.dark.text
  );
  root.style.setProperty(
    '--color-background',
    newValue === 'light' ? COLORS.light.background : COLORS.dark.background
  );
  root.style.setProperty(
    '--color-primary',
    newValue === 'light' ? COLORS.light.primary : COLORS.dark.primary
  );
}
```



<p align="center">
  <video  width="80%" muted autoplay loop loading="lazy"  src="https://www.joshwcomeau.com/videos/dark-mode/standard-toggle.mp4" ></video>
</p>  



在 button 中，這樣呼叫 toggle function:
```js
import { ThemeContext } from './ThemeContext';
const DarkToggle = () => {
  const { colorMode, setColorMode } = React.useContext(ThemeContext);
  return (
    <label>
      <input
        type="checkbox"
        checked={colorMode === 'dark'}
        onChange={(ev) => {
          setColorMode(ev.target.checked ? 'dark' : 'light');
        }}
      />{' '}
      Dark
    </label>
  );
};
```

上面 sample code 還有一個問題，`checkbox` 沒有以正確的 initial value 初始化 input 的值
- 別忘了，initial render 是在 server compile time，`colorMode` 一開始我們給的是 `undefined`
- 最間單的處理方法就是推遲 render，直到我們知道正確的 `colorMode`

```js
const DarkToggle = () => {
  const { colorMode, setColorMode } = React.useContext(ThemeContext);
  if (!colorMode) {
    return null;
  }
  return <label>{/* ... */}</label>;
};
```

(看看右上角的 checkbox)  
<p align="center">
  <video width="80%" muted autoplay loop loading="lazy" src="https://www.joshwcomeau.com/videos/dark-mode/checkbox-hole.mp4" ></video>
</p>  

到這邊就成功了，User 看到的第一個畫面就是正確的 mode。  

下面還有些改良的細節  

----------------

## 9. 一些小調整、優化


### Iteration
上面的 smaple code 是重複用好多 `setProperty`，而且還在兩個地方做
1. `gatsby-ssr.js`，用來建立 initial variables
2. `ThemeProvider` component，用來切換 mode 

但有機會用個 loop 來全面調整：
- 要做到這個，就要先事先定義好 design

這種重複的煩人之處在於，您需要記住在添加或更改設計令牌時更新兩個位置。 我們可以通過動態生成它們來解決這個問題：

```js
Object.entries(COLORS).forEach(([name, colorByTheme]) => {
  const cssVarName = `--color-${name}`;
  root.style.setProperty(cssVarName, colorByTheme[newValue]);
});
```


### No JavaScript
有沒有可能在`禁用 JavaScript `的情況下完成:
- Gatsby 的優點之一是許多 Gatsby 網站都禁用 JS
- 目前的解決方案沒有考慮到這一點
  - 如果在未啟用 JS 的情況下訪問，所有內容都會正確呈現，但沒有顏色



可以在 `gatsby-ssr.js` (build itme) inject `<style>` 到 `<head>` 來處理
- inject 一些預設值，這樣就能在 JS 被停用下能使用

```js
const FallbackStyles = () => {
  return (
    <style>
      {`
        html {
          --color-text: ${COLORS.text};
          --color-background: ${COLORS.background};
          --color-primary: ${COLORS.primary};
        }
      `}
    </style>
  );
};
export const onRenderBody = ({ setPreBodyComponents, setHeadComponents }) => {
  setHeadComponents(<FallbackStyles />);
  setPreBodyComponents(<MagicScriptTag />);
};
```



### Minification
實務上，前端的 code 都會經過 minification or uglification，來減少檔案大小
- 上面 inject 的 code 就沒有這段過程
- 下面用 [terser](https://www.npmjs.com/package/terser) 來加強這部分

```js
const outputCode = Terser.minify(inputCode).code;
```

但這樣做的效益其實不高，作者的 blog 前後差距只有約 `200 bytes`，可以視情況要不要做這段


### Script generation
在 `gatsby-ssr` 我們 inject 的 code 是用 string
- 用 string 寫 code 是遭了點，沒有 syntex highlight, Prettier, type check

可以寫個 function 來產出我們要的 code string
```js
function doStuff() {
  /* stuff */
}
String(doStuff);
```



但這有些問題！
- 我們希望用 IIFE，以防止洩漏到 global state
- 需要某種方式將 design value 傳給它
  - 與一般 function 不同，這種情況下將無法依賴 parent scopes，因為它將在完全不同的 context 中執行


```js
function setColorsByTheme() {
  const colors = '🌈';
  // Do stuff with `colors`, as if it was an object
  // that held everything!
}
const MagicScriptTag = () => {
  // Replace that rainbow string with our COLORS object.
  // We need to stringify it as JSON so that it isn't
  // inserted as [object Object].
  const functionString = String(setColorsByTheme).replace(
    "'🌈'",
    JSON.stringify(COLORS)
  );
  // Wrap it in an IIFE
  let codeToRunOnClient = `(${functionString})()`;
  // Inject it
  return <script dangerouslySetInnerHTML={{ __html: codeToRunOnClient }} />;
};
```

現在 IDE 能支援了
