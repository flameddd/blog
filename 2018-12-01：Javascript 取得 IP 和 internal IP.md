## 2018-12-01：Javascript 取得 IP 和 internal IP

#### npm: https://www.npmjs.com/package/binternalip
#### repo: https://github.com/flameddd/binternalip

這個議題算是最近才了解，做個小紀錄。
1. client side JS 是抓不到 user ip 的。
2. client side JS 要取得 ip 的目前唯一解是，往一個 server 打 request，讓該 server 回傳 ip。（網路上似乎有些在提供這種服務，不然只能自己架）
3. client side JS 是可以取到 internal IP （區網IP），程式碼如下。
4. 取區網IP 的做法，可能多多少少會有 browser 支援度的問題，真的要突破此問題，也是自己架一個 server 在區網裡面給大家打應該就能解了。  

client side 的可以 publish 一個  
server side 的可以包一個 docker  

```javascript
//compatibility for Firefox and chrome
// canisue: https://caniuse.com/#search=RTCPeerConnection 
window.RTCPeerConnection = window.RTCPeerConnection
  || window.mozRTCPeerConnection
  || window.webkitRTCPeerConnection;

const getMyIPAddr = new Promise(function (resolve, reject) {
  try {
    const pc = new RTCPeerConnection({ iceServers: [] });
    const noop = function () { };
    //create a bogus data channel
    pc.createDataChannel('');
    // create offer and set local description
    pc.createOffer(pc.setLocalDescription.bind(pc) , noop);
    pc.onicecandidate = function (ice) {
      if (ice && ice.candidate && ice.candidate.candidate) {
        const myIP = /([0-9]{1,3}(\.[0-9]{1,3}){3}|[a-f0-9]{1,4}(:[a-f0-9]{1,4}){7})/.exec(ice.candidate.candidate)[1] || ''
        resolve(myIP)
        pc.onicecandidate = noop;
      }
    };
  } catch{
    resolve('')
  }
});

getMyIPAddr
  .then(function (internalIP) {
    console.log('interanlIP => ' + internalIP) // interanlIP => 10.46.155.197
  })

```
