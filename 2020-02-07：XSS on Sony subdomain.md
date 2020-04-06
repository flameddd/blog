# XSS on Sony subdomain
## Gökhan Güzelkokar 2020 Jan 7
### https://medium.com/@gguzelkokar.mdbf15/xss-on-sony-subdomain-feddaea8f5ac

很清楚的文章  
也列出幾個實用工具，是寫網頁的我平常不會接觸的  
有學習價值  

## 先利用 crt.sh 掃描 Certificate Fingerprint
- https://crt.sh/

利用它，可以找到一些 **potential sub-domains**
 
- %my%.sony.net
- %jira%.sony.net
- %jenkins%.sony.net
- %test%.sony.net
- %staging%.sony.net
- %corp%.sony.net
- %api%.sony.net
- %ws%.sony.net
- %.%.%.sony.net
- %p%.sony.net
- %i%.sony.net
- %ff%.sony.net
- %co%.sony.net

## `assetfinder`, `httprobe`
選定一個 domain 後，接著用 `assetfinder` and `httprobe` 近一步掃描  
- https://github.com/tomnomnom/assetfinder
- https://github.com/tomnomnom/httprobe

執行  
```bash
assetfinder -subs-only ppf.sony.net | httprobe
```

![img](https://miro.medium.com/max/608/1*HVSSxnd3gWoh58vxXEem6A.png)  

下面 test 幾個結果看看
- assetfinder -subs-only soundcloud.com | httprobe
  - http://cdn-45f1.soundcloud.com
  - http://backstage.soundcloud.com
  - https://api-v2.soundcloud.com
  - http://l9bjkkhaycw6f8f4.soundcloud.com
  - http://static.soundcloud.com
  - http://creatorguide.soundcloud.com
  - https://l9bjkkhaycw6f8f4.soundcloud.com
  - http://p1.soundcloud.com
  - http://api-v2.soundcloud.com
  - http://cdn-334c.soundcloud.com
  - http://secure.soundcloud.com
  - http://swf.soundcloud.com
  - http://promote.soundcloud.com


## 最後用 dirsearch

執行
```bash
dirsearch.py -u “authtry.dev2.sandbox.dev.ppf.sony.net” -e html,json,php -x 403,500 -t 50
```

`dirsearch` 需要 `python3`，我 mac 裝不起來（說要升級 `xcode`，目前我不想這樣做）  
後來不死心，看看有沒有人包 docker，還真的找到了  
- https://github.com/jradikk/dirsearch-docker

```shell
docker pull jradik/dirsearch-docker:latest
docker run [container_id] -u "www.ettoday.net" -e html,json,php,js -x 403,500 -t 50
docker run [container_id] -u "firebase.google.com" -e html,json,php,js -x 403,500 -t 50
```

這樣重複執行的話，都會留下 container  
要看看能不能用互動模式進去執行，可能會更好  

![dirsearch_screenshot](/assets/img/dirsearch_screenshot.jpg)


最後，他查看了一個 index.php 的檔案，看到這樣的畫面  
![img](https://miro.medium.com/max/836/1*Ogs3_e10yXho21Oj2LjhWw.png)  

作者應該很熟悉 php，知道這樣有機會做 `XSS`  
所以他就試試看並且成功了，也拿到他的`bug bounty` first blood  

![img](https://miro.medium.com/max/1561/1*gzIpEDG2py5p8VXnIWuteg.png)  

### 關於 dirsearch
看來，大家在使用時，都會搭配 diction list 使用，官方有一個預設的  
下次可以試試看，搭配 `-wordlist` flag
