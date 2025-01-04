# 2024-12-08: 筆記 Liran Tal 的 Node.js CLI Apps Best Practices.md

## Liran Tal

### https://github.com/lirantal/nodejs-cli-apps-best-practices

---

對如何建立好的 CLI 工具有點興趣，意外看到這篇，快點來看看

作者 Liran Tal

- 是長期在 NodeJS Security 領域的人，這篇也一定有顧慮到這些方面
- 文章中似乎針對很多實作方面，都有推薦相應的 library 可以用

文章的內容雖然是以 NodeJS 為主，但其中關於 CLI 的經驗卻是完全通用的  
文章最後提到的其他參考資料，[https://clig.dev/](https://clig.dev/) 看起來超好料的，讓人感覺，做 CLI 就是必讀的程度

---

## 1 CLI 體驗: 開發良好 UX 的 NodeJS CLI best practice

- 1.1 尊重 POSIX 參數
- 1.2 構建同理心 args
- 1.3 Stateful 態資料
- 1.4 提供 colorful experience
- 1.5 豐富的互動
- 1.6 處處超連結
- 1.7 Zero configuration
- 1.8 尊重 POSIX signals

---

### 1.1 尊重 POSIX args

- ✅ 推薦做法: 使用符合 POSIX 的 CLI 參數語法，這是 CLI 工具的廣泛接受標準
- ❌ 否則: 當與 User 習慣的 Unix 不一致時，User 會感到沮喪
- ℹ️ 詳細說明
  - Unix-like OS 普及了 CLI tool，如 `awk`, `sed`
  - 這些工具標準化了 CLI flags, args 和其他操作數的行為

一些預期行為的例子:

- arguments 在 help 或 examples 中 `[]` 表示它們是 optional 的，用 `<>` 表示它們是 required 的
- 允許短格式單字母參數作為長格式參數的別名
  - 參考 [GNU Coding Standards](https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html)
- 短格式單數 `-` 指定選項可能包含一個字母或數字
- 可以將沒有值的多個選項組合在一起，如 `myCli -abc` 等同於 `myCli -a -b -c`

**CLI 的 high-level User 會期望你的 CLI 具有與其他 Unix 程式有相似的慣例**

📦 推薦的套件: 參考 Node.js packages:

- commander: https://github.com/tj/commander.js#readme
- yargs: https://github.com/yargs/yargs

---

### 1.2 同理心 CLI

- ✅ 推薦做法: 建立能夠幫助 User 成功互動的工作流程，否則將導致錯誤和沮喪
- ❌ 否則: 不能提供可操作幫助 User，將因缺乏使用 CLI 的能力而沮喪
- ℹ️ 詳細說明
  - 為 CLI 介面在保證其成功使用，與 Web UI 沒有什麼不同，應該盡你所能幫助 User

製作同理心 CLI 來改善成功的互動。例如

- `curl`，它期望 `URL` 作為其主要 input，而 User 未能提供它
  - 這會出現一條訊息是，希望 User 去查看 `curl --help`
- 一個互動式提示來，來取得 User 的 input，建構成功的互動

---

### 1.3 有狀態的 state

- ✅ 推薦做法: 在多次呼叫你的 CLI 之間提供 stateful 體驗，記住 state 以提供無縫的互動
- ❌ 否則: 要求 User 在多次呼叫 CLI 時，反覆提供相同的資料會讓 User 感到煩躁
- ℹ️ 詳細說明:
  - 你會發現需要為 CLI 提供 data persist
    - 如，多次呼叫 CLI 之間記住 username, email, API token, or other preferences
  - 用 configuration helper 來 persist 這些 User setting
  - 確保遵守 [XDG Base Directory 規範](https://specifications.freedesktop.org/basedir-spec/latest/)，讀取/寫入檔案(或用一個符合 spec 的 configuration helper)
    - 這樣可以讓 User 控制檔案寫入和管理位置

Reference projects:

- configstore: https://www.npmjs.com/package/configstore
- conf: https://www.npmjs.com/package/conf

---

### 1.4 提供 colorful UX

- ✅ 推薦做法: 在 CLI 中用顏色來突顯 output，並提供優雅的降級或顏色偵測
  - 確保能通過 option, env, flag 手動進行啟用/關閉
- ❌ 否則: 在 output 為 plain text 時，資料很容易丟失
- ℹ️ 詳細說明:
  - 如今大多 Terminals 與 CLI 都支援 colored text
    - 比如這些由特殊製作的 ANSI encoded characters 的顏色
  - 用顏色顯示可以進一步提供豐富的體驗和增加互動性
  - 需要注意的是，不支援顏色的 Terminal 可能會在螢幕上出現資料混亂的情況
  - 此外，CLI 可能會在不支援顏色輸出的 CI 作業中使用
  - 即使在 server 之外，CLI 也可能通過 IDE 的 console 使用，這可能無法處理某些字符
  - 因此，必須提供手動開關的選項

📦 推薦的 Node.js library:

- chalk: https://www.npmjs.com/package/chalk
- colors: https://www.npmjs.com/package/colors
- kleur: https://www.npmjs.com/package/kleur

---

### 1.5 豐富的互動

- ✅ 推薦做法: 利用豐富的 CLI 互動來提升 UX
- ❌ 否則: 當需要考慮的資料以封閉選項(如下拉選單)的形式出現時，User 需要輸入就比較麻煩
- ℹ️ Details
  - 豐富的互動可以通過 prompt inputs 來實現
    - 這比 text 更複雜，如下拉選單、單選按鈕切換、auto-complete、hidden password inputs
  - 另種豐富互動是動畫和進度條，在 async job 時提供更好的 UX
  - 許多 CLI 在不需要任何進一步互動的情況下提供預設的參數。不要強迫 Uuser 提供 CLI 可以自行解決的參數

📦 推薦的 Node.js library:

- enquirer: https://www.npmjs.com/package/enquirer
- ora: https://www.npmjs.com/package/ora
- ink: https://www.npmjs.com/package/ink
- prompts: https://www.npmjs.com/package/prompts

---

### 1.6 Hyperlinks everywhere

- ✅ 推薦做法: 在 text output 中對 URL(如 `https://www.github.com`)和原始碼(如 `src/Util.js:2:75`)使用正確格式化的超連結
  - Modern terminal 能夠將這些轉換為可點擊的連結
- ❌ 否則: 避免用像 `git.io/abc` 這樣的非互動連結，這需要 User 手動 copy/past
- ℹ️ 詳細說明
  - 如果你正在分享 URL，或者指向文件中的具體行號和列號，你可以提供正確格式化連結，點了就會在 browser or IDE 中打開

參考專案:

- open: https://github.com/sindresorhus/open

---

### 1.7 Zero configuration

- ✅ 推薦做法: 自動偵測所需的 config 和 arguments 來改善 UX
- ❌ 否則: 如果 argument 可以靠自動偵測，並且操作不明確需要 User 互動(如確認刪除)，不要強迫 User 使用者進行互動
- ℹ️ 詳細說明:
  - 目標是執行 CLI 時提供開箱即用的體驗
  - 例如，[POSIX 定義了用於不同目的的環境變數配置設定](https://pubs.opengroup.org/onlinepubs/009695399/basedefs/xbd_chap08.html)，如: `TMPDIR`, `NO_COLOR`, `DEBUG`, `HTTP_PROXY` 等
  - 自動偵測這些，並在必要時提示確認

Zero configuration 的參考專案:

- The [Jest JavaScript Testing Framework](https://jestjs.io/)
- [Parcel](https://parceljs.org/), a web application bundler

---

### 1.8 遵守 POSIX signals

- ✅ 推薦做法: 確保遵守 [POSIX signals](https://man7.org/linux/man-pages/man7/signal.7.html)，允許它與 User 或其他程式正確互動
- ❌ 否則: 程式將無法與其他程式良好互動
- ℹ️ 詳細說明
  - 對於 CLI，與 User input 互動是很常見的，不當管理 keyboard event 會導致無法回應 `SIGINT` 中斷信號
    - 這是 User 按下 `CTRL+C` 時的 signal
  - 不尊重 signal 在程式被非人類互動時會變得更糟
    - 例如，在 docker container 中執行的 CLI，但卻不遵守 interrupt signals

---

## 2 發行: 有關 distributing 和 packaging Node.js 的 best practice

- 2.1 偏好 small dependency
- 2.2 使用 `shrinkwrap`, `Luke`
- 2.3 清理 configuration files

---

### 2.1 越小的 dependency 越好

- ✅ 推薦做法: Minimize dependencies，以確保 CLI 的 bundle 小一點
  - 謹慎、避免過度優化 dep 的使用，不要重新發明輪子
- ❌ 否則: dependencies 大小和使用將影響 CLI 安裝時間，bad UX
- ℹ️ 詳細說明
  - 由 `npx` 呼叫 CLI 時，快速的 `npm install` 能提供更好的 UX
  - 一次性的 `global npm install` 慢速安裝，將提供一次性的不良體驗
  - 但 User 用 `npx` 呼叫 executable packages 時 performance 不會太好，因為 `npx` 總是從 registry 中 fetch 和安裝，體積大的 CLI，UX 感受就會更差

參考專案:

- [Bundlephobia](https://bundlephobia.com/) 能幫助找出 npm package size 的工具

---

### 2.2 使用 `shrinkwrap`, `Luke`

- ✅ 推薦做法: 用 `npm` 的 `npm-shrinkwrap.json` 作為 lockfile，以確保 User 安裝 package 下的 dependency versions (direct and transitive) propagate 能夠傳到你的最終 User
- ❌ 否則: 不固定 dependencies 的版本，`npm` 會在安裝期間解析它們，可能會引入你無法控制的 breaking change，導致 CLI 不正常
- ℹ️ 詳細說明
  - 用 `shrinkwrap`
  - 用 `npm shrinkwrap` 命令建立 `shrinkwrap` lockfile，其格式與 `package-lock.json` 相同
  - 安裝固定的 dependencies 版本，即使 dependencies 發了新版本，也不會被安裝
  - 這將責任轉移到維護者，需要保持對 dependencies 中的任何 security 更新，並定期發佈具有安全更新的 CLI
    - 考慮用 `Snyk Dependency Upgrade` 自動修復 dependency tree 中的安全問題

另種方法是將 dependencies 綁在已發佈的套件中

- 這具有加速安裝的優勢，因為減少了解析 dependencies 的需求以及 request/download
- 缺點是成為一個不透明的盒子，難以分析 dependency tree，導致像 Snyk 這樣的安全工具不會報告漏洞
- [ncc](https://www.npmjs.com/package/@vercel/ncc) 用於將 Node.js module compile 成一個包含所有 dependencies 的 檔案

---

### 2.3 清除 configuration files

- ✅ 推薦做法: 當 CLI uninstall 時，(可選的)清除 config
  - CLI 可以提示 User 保留 config，以便在下次安裝時跳過初始化階段，從而提供更好的 UX
- ❌ 否則: User config 會殘留(可能包含 identifiable data)
- ℹ️ 詳細說明
  - 如果你的 CLI 用 persistent storage 來保 config，那當 uninstall 時，也應該負責移除 config
  - 由於 `npm` 從 `v7`，就不提 uninstall hook，你的程式應該包括 uninstall 選項，用參數(如 `--uninstall`)或豐富互動來實現
  - 或者，可以提供 `pre` 和 `post` 的 uninstall [script](https://docs.npmjs.com/cli/v10/using-npm/scripts)
    - 如果 `npm v6` 或更低，這些 script 會自動呼叫
    - 範例: https://github.com/m-sureshraj/jenni/blob/master/src/scripts/pre-uninstall.js

---

## 3 Interoperability (互操作性): 無縫整合其他 CLI 工具，並遵循 CLI conventions 的 best practice

下面回答了以下問題:

- 我可以導出這個 CLI 的 output 以便於解析嗎？
- 我可以將這個 CLI 的 output 通過 pipe 傳到另個 CLI input 嗎？
- 我可以將另個工具的結果通過 pipe 傳到這個 CLI 嗎？

本節內容:

- 3.1 接受 STDIN input
- 3.2 啟用結構化 output
- 3.3 Cross-platform 禮儀
- 3.4 支援 config 的優先級

---

### 3.1 接受 `STDIN` input

- ✅ 推薦做法: 對於預期處理資料的 CLI，讓 User 能用 `STDIN` 傳入資料
- ❌ 否則: 其他 CLI 將無法直接將其 output 作為 input 提供給你的 CLI

常見的這類用法都會失敗:

```sh
$ curl -s "https://api.example.com/data.json" | your_node_cli
```

ℹ️ Details

- 如果 CLI 處理資料，例如對 JSON 執行某種任務，通常用 `--file <file.json>` 命令行參數來指定

[Node.js readline](https://nodejs.org/api/readline.html) 的例子如下:

```js
const readline = require("readline");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

rl.question("What do you think of Node.js? ", (answer) => {
  // TODO: Log the answer in a database
  console.log(`Thank you for your valuable feedback: ${answer}`);

  rl.close();
});
```

然後 pipe input 到上面的程式碼:

```sh
echo "Node.js is amazing" | node cli.js
```

---

### 3.2 啟用 structured output

- ✅ 推薦做法: 用一個 flag 來允許 structured output 結果，從而便於解析和簡單操作資料
- ❌ 否則: CLI 的 User 可能需要用複雜的 regex 解析和匹配 output
- ℹ️ 詳細說明
  - 對於 CLI 的 User 來說，解析資料並執行其他任務、整合其他工作都是很有用的
  - 能夠輕鬆的從 CLI output 抓感興趣的資料，給 User 好的 UX

---

### 3.3 Cross-platform

- ✅ 推薦做法: 如果預期 CLI 能跨平台上，必須適當關注命令 Shell 的語義和組件(如 file system)，以及 relevant programming constructs
- ❌ 否則: 在其他 OS 上因為像路徑符號不正確等因素而失效，即使功能上沒有區別
- ℹ️ 詳細說明

即使從程式的角度來看，功能沒有被削減，應在不同 OS 上良好運行

- 一些被忽視的細微差異可能使程式無法運作
- 我們來探討幾個必須遵守 Cross-platform 的情況

Cross-platform: 錯誤的建立命令

- 你可能需要建立一個 Node.js script。例如:

`program.js`:

```js
#!/usr/bin/env node

// the rest of your app code
```

這可以 work

```js
const cliExecPath = "program.js";
const process = childProcess.spawn(cliExecPath, []);
```

下面這種方式更好，為什麼它更好？

- `program.js` source code 以類 Unix 的 [Shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>) 標記法開頭
- 然而 Shebang 不是跨平台標準， Windows 不知道如何解釋它

```js
const cliExecPath = "program.js";
const process = childProcess.spawn("node", [cliExecPath]);
```

上面的議題，也適用於 `package.json` scripts

- 下面這種 npm script 就是 bad practice
- 因為 `myInstalls.js` 中的 Shebang，在 Windows 上沒用，Windows 不會知道要用 Node 去執行它

```json
"scripts": {
  "postinstall": "myInstall.js"
}
```

相反，follow 以下 best practice:

```json
"scripts": {
  "postinstall": "node myInstall.js"
}
```

Cross-platform: Shell 的 interpreters 有所不同

- 不同的 Shell interpreters 對所有 characters 的處理方式不完全相同
- 例如，Windows command 符對 single quote (`'`) 和 double quote (`"`) 的處理方式不同於 bash Shell
  - 它不能識別 single quote 內的整個字串屬於同一字串組，這會導致錯誤

下面這在 Windows command prompt 執行會失敗:  
`package.json`:

```json
"scripts": {
  "format": "prettier-standard '**/*.js'",
  ...
}
```

要修這個問題，使這個 `npm run` script 能在 Windows, macOS, Linux 間真正跨平台運行:

- 須使用 double quotes 並用 escape 轉義它

`package.json`:

```json
"scripts": {
  "format": "prettier-standard \"**/*.js\"",
  ...
}
```

Cross-platform: 避免連接路徑

- 不同 OS 構建路徑的方式不同。當它們通過連接字串手動構建時，將無法在不同平台之間互操作

下面是 bad practice:

- windows 上 path separator 不是 backslash

```js
const myPath = `${__dirname}/../bin/myBin.js`;
```

不要手動構建 file path，而是用 Node.js 自身的 `path` 來處理:

```js
const myPath = path.join(__dirname, "..", "bin", "myBin.js");
```

Cross-platform: 避免用分號(semicolons)組合命令

- 已知 Linux Shell 支援用分號(`;`)連結命令以順序運行，如: `cd /tmp; ls`
- 但 Windows 上會失敗

避免以下做法:

```js
const process = childProcess.exec(`${cliExecPath}; ${cliExecPath2}`);
```

相反，使用雙和號(double ampersand)或雙管道符號(double pipe ):

```js
const process = childProcess.exec(`${cliExecPath} || ${cliExecPath2}`);
```

---

### 3.4 支援 config 優先級

- ✅ 推薦做法: 支援根據優先級，來讀取多個來源的 config。CLI arguments 具有最高優先，其次是 Shell 變數，然後是不同級別的 config
- ❌ 否則: User 在自定義自己的 config 時，UX 很差
- ℹ️ 詳細說明
  - 偵測並支援使用 env 進行 config，這是許多工具呼叫 CLI 行為的常見方式

CLI config 優先順序如下:

1. CLI 呼叫時指定的命令行 arguments
2. Shell 的 env，以及 application 可用的任何其他 env
3. project 範圍 config，如: local directory `.git/config` 檔案
4. User 範圍 config，如: User 的主目錄: `~/.gitconfig` 或其 XDG 等效檔案: `~/.config/git/config`
5. 系統範圍 config，如: `/etc/gitconfig`。

Reference projects:

- cosmiconfig: https://github.com/cosmiconfig/cosmiconfig

---

## 4 無障礙(Accessibility)

希望 CLI 在開發者沒有預期到的環境上也能順利被使用

- 4.1 將 CLI Containerize
- 4.2 優雅降級
- 4.3 Node.js 版本相容性
- 4.4 Shebang 自動偵測 Node.js 執行環境

---

### 4.1 將 CLI 容器化

- ✅ 推薦做法: 建立 image，並發到 Docker Hub public registry，以便沒有 Node.js 的 User 也能用它
- ❌ 否則: 沒有 Node.js 環境的 User 將無法使用
- ℹ️ 詳細說明
  - 從 npm 安裝 Node.js CLI 通常會用 Node.js 的工具，如 `npm` 或 `npx`。這些工具在 JavaScript 和 Node.js 開發者中很常見
  - 然而，如果你打算讓一般 User 使用 CLI，無論他們是否熟悉 JavaScript，或者是否有這些工具，僅以 npm 發布分發的 CLI 將會受到限制
    - 如果你的 CLI 打算在構建或 CI 環境中使用，那這些環境也可能需要安裝與 Node.js 相關工具
  - 有許多方法可以打包和分發可執行文件，將其容器化是一種易於消費的替代方案(dependency-free )
    - (除了需要 Docker 環境)。

---

### 4.2 Graceful degradation (優雅降級)

- ✅ 推薦做法: 能讓 User 在不受支援的環境中，能夠自主關掉顏色跟動畫，例如能夠跳過互動並以 JSON 格式 output
- ❌ 否則: 強制有彩色 output，使用互動功能(如提示和其他顯示豐富界面)，會讓沒有支援的 terminal 的 User 不想使用
- ℹ️ 詳細說明
  - 提供豐富的 terminal display(如彩色輸出、ASCII、提示機制)很常見
  - 這些會提升 UX，但對於那些沒有受支援 terminal 的 User，可能會顯示混亂的文字或完全無法操作

為了這些 User 能使用 Node.js CLI，你可以:

- 自動偵測 terminal 能力，並在執行時評估是否降級 CLI 互動性
- 提供 User 切換模式，例如 `--json` 參數強制輸出原始資料
- 支援優雅降級(如 JSON )不僅對 Use r 及其不受支援的 terminal 有用，還對在 CI 環境中很有價值，使 User 能將程式 output 與其他程式 input 連接或在需要時導出資料

---

### 4.3 Node.js 版本相容性

- ✅ 推薦做法: 針對受支援和維護的 [Node.js 版本](https://nodejs.org/en/about/previous-releases)。
- ❌ 否則: 試圖與舊和不受支援的版本相容的 code 將難以維護，並失去語言和執行時的好處
- ℹ️ 詳細說明
  - 有時可能需要專門針對缺少新 ECAMScript 功能的舊 Node.js 版本
    - 例如，如果開發一個主要面向 DevOps 或 IT 的 Node.js CLI，他們可能沒有最新的 Node.js 環境
    - 如 Debian Stretch(舊穩定版)提供 [Node.js 8.11.1](https://packages.debian.org/search?suite=default&section=all&arch=any&searchon=names&keywords=nodejs)。
  - 如果確實需要針對 Node.js 8、6 等舊版本，最好用像 Babel 來確保建立的 code 符合這些版本的 V8 JavaScript 引擎和 Node.js 執行版本
  - 另個方案是 CLI containerized 以避免舊目標
  - 不要去寫舊版的 code 來去支援舊版 Node.js，這只會導致維護問題
  - 如果在不受支援的環境中，嘗試偵測並退出，並提供描述性錯誤資訊，顯示友好錯誤消息
  - 參考 dockly 範例
    - https://github.com/lirantal/dockly/blob/42d8c09631bc5348f108a50c3ce9601851fb760b/index.js#L25

---

### 4.4 Shebang 自動偵測 Node.js 執行環境

- ✅ 推薦做法: 在 [Shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>) 聲明 Node.js 環境，如 `#!/usr/bin/env node`，跟著 runtime environment
- ❌ 否則: 如果 hardcode Node.js 執行環境位置(如 `#!/usr/local/bin/node`)，這可能只適用於你的環境，可能會導致 CLI 在其他 Node.js 位置不同的環境中無法操作
- ℹ️ 詳細說明
  - 指定 `#!/usr/bin/env node` 為 best practice，仍假設 Node.js 執行環境被引用為 `node` 而不是 `nodejs` 或其他名稱，否則，可能會因為 nodejs executeable file 位置不一樣而出錯

---

## 5 測試

- 5.1 Put no trust in locales

---

### 5.1 Put no trust in locales

- ✅ 推薦做法: 測試可能在不同於你的系統或 English default 的系統上執行
- ❌ 否則: 開發者在不同於 English default 區域的系統上測試時，會測試失敗
- ℹ️ 詳細說明

在透過執行 CLI 並解析 output 來測試時，你可能傾向於 grep 特定功能以確保它們存在於 output 中

- 例如在 CLI 沒有參數執行時正確提供範例:

```js
const output = execSync(cli);
expect(output).to.contain("Examples:"));
```

當測試在非英語區的系統上時，如果 CLI argument parsing library 自動偵測區域並採用它，那這測試將會失敗

- 因為 `Examples` 被轉換為區域的等效語言並設為系統的默認區域

---

## 6 錯誤

目標是幫助 User 快速輕鬆地排除錯誤，而無需查文件 or source code 來理解錯誤

- 6.1 可追踪的錯誤
- 6.2 可操作的 errors
- 6.3 提供 debug 模式
- 6.4 使用正確的 exit codes
- 6.5 要讓 User 能輕鬆 report bug

---

### 6.1 可追踪的錯誤

- ✅ 推薦做法: 在 report error 時，提供可追踪的 error codes，可以在文件中查，並簡化錯誤消息的故障排除
  - 如果可能，為 trackable error codes 提供更多資訊，使其易於解析並使 context 清晰
- ❌ 否則: 通用的 error message 往往模糊不清，使 User 難搜尋解決方案
  - 解析和分析也不是那麼直觀，在文件中引用它們也不那麼清晰
- ℹ️ 詳細說明
  - 確保在返回 error message 時，包含參考號或特定的 error code，以便查詢
  - 就像 HTTP status code 一樣，CLI 也需要命名或編碼的錯誤

Example:

```sh
$ my-cli-tool --doSomething

Error (E4002): please provide an API token via environment variables
```

---

### 6.2 可操作的 errors

- ✅ 推薦做法: 失敗的 error message 應告訴 User 該怎麼處理，而不是單純的抱怨有錯
- ❌ 否則: 面對錯誤消息而沒有解決錯誤的行動提示的 User，無法成功用你的 CLI

Example:

```sh
$ my-cli-tool --doSomething

Error (E4002): please provide an API token via environment variables
```

---

### 6.3 提供 debug 模式

- ✅ 推薦做法: 允許 User 啟用更多詳細資料，以便 debug
- ❌ 否則: 盡量要有 debug 模式的功能，不然會更難從 User 那收集反饋，並使 User 難以找出錯誤原因
- ℹ️ 詳細說明
  - 使用 env 及 CLI arguments 來啟用 debug verbosity levels
  - 在 code 中適當的位置加上 message log，幫助 User 和維護者理解程式流程、input 和 output 以及其他使問題解決變得更容易的資料

📦 推薦的套件

- `debug`: https://www.npmjs.com/package/debug

---

### 6.4 使用正確的 exit codes

- ✅ 推薦做法: 用正確 exit codes 來終止程式，傳錯誤或退出狀態的語義意義
- ❌ 否則: 不正確的 exit codes 會沒法順利跟其他工具一起整合在 CI 中
- ℹ️ 詳細說明
  - CLI 通常用 shell 的 `$?` 來推斷程式的 status code 並據此行動
  - 這也在 CI 流程中用於確定步驟是否成功完成
  - 如果在錯誤情況下始終終止而沒有特定的 status code，那麼依賴它的 shell 和其他程式無法知道這一點

當錯誤發生導致程式終止時，應傳達這個意義。例如:

```js
try {
  // something
} catch (err) {
  // cleanup or otherwise
  // then exit with proper status code
  process.exit(1);
}
```

exit codes 簡短參考:

- exit codes 0 表示執行成功
- exit codes 1 表示失敗

也可以選擇使用具有程式語義的自定義 exit codes，但如果這樣做，請確保正確記錄它們

參考: [BASH shell 用的 exit codes list](http://www.tldp.org/LDP/abs/html/exitcodes.html)

---

### 6.5 要讓 User 能輕鬆 report bug

- ✅ 推薦做法: 提供 URL 打開問題並儘可能預填所需資料
  - [如 Github 的 Issue templates](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository)，可以指導 User 哪些是必要資料
- ❌ 否則: User 在搜尋如何 report 時會感到沮喪，可能最終提供很少有用的資料，或者根本不回報

---

## 7 開發和維護的 best practice

- 7.1 使用 bin object
- 7.2 使用相對路徑
- 7.3 使用 `files` 欄位

---

### 7.1 使用 bin object

- ✅ 推薦做法: 使用 object 來定義可執行文件的名稱及路徑
- ❌ 否則: 最終會將 package name 與可執行文件耦合在一起
- ℹ️ 詳細說明

以下 `package.json` 顯示了一個將可執行文件的名稱與 filename 及其在專案中的位置分離的範例:

```json
"bin": {
  "myCli-is-cool": "./bin/myCli.js"
}
```

---

### 7.2 使用相對路徑

- ✅ 推薦做法: 用 `process.cwd()` 存取 User 輸入路徑，用 `__dirname` 存取專案中的路徑
- ❌ 否則: 將會遇到錯誤的路徑，無法存取檔案
- ℹ️ 詳細說明
  - 你可能需要存取專案範圍內的檔案，或 User 輸入的檔案，如 log, JSON
  - 混著用 `process.cwd()` 或 `__dirname` 可能會導致錯誤，或者不用它們會產生問題

正確存取檔案方法:

- `process.cwd()`: 當需要存取的路徑依賴於 CLI 的相對位置時使用

  - 例子是當 CLI 支援路徑以建立 log 時，如: `myCli --outfile ../../out.json`
  - 如果 myCli 安裝在 `/usr/local/node_modules/myCli/bin/myCli.js`，那 `process.cwd()` 不會指向那個位置，而是指當前工作目錄，這是 User 在呼叫 CLI 時所在的目錄

- `__dirname`: 當需要從 CLI 的 source code 中存取檔案，並引用 code 所在檔案的相關位置時使用
  - 例如，當 CLI 需要存取另個目錄中的 JSON 時:
    - `fs.readFile(path.join(__dirname, '..', 'myDataFile.json'))`

---

### 7.3 使用 `files` 欄位

- ✅ 推薦做法: 用 `files` 欄位，包含發佈套件中所需的檔案
- ❌ 否則: 最終會包含不需要用來運行 CLI 的檔案的 package，如: test file, dev config 等
- ℹ️ 詳細說明
  - 為了保持發佈 package 的 small size，我們應只包括運行 CLI 所需的檔案
  - 詳情: https://nodejs.medium.com/publishing-npm-packages-c4c615a0fc6b

以下 `files` 欄位告訴 npm CLI 包括 src 目錄中的所有檔案，除了 `spec` 檔案

```json
"files": [
  "src",
  "!src/**/*.spec.js"
],
```

---

## 8 Analytics

8.1 嚴格的選擇加入分析

---

### 8.1 Strict Opt-in Analytics (嚴格的選擇加入分析)

8.1 嚴格的選擇加入分析

- ✅ 推薦做法: 始終以明確的方式提示、詢問或讓 User 選擇加入，以提供使用和產品分析到 server
- ❌ 否則: 隱私問題引起 User 意想不到的 CLI 行為。
- ℹ️ 詳細說明
  - 作為 CLI 維護者，會希望更好地了解 User 如何使用它
  - 然而，未經 User 同意就默默地進行搜集，將會受到批評

Guidelines:

- 讓 User 知道會收集哪些資料以及如何處理這些資料
- 注意隱私，避免收集可能識別個人的資料
- 說明資料儲存的方式、地點和期限

其他 CLI 專案收集分析資料的參考:

- [Angular CLI](https://v17.angular.io/cli/analytics) 和
- [Next.js CLI](https://nextjs.org/telemetry)

---

## 9 版本控制

- 9.1 包含 `--version` flag
- 9.2 採用 Semantic Versioning
- 9.3 在 `package.json` 中提供版本資料
- 9.4 在 error message 和 hele text 中顯示版本
- 9.5 向後相容性
- 9.6 在 `npm` 上發布版本化的發行版
- 9.7 更新程式的版本文件

---

### 9.1 包含 `--version` flg

- ✅ 推薦做法: 適當的 flag，User 能夠輕鬆檢查版本
- ❌ 否則: User 將不知道用的是哪個版本，使跟蹤更新或報告問題變得困難
- ℹ️ 詳細說明
  - 你需要一個功能來顯示版本
  - 用 flag 在 stdout 中顯示版本並退出程式(通常也會印出編譯的 configd detail)
  - 也可以用 `-V` short flag

Example:

```sh
$ my-cli-tool --version

my-cli-tool version 11.0.5, build 11.0.5-0ubuntu1~22.04.1
```

---

### 9.2. 採用 Semantic Versioning

9.2 採用語義版本控制

- ✅ 推薦做法: SemVer 提供清晰、標準化的方式來傳達版本資訊，User 更容易理解
- ❌ 否則: User 可能無法理解，導致困惑或意外行為

---

### 9.3. 在 `package.json` 提供版本資料

- ✅ 推薦做法: `package.json` 加版本資料，可以確保專案的一致性，並使自動化版本更新變容易
- ❌ 否則: 你和 User 將對追蹤 dependencies 關係和管理版本有困難
- ℹ️ 詳細說明
  - `npm` 或 `yarn` 提供 version management 功能，處理 dependencies 和版本控制變簡單
  - 用這些工具帶來的自動版本(如 `npm version`)減少人為錯誤風險並簡化過程

---

### 9.4. 在 Error Messages 和 help text 中顯示版本

- ✅ 推薦做法: 提供版本資料，讓 User 在回報問題時可以包含版本資料，有助於 debug
- ❌ 否則: debug 更困難，User 在尋求幫助時可能不知道該用哪個版本
  - 甚至在 User 將你的程式作為其他程式的 dependency 時也是如此。

---

### 9.5. 向後相容性

- ✅ 推薦做法: 確保程式與舊版本的向後相容性，User 升級時不會破壞工作流程
- ❌ 否則: 經常打破相容性會使 User 沮喪，妨礙採用
- ℹ️ 詳細說明
  - 確保在移除任何功能或更改 intput/output 格式時，也會為 User 提供描述
  - 文件上列出即將棄用或已棄用功能及其替代方案
  - 向 User 顯示，告知他們如何通過查看 release 說明來了解功能支援的結束

Example:

```sh
DEPRECATED: The blah-blah-feat is deprecated and will be removed in a future release.
            Install the blah-feat component to do the blah:
            <https://url/to/docs>
```

---

### 9.6 在 npm 上發布版本化的發行版

- ✅ 推薦做法: npm 上發布帶有版本 tag 的，使 User 能夠輕鬆安裝特定版本
- ❌ 否則: User 將無法訪問以前版本，這可能是兼容性或故障排除所必需的

---

### 9.7 更新程式的版本文件

- ✅ 推薦做法: 清晰的 release 說明來通知 User 每個版本中的更改
  - Change log 幫助 User 了解版本間的變化及其對使用的影響
- ❌ 否則: User 將不知道新版本中的內容，可能難以評估是否應該升級

---

## 10 安全性

- 10.1 最小化 Argument Injection

---

### 10.1 最小化 Argument Injection

- ✅ 推薦做法: 仔細考慮 CLI 啟用的命令行參數及其開放的命令
  - 盡量避免敏感的系統任務，如 fs read/write
- ❌ 否則: CLI 有被攻擊的風險，exploiting CLI argument flags 來進行 file read/write, command execution 等等
- ℹ️ 詳細說明
  - Argument injection 攻擊利用 CLI 解析 User intput 的漏洞
  - 當不受信任的 User input 被包括在執行的命令中時，攻擊就會發生
  - 攻擊者的命令參數和參數的輸入，以執行惡意操作或訪問未經授權的資料

CLI argument injection 的 case:

- Vulnerability in [git-interface](https://security.snyk.io/vuln/SNYK-JS-GITINTERFACE-2774028)
- Vulnerability in [git-pull-or-clone](https://security.snyk.io/vuln/SNYK-JS-GITPULLORCLONE-2434307)
- Vulnerability in [ungit](https://security.snyk.io/vuln/SNYK-JS-UNGIT-2414099)
- Vulnerability in [simple-git](https://security.snyk.io/vuln/SNYK-JS-SIMPLEGIT-2421199)

References for

- [Blamer npm package vulnerable to argument injection](https://www.nodejs-security.com/blog/destroyed-by-dashes-how-two-hyphens-cause-argument-injection-vulnerability-in-blamer-npm-package), and
- [Node.js Secure Coding: Defending Against Command Injection](https://www.nodejs-security.com/book/command-injection) book.

---

## 11 Appendix: CLI Frameworks

11.1 CLI Frameworks Table

|              |                                                                             |                                                                   |
| :----------- | :-------------------------------------------------------------------------- | :---------------------------------------------------------------- |
| `oclif`      | 做 CLI 的 framework                                                         | https://github.com/oclif/oclif                                    |
| `inquirer`   | A collection of common interactive command line user interfaces.            | https://github.com/SBoudrias/Inquirer.js                          |
| `ink`        | 用 React 的寫法來寫 CLI                                                     | https://github.com/vadimdemedes/ink                               |
| `pastel`     | 用 Ink 做的，一個類似 Next.js 的 CLI framework                              | https://github.com/vadimdemedes/pastel                            |
| `ink UI`     | CLI customizable UI components Collection、Ink 開發的                       | https://github.com/vadimdemedes/ink-ui                            |
| `blessed`    | A curses-like library with a high level terminal interface API for node.js. | https://github.com/chjj/blessed                                   |
| `prompts`    | Lightweight, beautiful and user-friendly interactive prompts                | https://github.com/terkelg/prompts                                |
| `vue-termui` | ue.js based terminal UI framework                                           | https://github.com/vue-terminal/vue-termui                        |
| `clack`      | 輕鬆做 CLI apps                                                             | https://github.com/bombshell-dev/clack/tree/main/packages/prompts |

---

## 12 Appendix: CLI educational resources

- https://clig.dev/
- https://primer.style/cli/getting-started/principles
