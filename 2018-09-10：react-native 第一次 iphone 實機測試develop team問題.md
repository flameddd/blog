## 測試 cool-wallet app
  程式碼是現成的，要在我的機器上開啟需要改一些設定，今天踩了一些問題才順利有機會開啟。

## Failed to create provisioning profile
我已經改掉 develop team 了之後，Run 出現此錯誤。  
把上面的（應該一起都是在 build settings 頁面的）「bundle identifier」欄位也替換掉。 i.e. if you had HelloWorld change it to HelloWorld12345  
因為我拿別人的 code ，可能我機器上沒有此 team 的設定

## develop team 設定調整
 - 參考網址(作者：超级大柱子)：[ReactNative iOS真机调试注意事项](https://www.jianshu.com/p/65692f59063b)

以下步骤, 前3步不能少：
1. 设定开发者账号，并且修改公司名为com.ymblender.项目名。
2. Build Settings -> Signing 中所有都改为iOS Developer：
3. 清除缓存(cmd+shift+k)，重启Xcode，编译到真机
4. 如果提示：No account for team "UBF8T346G9". Add a new account in the Accounts preference pane or verify that your accounts have valid credentials.

表示这个工程之前的teamID没有被切换。

5. 使用旧的TeamID(UBF8T346G9)搜索（會找出該欄位，有 dropdown menu 可以改值），并且把这个team改为自己的team
6. 清除缓存，重启Xcode(这次可以不需要重启)，编译到真机器
7. 搜索localhost，找到RCTWebSocketExecutor.m，把localhost改为自己的本机ip

## iphone 實機要「信任」App
 - 到「設定」>「一般」>「裝置管理」>「開發者App」，並找到尚未信任的描述檔，並點選信任
