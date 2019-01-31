## 測試 Immutable.js, persistent data structures 的效能
## 文章來源： https://medium.com/@dtinth/immutable-js-persistent-data-structures-and-structural-sharing-6d163fbd73d2
## github: https://github.com/flameddd/Immutable_persistent_data_structure

讀完上面這篇 `Immutable.js, persistent data structures and structural sharing` 之後，做點測試。測試的有
 1. 原生 Object assign
 2. immutableJS set
 3. Seamless-immutable set
 4. immer produce update

object size 5000 key:value

## install and run
 > step1. .\npm install  
 > step2. .\npm start

## result
 1. 測試_Object_Assign: `1.822ms`
 2. 測試ImmutableJS_set: `0.783ms`
 3. 測試Seamless-Immutable_set: `4.477ms`
 4. 測試 immer produce update: `29.253ms`

## 結論 Immutable 真的快。
但作者也有一段話很重要
> Not always…
Please don’t take this article to mean “you should always use Immutable.js.” No, I’m just trying to highlight its benefits in this article and explain why it’s recommended a lot.

> Data structure matters, but when I write software, I try to do the simplest thing first. I’d use plain arrays and objects, and then use Immutable.js when I need a speed boost, or when I have a very strong feeling that I will need it for sure (e.g. I anticipate my collection to hold 10,000 items). I almost never use Immutable.js with small objects or collections that I know will have just only a few items in it.

所以看情況使用吧。

```js
/// test source code
const Immutable = require('immutable')
const SeamImmutable = require("seamless-immutable").static;

const testData = Array
  .from({ length: 5000 }, (v, i) => i)
  .reduce((acc ,cur) => Object.assign(
    {},
    acc,
    {[cur]: `${Math.floor(Math.random() * 99999) + 1}` }
  ), {})

const immutableTestData = Immutable.fromJS(testData)
const seamImmutableTestData = SeamImmutable(testData);

console.time('1.測試_Object_Assign')
const test1 = Object.assign(
  {},
  testData,
  { '99': '測試 Object Assign' }
)
console.timeEnd('1.測試_Object_Assign')
console.log(test1['99'])

console.time('2.測試ImmutableJS_set')
const setImmutableData = immutableTestData.set('99', '測試 immutableJS set')
console.timeEnd('2.測試ImmutableJS_set')
console.log(setImmutableData.get('99'))

console.time('3.測試Seamless-Immutable_set')
const setSeamImmutableData = SeamImmutable
  .set(seamImmutableTestData, "99", "測試 Seamless-Immutable set");
console.timeEnd('3.測試Seamless-Immutable_set')
console.log(setSeamImmutableData['99'])
```

