# TomNomNom: Bug Bounties With Bash
## TomNomNom 2020.04.18
### https://www.youtube.com/watch?v=s9w0KutMorE

talk 示範用 bash 來協助 Subdomain brute force

subdomains.txt
```
admin
test
qa
dev
www
m
formus
invalid
blog
shop
news
mail
ftp
```

### 第一步，確認 domain 能不能 resolve
用 `host`

```
> host example.com
example.com has address 93.184.216.34
example.com has IPv6 address 2606:2800:220:1:248:1893:25c8:1946
example.com mail is handled by 0 .

> host lolwtfdoing.com
Host lolwtfdoing.com not found: 3(NXDOMAIN)
```

上面的結果，人來讀的話，很快理解，但用程式來讀呢？  

在 command line 執行的指令，都有一個 exit code  
- 用 `echo $?` 來查看，如果是 `0` 代表 command work，其他數字代表其他的結果 (not work)

```
> ls
...

> echo $?
0
```

這樣就能寫 condition
```bash
if this-command-work; then
    run-this-command
fi
```

```bash
if host example.com; then echo "It resolves!"; fi
```

可以結合 IO redirection，像是 `&>` 
- `&>`: https://www.gnu.org/software/bash/manual/bash.html#Redirecting-Standard-Output-and-Standard-Error
```bash
if host example.com &> /dev/null; then echo "It resolves"; fi
```

把結果導去 `/dev/null` 黑洞中，這樣就不會有顯示輸出結果  
適用於有時候不想看到過程，只想看到結果  


Loop 的做法

```bash
while this-command-works; do
    this-command
done
```

`read` 指令，讀取 stand input 的 single line  
```bash
while read sub; do
    echo "$sub.google.com";
done < subdomains.txt
```  
印出組合的 subdomain  

利用 `sh` file 來整理 code  
- 第一行可以告訴 terminal 說要用哪一種 shell
```shell
#!/usr/bin/bash

domain=$1
while read suh; do
    if host "$sub.$domain" &> dev/null; then
        echo "$sub.$domain"
    fi
done
```

可能需要 update execute permission
```
chmod +x ./brute.sh
```

舉例
```bash
#!/usr/bin/bash

while read sub; do
  if host $sub.sbtuk.net &> /dev/null; then
    echo "$sub.sbtuk.net";
  fi
done < subdomain.txt
```

`$1` 就是代表第一個參數、`$2` 第二個   
這例子，就能這樣執行 (code 參考上面一點的範例)
```
./brute.sh sbtuk.net
```

stand input 可以從外面輸入  
上面的範例拿掉 `done` 後面的 `< subdomain.txt`  

指令輸入時，改成  
```
./brute.sh sbtuk.net < subdomain.txt
```

或者 pipe `|`
```shell
cat subdomain.txt | ./brute.sh sbtuk.net
```

### Plan
1. check subdomain for CNAME records
2. check if those CNAMEs resolve
3. ...profilt?
4. demo

```
> host -t CNAME invalid.sbtuk.net
invalid.sbtuk.net is an alias for lolifyouregisteredthisyouwastedyourmoney.com.

> host lolifyouregisteredthisyouwastedyourmoney.com
Host lolifyouregisteredthisyouwastedyourmoney.com not found: 3(NXDOMAIN)
```

接著用 `grep` 來過濾有找到 CNAME  的結果
```
> host -t CNAME invalid.sbtuk.net | grep 'an alias'
```

在用 `awk` 來取 field, `awk` 是用來 text process 的
- `$NF` 是 `awk` 的變數ㄜ，用來取最後一了欄位
- `$(NF-1)` 是倒數第二個欄位
- `$1` 是第一、`$2` 是第二，依此類推
```
> host -t CNAME invalid.sbtuk.net | grep 'an alias' | awk '{print $NF}'
lolifyouregisteredthisyouwastedyourmoney.com.
```

結合 `$()`，就能把上面的指令變成變數傳回  

```
> cname=$(host -t CNAME invalid.sbtuk.net | grep 'an alias' | awk '{print $NF}')
> echo $cname
```

結合到前面的 script  
- (`if [ -z "$cname" ]; then` 判斷如果 `cname` 是 zero line 的話，就跳過)

```bash
#!/usr/bin/bash

domain=$1
while read sub; do
  cname=$(host -t CNAME $sub.$domain | grep 'an alias' | awk '{print $NF}')

  if [ -z "$cname" ]; then
    contiune
  fi

  if ! host $cname &> /dev/null; then
    echo "$cname did not resolve ($sub.$domain)";
  fi
done
```

然後這樣執行
```
cat subdomain.txt | ./cname.sh yahoo.com
```

### Fetch All The things

回到這邊開始，用指令印出 subdomain list
```
> cat subdomains.txt | ./brute.sh yahoo.com
test.yahoo.com
admin.yahoo.com
m.yahoo.com
blog.yahoo.com
```

結合 `curl` 來 fetch url
- `-L` flag: follow redirect
- `-o` outpur file
- `-s` silent flag
- `-k` igore significant error

調整 command
- `awk` 來加上 https prefix
- `>` redirect output to file

```
cat subdomains.txt | ./brute.sh yahoo.com | awk '{print "https://" $1}' > url
```

fetch 所有 url
- mkdir 的 `-p` 用來 create nested folder、已存在的話，就忽略
- `tee` 將結果同時輸出到螢幕和檔案。把 filename 跟 url 對稱印出來，這樣才知道哪個檔案是哪個 url
  - `-a` 是 overrite

```bash
#!/usr/bin/bash

mkdir -p out

while read url; do
  filename=$(echo $url | sha256sum | awk '{print $1}')
  filename="out/$filename"
  echo "$filename $url" | tee -a index
  curl -sk "$url" -o $filename
done
```

在 vim 中，可以這樣呼叫現在的檔案，這樣就能不用跳出去，也能執行 bash 
- `cat urls` 只是用來印出 url list 而已
- `%` 表示目前的檔案
```
:!cat urls | ./%
```

更改 permission 也可以這樣做
```
:!chmod +x %
```

### Grep title
fetch 完所有 html 之後，如果我們要找出所有 title 呢？  

grep
- `-E` 表示用正規表達式
- `-i` ingore 大小寫
- `-o` 只找出有 match 的地方（不然會找出整行、通常 mini 過的檔案，整行就是幾乎就是整隻檔案了)
```
grep -oiE '<title>(.*)</title>' *
```

### Grep header

上面的 `curl` 加上 `-v`
- 然後 verbose 的結果並不會在 stand ouput 中。所以用 `&>` 取代 `-o` 來導出所有 output
- `curl -sk "$url" -o $filename` -> `curl -sk -v "$url" &> $filename`

這樣就有把 header 一起存了  

這時候，存的格式會有 `< ` prefix，可以這樣抓 header
- `grep -hE '^< ' *`: 抓出用 `< ` 起始的 data
  - `-h` 會略過 filename ，這樣輸出載畫面上好看一點  

`grep -hE '^< ' * | sort` 做點排序，這樣能馬上看到最後面的輸出，讓我們一眼看出有哪些 **custom header** or 特別的 header  

### 一些 vim 的技巧，來快速瀏覽檔案

一次開啟多個檔案 or 特定檔案
- `vim`: `-p[N]`: Open N tab pages.  If [N] is not given, one tab page is opened
		for every file given as argument.
- `find` 的 `-type` 可以搭配一些選項
  - `d`：目錄
  - `p`：具名的 pipe（FIFO）
  - `f`：一般的檔案。
  - `l`：連結檔，如果與 `-L` 或 `-follow` 參數同時使用時，就只會搜尋到有問題的連結檔，如果想要與 `-L` 同時使用，請改用 `-xtype`
  - `s`：socket 檔案
```
vim -p *
vim -p $(find . -type f)
```

vim 中
- `:n` 下一個 tab
- `:N` 上一個 tab
- `G` 文件最後一行看看有什麼東西

### xargs
用來讀取 stand input 的每行字串並轉換為指令  
- `-n1`: 「一行」做為分界當作輸入的參數，`n3` 就是三行
- `-P10`: 平行執行 10 個 process 

```
cat subdomain.txt | awk '{print $1 ".sbtuk.com"}' | xargs -n1 -P10 ./sub.sh
```

```bash
#!/usr/bin/bash

domain=$1
while read sub; do
  echo $sub.$domain
done | xargs -n1 -P10 ./sub.sh
```

```
cat subdomain.txt | ./parsub.sh yahoo.com
```