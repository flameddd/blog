# 2024-11-10: 筆記 Hannah Stryker 的 Linux Built-In Tools Are So Powerful, You Can Build a Database With Them.md

## Hannah Stryker

### https://www.howtogeek.com/build-a-database-with-powerful-linux-built-in-tools/

---

對 Linux 的 cli command 一直沒有很深入，看到這篇有興趣，所以讀讀。看看怎麼樣用 Linux command 來做類似 SQL 的事情

---

## 會用到哪些工具？

[Linux 有非常多實用的 commands](https://www.howtogeek.com/412055/37-important-linux-commands-you-should-know/) 接下來我們會用到這些:

- **grep**: 通過 input 進行搜尋並選擇 match 一個或多個 pattern 的 line
- **cut**: extracts 每行的選定部分並將它們寫入 standard output
- **awk**: 功能更強大的 pattern scanning and processing 工具
- **sort**: 對特定列進行排序
- **head/tail**:允許從 input 中 extract 一個 row 的切片
- **join**: 可以從多個案中建立連結

---

### 建立 Table

最簡單的結構化 text format 之一是 [DSV, or delimiter-separated values](https://en.wikipedia.org/wiki/Delimiter-separated_values)

- 這是種通用的 CSV —comma-separated values—format
- 在 Linux 上，structured text files 通常用空格或冒號 `:` 來分隔字
- `/etc/passwd` 檔案就是個經典範例

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_01.avif)

你可以用這種格式存許多種類型的資料，如 todo list:

```
Buy milk:2024-10-21:2:open
Call bank:2024-10-20:1:closed
```

你可以用 text editor 來更新你的 DB

- 這是 plain text 的另個優點

甚至可以用 command line 來 echo 的輸出到檔案來直接 add items:

```sh
echo "Take out the trash:$(date -I):3:open" > tasks
```

這相當於 SQL 的:

```sql
INSERT INTO tasks VALUES('Take out the trash', CURDATE(), '3', 'open')
```

---

### Fetch 整個 Table

Selecting data 是最常見的 DB task。基本情況是從 Table 中選擇所有內容，即:

```sql
SELECT * FROM tasks
```

這會從 DB 的每一 columns 中提取所有 row。對於基於文件的 DB，等效操作非常簡單:

```sh
cat tasks
```

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_02.avif)

---

### 用 `cut` 來 select 列 (Columns)

更進一步，可以將選擇範圍縮小到特定的 columns。SQL 中，這是這樣的:

```sql
SELECT task FROM tasks
```

用 `cut`:

- `-d` 指定分隔符(delimiter)
- `-f` 選項允許選擇特定 fields

```sh
cut -d':' -f1 tasks
```

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_03.avif)

---

### 用 `grep` 或 `awk` 選擇行 (row)

從 DB 中取每一 row

- 你通常會希望限制結果。根據值進行過濾是最常見的需求的要求

```sql
SELECT * FROM tasks WHERE status=open
```

在這情況，`grep` 是完美的替代品

- 它可以根據 regex match rows
- 如，可以找到所有狀態為 `"open"` 的任務

```sh
grep 'open$' tasks
```

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_04.avif)

這利用了 status 是每行最後一個字的情況

- `$` match 字串的尾

對位於中間的字，可能需要用更複雜的 regex。如

- 獲取所有優先級為 2 的 row

```js
grep ':2:[^:]*$' tasks
```

但 `grep` 只能 match text patterns，不能處理這樣更複雜的條件:

```sql
SELECT status, task FROM tasks WHERE date<2024-10-21
```

這個 SQL 用來取某個日期之前的任務，但開始超出了 `grep` 的能力

- 需要的是更強大的工具 `awk`:

```sh
awk -F':' '$2<"2024-10-21" {print $1 ":" $2 }' tasks
```

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_05.avif)

`Awk` 可以同時完成 `grep` 和 `cut` 的工作

- 在這個例子中，讀取的部分:

```
$2<"2024-10-21"
```

上面這是個先決條件，意味著只有日期較早的行才會 match

- 然後該命令會印出每行的前兩列

---

### 用 `tail` 和 `head` 做到 Pagination

SQL 的 LIMIT 允許取特定數量的結果。要取前兩行:

- 可以用 `tail` 來取最後 n 行
- 與 `head` 一起用，這允許模擬 `LIMIT`。例如，取第 `2-3` 行:

```sh
head -2 tasks
```

```sh
head -3 tasks | tail -2
```

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_06.avif)

---

### 用 sort 排序

`ORDER BY`: Linux 有個很棒的等效命令: `sort`

- 可以通過指定分隔符和數字，標誌用不同的字母
- 這次，`t` 指定分隔符，`k` 指定字段號:

```sh
sort -t':' -k2 tasks
```

這將按日期顯示所有記錄的排序結果:

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_07.avif)

---

### 用 `join` 來 join Tables 連接表

先來多加些資料，在原始 tasks file 中加一個新的姓名列，看起來像這樣:

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_08.avif)

接著新增 people file 來存 User 詳細資料:

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_09.avif)

現在可以用 `join` 並通過 `t` 選項指定分隔符:

```sh
join -t':' -1 5 -2 1 tasks people
```

`-1` 和 `-2` 分別指定要從每個文件中連接的第幾個字段

- 這裡，它們分別是第 `5` 和第 `1` 字段
- join 默認用第一個字段，因此可以將其簡化為:

```sh
join -t':' -1 5 tasks people
```

結果:

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_10.avif)

要清理輸出，可以 pipe 給 cut 並省略姓名:

```sh
join -t':' -1 5 tasks people | cut -d':' -f2-
```

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_11.avif)

可以用 `awk` 將兩個名字合併為一個:

```sh
join -t':' -1 5 tasks people | awk -F':' '{print $2":"$3":"$4":"$5":"$6" "$7}'
```

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_12.avif)

---

### 整合所有內容

最後，考慮個更加複雜的 SQL

- 這是個 join 兩個表以取名字，選擇特定列，並提取具有特定優先級的行
- 它按日期排序，最後只提取第一個匹配的行:

```sql
SELECT task, date, priority, status, first_name, last_name
FROM tasks t
LEFT JOIN people p ON t.name=p.name
WHERE priority=2
ORDER BY date
LIMIT 1
```

等效的命令 pipeline 可能難理解，一旦熟悉了這些核心工具，它也不會更複雜:

```sh
join -t':' -1 5 -2 1 tasks people \
  | awk -F':' '{print $2":"$3":"$4":"$5":"$6" "$7}' \
  | grep ':2:' \
  | sort -t ':' -k2 \
  | head -1
```

![alt text](assets/img/Linux_Built-In_Tools_Are_So_Powerful_You_Can_Build_a_Database_With_Them_13.avif)
