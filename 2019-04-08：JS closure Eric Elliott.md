### JS closure Eric Elliott
#### Eric Elliott
#### Jan 8, 2016
#### https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36

### closure
Closure 是「function」以及「該function被宣告」時所在的作用域環境（lexical environment）的組合。
- 宣告一個 function 在另一個 function 裡面，然後 expose it
- inner function 能夠存取到 outer function 的變數，即使 outer 已經執行 return 了

### Using Closures (Examples)
- closures 常用來建立**data privacy**
- Data privacy is an essential property that helps us program to an interface, not an implementation
- because implementation details are more likely to change in breaking ways than interface contracts.

enclosed 的變數 variables 能存取的範圍只有在 outer function 裡面  
這樣就能實現私有變數  
```js
const getSecret = (secret) => {
  return {
    get: () => secret
  };
};

const obj = getSecret(1);
const actual = obj.get();
// secret 是從外部存取不到的。這 ex 寫法有點奇妙
```

除了 Object 實現 data privacy 做法之外，Closures 還可以實作出 `stateful function`來達成。  
`stateful function` 意思是 function 的結果(return) 會受到 internal 的 state 影響。

```js
const secret = msg => () => msg;
```

### Closures 在 functional programming 的應用
- partial application
- currying

Currying（柯里化），又稱為 parital application 或 partial evaluation，是個「將一個接受 n 個參數的 function，轉變成 n 個只接受一個參數的 function」的過程。  

原理是將傳入 function 的參數，利用 closure（閉包）特性，將它們存放在另一個 function 中並當做回傳值，而這些 function 會形成一個鏈（chain），待最後參數傳入，完成運算。  

這樣做的好處是
- 簡化參數的處理，基本上是一次處理一個參數，藉以提高程式的彈性和可讀性
- 將程式碼依功能拆解成更細的片段，有助於重複利用
- 這對於整理冗長程式碼、需要詳客製化的 function 和非同步呼叫的處理等有很大的幫助。

```js
function curriedMultiply(x) {
  return function(y) {
    return x * y;
  }
}

var multipleOfThreeAndNumberY = curriedMultiply(3);

multipleOfThreeAndNumberY(5); // 15
multipleOfThreeAndNumberY(10); // 30
```