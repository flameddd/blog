## firebase china

哎，開始了才想到，firebase 會被 china 擋住  
google 後的方案都是 proxy

記錄一些 ref 未來有機會參考  

下面我沒有 review 過  
只是存在來可能做參考而已  

未來就多用 `firebase china proxy` 當做 keyword 吧  

- https://codesandbox.io/s/o9r2lz1n69?fontsize=14
```js
var http = require("http");
var proxyFirebaseConfig = require("./proxy-firebase-config");

const html = `
  <!DOCTYPE html>
    <head><title>china firebase proxy client</title></head>
    <body>
      <script type="text/javascript">
        fetch('https://o9r2lz1n69.sse.codesandbox.io:8118')
          .then(res => {
            console.log(res)
          })
      </script>
    </body>
  </html>
`

http
  .createServer(function(req, res) {
    res.writeHeader(200, {"Content-Type": "text/html"});  
    res.write(html);  
    res.end(); // end the response
  })
  .listen(8080); //the server object listens on port 8080

```

```js
const caw = require("caw");
const https = require("https");
const admin = require("firebase-admin");
const serviceAccount = require(`../config/service-account-key.json`);

console.log("begin testing proxy Firebase");

// Your Proxy
const firebaseAgent = caw("http://127.0.0.1:8118", {
  protocol: "https"
});

const app = admin.initializeApp({
  credential: admin.credential.cert(serviceAccount, firebaseAgent),
  databaseURL: "https://china-proxy-1c2da.firebaseio.com",
  agent: firebaseAgent, // proxy agent
  logging_enabled: true // open logging info
});

// Use the shorthand notation to retrieve the default app's services
var defaultAuth = app.auth();
var defaultDatabase = app.database();
var db = admin.firestore();

var myTestRef = db
  .collection('my-test-collection')
  .doc('my-test-document');
var getDoc = myTestRef
  .get()
  .then(doc => {
    if (!doc.exists) {
      console.log('No such document!');
    } else {
      console.log('Document data:', doc.data());
    }
  })
  .catch(err => {
    console.log('Error getting document', err);
  });

console.log(`Firebase App Name: ${app.name}`);
console.log(`Default database: ${JSON.stringify(Object.keys(defaultDatabase))}`);
console.log("end of the testing proxy Firebase");

```


- https://gist.github.com/urish/4a79ad2a48ad68ae6cac6bd0bc4e881c
```js
// To use: npm install ws

'use strict';

var WebSocket = require('ws');
var WebSocketServer = WebSocket.Server;
var wss = new WebSocketServer({port: 5000});

wss.on('connection', function (ws) {
	var clientWs = new WebSocket(
    'wss://s-dal5-nss-22.firebaseio.com/.ws?v=5&ns=firebase-name',
    {protocolVersion: 8, origin: 'http://localhost'}
  );

	clientWs.on('message', function(data) {
		console.log('server', data);
		ws.send(data);
	});

	ws.on('message', function(data) {
		console.log('client', data);
		clientWs.send(data);
	});

	console.log('conn');
});
```