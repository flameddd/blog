# Fibonacci Javascript
## devlucky Mar 4, 2016
### https://medium.com/developers-writing/fibonacci-sequence-algorithm-in-javascript-b253dc7e320e

```
F(n) = F(n-1) + F(n-2)
```

### 遞迴 Recursive
- Time complexity: O(2^N)
- Space complexity: O(n)
- Function calls: 20.365.011.074
- Time needed: 176.742ms

```js
function fibonacci(num) {
  if (num <= 1) return 1;

  return fibonacci(num - 1) + fibonacci(num - 2);
}
```

### Memoization
- Time complexity: O(2N)
- Space complexity: O(n)
- Function calls: 99
- Time needed: 0.000001ms

```js
const memo = {};

function fibonacci(num) {

  if (memo[num]) return memo[num];
  if (num <= 1) return 1;

  memo[num] = fibonacci(num - 1) + fibonacci(num - 2)
  return memo[num];
}
```

### while loop
- Time complexity: O(N)
- Space complexity: Constant
- Function calls: 51
- Time needed: 0.000001ms

```js
function fibonacci(num){
  let a = 1
  let b = 0
  let temp;

  while (num >= 0){
    temp = a;
    a = a + b;
    b = temp;
    num--;
  }

  return b;
}
```