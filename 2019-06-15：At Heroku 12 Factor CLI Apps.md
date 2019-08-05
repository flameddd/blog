## At Heroku 12 Factor CLI Apps
### Jeff Dickey Oct 9, 2018  
#### https://medium.com/@jdxcode/12-factor-cli-apps-dd3c227a0e46

### 1. Great help is essential help 是最關鍵的
- good help documentation 是最重要的
- CLI 要提供 `in-CLI help` 跟 `help on the web` (READMEs 是個好選擇)
  - 讓 user 不用離開 cli 就能查
  - 也讓 user 能夠 google 來查詢
  - 確保 doc 能被 google 查到
  
提到了一個工具 oclif
- https://github.com/oclif/oclif
  - Node.JS Open CLI Framework
  - 用來寫 CLI 的 framework

這些指令 must displays the help
- 沒法控制 user 的輸入行為，所以都支援就對了

```
# list all commands
$ mycli
$ mycli --help
$ mycli help
$ mycli -h

# get help for subcommand
$ mycli subcommand --help
$ mycli subcommand -h
```

`-h,--help` 應該要是個（唯一一個）reserved flag。  
幫助查詢 `subcommand` 的 help  
- $ mycli subcommand help
- Shell completion 也是個好東西，可以幫助 user
  - https://github.com/oclif/plugin-autocomplete

幾個方向
- show a description of the command
- description of the arguments
- description of all the flags

最最最重要的
- provide examples of common usage of the CLI

![heroku CLI](https://cdn-images-1.medium.com/max/1200/1*UdKDss6k-v36qrbcF4oBNw.png)


#### oclif
看來是神物了，能做到
- Online docs
- in-CLI docs
- autocomplete


### 2. Prefer flags to args
Flags require a bit more typing, but make the CLI much clearer

舉例，有個指令是 `heroku fork`、took in a source app to copy from and a destination app to copy to  
一開始是這樣使用

```
$ heroku fork FROMAPP --app TOAPP
```

一個 flag 跟 一個參數
- it was really confusing which was the source and which was the destination app

後續改為，兩邊都使用 flag
```
$ heroku fork --from FROMAPP --to TOAPP
```

當 argument （用途）很明顯時（`$ rm file_to_remove`），多個參數大致上沒有問題  
原則上
- 1個 type of argument is fine 
- 2個 types are very suspect
- 3個 are never good

當 arguments 的數量是變動時  
- 多個 arguments 是 OK的
  - `$ rm file1 file2 file3`
- 只是，當 arguments 是不同 types，就會 confusing to the user

the flag parser
- should accept a `--` argument
  - to denote that it should stop parsing and simply pass everything down as an argument.
  
這可以這樣執行
```
heroku run -a myapp -- myscript.sh -a arg1
```
  This allows you to run a command like heroku run -a myapp -- myscript.sh -a arg1 (This shows how -a can be a flag for heroku run but also a different -a is passed to the dyno).

### 3. What version am I on?
確保 version 可以用這些指令查
```
$ mycli version # multi only
$ mycli --version
$ mycli -V
```

除非是個 single-command CLI，也會有一個 `-v,--verbose` flag  
`$ mycli -v` 也應該顯示 CLI version  


#### version info for debug
- version 其中主要用途時，叫 user 提供此資訊來幫忙 debug。

I also suggest sending the version string as the User-Agent so you can debug server-side issues. (Assuming your CLI uses an API of some sort)

![](https://cdn-images-1.medium.com/max/1200/1*4kwxuIP4XyM6TGQAd_td9g.png)

### 4. Mind the streams
`Stdout` and `stderr` 讓你 output messages 給 user  
也能把 **redirect** content to a file

```
$ myapp > foo.txt
Warning: something went wrong
```

- stdout is for output
- stderr is for messaging.

![](https://cdn-images-1.medium.com/max/1200/1*Q6FL70WgULj4-tLOvx7otg.gif)  

如果是 subcommand 
- 確保 `pipe the stderr of that subcommand` 是讓 user 來決定

This way any issues are surfaced ultimately to the user’s screen.

### 5. Handle things going wrong

CLIs 比 web app 更容易有問題
- CLI 沒有 UI to guide the user
- the only thing can do is **display an error to the user**
  - This is expected behavior and part of using any CLI.

First and foremost
- 使 errors 是 informative

#### A great error message should contain the following:
好的 error message 應該要有

1. Error code
2. Error title
3. Error description (Optional)
4. How to fix the error
5. URL for more information


For example
```
$ myapp dump -o myfile.out
Error: EPERM - Invalid permissions on myfile.out
Cannot write to myfile.out, file does not have write permissions.
Fix with: chmod +w myfile.out
https://github.com/jdxcode/myapp
```

有時還是有你無法預期的錯誤發生，這時
- 要有一個辦法來盡可能
  - full traceback information 
  - debug output with environment variables
  
oclif 採用 `debug`
- https://www.npmjs.com/package/debug

allows us to output debug statements grouped by component  
if the `DEBUG` environment variable is set
- We have a lot of verbose logging

if debug is fully enabled
- which is incredibly valuable to us when debugging issues.

ensure Error logs
- have timestamps
- truncate them occasionally so they don’t eat up space on disk
- make sure they **don’t contain ansi color codes**

### 6. Be fancy!
spinner example
![spinner](https://cdn-images-1.medium.com/max/900/1*ruLBiz3x3Fg7XN2pVUC31g.gif)

be afraid to show off
- Use colors/dimming to highlight important information
- Use spinners and progress bars to show long-running tasks
  - to tell the user you’re still working
  - only works on a screen. You never want to output those codes to a file
- Leverage OS notifications when a very long-running task is done.

也是有 user 需求是**不要 facny**，所以可以
- TERM=dumb, NO_COLOR is set
- or specify --no-color

建議加入一個 app-specific `MYAPP_NOCOLOR=1` 的 environment variable
- user 可能只想要關掉你這個 cli 的顏色

### 7. Prompt if you can
For accepting input, if stdin is a tty then prompt rather than forcing the user to specify a flag.  
Never require a prompt though. The user needs to be able to automate your CLI in a script so allow them to override prompts always.  

上面兩句話不是很懂，大概的理解是，當情況是需要 user 輸入特定 flag (and args) 時，可以不要用強迫的方式（出 error 告訴 user 錯誤）。而是用選項給 user 選。  
但也不能做成「只能用選項」的方式操作，因為要給 User 有權利自己控制、override prompt  

![prompts](https://cdn-images-1.medium.com/max/1200/1*X-PnBs4mM7BI2vTAjDXS6A.gif)

這種時候也適合用 prompts
- confirmation dialogs for dangerous actions
- For example when destroying a Heroku app
  - you’ll need to type the app name again to confirm:

![prompts](https://cdn-images-1.medium.com/max/1200/1*w_OYC5FXnBub9QPJQGJuQg.gif)


`Checkboxes` and `radio buttons` 都能改善 CLI UX  
![prompts](https://cdn-images-1.medium.com/max/1200/1*ODEhiWAn3qoZ2SSIw4yINw.gif)


### 8. Use tables
- https://github.com/oclif/cli-ux#clitable

table 也是個好方法來呈現資訊
- It’s important that each row of output is a single ‘entry’ of data.
- Never output table borders
  - It’s noisy and a huge pain for parsing
  
**這張圖是舉例 不該 把 borders 印出來**  
![prompts](https://cdn-images-1.medium.com/max/1200/1*AybS19Drx08ihW4KsrNhpw.png) 

保持每一 row 是 single entry，這樣就能 `pipe` 處理，例如
- wc, grep

![prompts](https://cdn-images-1.medium.com/max/1200/1*0mJtpvzUtursztCIrZoKrA.png) 

要注意 screen width 的概念
- Only show a few columns by default
- but allow the user to pass --columns
  - with a comma-separated list of column names to add less common types
  
其他
- Truncate rows that are going to spill over the current screen width
  - unless `--no-truncate` is set.  
- Show column headers by default
  - but allow them to be hidden with `--no-headers`.
- Allow users to pass --filter to filter specific columns
  - (grep can usually do this, but a flag can filter on specific cell values)
- Allow sorting by column with `--sort`
  - Allow inverse and multi-column sort as well.
- Allow output in csv or json
- Displaying raw output as json is a great way to output structured data

It can be manipulated with jq. While jq is incredibly useful, cut and awk are simpler tools that work better with csv data.
- https://stedolan.github.io/jq/


### 9. Be speedy
CLI 也是要講究效能的，利用 time 來 benchmark
- `$ time mycli`
- `$ time ls`

一些指標
- <100ms: very fast (sadly, not feasible for scripting languages)
- 100ms–500ms: fast enough, aim here
- 500ms-2s: usable, but not going to impress anyone
- 2s+: languid, users will prefer to avoid your CLI at this point

```
#$ time ls
real	0m0.089s
user	0m0.003s
sys	0m0.029s
```

當然如果執行一些久的 task，例如 download large file，這時候
- make sure to show a progress bar or at least a spinner
- Even just a spinner will give the impression the CLI is much faster than it is.

### 10. Encourage contributions
- Keep code open source
  - This allows users to poke around and diagnose problems themselves
- It’s healthy to the community to offer a sample in case
  - it’s useful to others
- It makes organizations look great as well.
- Make sure you pick a license of course
- GitHub and GitLab are great places to put your CLI
- README gives you a perfect place to provide an overview of the CLI.

Write up
- how to run the CLI locally and run the test suites
- Offer a contribution guideline doc
  - to tell contributors what you expect
    - in terms of commit syntax
    - code quality
    - tests
    - or whatever else is important for them to know.
- Add a code of conduct
  - Even if you don’t feel that it’s necessary
  - It’s important to some people and they’ll feel much better by seeing one

refs
- https://choosealicense.com/
- https://help.github.com/en/articles/setting-guidelines-for-repository-contributors
- https://help.github.com/en/articles/adding-a-code-of-conduct-to-your-project


### 11. Be clear about subcommands
There are 2 types of CLIs:
- single
- multi-command

#### Single-command CLIs
are basic UNIX-style CLIs like `cp` or `grep`  
only performs one basic task

#### Multi-commands
are more like `git` or `npm`
- accept a subcommand as the first argument.

if the user doesn’t pass anything arguments to the CLI
- it’s always better to list the subcommands (for multi)
  - or display the help (for single)
- rather than do some default behavior

Git chooses to separate subcommands from topics with spaces:
```
$ git submodule add git@github.com:oclif/command
```

Where in the Heroku CLI we use `colons`:
```
$ heroku domains:add www.myapp.com
```

Colons are preferable to help delineate the command between the arguments passed to the command.  
- The user quickly learns that **argument 1 is the command**


### 12. Follow XDG-spec
XDG-spec is a great standard that should be used to find out where to put files.  

Unless environment variables like `XDG_CONFIG_HOME` say otherwise,
- use ~/.config/myapp for config files
- and ~/.local/share/myapp for data files.

For cache files though
- use ~/.cache/myapp on Unix
- but on MacOS it’s better to default to `~/Library/Caches/myapp`
- On Windows you can use `%LOCALAPPDATA%\myapp`

ref link
- https://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html
 