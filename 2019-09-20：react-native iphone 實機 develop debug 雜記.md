## 2019-09-20：react-native iphone 實機 develop debug 雜記
這兩天一直在弄 JS remote debug 的問題。
撞到的問題有
 1. 開啟 debug 後 ontouch 的觸發非常非常慢，可能要快 1分鐘 才有反應。
 2. fetch 會失敗，出現錯誤訊息"unsupported BodyInit type"
 3. CORS 問題

CORS 問題我不確定我是否解決了，"unsupported BodyInit type" 是 react-native debug 官方的 bug。  
雖說後面版本有修正了，但我現況無法花這麼多時間去升級 react-natve。  
在 fetch 執行時設定 header 也是可行方法，但也不適用我這邊。  
  
最後聽從別人建議，改用 `react-native debugger` 來 debug。確實就沒有"unsupported BodyInit type"的問題了（看來 inspect network 方面有不同作法）
  
ontouch 非常慢問題，網路上說是 `react dev tool` 造成的，我在 `react-native debugger` 的 menu 把 `react` 跟 `redux` 的 dev tool 關掉後，確實就順了！！（另外我是用無痕模式）

[react-native-debugger](https://github.com/jhen0409/react-native-debugger)
 - 安裝方法 「brew update && brew cask install react-native-debugger」
 - 開啟方法 「open "rndebugger://set-debugger-loc?host=localhost&port=8081"」