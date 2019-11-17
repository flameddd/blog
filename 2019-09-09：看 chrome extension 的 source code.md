## 看 chrome extension 的 source code
#### 參考 http://www.chrisl e.me/2012/12/how-to-get-the-source-code-to-any-chrome-extension/

Mac 下找
### Chrome
查 `extension ID`
- 進入管理擴充功能
- 找到要看的 extension
- 查到它的 ID, e.x. ophjlpahpchlmih****************jc

Chrome 還可以開 `devTool`！，這樣能看 network 之類的

### Mac
開 finder
- menubar 選「前往(goto)」，然後「前往資料夾（Go To Folder）」
- 去 `Library`
  - 要找一下，我的就在 UserID 底下的「資源庫」，不是在最外層的「資源庫」
- 找到底下的這個路徑 `Application Support/Google/Chrome/Default/Extensions`
- 查看上面查到的 `extension ID` 就有了
