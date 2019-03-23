## Using Events In Node.js The Right Way
## Usama Ashraf
## https://dev.to/usamaashraf/using-events-in-nodejs-the-right-way-449b

程式之間要溝通，直覺的方法就是直接呼叫對方。  
但 event-driven 的方式思維是「回應對方」，而不是被呼叫。

### Eventing 的優點
- 更解耦合 decoupled
- 這架構下，擴展 feature 時更容易，只要新增更多特定的 listeners 就能溝通了。  

這邊要聊的，本質上就是  Observer pattern.

### 設計 Event-Driven 架構

- 區別 events 是一個重點，因為我們不想到頭來去 remove/replace 已存在的 events 。
- 基本準則是，當**一組商業邏輯結束時，才去 fire an event**。

NodeJS 中有 `emitters` 可以 `emit` 出**已被命名的 events**，來執行所為的 listeners function  
所有 emit events 都是 EventEmitter class 的實例。  

### Example

```js
// my_emitter.js
const EventEmitter = require('events');
const myEmitter = new EventEmitter();
module.exports = myEmitter;
```

```js
// registration_handler.js
const myEmitter = require('./my_emitter');

// Perform the registration steps
// Pass the new user object as the message passed through by this event.
myEmitter.emit('user-registered', user); // user 註冊流程跑完了，接著 fire event
```

```js
// listener.js 另外就會有專門寫這隻 listener。這樣邏輯清楚分開，程式也好維護

const myEmitter = require('./my_emitter');

myEmitter.on('user-registered', (user) => {
  // Send an email or whatever.
});
```

訂閱也拆一隻
```js
// subscriptions.js

const myEmitter = require('./my_emitter');
const sendEmailOnRegistration = require('./send_email_on_registration');
const someOtherListener = require('./some_other_listener');

myEmitter.on('user-registered', sendEmailOnRegistration);
myEmitter.on('user-registered', someOtherListener);
```

- 上面這樣的架構，讓 `listener` 能 re-usable。未來其他地方要用，就能重複這樣操作。
- 多個  `listener` attache 到單一一個 event，它們會 synchronously 執行和 follow attache 的順序。

要 asynchronously 的話，listener 用 setImmediate 包：
```js
// send_email_on_registration.js

module.exports = (user) => {
  setImmediate(() => {
    // Send a welcome email or whatever.
  });
}
```

### 保持 Listeners 乾淨
- 寫 Listeners 時，保持 Single Responsibility 原則
- 一個 listeners 只做一件事情、把它做好。

例如，寫太多 conditionals 在一隻 listener 中。  

```js
// registration_handler.js
const myEmitter = require('./my_emitter');

// Perform the registration steps
// The application should react differently if the new user has been activated instantly.
if (user.activated) {
  myEmitter.emit('user-registered:activated', user);
} else {
  myEmitter.emit('user-registered', user);
}
```

```js
// subscriptions.js

const myEmitter = require('./my_emitter');
const sendEmailOnRegistration = require('./send_email_on_registration');
const someOtherListener = require('./some_other_listener');
const doSomethingEntirelyDifferent = require('./do_something_entirely_different');

myEmitter.on('user-registered', sendEmailOnRegistration);
myEmitter.on('user-registered', someOtherListener);

myEmitter.on('user-registered:activated', doSomethingEntirelyDifferent);
```

### 有必要時，就 Detaching Listeners
上面的 case，listeners 都是獨立的 function，但有時候是跟 object 有關係的（可能是 object 的 method）  

> 這種時候就有需要手動 detache，不然 GC 永遠不會回收，因為還有 referenced，這樣可能會有 memory-leak

舉例，一位 user connect 進入聊天室，我們想發通知
```js
// chat_user.js

class ChatUser {

  displayNewMessageNotification(newMessage) {
    // Push an alert message or something.
  }

  // `chatroom` is an instance of EventEmitter.
  connectToChatroom(chatroom) {
    chatroom.on('message-received', this.displayNewMessageNotification);
  }

  disconnectFromChatroom(chatroom) {
    chatroom.removeListener('message-received', this.displayNewMessageNotification);
  }
}
```
當這位 user 離開（斷線 or 關分夜那種）時，這時候去呼叫 `this.displayNewMessageNotification` 給一位 offline 的 user 就沒甚麼意義了。沒清除的話，Server 就會去執行

### 提醒
event-driven architectures 雖然 loose coupling，但一不注意也會增加複雜度。尤其程式上變成了一種  
events trigger chains (在 listener 裡面去 emit event)，要追蹤這種 code 很痛苦。

