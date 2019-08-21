## JavaScript Clean Code - Best Practices
### Milos Protic May, 19 2019
#### https://devinduct.com/blogpost/22/javascript-clean-code-best-practices

### 1. Strong type checks

```js
// bad
0 == false // true

// good
0 === false // false
```

### 2. Variables 有意義的命名
```js
// bad
let daysSLV = 10;
let y = new Date().getFullYear();

let ok;
if (user.age > 30) {
  ok = true;
}

// Good
const MAX_AGE = 30;
let daysSinceLastVisit = 10;
let currentYear = new Date().getFullYear();

const isUserOlderThanAllowed = user.age > MAX_AGE;
```

也不要多餘的命名
```js
// bad
let nameValue;
let theProduct;

// bad
const user = {
  userName: "John",
  userSurname: "Doe",
  userAge: "28"
};

user.userName;

//Good
let name;
let product;

// good
const user = {
  name: "John",
  surname: "Doe",
  age: "28"
};

user.name;
```

不要強迫人去記變數的內容是什麼
```js
// bad
const users = ["John", "Marco", "Peter"];
users.forEach(u => {
  doSomething();
  // ...
  // ...
  // Here we have the WTF situation: WTF is `u` for?
  register(u);
});

// good
const users = ["John", "Marco", "Peter"];
users.forEach(user => {
  doSomething();
  // ...
  // ...
  register(user);
});
```

### 3. Functions
採用 **長的、描述性的 名字**  
把 name 當作它要具體行為  
a function name should be a verb or a phrase fully exposing the intent behind it as well as the intent of the arguments  

```js
// Bad:
function notif(user) {
  // implementation
}

// Good
function notifyUser(emailAddress) {
  // implementation
}
```

避免**過多的參數**  
用 **default參數** 取代 conditionals  
```js
//Bad
function getUsers(fields, fromDate, toDate) {
  const usrFromDate = fromDate || "2019-01-01";
  // implementation
}

//Good
function getUsers({ fields, fromDate, toDate }) {
  // implementation
}

getUsers({
  fields: ['name', 'surname', 'email'],
  fromDate: '2019-01-01',
  toDate: '2019-01-18'
});
```

function 中別做多件事情  
```js
// Bad
function notifyUsers(users) {
  users.forEach(user => {
    const userRecord = database.lookup(user);
    if (userRecord.isVerified()) {
      notify(user);
    }
  });
}

// Good

function notifyVerifiedUsers(users) {
  users.filter(isUserVerified).forEach(notify);
}

function isUserVerified(user) {
  const userRecord = database.lookup(user);
  return userRecord.isVerified();
}
```

用 `Object.assign` 來設 `default objects`  
```js
// Bad
const shapeConfig = {
  type: "cube",
  width: 200,
  height: null
};

function createShape(config) {
  config.type = config.type || "cube";
  config.width = config.width || 250;
  config.height = config.width || 250;
}

createShape(shapeConfig);

//Good
const shapeConfig = {
  type: "cube",
  width: 200
  // Exclude the 'height' key
};

function createShape(config) {
  config = Object.assign(
    {
      type: "cube",
      width: 250,
      height: 250
    },
    config
  );

  //...
}

createShape(shapeConfig);
```

不用使用 `flags` 的參數  
`flags` 就是在告訴你，這 function 做了多出這 function 該做的事情
```js
// Bad:
function createFile(name, isPublic) {
  if (isPublic) {
    fs.create(`./public/${name}`);
  } else {
    fs.create(name);
  }
}
// Good:
function createFile(name) {
  fs.create(name);
}

function createPublicFile(name) {
  createFile(`./public/${name}`);
}
```

**不要污染 globals**  
如果需要 extend 已存在的 object，那用 **ES Classes 繼承**  


```js
//Bad
Array.prototype.myFunc = function myFunc() {
  // implementation
};

//Good
class SuperArray extends Array {
  myFunc() {
    // implementation
  }
}
```

### 4. Conditionals

條件式要避免**負面表列**  
寫 code 的時候，一定要想著，未來總有人會來讀這邊、理解這邊  
這個人可能是未來的你～  

> method 和 變數名稱的負面條件、否定詞 都會阻礙理解

```js
//Bad
function isUserNotBlocked(user) {
  // implementation
}

if (!isUserNotBlocked(user)) {
  // implementation
}

//Good
function isUserBlocked(user) {
  // implementation
}

if (isUserBlocked(user)) {
  // implementation
}
```

Use conditional shorthands. 

```js
// Bad
if (isValid === true) {
  // do something...
}

if (isValid === false) {
  // do something...
}
// Good
if (isValid) {
  // do something...
}

if (!isValid) {
  // do something...
}
```

可以不用 conditionals 時，就避掉吧  
用 polymorphism and inheritance 取代  

```js
// Bad
class Car {
  // ...
  getMaximumSpeed() {
    switch (this.type) {
      case "Ford":
        return this.someFactor() + this.anotherFactor();
      case "Mazda":
        return this.someFactor();
      case "McLaren":
        return this.someFactor() - this.anotherFactor();
    }
  }
}

//Good
class Car {
  // ...
}

class Ford extends Car {
  // ...
  getMaximumSpeed() {
    return this.someFactor() + this.anotherFactor();
  }
}

class Mazda extends Car {
  // ...
  getMaximumSpeed() {
    return this.someFactor();
  }
}

class McLaren extends Car {
  // ...
  getMaximumSpeed() {
    return this.someFactor() - this.anotherFactor();
  }
}
```

### 5. ES Classes

```js
// Bad
const Person = function(name) {
  if (!(this instanceof Person)) {
    throw new Error("Instantiate Person with `new` keyword");
  }

  this.name = name;
};

Person.prototype.sayHello = function sayHello() { /**/ };

const Student = function(name, school) {
  if (!(this instanceof Student)) {
    throw new Error("Instantiate Student with `new` keyword");
  }

  Person.call(this, name);
  this.school = school;
};

Student.prototype = Object.create(Person.prototype);
Student.prototype.constructor = Student;
Student.prototype.printSchoolName = function printSchoolName() { /**/ };


// Good
class Person {
  constructor(name) {
    this.name = name;
  }

  sayHello() {
    /* ... */
  }
}

class Student extends Person {
  constructor(name, school) {
    super(name);
    this.school = school;
  }

  printSchoolName() {
    /* ... */
  }
}
```


使用 method chaining
- code will be less verbose

class 的每一個 function 就 return `this`  
就能做到這功能  

```js
// Bad
class Person {
  constructor(name) {
    this.name = name;
  }

  setSurname(surname) {
    this.surname = surname;
  }

  setAge(age) {
    this.age = age;
  }

  save() {
    console.log(this.name, this.surname, this.age);
  }
}

const person = new Person("John");
person.setSurname("Doe");
person.setAge(29);
person.save();

// Good
class Person {
  constructor(name) {
    this.name = name;
  }

  setSurname(surname) {
    this.surname = surname;
    // Return this for chaining
    return this;
  }

  setAge(age) {
    this.age = age;
    // Return this for chaining
    return this;
  }

  save() {
    console.log(this.name, this.surname, this.age);
    // Return this for chaining
    return this;
  }
}

const person = new Person("John")
  .setSurname("Doe")
  .setAge(29)
  .save();
```

### 6. Avoid In General
- you should do your best not to repeat yourself
- not to leave tails behind you such as unused functions and dead code
