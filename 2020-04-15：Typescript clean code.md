# clean-code-typescript
### en: https://github.com/labs42io/clean-code-typescript
### cn: https://github.com/beginor/clean-code-typescript

（這篇針對 ts 的收穫，實在不多...，跟一般程式文章、OO 的比較相關）

前面很多部分跟一般程式語言、javascript 都差不多的建議，跟 ts 其實沒有太多關係
- 當覺得跟 ts 沒太多關係時，當作 js 的建議也可以

## 變數名稱
命名可辨識的名稱

```ts
// bad: a1, a2, a3 這種無法辨識
function between<T>(a1: T, a2: T, a3: T): boolean {
  return a2 <= a1 && a1 <= a3;
}

// good 比較 value 是否在 left, right 之間
function between<T>(value: T, left: T, right: T): boolean {
  return left <= value && value <= right;
}
```

## 變數名稱，要命名成可以「念」出來

```ts
// bad: 無法念出來，這樣會不方便口頭討論
type DtaRcrd102 = {
  genymdhms: Date;
  modymdhms: Date;
  pszqint: number;
}

// good
type Customer = {
  generationTimestamp: Date;
  modificationTimestamp: Date;
  recordId: number;
}
```

## Use the same vocabulary for the same type of variable
```ts
// bad
function getUserInfo(): User;
function getUserDetails(): User;
function getUserData(): User;

// good
function getUser(): User;
```


## 變數要命名成可以 search
```ts
// bad: What the heck is 86400000 for?
setTimeout(restart, 86400000);

// good: Declare them as capitalized named constants
const MILLISECONDS_IN_A_DAY = 24 * 60 * 60 * 1000;
setTimeout(restart, MILLISECONDS_IN_A_DAY);
```

## 使用能解釋的變數名稱
```ts
// bad
declare const users: Map<string, User>;

for (const keyValue of users) {
  // iterate through users map
}

// good
declare const users: Map<string, User>;

for (const [id, user] of users) {
  // 比起 "keyValue"，"id, user" 更能解釋，但這邊我覺得是 users, user 的命名不好
  // iterate through users map
}
```

## 避免心理、聯想的變數命名
- Clarity 才是王道
- Clarity 的敵人是 clever

```ts
// bad
const u = getUser();
const s = getSubscription();
const t = charge(u, s);

// good
const user = getUser();
const subscription = getSubscription();
const transaction = charge(user, subscription);
```

## Don't add unneeded context
```ts
// bad
type Car = {
  carMake: string;
  carModel: string;
  carColor: string;
}

function print(car: Car): void {
  console.log(`${car.carMake} ${car.carModel} (${car.carColor})`);
}

// good
type Car = {
  make: string;
  model: string;
  color: string;
}

function print(car: Car): void {
  console.log(`${car.make} ${car.model} (${car.color})`);
}
```

## 用 default 變數，來代替 short circuiting or conditionals

```ts
// bad
function loadPages(count?: number) {
  const loadCount = count !== undefined ? count : 10;
  // ...
}

// good
function loadPages(count: number = 10) {
  // ...
}
```

## 利用 `enum` 來幫助註記 cpde
- `Enums` 可以像是 document，幫助表達 code 的意圖

```ts
// bad
const GENRE = {
  ROMANTIC: 'romantic',
  DRAMA: 'drama',
  COMEDY: 'comedy',
  DOCUMENTARY: 'documentary',
}

projector.configureFilm(GENRE.COMEDY);

class Projector {
  // declaration of Projector
  configureFilm(genre) {
    switch (genre) {
      case GENRE.ROMANTIC:
        // some logic to be executed
    }
  }
}

// good
enum GENRE {
  ROMANTIC,
  DRAMA,
  COMEDY,
  DOCUMENTARY,
}

projector.configureFilm(GENRE.COMEDY);

class Projector {
  // declaration of Projector
  configureFilm(genre) {
    switch (genre) {
      case GENRE.ROMANTIC:
        // some logic to be executed
    }
  }
}
```


## 關於 function 的參數
- 理想上不要超過 2 個
- 3 個以上變數，改用 `object`、用解構方法取值
  - 清楚辨別參數、不需要管順序
  - 用解構的話，ts 會警告出沒使用的變數

```ts
// bad
function createMenu(title: string, body: string, buttonText: string, cancellable: boolean) {
  // ...
}

createMenu('Foo', 'Bar', 'Baz', true);

// good
function createMenu(options: { title: string, body: string, buttonText: string, cancellable: boolean }) {
  // ...
}

createMenu({
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
});
```

## function 應該只做一件事情
```ts
// bad
function emailClients(clients: Client) {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}

// good
function emailClients(clients: Client) {
  clients.filter(isActiveClient).forEach(email);
}

function isActiveClient(client: Client) {
  const clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```

## function name 說明它的功能

```ts
// bad: 難辨識
function addToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();
addToDate(date, 1);

// good
function addMonthToDate(date: Date, month: number): Date {
  // ...
}

const date = new Date();
addMonthToDate(date, 1);
```

## Functions 應該只有一層的 abstraction
- 多層的話，難 reuse、難 test，應該要拆開

```ts
// bad
function parseCode(code: string) {
  const REGEXES = [ /* ... */ ];
  const statements = code.split(' ');
  const tokens = [];

  REGEXES.forEach((regex) => {
    statements.forEach((statement) => {
      // ...
    });
  });

  const ast = [];
  tokens.forEach((token) => {
    // lex...
  });

  ast.forEach((node) => {
    // parse...
  });
}

// good
const REGEXES = [ /* ... */ ];

function parseCode(code: string) {
  const tokens = tokenize(code);
  const syntaxTree = parse(tokens);

  syntaxTree.forEach((node) => {
    // parse...
  });
}

function tokenize(code: string): Token[] {
  const statements = code.split(' ');
  const tokens: Token[] = [];

  REGEXES.forEach((regex) => {
    statements.forEach((statement) => {
      tokens.push( /* ... */ );
    });
  });

  return tokens;
}

function parse(tokens: Token[]): SyntaxTree {
  const syntaxTree: SyntaxTree[] = [];
  tokens.forEach((token) => {
    syntaxTree.push( /* ... */ );
  });

  return syntaxTree;
}
```


(看到這邊，很多跟 js 的 clean code 建議一模一樣，跳過幾個)

## Avoid type checking
- Always prefer to specify types of variables, parameters and return values to leverage the full power of TypeScript features
  - 這會有助於重構

```ts
// bad
function travelToTexas(vehicle: Bicycle | Car) {
  if (vehicle instanceof Bicycle) {
    vehicle.pedal(currentLocation, new Location('texas'));
  } else if (vehicle instanceof Car) {
    vehicle.drive(currentLocation, new Location('texas'));
  }
}

// good
type Vehicle = Bicycle | Car;

function travelToTexas(vehicle: Vehicle) {
  vehicle.move(currentLocation, new Location('texas'));
}
```

## 使用 `protected` 和 `private`
```ts
// bad
class Circle {
  radius: number;

  constructor(radius: number) {
    this.radius = radius;
  }

  perimeter() {
    return 2 * Math.PI * this.radius;
  }

  surface() {
    return Math.PI * this.radius * this.radius;
  }
}

// good
class Circle {
  constructor(private readonly radius: number) {
  }

  perimeter() {
    return 2 * Math.PI * this.radius;
  }

  surface() {
    return Math.PI * this.radius * this.radius;
  }
}
```

## 偏好 immutability
- ts 有 `readonly`，有機會就可以使用

```ts
// bad
interface Config {
  host: string;
  port: string;
  db: string;
}

// good
interface Config {
  readonly host: string;
  readonly port: string;
  readonly db: string;
}
```

## type vs interface
- 沒有誰好誰壞，看情況使用

## class
- class 應該要小一點、拆分開

```ts
// bad
class Dashboard {
  getLanguage(): string { /* ... */ }
  setLanguage(language: string): void { /* ... */ }
  showProgress(): void { /* ... */ }
  hideProgress(): void { /* ... */ }
  isDirty(): boolean { /* ... */ }
  disable(): void { /* ... */ }
  enable(): void { /* ... */ }
  addSubscription(subscription: Subscription): void { /* ... */ }
  removeSubscription(subscription: Subscription): void { /* ... */ }
  addUser(user: User): void { /* ... */ }
  removeUser(user: User): void { /* ... */ }
  goToHomePage(): void { /* ... */ }
  updateProfile(details: UserDetails): void { /* ... */ }
  getVersion(): string { /* ... */ }
  // ...
}

// good
class Dashboard {
  disable(): void { /* ... */ }
  enable(): void { /* ... */ }
  getVersion(): string { /* ... */ }
}

// split the responsibilities by moving the remaining methods to other classes
```

## 優先使用 composition，沒辦法才用 inheritance

一些判斷思維
- 繼承的關係是「是一個 ...」，而不是「有一個 ...」
  - (人類 -> 動物 (人類是一種動物)，使用者 -> 使用者詳細資料(使用者，有詳細資料))

```ts
// bad
class Employee {
  constructor(
    private readonly name: string,
    private readonly email: string) {
  }

  // ...
}

// bod 是因為，「Employee 有 EmployeeTaxData」，而不是「Employee 是一個 EmployeeTaxData」
class EmployeeTaxData extends Employee {
  constructor(
    name: string,
    email: string,
    private readonly ssn: string,
    private readonly salary: number) {
    super(name, email);
  }

  // ...
}

// good
class Employee {
  private taxData: EmployeeTaxData; // 這樣 composition 就可以了

  constructor(
    private readonly name: string,
    private readonly email: string) {
  }

  setTaxData(ssn: string, salary: number): Employee {
    this.taxData = new EmployeeTaxData(ssn, salary);
    return this;
  }

  // ...
}

class EmployeeTaxData {
  constructor(
    public readonly ssn: string,
    public readonly salary: number) {
  }

  // ...
}
```

## 善用 function chain (這跟 ts 也沒有直接相關，但順便參考看看 ts 的寫法)

```ts
// bad
class QueryBuilder {
  private collection: string;
  private pageNumber: number = 1;
  private itemsPerPage: number = 100;
  private orderByFields: string[] = [];

  from(collection: string): void {
    this.collection = collection;
  }

  page(number: number, itemsPerPage: number = 100): void {
    this.pageNumber = number;
    this.itemsPerPage = itemsPerPage;
  }

  orderBy(...fields: string[]): void {
    this.orderByFields = fields;
  }

  build(): Query {
    // ...
  }
}

// ...

const queryBuilder = new QueryBuilder();
queryBuilder.from('users');
queryBuilder.page(1, 100);
queryBuilder.orderBy('firstName', 'lastName');

const query = queryBuilder.build();

// good
class QueryBuilder {
  private collection: string;
  private pageNumber: number = 1;
  private itemsPerPage: number = 100;
  private orderByFields: string[] = [];

  from(collection: string): this {
    this.collection = collection;
    return this;
  }

  page(number: number, itemsPerPage: number = 100): this {
    this.pageNumber = number;
    this.itemsPerPage = itemsPerPage;
    return this;
  }

  orderBy(...fields: string[]): this {
    this.orderByFields = fields;
    return this;
  }

  build(): Query {
    // ...
  }
}

// ...

const query = new QueryBuilder()
  .from('users')
  .page(1, 100)
  .orderBy('firstName', 'lastName')
  .build();
```


## Single Responsibility Principle (SRP)
- Clean Code, "There should never be more than one reason for a class to change"
  - 沒有很能體會這句話，可能要去看書才會懂，但這邊主要就是說，別讓一個東西負責多個事情

```ts
// bad
class UserSettings {
  constructor(private readonly user: User) {
  }

  changeSettings(settings: UserSettings) {
    if (this.verifyCredentials()) {
      // ...
    }
  }

  verifyCredentials() { // 這邊還負責了 auth 功能
    // ...
  }
}

// good
class UserAuth {
  constructor(private readonly user: User) {
  }

  verifyCredentials() {
    // ...
  }
}


class UserSettings {
  private readonly auth: UserAuth;

  constructor(private readonly user: User) {
    this.auth = new UserAuth(user); // auth 拆出去做
  }

  changeSettings(settings: UserSettings) {
    if (this.auth.verifyCredentials()) {
      // ...
    }
  }
}
```

## Open/Closed Principle (OCP)
- Bertrand Meyer 說 software entities (classes, modules, functions, etc.) 應該是
  - be open for extension
  - 但是 closed for modification
- 意思是，要讓 developer 能夠不修改程式，也能加入新的功能

```ts
// bad
class AjaxAdapter extends Adapter {
  constructor() {
    super();
  }

  // ...
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
  }

  // ...
}

class HttpRequester {
  constructor(private readonly adapter: Adapter) {
  }

  async fetch<T>(url: string): Promise<T> {
    if (this.adapter instanceof AjaxAdapter) {
      const response = await makeAjaxCall<T>(url);
      // transform response and return
    } else if (this.adapter instanceof NodeAdapter) {
      const response = await makeHttpCall<T>(url);
      // transform response and return
    }
  }
}

function makeAjaxCall<T>(url: string): Promise<T> {
  // request and return promise
}

function makeHttpCall<T>(url: string): Promise<T> {
  // request and return promise
}

// good
abstract class Adapter {
  abstract async request<T>(url: string): Promise<T>;

  // code shared to subclasses ...
}

class AjaxAdapter extends Adapter {
  constructor() {
    super();
  }

  async request<T>(url: string): Promise<T>{
    // request and return promise
  }

  // ...
}

class NodeAdapter extends Adapter {
  constructor() {
    super();
  }

  async request<T>(url: string): Promise<T>{
    // request and return promise
  }

  // ...
}

class HttpRequester {
  constructor(private readonly adapter: Adapter) {
  }

  async fetch<T>(url: string): Promise<T> {
    const response = await this.adapter.request<T>(url);
    // transform response and return
  }
}
```

(看不懂這邊的舉例...)
對 OO 沒功力
後面都是跟 OO 的比較相關
還有一些純粹 js 的

感覺跟 ts 沒有太多關係，就不繼續看了
