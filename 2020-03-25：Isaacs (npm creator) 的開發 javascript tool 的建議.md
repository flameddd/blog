# Isaacs (npm creator) 的開發 javascript tool 的建議
## Mar 24, 2020  Isaacs (npm creator, CEO)
### https://twitter.com/izs/status/1242288484649283586

Isaacs: I could write 1000 pages on the details, if I had the time and inclination.

Some bullets tho:
- 100% test coverage from day 1
- abstract out the decisions you might want to revisit/replace
- keep the parts small
- loose coupling between parts
- don't be afraid to rewrite
- a loop and switch is a perfectly fine arg parser
- config files: 1 or zero
- For each new feature:
  - write a dirty hack of a script to try out an idea. Then a test. Then throw away the proof of concept and do it right.
- Shim the fuck out of the `fs` in tests. Monkey patching is fine. Just write tests, 100% coverage always.
- not done without docs and tests
- don't sweat performance early,
  - but DO sweat data structures and algorithmic complexity. Then measure perf with benchmarks.
  - Bad performance is a bug, and means something is doing too much stuff.
- to improve performance, do less stuff. Only do it once.

Every tool I've created
- I breaks at least one of these rules
  - usually many more than one.
- I've regretted breaking these rules most of the time, but not all of the time.
- I've always regretted breaking the 100% test coverage rule

more:
- Write pseudo code in markdown before writing js.
  - It's easier to see algorithmic problems in a bulleted list
  - and easier to throw away without feeling like you're losing something
  - and it means you get docs for free.

## 我有發問請教他

```
Hi, 
@izs
 
out of curiosity, did you _ALWAYS_ follow this rule:
- 「100% test coverage from day 1」

when build some tools or utils library？

(It's hard for me, bcuz I know there is chance to do some refactoring in the  near future) 
```

@izs 回答
> I didn't always. `But for the last few years, yes, I have.` Refactoring is easier and safer if you have full test coverage. Definitely do it first, **especially** if you have some refactoring coming!

順便提到他說他之前有個 talk  
- [Isaac Schlueter (npm)-99% is not enough-full test coverage and why it's awesome](https://www.youtube.com/watch?v=hJ4XHQHhmw8)

影片中就是有提到 `mutate-fs`
- https://github.com/isaacs/mutate-fs  
- 就是用來做上面提到的，假造你 fs 讀檔案失敗 (Monkey patching) or 各種狀態，來提高 test 品質。