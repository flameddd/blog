# 2024-12-08: 筆記 Command Line Interface Guidelines.md

## Aanand Prasad, Ben Firshman, Carl Tashian, Eva Parish

### https://clig.dev

---

對如何建立好的 CLI 工具有點興趣，從 Liran Tal 的文章看到這篇，感覺非常有料，快點來看看

作者群:

- Aanand Prasad: Squarespace 工程師，Docker Compose 共同創作者
- Ben Firshman Replicate 共同創作者，Docker Compose 共同創作者
- Carl Tashian: Smallstep Offroad 工程師，Zipcar 首位工程師，Trove 共同創辦人
- Eva Parish: Squarespace 技術作家，O'Reilly 撰稿人

都感覺是很有硬實力的人，所以這篇我很期待

---

## Foreword

在 1980 年代

- 如果你想要電腦為你做些什麼，你需要知道 `C:\>` 或 `~$` 該輸入什麼
- 此時只有一大本手冊、模糊不清錯誤資訊，沒有 Stack Overflow 來拯救你
- 但如果你夠幸運擁有 Internet，你可以從 Usenet 獲得幫助
  - 一個早期的網際社群，充滿和你一樣沮喪的人。他們可以幫助你解決問題，或者至少提供一些支持和夥伴情誼

四十年後

- 電腦變得對每個人都更加易於使用
- 往往是犧牲 low level end user control 為代價
- 在許多設備上，根本沒有 CLI，部分原因是它違背了企業利益

今天，大多數人不知道 CLI 是什麼，更不用說為什麼他們會想要去麻煩它

- Computing pioneer `Alan Kay` 在 2017 年採訪中說:
  - `因為人們不理解 computing 是什麼，所以他們認為在 iPhone 中有了它，這種錯覺就像認為 Guitar Hero 和真正的吉他是一樣的。`

然而，雖然有著數十年來的限制和無法解釋的怪癖

- CLI 仍然是電腦中最通用的角落，並且以 GUI 無法承擔的複雜和深度與機器進行創造性的互動
- 它在任何電腦上都可以使用
- 對任何想要學習的人都開放。它可以互動使用，也可以自動化
- 而且，它的變化速度不像系統的其他部分那樣快。它的穩定性有創造價值

因此，既然我們還擁有它，我們應努力最大限度地利用它的 utility 和 accessibility

過去的 CLI 是以機器為中心的:

- 只是 `REPL` 之上的一個 scripting platform
- 但隨著通用解釋性語言的繁榮，shell script 角色縮小了

今天的 CLI 是以人為中心的:

- 一種基於 text 的 UI，提供了對各種工具、系統和平台的訪問

受到傳統 UNIX 哲學的啟發，出於對更愉快和可訪問的 CLI 環境的興趣，以及我們作為程式設計師的經驗指導

- 我們決定是時候重新審視構建 CLI 的 `best practices` 和 `design principles`

Command Line 萬歲！

---

## Introduction

這份文件涵蓋

- high-level 的設計哲學和具體的指南
- 指南的部分更多，因為我們作為實踐者
  - 我們相信通過例子來學習，因此提供很多例子

誰適合讀這份文件？

- 如果你正在建立 CLI，並且尋找它的 UI 設計原則和具體的最佳實踐
- 如果你是專業的 CLI UI 設計師，那太棒了，我們希望向你學習
- 如果你想避免一些明顯的錯誤，那些違背了 40 年 CLI 設計約定的錯誤
- 如果你希望通過你良好設計的程式來幫助別人

如果你正在建立 GUI

- 這個指南不適合你，如果你決定讀它，可能會學到一些 GUI 的 anti pattern

如果你正在設計一個沉浸式的全螢幕 CLI (like Vim)版本的 Minecraft

- 這個指南不適合你

---

## Philosophy

以下是我們認為良好 CLI 設計的 fundamental principles

---

## 以人為本的設計

傳統上 UNIX 命令是基於它們主要被其他程式使用的假設寫的

- 它們與程式語言中的 function 有更多共同之處，而不是 graphical application
- 今天，即使許多 CLI 專門由人類使用，它們的很多 interaction design 仍然承載著過去的包袱
  - 是時候拋棄這些包袱了: 主要由人類使用，它應該首先為人設計

---

## 簡單的組件，一起工作

原始 UNIX 哲學的核心信條([the original UNIX philosophy](https://en.wikipedia.org/wiki/Unix_philosophy))是

- 小而簡單且具有乾淨界面的程式可以組合起來構建更大的系統
- 與其向這些程式中加越來越多的功能，不如使程式足夠 modular 化，以便根據需要重新組合

在過去

- pipes 和 shell script 在組合的過程中起著關鍵作用
- 隨著通用解釋性語言的興起，它們的角色可能已經減少，但它們沒有消失
- 大規模自動化，以 CI/CD, orchestration and configuration management 的形式蓬勃發展
  - 使程式可組合比以往任何時候都重要

幸運的是

- 為此目的設計的 UNIX 環境的長期約定(習慣)今天仍然有助我們
- Standard `in/out/err`, `signals`, `exit codes` 和其他機制確保了不同程式可以很好地配合在一起
- 簡單的基於行的 text 易於在 CLI 間傳輸

最重要的是，為可組合性設計不需要與為人類優先設計相抵觸

- 這篇介紹很多建議都是關於如何同時實現這兩者的

---

## 一致性貫穿於程式

Terminal 的慣例已經深植於我們的指尖

- 我們必須支付前期成本，通過學習 CLI 語法、flags、environment variables 等來換取長期的效率，前提是程式一致。

在可能的情況下，

- **CLI 應遵循已存在的模式，這是使 CLI 直觀且可猜測的主因**
  - 這也是使使用者高效的原因。

話雖如此，有時一致性與易用性相衝突

- 例如，許多長期存在的 UNIX commands 預設不輸出大量資訊，這會讓不太熟悉 CLI 的人感到困惑

當遵循慣例會損害程式的可用性時

- 可能是時候打破它了，但決定應謹慎做出

---

## 說出(剛好)足夠的資訊就好

Terminal 是個純資訊的世界

- 你可以認為資訊即是 interface，並且，就像任何 interface 一樣，它往往資訊過多或過少
- 當一個 CLI 跑了數分鐘時，User 開始懷疑它是否 crash 了
  - 此時這個 CLI 說的太少了
- 當一個 CLI 出大量的 debug log，淹沒真正重要的內容時
  - 這個 CLI 說的太多了
- 最終結果是一樣的: **缺乏清晰度，讓 CLI 感到困惑和惱火**

找到平衡點非常困難

- 但如果軟體要賦能並服務其使用者，這是絕對至關重要

---

## 易於發現

在「使功能可以被發現」方面，GUI 具有優勢

- 你可以做的所有事情都列在螢幕，所以你在不需要學習任何東西的情況下找到需要的東西
- 甚至可能發現不知道的功能

It is assumed that command-line interfaces are the opposite of this—that you have to remember how to do everything. The original Macintosh Human Interface Guidelines, published in 1987, recommend “See-and-point (instead of remember-and-type),” as if you could only choose one or the other.

人們認為 CLI 與此相反，你必須記住如何做所有事情

- 這些事情不必互相排斥
- 使用 CLI 的效率來自於記住命令，但沒有理由 CLI 不能幫助你學習和記住

Discoverable CLI

- 擁有全面的 help texts，提供大量範例
- 建議下一步要執行的命令
- 建議出錯時該怎麼做
- 有很多可以從 GUI 借鑑的想法，使 CLI 更易於學習和使用

---

## 把對話視為常態

GUI 設計，尤其在早期

- 廣泛使用隱喻: 桌面、檔案、資料夾、回收桶。這非常有意義，因為電腦仍在努力使自己正當化
- 隱喻的實作容易是 GUI 相較於 CLI 的一大優勢
- 然而，諷刺的是，CLI 一直體現著一個意外的隱喻: **它是一場對話**

除了最簡單的 commands 之外，執程式通常涉及不只一次呼叫

- 這是因為很難一次就做對:
  1. 使用者輸入命令，出錯
  2. 改變命令，再次出錯
  3. 如此反覆，直到成功
- 這種通過反覆失敗學習的模式就像是 User 與程式的對話

反覆試驗(Trial-and-error)不是唯一的對話互動類型。還有其他的:

- 執行一個命令來設置工具，然後學習實際開始使用它的命令
- 執行多個命令來設置操作，然後執行最終命令(如多次 `git add`，接著是 `git commit`)
- 探索系統，例如，大量的 `cd` 和 `ls` 了解目錄結構，或 `git log` 和 `git show` 探索歷史
- 在實際執行之前進行複雜操作的 dry-run

CLI 互動的對話性質意味著你可以在其設計中引入相關技術

- 當 User 輸入無效時，可以建議可能的更正
- 當 User 進行多步驟過程時，可以清楚地顯示中間狀態
- 可以在他們做出可怕的操作之前向他們確認一切看起來都很好

無論你是否有意，User 都在與你的軟體對話

- 最糟糕的情況是，它是一場使他們感到愚蠢和憤怒的敵對對話
- 最好的情況是，它是一場愉快的交流，使他們帶著新知識和成就感快速前行

Further reading: `The Anti-Mac User Interface (Don Gentner and Jakob Nielsen)`

- https://www.nngroup.com/articles/anti-mac-interface/

---

## Robustness (穩健性)

主觀的穩健性需要注意細節，並認真考慮可能出錯的地方。這包含許多小事:

- 讓 User 了解正在發生的事情
- 解釋常見錯誤的含義
- 不要 print 看起來令人害怕的 stack traces
- 一般來說，保持簡單也可以帶來穩健性。許多特殊情況和複雜的 code 往往使程式脆弱

---

## 同理心

CLI 是程式設計師的創意工具包，因此它們應該是令人愉快的使用工具

- 這並不意味著將它們變成遊戲，或使用大量表情符號(表情符號本身沒有什麼錯 😉)
- 給 User 一種感覺，讓他們感覺你在他們一邊，你希望他們成功，你已經仔細考慮了他們的問題以及如何解決它們

沒有一個 checklist 可以確保 User 有這種感覺，儘管我們希望遵循建議能幫助你部分地達到這目標

- 使 User 到愉快意味著在每一步都超出他們的預期，而這始於同理心

---

## Chaos

Terminal 世界的混亂、隨處可見的 inconsistencies，拖慢了我們的速度

- 然而，不可否認的是，這種混亂一直是力量的源泉
- 像 UNIX 衍生的計算環境一樣，Terminal 的限制非常少，各種發明蓬勃發展

這篇文章

- 呼籲你遵循現有的模式
- 給出了與幾十年的命令行傳統相矛盾的建議，這是很諷刺的
- 我們打破規則的罪名與其他人一樣
- 可能有一天，你也需要打破規則。請有意圖地和目標明確地這樣做

Jef Raskin, The Humane Interface:

- `當某個標準顯然對生產力或使用者滿意度有害時，應當放棄該標準。`
  - https://en.wikipedia.org/wiki/The_Humane_Interface

---

## 指南

下面是讓開發者設計更好的 CLI 具體建議

- 第一部分包含需要 follow 的最根本的事項。如果這些做錯了，你的 CLI 會很難用、糟糕
- 其餘的 nice-to-haves。如果有加這些功能，你的程式將比一般程式好得多

如果你不想花太多時間去思考這些細節

- 那就直接 follow 這些規則，你的程式可能會很好
- 如果你經過思考並認為某個規則不適合你的程式，那也沒關係
  - (沒有人會因為不遵守任意規則而拒絕你的程式)

此外

- 這些規則不是一成不變的。如果你有充分的理由不同意某些普遍規則，希望你能提出建議
- https://github.com/cli-guidelines/cli-guidelines

---

## 基本原則

有些基本規則你需要 follow，如果這些做錯了，你的程式要麼非常難用，要麼完全壞掉

**一定要使用 CLI argument parsing library**

- 使用你的語言 build-in 的，或者好的 library
- 它們通常會以合理的方式處理 arguments、標誌 flag、help text，甚至拼寫建議

這裡有些推薦:

- Multi-platform: docopt
- Bash: [argbash](https://argbash.dev/)
- Go: [Cobra](https://github.com/spf13/cobra), [cli](https://github.com/urfave/cli)
- Haskell: [optparse-applicative](https://hackage.haskell.org/package/optparse-applicative)
- Java: [picocli](https://picocli.info/)
- Julia: [ArgParse.jl](https://github.com/carlobaldassi/ArgParse.jl), [Comonicon.jl](https://github.com/comonicon/Comonicon.jl)
- Kotlin: [clikt](https://ajalt.github.io/clikt/)
- Node: [oclif](https://oclif.io/)
- Deno: [flags](https://deno.land/std@0.224.0/flags/mod.ts)
- Perl: [Getopt::Long](https://metacpan.org/pod/Getopt::Long)
- PHP: [console](https://github.com/symfony/console), [CLImate](https://climate.thephpleague.com/)
- Python: [Argparse](https://docs.python.org/3/library/argparse.html), [Click](https://click.palletsprojects.com/en/stable/), [Typer](https://github.com/fastapi/typer)
- Ruby: [TTY](https://ttytoolkit.org/)
- Rust: clap
- Swift: [swift-argument-parser](https://github.com/apple/swift-argument-parser)

**成功時 `return zero exit code`，失敗時 `return non-zero`**

- `Exit codes` 是 script 判斷程式是否成功的方式
- 所以一定要正確回報這點。將 `non-zero` exit codes map 到最重要的失敗模式

**將 output 發送到 `stdout`**

- 你的 CLI 的主要 output 應該發送到 `stdout`
- 任何可機器讀取的內容也應該發送到 `stdout`
- 這是 piping 預設發送內容的地方

**將 message 發送到 `stderr`**

- Log message, errors 等都應該發送到 `stderr`
- 這意味著當命令被管道連接在一起時，這些消息會顯示給使用者，而不是傳輸到下一個命令。

---

## Help

- 當 User 沒有輸入 option 或輸入 `-h`/`--help` flag 時，提供 help text
  - 當執行 `myapp` 或 `myapp subcommand` 並沒有參數時，顯示 help text
- 預設顯示簡潔的 help text
- 如果 command or sub-command 非常簡單且不需要參數(例如 `ls`, `git pull`)，或者它預設是互動的(例如 `npm init`)，就可以忽略這個建議

簡潔的 help text 應該只需要包含:

- 描述你的程式的作用
- 一、兩兩個使用範例
- flags 標誌的描述，除非有很多 flags
- 一條指示，要求傳 `--help` flag 來查更多資訊

`jq` 做得很好

- 當輸入 `jq` 時，它顯示一個簡介說明和一個範例
- 當輸入 `jq --help` 時，會有完整的 flags list:

```
$ jq
jq - commandline JSON processor [version 1.6]

Usage:    jq [options] <jq filter> [file...]
    jq [options] --args <jq filter> [strings...]
    jq [options] --jsonargs <jq filter> [JSON_TEXTS...]

jq is a tool for processing JSON inputs, applying the given filter to
its JSON text inputs and producing the filter's results as JSON on
standard output.

The simplest filter is ., which copies jq's input to its output
unmodified (except for formatting, but note that IEEE754 is used
for number representation internally, with all that that implies).

For more advanced filters see the jq(1) manpage ("man jq")
and/or https://stedolan.github.io/jq

Example:

    $ echo '{"foo": 0}' | jq .
    {
        "foo": 0
    }

For a listing of options, use jq --help.
```

下面這些都應該顯示 help text:

- 傳 `-h` 和 `--help` 時顯示完整的 help

```
$ myapp
$ myapp --help
$ myapp -h
```

當 `-h`/`--help` 跟其他 flag 一起傳進來時，忽略其他 flag

- 你應該能夠在任何命令結尾加 `-h` 並顯示 help
- 不要過度使用 `-h`

如果你的程式類似 `git`，則以下情況也應該提供 help:

```
$ myapp help
$ myapp help subcommand
$ myapp subcommand --help
$ myapp subcommand -h
```

**提供可以讓 User feedback 或提問的地方**

- 常見的是在 top help text 中提供網站或 GitHub 連結

**在 help text 中提供連結到網路版的文件**

- 如果你有 sub-command 的特定頁面或 anchor，就直接連結到那裡
- 如果網路上有更詳細的文件或可能解釋某些行為的資訊，這是特別有用的

**以範例為主**

- 使用者往往是直接使用範例，而不是其他形式的文件
- 所以首先在 help page 中提供這些範例，特別是常見的複雜 case

**如果你有很多範例，將它們放在其他地方**

- 例如備 cheat sheet command 或 web page 中
- 擁有詳盡的 advanced examples 是很有用的
- 但你不想讓你的 help text 變得非常長

對於更複雜的使用情況，例如與其他工具整合時

- 可能適合寫一個完整的教學

**在 help text 的開頭顯示最常見的 flag 和 command**

- 如果你有一些非常常見的 flag，一開始就先顯示它們
- 如 `Git` 首先顯示入門命令和最常用的 sub-command

```
$ git
usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]

These are common Git commands used in various situations:

start a working area (see also: git help tutorial)
   clone      Clone a repository into a new directory
   init       Create an empty Git repository or reinitialize an existing one

work on the current change (see also: git help everyday)
   add        Add file contents to the index
   mv         Move or rename a file, a directory, or a symlink
   reset      Reset current HEAD to the specified state
   rm         Remove files from the working tree and from the index

examine the history and state (see also: git help revisions)
   bisect     Use binary search to find the commit that introduced a bug
   grep       Print lines matching a pattern
   log        Show commit logs
   show       Show various types of objects
   status     Show the working tree status
…
```

**在 help text 中使用些格式**

- 粗體標題使其更容易掃描
- 但要嘗試以與 Terminal 無關的方式進行，這樣你的 User 就不會看到一堆跳脫字元

![alt text](assets/img/Command_Line_Interface_Guidelines_01.png)

(Note: When `heroku apps --help` is piped through a pager, the command emits no escape characters.)

**如果 User 做錯了什麼而你可以猜出他們的意思，提供他們建議**

- 例如 `brew update jq` 告訴你應該執行 `brew upgrade jq`
- 可以問 User 是否想執行建議的命令，但不要強制他們。例如:

```
$ heroku pss
 ›   Warning: pss is not a heroku command.
Did you mean ps? [y/n]:
```

與其建議正確的語法，你可能會忍不住直接為他們執行它，就像 User 一開始就輸入正確一樣

- 有時這是正確的做法，但並不總是如此。
- 首先，無效輸入不一定意味著簡單的拼寫錯誤
- 它通常意味著 User 犯了邏輯錯誤，或錯誤使用了 shell 變數
- 假設他們的意思可能是危險的，尤其是修改狀態
- 其次，請注意，如果你改了 User 輸入的內容，他們不會學到正確的語法

實際上

- 你是在認定他們輸入的方式是有效且正確的，並承諾無限期支持該方式
- 要有意識地做出這一決定，並記錄兩種語法

Further reading: `Do What I Mean`

- http://www.catb.org/~esr/jargon/html/D/DWIM.html

**如果你的命令預期會接收 piped 傳輸的內容，而且 `stdin` 是個 interactive Terminal，立即顯示 help 並退出**

- 這意味著它不會像 `cat` hang 在那邊
- 或者，你可以向 `stderr` 印個 log message

---

## Documentation

`Help text` 的目的是工具的簡短、即時的資訊

- 了解有哪些選項以及如何執行最常見的任務

而 `document` 則是進行詳細說明的地方

- 它是 User 了解你的工具的用途、非用途、工作原理
- 以及如何完成他們可能需要的所有操作的地方

**提供 web-based 的文件**

- User 需要能夠在網路上搜尋工具的文件，並將其連結到特定部分
- 網路是目前最具包容性的文件格式

**提供 Terminal-based 的文件**

- Terminal-based 的文件有幾個不錯的特性:
- 訪問快速、而且不需要網路，並且與特定安裝版本的工具保持同步

考慮提供 [man pages](https://en.wikipedia.org/wiki/Man_page)

- `man page` 是 UNIX 的原始文件系統，今天仍在使用
- 許多 User 會自然而然地檢查 `man mycmd` 作為了解工具的第一步
- 為了容易建立 `man page`，你可以用類似 [ronn](https://rtomayko.github.io/ronn/ronn.1.html) 的工具
  - (它也可以產生 web docs)

然而，並非每個人都知道 `man`，而且它並不適用於所有平台

- 所以你也應確保 Terminal-based 的文件 可以通過工具本身訪問
- 如，`git` 和 `npm` 用 help 可以開他們的 `man`
  - `git branch help` 等同於 `man git-branch`

```
GIT-BRANCH(1)                                       Git Manual                                       GIT-BRANCH(1)

NAME
       git-branch - List, create, or delete branches

SYNOPSIS
       git branch [--color[=<when>] | --no-color] [--show-current]
               [-v [--abbrev=<n> | --no-abbrev]]
               [--column[=<options>] | --no-column] [--sort=<key>]
               [--merged [<commit>]] [--no-merged [<commit>]]
               [--contains [<commit>]] [--no-contains [<commit>]]
               [--points-at <object>] [--format=<format>]
               [(-r | --remotes) | (-a | --all)]
               [--list] [<pattern>...]
       git branch [--track[=(direct|inherit)] | --no-track] [-f]
               [--recurse-submodules] <branchname> [<start-point>]
       git branch (--set-upstream-to=<upstream> | -u <upstream>) [<branchname>]
       git branch --unset-upstream [<branchname>]
       git branch (-m | -M) [<oldbranch>] <newbranch>
       git branch (-c | -C) [<oldbranch>] <newbranch>
       git branch (-d | -D) [-r] <branchname>...
       git branch --edit-description [<branchname>]

DESCRIPTION
       If --list is given, or if there are no non-option arguments, existing branches are listed; the current
       branch will be highlighted in green and marked with an asterisk. Any branches checked out in linked
       worktrees will be highlighted in cyan and marked with a plus sign. Option -r causes the remote-tracking
       branches to be listed, and option -a shows both local and remote branches.

       ...
```

---

## 輸出格式

**人類可讀輸出格式至關重要**

- 人類第一，機器第二
- 判斷某個特定輸出流(`stdout` 或 `stderr`)是否由人來讀的最簡單準則是判斷是它是否為 TTY
- 無論用什麼語言，它都會有個程式或 library 來執行操作(例如 Python, Node, Go)

Further reading on `what a TTY is`.

- https://unix.stackexchange.com/questions/4126/what-is-the-exact-difference-between-a-Terminal-a-shell-a-tty-and-a-con/4132#4132

在不影響可用性的情況下提供 machine-readable 的輸出

- Streams of text 是 UNIX 的通用介面
- 程式通常輸出 `lines of text`，程式通常也期望 `lines of text` 作為輸入
- 因此你可以將多個程式組合在一起，這通常是為了寫 script，但也可以幫助使用程式的人
- **例如，User 應該能夠將 output 通過 pipe 送到 `grep`**

如果 human-readable 的 output 破壞了 machine-readable 的 output

- 用 `--plain` 以 pure text 格式輸出，便於與整合 `grep` 或 `awk` 等工具
- 在某些情況下，可能需要以不同的方式輸出資訊以使其對人可讀

例如，如果你顯示的是 line-based table，你可能會將一個 cell 分成多行，保持在螢幕寬度內容納更多資訊

- 這打破了每行只有一條資料的預期行為
- 因此你應該為 script 提供 `--plain`，禁止所有這種操作並每行輸出一條記錄

**`--json` 來提供 JSON 輸出**

- JSON 提供更多結構，因此更容易處理複雜資料結構
- `jq` 是個常用於 CLI 處理 JSON 的工具，有完整的工具生態系統來輸出和操作 JSON
  - https://ilya-sher.org/2018/04/10/list-of-json-tools-for-command-line/
- JSON 也在網路上廣泛使用，你可以用 `curl` 直接與 web service 進行 pipe 傳輸

**在 task 成功時，顯示成功的資訊，但要保持簡短**

- 傳統上，當沒有問題時，UNIX command 不向使用者顯示任何 output
- 這在 script 時是有意義的，但當人類使用時可能會讓 command 看起來像卡住了
  - 例如，`cp`，即使它需要很長時間，它也不會印任何資訊
- 「完全不印任何資訊」作為預設行為是很少見的

在你確實想要 no output 的情況下(例如，在 shell script 使用時)

- 為了避免將 `stderr` 導到 `/dev/null`，可以提供 `-q` 選項來關閉所有非必要 output

**如果你改變了狀態，那就要告訴 User**

- 當 command 改變系統的狀態時，解釋剛發生的事情特別有價值
- 這樣 User 可以在腦海中模擬系統的狀態
  - 特別是如果結果不直接對應於 User 的 request 時

例如，`git push` 會精確告訴你它在做什麼，以及 remote branch 的新狀態是什麼:

```
$ git push
Enumerating objects: 18, done.
Counting objects: 100% (18/18), done.
Delta compression using up to 8 threads
Compressing objects: 100% (10/10), done.
Writing objects: 100% (10/10), 2.09 KiB | 2.09 MiB/s, done.
Total 10 (delta 8), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (8/8), completed with 8 local objects.
To github.com:replicate/replicate.git
 + 6c22c90...a2a5217 bfirsh/fix-delete -> bfirsh/fix-delete
```

**要很容易可以查看系統現在的狀態**

- 如果程式進行了很多複雜的狀態變更，並且這些變更在文件系統中不能立即可見，要確保這些變更容易查的到

如，`git status` 會盡可能告訴你當前的狀態，並提供一些修改狀態的提示:

```
$ git status
On branch bfirsh/fix-delete
Your branch is up to date with 'origin/bfirsh/fix-delete'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   cli/pkg/cli/rm.go

no changes added to commit (use "git add" and/or "git commit -a")
```

**建議 User 應該執行的命令**

- 當多個命令構成一個工作流程時，向 User 建議可以執行的下個命令
- 這有助於 User 學習如何使用程式並發現新的功能
- 如上面的 `git status` output 中，它建議了可以執行的命令來修改你正在查看的狀態

**超出程式內部世界的邊界的操作通常應該是明確的**，這包括:

- Read/Write User 未明確傳為參數的檔案(除非這些檔案存的是內部程式狀態，如 cache)
- 與 server 互動，如下載檔案

**`ASCII art` 可以增加資訊密度**

- 例如，`ls` 以可掃描的方式顯示 permissions
- 當你第一次看到它時，你可以忽略大多數資訊。隨著你學習它的工作原理，會隨著時間的推移識別出更多模式

**有意義地使用顏色**

- 你可能想要突出資訊，使 User 注意到它，或用紅色來指示錯誤
- 不要過度使用它
  - 如果每個東西都是不同的顏色，顏色就失去了意義，只會使閱讀變得更加困難

**如果程式不在 Terminal 中執行或使用者要求禁用，就禁用顏色**，以下情況應該禁用顏色:

- `stdout` 或 `stderr` 不是 interactive Terminal (a TTY)
  - 最好單獨檢查，如果你正在將 `stdout` pipe 到另個程式，仍然希望在 `stderr` 上有顏色
- 設了 `NO_COLOR` env
- `TERM` env 的值為 `dumb`
- User 傳 `--no-color` 參數
- 你可能還希望加個 `MYAPP_NO_COLOR` env，以防 User 希望專門為你的程式禁用顏色

進一步閱讀:

- `no-color.org`: https://no-color.org/
- `12 Factor CLI Apps`: https://medium.com/@jdxcode/12-factor-cli-apps-dd3c227a0e46

**如果 `stdout` 不是一個 interactive Terminal 的話**

- 不要顯示任何動畫。(log 裡面就會大亂了)

**有時候，一些符號能幫助 User 更清楚區分資訊**

- 圖片如果需要，能使多個事物區分開來、引起使用者注意或僅僅增加一點特色，則比文字更好
- 但是要小心，很容易過度使用，讓程式看起來雜亂無章

如，`yubikey-agent` 用表情符號為輸出增加結構

- 用 ❌ 吸引你的注意力到重要的資訊:

```
$ yubikey-agent -setup
🔐 The PIN is up to 8 numbers, letters, or symbols. Not just numbers!
❌ The key will be lost if the PIN and PUK are locked after 3 incorrect tries.

Choose a new PIN/PUK:
Repeat the PIN/PUK:

🧪 Retriculating splines …

✅ Done! This YubiKey is secured and ready to go.
🤏 When the YubiKey blinks, touch it to authorize the login.

🔑 Here's your new shiny SSH public key:
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCEJ/
UwlHnUFXgENO3ifPZd8zoSKMxESxxot4tMgvfXjmRp5G3BGrAnonncE7Aj11pn3SSYgEcrrn2sMyLGpVS0=

💭 Remember: everything breaks, have a backup plan for when this YubiKey does.
```

**預設情況下，不要 output 只有 creator 才能理解的資訊**

- 如果某個 output 僅用於幫助 creator 理解軟體在做什麼，則幾乎不應該預設顯示給普通 User
- 只在詳細模式下顯示

邀請外部人士提供 usability feedback

- 會幫助你看到你因離 code 太近而忽視的重要問題

**不要將 `stderr` 視為 log file**

- 至少在 default 情況下不要這樣做
- 除非在詳細模式下，否則不要打 print log 的 level tag (ERR，WARN ...)或額外的 context

**如果輸出大量 text，使用 pager (分頁功能) (例如 less)**

- 如 `git diff` 預設執行此操作
- 使用 pager 可能容易出錯，因此在實作時要小心，避免 UX 變差
- 如果 `stdin` 或 `stdout` 不是 interactive Terminal，則不該用 pager

用 `less` 的良好合理的選項是 `less -FIRX`

- 這當內容佔滿一個螢幕時不會分頁
- 搜尋時忽略大小寫
- 啟用顏色和格式化，並在 `less` 退出時保留內容在螢幕上

在你的語言中可能有比通過 pipe 傳到 less 更優秀的 library

- 例如，Python 中的 `pypager`
  - https://github.com/prompt-toolkit/pypager

---

## Errors

User 最常來問問題 or 查文件的主因就是要 fix errros

- 如果能將 Errors 變成 documentation，這節省 User 使用者大量時間

**Catch errors 並且重新組織成方便人理解的資訊**

- 如果是 expeexted error 發生，catch 它，然後重新編寫錯誤訊息
- 讓訊息變成有用
- 將其視為一場對話
  - User 做錯事，程式在引導 User 朝正確方向前進
- 如: `Can’t write to file.txt. You might need to make it writable by running ‘chmod +w file.txt’.`

**信號:噪音，兩者間的比例相當重要**

- 產生越多無關的 output，User 需要更多時間才能弄清楚他們做錯了什麼
- 如果程式產生多個相同類型的 errors，考慮將它們組成單個解釋標題，而不是印出許多相似的訊息

**考慮 User 首先會看的地方**

- 將最重要的資訊放在輸出的最尾端
- 眼睛會被紅色字吸引，有意識且謹慎地使用它

**如果有意外或無法解釋的錯誤，提供 debug 和 traceback 資訊，以及如何 submit bug report**

- 但不要忘記「信號:噪音」比例
- 你不希望 User 被不理解的資訊淹沒
- 考慮將 debug log 寫入文件，而不是印在畫面上

**讓 User 可以輕鬆的 submit bug report**

- 一個不錯的方法是提供 URL，並且預先填盡可能多的資訊

Further reading:

- Google: Writing Helpful Error Messages
  - https://developers.google.com/tech-writing/error-messages
- Nielsen Norman Group: Error-Message Guidelines
  - https://www.nngroup.com/articles/error-message-guidelines/

---

## Arguments (參數) 跟 flags (標誌)

關於術語的說明:

- `Arguments`, or `args` 是命令的定位參數
  - 如，提供給 `cp` 的路徑是 args
  - args 的順序通常很重要: `cp foo bar` 與 `cp bar foo` 意思不同
- `Flags` 是命名參數
  - 用連字符和單字母名稱(`-r`)或雙連字符和多字母名稱(`--recursive`)表示
  - 它們可能包括或不包括 User 指定的值
    - (`--file foo.txt` 或 `--file=foo.txt`)
  - 順序，一般來說，不影響語義

**偏好用 `flags` 而不是 `args`**

- 雖然這需要多輸入些內容
- 但它使正在發生的事情更加清晰
- 它還使將來更改接受輸入的方式變得更容易
- 有時用 `args` 時，無法加新的輸入而不破壞現有行為或造成歧義

引用: 12 Factor CLI Apps.

- https://medium.com/@jdxcode/12-factor-cli-apps-dd3c227a0e46

**所有 `flags` 都應該有全長度的版本**

- 如，應同時具有 `-h` 和 `--help`
- 在 script 中，全長版本是有用的，因為 User 希望它們詳細且具描述性，不必到處查含義

引用: GNU Coding Standards

- https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html

**常用的 flag，只採用單個字母的 flag**

- 特別是在 top-level 時，使用 subcommand
- 這樣，你不會污染 short flag 的名稱空間
  - 不然會迫使在未來加 flag 時，使用複雜的字母和大小寫

**對於多個檔案的簡單操作，多個 arguments 是可以接受的**

- 如，`rm file1.txt file2.txt file3.txt`
- 這使得它可以與 globbing 一起用: `rm *.txt`

**如果有兩個或更多 arguments 用於不同的事情，你可能設計出錯了**

- 例外情況是常見的主要操作，這種簡潔是值得記住的
- 例如，`cp <source> <destination>`

引用: 12 Factor CLI Apps.

- https://medium.com/@jdxcode/12-factor-cli-apps-dd3c227a0e46

**如果有標準名稱，就用標準名稱來命名 flag**

- 如果另個常用命令使用了 flag name，最好遵循現有模式
- 這樣， User 不必記住兩個不同的選項(及其適用的命令)
  - User 甚至可以猜測選項而不必查

這裡有些常用選項的清單:

- `-a, --all`: 顯示全部，例如 `ps`, `fetchmail`
- `-d, --debug`: 顯示 debug output
- `-f, --force`: 如，`rm -f` 強制刪除，即使它認為自己沒有權限這樣做
  - 這對於那些通常需要 User 確認的破壞性操作很有用
- `--json`: 顯示 JSON output，參考 `output` 章節的介紹
- `-h, --help`: 這應該只意味著 help。參考 `help` 章節的介紹
- `--no-input`: 參考 `interactivity` 章節的介紹
- `-o, --output`: 輸出檔案。如 `sort`, `gcc`
- `-p, --port`: Port，如 `psql`, `ssh`
- `-q, --quiet`: 顯示較少的 output
  - 這在給人類的 output 中特別有用
  - 你可能希望在 script 中執行時隱藏它們
- `-u`, `--user`: User。如 `ps`, `ssh`
- `--version`: 版本
- `-v`: 可能是 `verbose` 或 `version`
  - 可以用 `-d` 表示 `verbose` (detail)，`-v` 給 `version`

**把 default 的 config 設計成大多數 User 的最佳選擇**

- 把東西設計成 configurable 是好的，但大多 User 不會找到正確的 flag 並記住用它
- 如果這不是 default ，會使大多 User 的 UX 變差
- 如，`ls` 有簡潔的預設 output 和其他歷史原因，但如果今天設計，可能會預設為 `ls -lhF`

**提示 User 輸入**

- 如果 User 沒有傳 argument or flag，提示 User

**Never require a prompt.**

- 始終提供一種通過 flags or arguments 傳 input 輸入的方式
- 如果 `stdin` 不是 interactive Terminal，跳過提示並要求這些 flags/args

**在執行任何危險操作之前，要跟 User 確認**

- 常見的慣例是提示 User 輸入 `y` 或 `yes`
- 否則要求傳 `-f` 或 `--force`

危險是主觀的，存在不同程度的危險:

- 輕微:
  - 如刪除檔案，這種小型更改。你可能想提示確認，可能不需要
  - 例如，如果 User 明確執行刪除的命令，你可能不需要詢問
- 中等:
  - 例如刪除 folder 這種大型更改，刪除某種資源的 remote 遠程更改，或者不能輕易取消的批次修改
  - 這種情況下，你通常希望提示確認
  - 考慮給 User 提供 `dry run`，讓他們可以看到會發生什麼
- 嚴重:
  - 刪除複雜的東西，如整個 remote app or server
  - 在這情況下，你不僅希望提示確認，還希望 User 不會意外的確認送出
  - 考慮要求 User 輸入些非平凡的內容，例如他們要刪除東西的名稱
    - 可以讓他們傳個 flag ，如 `--confirm="name-of-thing"`，這樣它仍然可以被 script 化

**如果 input 或 outpur 是檔案，支援 `-` 指令，可以從 `stdin` 讀或寫入 `stdout`**

- 這讓另個命令的輸出成為你命令的輸入，反之亦然，無需使用臨時文件
- 例如，`tar` 可以從 `stdin` 解壓

```
$ curl https://example.com/something.tar.gz | tar xvf -
```

**如果 flag 可以接受 optional value，允許一個特殊詞如 `none`**

- 例如，`ssh -F` 接受 alternative ssh_config 檔案的可選檔案名
- 而 `ssh -F none` 在沒有 config 檔案的情況下執行 SSH
- 不要只使用空值，這會使 arguments 是否為 flag 或 arguments 變得模糊

**如果可能，讓 arguments, flags 和 subcommands 沒有順序分別**

- 許多 CLI，特別是有 subcommands 的，對可以放置各種參數的位置有規則
- 例如，一個命令可能有個 `--foo` flag，只在將它放在 subcommands 之前時才有效
- 這對 User 來說非常困惑
  - 特別是考慮到 User 嘗試讓 command 正常工作時，最常做的事就是按向上箭頭，呼叫上次的指令，然後在末尾加另個選項並再次執行
- 如果可能，嘗試使兩種形式等效

```
mycmd --foo=1 subcmd
works

$ mycmd subcmd --foo=1
unknown flag: --foo
```

**不要直接從 flags 讀 secrets**

- 當 command 接受 secret，例如 `--password`，值會將 secret 洩露到 ps output 中，並可能洩露到 shell history 中
- 而且，這種 flag 鼓勵使用不安全的環境變數來存 secret
- 考慮只接受檔案的方式來接敏感資料
- 用 `--password-file` or `stdin`
- `--password-file` 允許在多種情況下以不顯眼的方式傳機密

(Bash 中可以用 `--password $(< password.txt)` 將檔案內容傳給 flag。但這方法有同樣的安全問題，會將內容洩露到 `ps` output 中。最好避免這樣做。)

---

## 互動性 (Interactivity)

**只有當 `stdin` 是 interactive Terminal (TTY) 時，才使用提示或互動元素**

- 這是相當可靠的方法，來判斷你是將資料通過 pipe 傳輸到 command，還是它正在 script 中執行
- 在這種情況下，prompts 將不起作用，應該拋出錯誤並告訴 User 應該傳什麼 flag

**如果傳了 `--no-input`，不要提示或執行任何互動操作**

- 這為 User 提供了關閉所有 prompts 的方法
- 如果需要輸入命令，就 fail 並告訴 User 如何以 flag 傳資訊

**如果提示輸入密碼，不要在 User 輸入時顯示密碼**

- 去 turning off Terminal 的 echo 就能做到了
- 你用程式語言應該有相應的輔助工具

**讓 User 能夠退出**

- 明確說明如何退出。(不要像 vim 那樣)
- 如果程式在網路 I/O 等操作上 hangs on，確保 `Ctrl-C` 仍然可用
- 如果它是程式執行的包裝程式(如 `SSH`, `tmux`, `telnet` 等)且 `Ctrl-C` 不能退出，請明確說明如何退出
  - 如，`SSH` 允許用 `~` escape character

---

## Sub-commands

如果工具足夠複雜

- 可以通過 subcommands 來降低其複雜性
- 如果你有幾個非常密切相關的工具，組合它們成一個命令可以，使它們更易於使用和發現

對 subcommands 共享一些東西很有用

- global flags
- help text
- configuration
- storage mechanisms

**Be consistent across subcommands.** Use the same flag names for the same things, have similar output formatting, etc.

**Use consistent names for multiple levels of subcommand.** If a complex piece of software has lots of objects and operations that can be performed on those objects, it is a common pattern to use two levels of subcommand for this, where one is a noun and one is a verb. For example, `docker container create`. Be consistent with the verbs you use across different types of objects.

**在 subcommands 之間保持一致性**

- 對相同的事物使用相同的 flag name，具有類似的輸出格式等

**對 multiple levels of subcommand 使用一致的名稱**

- 如果複雜的軟體有很多 object 和可以對這些 object 執行的操作，通常是對此用兩級 subcommand
- 其中一個是名詞，一個是動詞
- 如，`docker container create
- 對於不同類型的 object，使用一致的動詞

無論是 `名詞 動詞` 還是 `動詞 名詞` 順序都可以

- 但 `名詞 動詞` 似乎更常見

Further reading: `User experience, CLIs, and breaking the world, by John Starich.`

- https://uxdesign.cc/user-experience-clis-and-breaking-the-world-baed8709244f

**不要模棱兩可的命令**

- 如，有名為 `update` 和 `upgrade` 的 subcommand 是相當令人困惑
  - 你可能需要使用不同的詞，或通過額外的詞來消除歧義

---

## Robustness

**驗證 User 的 input**

- 程式接受 User 的 input，使終都會收到錯誤的資料
- 及早檢查並在發生任何錯誤之前退出，並使錯誤易於理解

**Responsive 比速度更重要**

- 在 `<100ms` 內向 User 提供一些資訊
- 如果正在發網路請求，在執行之前印一些內容，這樣它不會 hang 著像是壞掉一樣

**如果操作需要很長時間，顯示進度條**

- 如果程式在一段時間內不顯示 output，會讓人覺得卡住了
- spinner or progress indicator 可以讓程式看起來比實際更快

如果 progress bar 卡著

- User 不會知道是否還有正常執行
- 顯示預計剩餘時間，或者有個動畫，來安撫 User 你還在工作。

有許多好的 library，如

- [tqdm](https://github.com/tqdm/tqdm) for Python
- [schollz/progressbar](https://github.com/schollz/progressbar) for Go
- [node-progress](https://github.com/visionmedia/node-progress) for Node.js.

**盡可能 parallel 執行，但要慎重**

- 在 shell 中報告進度已經很難了；parallel processes 難十倍
- 確保它是穩健的，並且 output 不會混亂交錯
- 如果可以，就用 library
  - [tqdm](https://github.com/tqdm/tqdm) for Python
  - [schollz/progressbar](https://github.com/schollz/progressbar) for Go
  - support multiple progress bars natively

好處是，它可以大大提高 usability

- 如，`docker pull` 的多個 progress bars 讓人對所有事情有清楚的了解

```
$ docker image pull ruby
Using default tag: latest
latest: Pulling from library/ruby
6c33745f49b4: Pull complete
ef072fc32a84: Extracting [================================================>  ]  7.569MB/7.812MB
c0afb8e68e0b: Download complete
d599c07d28e6: Download complete
f2ecc74db11a: Downloading [=======================>                           ]  89.11MB/192.3MB
3568445c8bf2: Download complete
b0efebc74f25: Downloading [===========================================>       ]  19.88MB/22.88MB
9cb1ba6838a0: Download complete
```

當事情進展順利時，將 log 藏在 progress bars 後面

- 使 User 更容易理解正在發生的事情
- 但如果有錯誤，確保印出 log。否則，將非常難以 debug

**設置 time out**

- 允許 config network timeouts，並設合理的預設值，以免 hang forever

**Make it recoverable**

- 如果程式因某些臨時原因失敗(如，網路中斷)，你應該能夠按 `<up>` 和 `<enter>` 鍵，它應該從中斷處繼續

**使其 crash-only**

- 這是冪等性(idempotence)的下一步
- 如果你能夠避免在操作後進行任何清理，或者可以將清理推遲到下一次執行
- 那麼程式可以在失敗或中斷時立即退出。這使它既更穩健又更具響應性

Citation: `Crash-only software: More than meets the eye.`

- https://lwn.net/Articles/191059/

**User 會誤用你的程式**

- 為此做好準備。他們會將它包裝在 sctipt 中
- 在網路不良的情況下使用，並同時執行多個 instance
- 具有你未預料到的怪癖

---

## 未來考量

在任何程式中，確保 interfaces 不會輕易修改 deprecation 是非常重要的

- 進行 deprecation 之前，一定要有在文件中說明「什麼功能會 deprecation」很長一段時間
- Subcommands, arguments, flags, configuration files, environment variables，這些都是 interfaces
- 你承諾要讓它們繼續運作
- `Semantic versioning` 只能承擔有限的變更
  - 如果你每月都推出一個 major，這是沒有意義的

**儘可能保持 additive 方式的更改**

- 與其以向後不相容的方式修改 flag 的行為
- 不如加個新 flag，只要它不會使 interface 過於膨脹

**要在進行 non-additive change 之前，發出警告**

- 最終你無法避免破壞 interface
- 在這樣做之前，在程式本身中預先警告 User
- 當 User 傳要棄用的 flag 時，告訴他們它即將更改
- 確保 User 今天有辦法修改，使用以使其具有未來保證，並告訴 User 如何操作

**對於人類的 output 進行修改通常是可以的**

- 使 interface 變好用的唯一方法是對其進行迭代
- 如果 output 被認為是個 interface，那麼就無法對其進行迭代
- 鼓勵你的 User 在 script 中用 `--plain` 或 `--json` 來保持 output 穩定性

**不要有包羅萬象的 subcommand**

- 如果有個可能是使用最多的 subcommand，你可能會想讓 User 完全省略它以節省時間
- 假設你有一個 `run` 命令來包裝任何的 `shell` 命令:

```sh
$ mycmd run echo "hello world"
```

You could make it so that if the first argument to `mycmd` isn’t the name of an existing subcommand, you assume the user means `run`

所以他們可以這樣輸入:

```sh
$ mycmd echo "hello world"
```

但有個嚴重的缺點:

- 現在就很難加名為 `echo` 或任何東西的 subcommand，會有破壞現有用法的風險
- 如果外面有個 script 用 `mycmd echo`，那該 User 升級到新版本後，它將完全不同

**不要允許任意 subcommands 的縮寫**

- 假設你有個 install subcommands
  - 你想節省 User 輸入，所以允許輸入任何不含糊的 prefix
  - 如 `mycmd ins`，甚至是 `mycmd i`，並作為 `mycmd install` 的別名
- 現在你被困住了: 你不能再加任何以 i 開頭的命令，因為有些 script 假設 `i` 代表 `install`
- aliases—saving 本身沒有錯，節省輸入是好的，但應該是明確的並保持穩定

**不要建立 time bomb**

- 20 年後。command 是否仍然以今天的方式執行
- 或者它會因為網路上的某些外部依賴發生變化或不再維護而停止工作？
- 最有可能在 20 年內不再存在的 server 是你現在正在維護的那個
  - (But don’t build in a blocking call to Google Analytics either.)

---

## Signals 和 control characters

**如果 User 按 `Ctrl-C (INT signal)`，盡快退出**

- 立即說些什麼，在開始清理之前。為任何 clean-up code 設定 timeout，以免它 hang forever

**如果 User 在可能需要很長時間的清理操作期間按 `Ctrl-C`，那就跳過它們**

- 告訴 User 當再次按下 `Ctrl-C` 時會發生什麼情況，以防這是個破壞性操作

例如，當退出 `Docker Compose` 時，可以第二次按下 `Ctrl-C` 強制 container 立即停止，而不是優雅關閉

```sh
$  docker-compose up
…
^CGracefully stopping... (press Ctrl+C again to force)
```

你的程式應該預期在 clean-up 未執行的情況下啟動

- (See [Crash-only software: More than meets the eye.](https://lwn.net/Articles/191059/))

---

## Configuration

- CLI 有許多不同類型的 config，以及許多不同的提供方式 (flags, environment variables, project-level config files)
- 設計「提供每個 config 的最佳方式」，取決於幾個因素，主要是特定性、穩定性和複雜性

Config 通常分為幾類:

1. 從「這次執行」，到「下次執行」會有所變化

   - 舉例:
     - 設定 deubug output level
     - enable safe mode or dry-run
   - 建議: 用 `flag`。`Environment variables` 也可能有用

2. 從「這次執行」到「下次執行」通常不變、穩定的使用，但並非總是如此。可能在 project 間有所不同。而且，不同的 User 使用的方法絕對不一樣，特別是在同 project 上工作時

   - 這種類型的 config 通常針對個別 computer

     - 舉例:
     - 提供程式啟動所需 project 的非預設路徑
     - 指定 output 中是否應出現顏色
     - 指定 HTTP proxy

   - 建議: 使用 `flag` 和可能的 `environment variables`
   - User 可能希望在 shell config 中 global 設這些變數，或在 `.env` 中為特定 project 設定
   - 如果這 config 足夠複雜，可能需要專門的 config 文件，但環境變數通常已經足夠

3. Stable within a project, for all users.

   - This is the type of configuration that belongs in version control. Files like `Makefile`, `package.json` and `docker-compose.yml` are all examples of this.
   - Recommendation: **Use a command-specific, version-controlled file.**

4. 在 project 內對所有 User 都穩定 config
   - 這種類型的 config 屬於版本控制。如 `Makefile`, `package.json`, `docker-compose.yml` 都是這類
   - 建議: 用特定於命令的版本控制文件

**遵循 XDG 規範**

- [freedesktop.org(X Desktop Group)](https://www.freedesktop.org/wiki/)開發了關於 config 可能位於的基本目錄位置的 spec
- 其目標之一是通過支持通用的 `~/.config` 資料夾來限制 User 主目錄中點檔案變多
- XDG 基本目錄規範得到了 yarn, fish, wireshark, emacs, neovim, tmux 和許多其他 project 的支持
- [full spec](https://specifications.freedesktop.org/basedir-spec/latest/)
- [summary](https://wiki.archlinux.org/title/XDG_Base_Directory#Specification)

**如果你自動修改不是你的程式的 config，要讓 User 同意並告訴他們你在做什麼**

- 優先建立新的 config (如 `/etc/cron.d/myapp`)
- 而不是附加到現有的 config (如 `/etc/crontab`)
- 如果你必須附加或修改系統範圍的 config，要在該檔案中使用帶日期的註解來劃分你的新增

**按優先順序來採用 config 參數**，以下是優先順序，從高到低:

1. Flags
2. The running shell’s environment variables
3. Project-level configuration (e.g. `.env`)
4. User-level configuration
5. System wide configuration

---

## 環境變數 (Environment variables)

**環境變數會隨著 commnad 的 context 改變而改動**

- 環境變數的「環境」是 Terminal session、command 執行的 context
- 因此，環境變數可能會在每次 command 執行時、更換 Terminal session 或不同機器之間變化

使用前

- 看看 configuration 章節，了解常見 config 類型和何時最合適用環境變數的建議

**為了最大限度提高 portability，環境變數名稱只能包含**

- 大寫字母
- 數字和下底線
- 且不能以數字開頭
- 這意味著 `O_O` 和 `OWO` 是唯一也是有效的表情符號環境變數 ...

**目標是單行環境變數的值**

- 雖然 multi-line values 是可能的，但它們會在使用 `env` command 時，有 usability 問題

**避免使用廣泛使用的名稱**

- 對照 POSIX 標準環境變數列表
- https://pubs.opengroup.org/onlinepubs/009695399/basedefs/xbd_chap08.html

**盡可能檢查 general-purpose 的環境變數的值:**

- `NO_COLOR`: 禁用顏色 or `FORCE_COLOR` 啟用顏色並忽略邏輯檢測
- `DEBUG`: 啟用更詳細的 output
- `EDITOR`: 如果你需要提示 User 編輯文件或輸入多於一行的內容
- `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY` 和 `NO_PROXY`: 如果要進行網路操作(你正在使用的 HTTP library 可能已經檢查這些變數)
- `SHELL`: 打開 User 首選 Shell(如果需要執行 Shell script，用特定的解釋器如 `/bin/sh`)
- `TERM`, `TERMINFO` 和 `TERMCAP:` 如果要使用 Terminal-specific 的逃逸序列
- `TMPDIR`: 如果要建立 temp file
- `HOME`: 定位 config file
- `PAGER` 如果想自動分頁輸出
- `LINES` 和 `COLUMNS` 用於依賴螢幕大小的 ouptu (例如表格)

**在適當的情況下從 `.env` 讀取環境變數**

- 如果 command 定義了 env，而這些變數只要 User 在特定目錄中工作就不會更改，那麼它應該也從 local `.env` 讀取
  - 這樣 User 可以為不同 project 配不同的 env，不需要每次都指定
- 許多語言都有讀取 .env 文件的 library

**不要將 `.env` 作為 config 文件的替代品，`.env` 檔案有很多限制:**

- `.env` 通常不存在版本控制中
- 它只有一種資料類型: `string`
- 它容易組織很容易很差
- 它容易引入 encode 問題
- 它通常包含敏感 credentials, key，這些應該更安全的存

如果這些限制似乎會妨礙 usability 或 security

- 那專門的 config 檔案會更合適。

**不要從 env 讀 secrets，雖然 env 方便於存，但已被證明過於容易泄漏:**

- **Exported env 會發送到每個 process，而容易泄漏到 log 或被竊取**
- 像 `curl -H "Authorization: Bearer $BEARER_TOKEN"` 這樣的 Shell 會 leak 到 globally-readable process 中
  - (`cURL` 提供 `-H @filename` 替代方案，從檔案讀感頭資訊)
- Docker 容器 env 可以被任何具有 Docker daemon 訪問權限的人查看，`docker inspect`
- `systemd` 單元中的 env 通過 `systemctl show` 可以被 globally readable

`Secrets` 應僅通過 credential 檔案、pipes、AF_UNIX sockets、secret management services 或其他 IPC mechanism

---

## 命名

“Note the obsessive use of abbreviations and avoidance of capital letters; `[Unix]` is a system invented by people to whom repetitive stress disorder is what black lung is to miners. Long names get worn down to three-letter nubbins, like stones smoothed by a river.” — Neal Stephenson, [In the Beginning was the Command Line](https://web.stanford.edu/class/cs81n/command.txt)

The name of your program is particularly important on the CLI: your users will be typing it all the time, and it needs to be easy to remember and type.

不要過度用縮寫和避免使用大寫字母

- Naming 在 CLI 上特別重要: User 將一直輸入它，它需要易於記憶和輸入

**簡單、令人難忘的詞**

- 但不要太通用，否則會踩到其他命令，User 感到困惑
- 如，`ImageMagick` 和 `Windows` 都用了 `convert` 命令

**只用小寫字母，如果真的需要，可以使用破折號**

- `curl` 是個好名字，`DownloadURL` 則不是

**簡短**

- User 將一直輸入它
- 也不能太短，最短的命令最好保留給經常使用的通用工具，例如 `cd`, `ls`, `ps`

**讓它容易輸入**

- 如果期望 User 整天輸入你的命令，讓它對 hands 友好

Further reading: `The Poetics of CLI Command Names`

- https://smallstep.com/blog/the-poetics-of-cli-command-names/

---

## 發行

**如果可能，發行 single binary**

- 如果你的語言不能標準編譯二進制可執行文件，看它是否有類似 [PyInstaller](https://pyinstaller.org/en/stable/) 的工具
- 如果真的不能發行 single binary，用平台的 native package installer
  - 以避免在硬碟上殘留散落，無法輕易刪除的東西，這會讓 User 惱怒

如果你製作特定語言的工具，如 code 檢查工具

- 那麼此規則不適用
- 可以假設 User 的計算機上安裝了該語言的 interpreter

**要讓工具也能很容易解除安裝**

- 如果需要指導，將其放在安裝指導的底部
- User 想要解除安裝的最常見時間之一就是剛安裝後

---

## 分析

metrics 可以了解 User 如何使用程式，如何改進它以及在哪裡集中精力

- 但是，與網站不同，CLI 的 User 希望掌控他們的環境
- 當程式在背後進行某些操作而未告知時，User 會感到驚訝

**在未經同意的情況下，不要收集使用或 crash 資料**

- User 會發現並會感到憤怒
- 非常明確地說明你收集了什麼，為什麼收集它，它有多匿名以及你如何匿名化它、保留多長時間

理想情況下

- 問 User 是否願意提供資料(`opt-in`)
- 如果你選擇預設情況下收集資料，那麼在網站上或首次執行時明確告訴 User
  - 而已要容易關掉它

Examples projects

- Angular.js: [collects detailed analytics using Google Analytics](https://v17.angular.io/cli/analytics)
  - 你必須明確選擇加入。如果你想跟踪內部 Angular 使用情況，可以將跟 track ID 改為你自己的 GA
- `Homebrew`: 將 metrics 送給 GA
  - [有詳細的 FAQ](https://docs.brew.sh/Analytics)
- Next.js [匿名收集使用統計資料，預設啟用](https://nextjs.org/telemetry)

**考慮收集分析的替代方案**

- 從 web docs 下手，如果想了解 User 如何使用你的 CLI，製作最好理解的 user case doc，看看它們隨時間的表現。查看 User 在 doc 中搜尋了什麼
- 從 download 下手，有粗略的 metrics 來了解使用情況和 User 執行的 OS
- 與 User 交流。接觸並詢問如何使用你的工具

Further reading: [Open Source Metrics](https://opensource.guide/metrics/)

---

## Further reading

- [The Unix Programming Environment](https://en.wikipedia.org/wiki/The_Unix_Programming_Environment), Brian W. Kernighan and Rob Pike
- [POSIX Utility Conventions](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html)
- [Program Behavior for All Programs](https://www.gnu.org/prep/standards/html_node/Program-Behavior.html), GNU Coding Standards
- [12 Factor CLI Apps](https://medium.com/@jdxcode/12-factor-cli-apps-dd3c227a0e46), Jeff Dickey
- [CLI Style Guide](https://devcenter.heroku.com/articles/cli-style-guide), Heroku
