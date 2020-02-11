# try to find out useless javascript dependencies in package.json
(嘗試找出沒用到的 javascript library)

# use `depcheck` to find out useless javascript dependencies
not 100% correctly, but info is real useful
- https://www.npmjs.com/package/depcheck

I think `devDependencies` may not correct, cuz lot of them work in cli. They may not have `import`, `require` in any files.

```shell
npx depcheck
```

### screen shot
![img](/assets/img/depcheck_screenshot.jpg)  


# `npm-check` looks very useful too
Check for **outdated**, **incorrect**, and **unused** dependencies.  
- https://www.npmjs.com/package/npm-check

never use before, but I guess it command is work
```shell
npx npm-check
```

### screen shot
![img](https://i.stack.imgur.com/ldoRq.png)