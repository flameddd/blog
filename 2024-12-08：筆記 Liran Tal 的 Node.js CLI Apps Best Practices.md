# 2024-12-08：筆記 Liran Tal 的 Node.js CLI Apps Best Practices.md
## Liran Tal
### https://github.com/lirantal/nodejs-cli-apps-best-practices

---

對如何建立好的 CLI 工具有點興趣，意外看到這篇，快點來看看   

作者 Liran Tal
- 是長期在 NodeJS Security 領域的人，這篇也一定有顧慮到這些方面  
- 文章中似乎針對很多實作方面，都有推薦相應的 library 可以用

文章的內容雖然是以 NodeJS 為主，但其中關於 CLI 的經驗卻是完全通用的  
文章最後提到的其他參考資料，[https://clig.dev/](https://clig.dev/) 看起來超好料的，讓人感覺，做 CLI 就是必讀的程度   

---

## 1 Command Line Experience
This section deals with best practices concerned with creating beautiful and high-value user experience Node.js command line applications.

In this section:

1.1 Respect POSIX args
1.2 Build empathic CLIs
1.3 Stateful data
1.4 Provide a colorful experience
1.5 Rich interactions
1.6 Hyperlinks everywhere
1.7 Zero configuration
1.8 Respect POSIX signals

### 1.1 Respect POSIX args
✅ Do: Use POSIX-compliant command line argument syntax, which is widely accepted as a standard for command line tools.

❌ Otherwise: Users may get frustrated when a CLI's syntax for arguments, options, or command parameters deviate from the de facto Unix standards they are used to.

ℹ️ Details

Unix-like operating systems popularized the use of the command line and tools such as awk, sed. Such tools have effectively standardized the behavior of command line options (aka flags), options-arguments, and other operands.

Some examples of expected behavior:
- option-arguments or options can be notated in help or examples as square brackets (`[]`) to indicate they are optional, or with angle brackets (`<>`) to indicate they are required.
- allow short-form single letter arguments as aliases for long-form arguments (see reference from the [GNU Coding Standards](https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html)).
- options specified using the short form singular `-` may contain one alphanumeric character.
- specifying multiple options with no values may be grouped, such as `myCli -abc` being equivalent to `myCli -a -b -c`.

Command line power-users will expect your command line application to have similar conventions as other Unix apps.

📦 Recommended packages

Reference to Open Source Node.js packages:
- commander: https://github.com/tj/commander.js#readme
- yargs: https://github.com/yargs/yargs

### 
1.2 Build empathic CLIs
✅ Do: Put workflows in place that assist the user to interact with the CLI successfully, when otherwise such interactions would result in errors and frustration.

❌ Otherwise: Failing to provide actionable assistance in supporting the user will result in frustration due to the lack of capability to operate the CLI.

ℹ️ Details

A command line interface for your program is no different than a web user interface in the sense of doing as much as you can as the program author to ensure that it is being used successfully.

Optimize for successful interactions by building empathic CLIs that support the user. As an example, let's explore the case of the curl program that expects a URL as its primary data input, and the user failing to provide it. Such failure will lead to reading through a (hopefully) descriptive error messages or reviewing a curl --help output. However, an empathic CLI would have presented an interactive prompt to capture input from the user, resulting in a successful interaction.

### 1.3 Stateful data
✅ Do: Provide a stateful experience between multiple invocations of your CLI app, and remember values and data in order to provide seamless interaction.

❌ Otherwise: Requiring your user to repeatedly provide the same information with multiple invocations of the CLI will annoy your user.

ℹ️ Details

It may happen that you find yourself needing to provide storage persistence for your CLI application, such as remembering a username, email, API token, or other preferences between multiple invocations of the CLI. Use a configuration helper that allows the app to persist such user settings. Be sure to follow the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/latest/) when reading/writing files (or choose a configuration helper that respects the spec). These keeps the user in control of where files are written and managed.

Reference projects:
- configstore: https://www.npmjs.com/package/configstore
- conf: https://www.npmjs.com/package/conf

### 1.4 Provide a colorful experience
✅ Do: Make use of colors in your CLI application to highlight parts of your app's output, and provide a graceful degradation, or color detection, to allow automatic opt-out so that output isn't garbled. Ensure manual opt-in and opt-out is possible via CLI option, environment variable, and/or config file.

❌ Otherwise: Information may easily get lost in pale program output, especially when the output is text-heavy.

ℹ️ Details

Most terminals used today to interact with command line applications support colored text such as these enabled by specially crafted ANSI encoded characters.

A colorful display in your command line application output may further contribute to a richer experience and increased interaction. That said, unsupported terminals may experience a degraded output in the form of garbled information on the screen. Furthermore, a CLI may be used in a continuous integration build job which may not support colored output. Even outside of build servers, a CLI may be used through an IDE's console that may not handle certain characters. Manual opt-out must be available.

📦 Recommended packages

Reference to Open Source Node.js packages:
- chalk:https://www.npmjs.com/package/chalk
- colors: https://www.npmjs.com/package/colors
- kleur: https://www.npmjs.com/package/kleur

---------

### 1.5 Rich interactions
✅ Do: Leverage the use of rich command line interactions beyond the basics of text input prompt to provide a smoother experience for CLI users.

❌ Otherwise: A text prompt as input may prove cumbersome for users when data to reason about is in the form of closed options (i.e: dropdowns).

ℹ️ Details

Rich interactivity can be introduced in the form of prompt inputs, which are more sophisticated than free text, such as dropdown select lists, radio button toggles, rating, auto-complete, or hidden password inputs.

Another type of rich interactivity is in the form of animated loaders and progress-bars which provide a better experience for users when asynchronous work is being performed.

Many CLIs provide default command line arguments without requiring any further interactive experience. Don't force your users to provide parameters that the app can work out for itself.

📦 Recommended packages

Reference to Open Source Node.js packages:

- enquirer: https://www.npmjs.com/package/enquirer
- ora: https://www.npmjs.com/package/ora
- ink: https://www.npmjs.com/package/ink
- prompts: https://www.npmjs.com/package/prompts


----------

### 1.6 Hyperlinks everywhere
✅ Do: Use properly formatted hyperlinks in text output for both URLs (e.g: `https://www.github.com`), as well as source code (e.g: `src/Util.js:2:75`) - both of which a modern terminal is able to transform into a clickable link.

❌ Otherwise: Avoid broken and non-interactive links like git.io/abc which requires your user to copy and paste manually.

ℹ️ Details

If you are sharing links to URLs, or pointing to a file and a specific line number and column in the file, you can provide properly formatted links to both of these examples that, once clicked, will open up the browser, or an IDE at the defined location.

Reference projects:
- open: https://github.com/sindresorhus/open

--------

### 1.7 Zero configuration
✅ Do: Optimize a plug-and-play experience by auto-detecting required configuration and command line arguments values.

❌ Otherwise: Don't force user interactivity if a command-line argument can be auto-detected in a reliable way, and the action invoked doesn't explicitly require user interaction (such as confirming a deletion).

ℹ️ Details

Aim to provide a "works out of the box" experience when running the CLI application. For example, [POSIX defines a standard for environment variable configuration](https://pubs.opengroup.org/onlinepubs/009695399/basedefs/xbd_chap08.html) used for different purposes, such as: `TMPDIR`, `NO_COLOR`, `DEBUG`, `HTTP_PROXY` and others. Detect these automatically and prompt for confirmation when necessary.

Reference projects which are built around Zero configuration:

- The [Jest JavaScript Testing Framework](https://jestjs.io/)
- [Parcel](https://parceljs.org/), a web application bundler


--------

### 1.8 Respect POSIX signals
✅ Do: Ensure your program respects [POSIX signals](https://man7.org/linux/man-pages/man7/signal.7.html) to allow it proper interaction with users or other programs.

❌ Otherwise: Your program will not play well with other programs and introduce unexpected behavior.

ℹ️ Details

Especially for CLI applications, it is common to interact with user input and improperly managing keyboard events may result in your app failing to respond to SIGINT interrupts, commonly used by users when they hit the `CTRL+C` keys.

The problem of not respecting process signals worsens when the program is being orchestrated by non-human interaction. For example, a CLI that runs in a docker container but will not respond to software interrupt signals sent to it.


---

## 2 Distribution
This section deals with best practices concerned with distributing and packaging a Node.js command line application in an optimal matter for consumers.

In this section:
- 2.1 Prefer a small dependency footprint
- 2.2 Use the shrinkwrap, Luke
- 2.3 Cleanup configuration files

--------


### 2.1 Prefer a small dependency footprint
✅ Do: Minimize your use of production dependencies, use alternative dependencies which are smaller, and vet your dependencies' footprints as well for transitive dependencies to ensure a small bundle of the Node.js CLI. Balance this by being careful to not over-optimize use of dependencies by reinventing the wheel.

❌ Otherwise: The size and use of dependencies in the application will impact the install time of your Node.js CLI, potentially providing a poor user experience.

ℹ️ Details

A fast `npm install` for Node.js CLIs invoked with `npx` will provide a better user experience. This is made possible when the overall dependency, and transitive dependency, footprint is kept to a reasonable size.

A one-off global `npm` installation of a slow-to-install npm package will offer a one-off poor experience, but the use of npx by users to invoke executable packages will make the degraded performance, due to npx always fetching and installing packages from the registry, more significant and obvious.

Reference projects:
- [Bundlephobia](https://bundlephobia.com/) is a tool to help you find the cost of a npm package.



-----

### 2.2 Use the shrinkwrap, Luke
✅ Do: Use npm's npm-shrinkwrap.json as a lockfile to ensure that pinned-down dependency versions (direct and transitive) propagate to your end users when they install your npm package.

❌ Otherwise: Not fixing the versions of your app's dependencies will mean that the package manager (e.g. npm) will resolve them during installation, and transitive dependencies installed via version ranges may introduce breaking changes that you can't control, that may result in your Node.js CLI application failing to build or run.

ℹ️ Details

Use the  shrinkwrap, Luke!

Typically, an npm package only defines its direct dependencies, and their version range, when being installed, and the npm package manager will resolve all the transitive dependencies' versions upon installation. Over time, the resolved versions of dependencies will vary, as new direct, and transitive dependencies will release new versions.

Even though [Semantic Versioning](https://semver.org/) is broadly accepted among maintainers, npm is [known to introduce many dependencies](https://snyk.io/blog/how-much-do-we-really-know-about-how-packages-behave-on-the-npm-registry/) for an average package being installed, which adds to the risk of a package introducing changes that may break your application.

The flip side of using `npm-shrinkwrap.json` is the security implications you are forcing upon your consumers. The dependencies being installed are pinned to specific versions, so even if newer versions of these dependencies are released, they won't be installed. This moves the responsibility to you, the maintainer, to stay up-to-date with any security fixes in your dependencies, and release your CLI application regularly with security updates. Consider using `Snyk Dependency Upgrade` to automatically fix security issues across your dependency tree. Full disclosure: I am a developer advocate at Snyk.

> 👍 Tip
> Use `npm shrinkwrap` command to generate the shrinkwrap lockfile, which is of the same format as that of a `package-lock.json` file.

Another method for vendoring dependencies is to bundle them within the published package, which has the advantage of speeding up installations as it reduces the need to resolve dependencies as well as network requests and bandwidth for download, yet it comes with the disadvantages of being an opaque box for which it is difficult to analyze the dependency tree of the project and result in security tools like Snyk, not reporting vulnerabilities (because Snyk ignores devDependencies by default, to reduce noise for developers)

Packages are declared as `devDependencies`, so that the package managers will not find any production dependencies to install.
The [ncc](https://www.npmjs.com/package/@vercel/ncc) is used to compile a Node.js module into a single file with all of its dependencies in-lined.
References:

Do you really know how a lockfile works for yarn and npm packages?
Yarn docs: Should lockfiles be committed to the repository?


------

### 2.3 Cleanup configuration files
✅ Do: Cleanup configuration files when the CLI application gets uninstalled. Optionally, CLI applications can prompt your users to keep the configuration files to skip the re-initialising phase on the next installation for a better user experience.

❌ Otherwise: The user's file system may contain residue in the form of orphaned configuration files, and identifiable data which the CLI tool introduced when installed.

ℹ️ Details

As mentioned in the stateful data section, if your CLI application uses persistent storage, such as to save configuration files, then the CLI application should also be responsible for removing its configuration files when it gets uninstalled.

Due to npm package manager not providing uninstall hook since npm v7, your program should include an uninstallation option, either via arguments (e.g. --uninstall) or via rich interaction.

> Tip
> Optionally you can provide `pre` and `post` uninstall [script](https://docs.npmjs.com/cli/v10/using-npm/scripts) which will be automatically called if the npm version is 6 or lower. You can find a working [example in this repository](https://github.com/m-sureshraj/jenni/blob/master/src/scripts/pre-uninstall.js).

---

## 3 Interoperability
This section deals with best practices concerned with making your Node.js CLI seamlessly integrate with other command line tools, and following conventions that are natural for CLIs to operate in.

This section answers questions such as:

Can I export the output of this CLI for easy parsing?
Can I pipe the output of this CLI to the input of another command line tool?
Can I pipe the result of another tool to this CLI?
In this section:
- 3.1 Accept input as STDIN
- 3.2 Enable structured output
- 3.3 Cross-platform etiquette
- 3.4 Support configuration precedence

------

###  3.1 Accept input as STDIN
✅ Do: For command line applications that are expected to work with data, make it easy for a user to pipe the data to standard input (STDIN).

❌ Otherwise: Other command line applications will not be able to provide their result, directly as input for your CLI, which prevents common UNIX one-liners such as:

```sh
$ curl -s "https://api.example.com/data.json" | your_node_cli
```

ℹ️ Details

If the command line application works with data, such as performing some kind of task on a JSON file that is usually specified with `--file` `<file.json>` command line argument.

An example that is based on the official [Node.js API docs for the for readline module](https://nodejs.org/api/readline.html) of how taking input from a command pipe is as follows:

```js
const readline = require("readline");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

rl.question("What do you think of Node.js? ", (answer) => {
  // TODO: Log the answer in a database
  console.log(`Thank you for your valuable feedback: ${answer}`);

  rl.close();
});
```

Then pipe the input to the above Node.js application:

```sh
echo "Node.js is amazing" | node cli.js
```


------

###  3.2 Enable structured output
✅ Do: Enable a flag to allow structured output of the app's result, if such result is available, to enable parsing and easy manipulation of the data.

❌ Otherwise: Users of the CLI may need to apply complicated regex parsing and matching techniques to extract the output data provided by your CLI.

ℹ️ Details

It is often useful for users of a command line application to parse the data and perform other tasks with it, such as using it to feed web dashboards, or email notifications.

Being able to easily extract the data of interest from a command line output provides a friendlier experience to users of your CLI.

------

###  3.3 Cross-platform etiquette
✅ Do: If a CLI is expected to function across platforms, proper attention must be given to semantics of command shells, and components such as file systems, along with developers leveraging relevant programming constructs.

❌ Otherwise: The CLI will break on other operating systems due to factors such as incorrect path separator characters, even though there are no functional differences in the code.

ℹ️ Details

Even though, from a program's perspective, the functionality isn't being stripped down and should execute well in different operating systems, some missed nuances may render the program inoperable. Let's explore several cases where cross-platform ethics must be honored.

Wrongly spawning a command
You might need to spawn a process that runs a Node.js program. For example, you have the following script:

`program.js`:  
```bash
#!/usr/bin/env node

// the rest of your app code
```

This works:
```js
const cliExecPath = 'program.js'
const process = childProcess.spawn(cliExecPath, [])
```

This is better:

```js
const cliExecPath = 'program.js'
const process = childProcess.spawn('node', [cliExecPath])
```

Why is it better? The `program.js` source code begins with the Unix-like [Shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) notation, however Windows doesn't know how to interpret this due to the fact that Shebang isn't a cross-platform standard.

This is also true for `package.json` scripts. Consider the following bad practice of defining an npm run script:

```json
"scripts": {
  "postinstall": "myInstall.js"
}
```
This is due to the fact that the Shebang in `myInstalls.js` will not help Windows understand how to run this with the node interpreter.

Instead, follow the best practice of:

```json
"scripts": {
  "postinstall": "node myInstall.js"
}
```
Shell interpreters vary
Not all characters are treated the same across different shell interpreters.

For example, the Windows command prompt doesn't treat a single quote the same as a double quote, as would be expected on a bash shell, and so it doesn't recognize that the whole string inside a single quote belongs to the same string group, which will lead to errors.

This will fail in a Windows Node.js environment that uses the Windows command prompt:

`package.json`:
```json
"scripts": {
  "format": "prettier-standard '**/*.js'",
  ...
}
```

To fix this so that this `npm run` script will indeed be cross-platform between Windows, macOS and Linux:

`package.json`:
```json
"scripts": {
  "format": "prettier-standard \"**/*.js\"",
  ...
}
```

In this example we had to use double quotes and escape them with the JSON notation.

Avoid concatenating paths
Paths are constructed differently across different platforms. When they are built manually by concatenating strings they are bound not to be interoperable between different platforms.

Let's consider the following bad practice example:

```js
const myPath = `${__dirname}/../bin/myBin.js`;
```

It assumes that forward slash is the path separator where on Windows, for example, a backslash is used.

Instead of manually building filesystem paths, defer to Node.js's own `path` module to do this:

```js
const myPath = path.join(__dirname, "..", "bin", "myBin.js");
```

Avoid chaining commands with semicolons
Linux shells are known to support a semicolon (`;`) to chain commands to run sequentially, such as: `cd /tmp; ls`. However, doing the same on Windows will fail.

Avoid doing the following:

```js
const process = childProcess.exec(`${cliExecPath}; ${cliExecPath2}`);
```

Instead, use the double ampersand or double pipe notations:

```js
const process = childProcess.exec(`${cliExecPath} || ${cliExecPath2}`);
```

------

###  3.4 Support configuration precedence
✅ Do: Allow configuration to be obtained from multiple sources, by order of precedence. Command line arguments take highest priority, followed by shell variables, and then different levels of configuration.

❌ Otherwise: Users face frustration when customizing their experience with the CLI.

ℹ️ Details

Detect and support configuration setting using environment variables as this will be a common way in many toolchains to modify the behavior of the invoked CLI application.

Configuration order of precedence for command line applications should follow this:

- Command line arguments specified when the application is invoked.
- The spawned shell's environment variables, and any other environment variables available to the application.
- The project scope configuration, e.g: a local directory `.git/config` file.
- The user scope configuration, e.g: the user's home directory configuration file: `~/.gitconfig` or its XDG equivalent: `~/.config/git/config`.
- The system scope configuration, e.g: `/etc/gitconfig`.

Reference projects:
- cosmiconfig: https://github.com/cosmiconfig/cosmiconfig

---

## 4 Accessibility
This section deals with best practices concerned with making a Node.js CLI application available to users who wish to consume it, but who are lacking the environment for which the maintainer designed the application.

In this section:
- 4.1 Containerize the CLI
- 4.2 Graceful degradation
- 4.3 Node.js versions compatibility
- 4.4 Shebang autodetect the Node.js runtime


--------

### 4.1 Containerize the CLI
✅ Do: Create a docker image for the CLI and publish it to a public registry like Docker Hub so that users without a Node.js environment can use it.

❌ Otherwise: Users without a Node.js environment will not have npm or npx available, and so won't be able to run your CLI application.

ℹ️ Details

Installing Node.js CLI applications from the npm registry will typically be done with Node.js native toolchain such as `npm` or `npx`. These are common across JavaScript and Node.js developers, and are expected to be referenced within install instructions.

However, if you are targeting a CLI application to be consumed by the general public, regardless of their familiarity with JavaScript, or availability of this tooling, then distributing your CLI application only in the form of an install from the npm registry will be restricting. If your CLI application is intended to be used in a build or CI environment then those may also be required to install Node.js related toolchain dependencies.

There are many ways to package and distribute an executable, and containerizing it as a Docker container that is pre-bundled with your CLI application is an easily consumable alternative and dependency-free (aside of requiring a Docker environment).

--------

### 4.2 Graceful degradation
✅ Do: Provide users with the ability to opt-out of colorful and animation-rich display in unsupported environments, such as by skipping interactivity and providing formatted output in the form of JSON.

❌ Otherwise: Having colorful output, using terminal interactive such as prompts and other display-rich interfaces may significantly degrade the end user experience for users not having a supported terminal and deter them away from using your CLI application.

ℹ️ Details

It is common to provide a rich terminal display in the form of colorful output, ascii charts, or even animation on the terminal and powerful prompt mechanism. These may contribute to a great user experience for those who have a supported terminal, however it may display garbled text or be completely inoperable for those without it.

To enable users with an unsupported terminal to properly use the Node.js CLI application, you may choose to:

- Auto-detect a terminal capability and evaluate during run-time whether to degrade the CLI interactivity
- Provide an opt-in for users to explicitly toggle a graceful degradation, such as by providing a `--json` command line argument to force it to output raw data.

> 👍 Tip
> Supporting graceful degradation such as JSON output isn't only useful for end-users and their unuspported terminals, but is also valuable for running in continuous integration environments, as well as enabling users the ability to connect your program's output with other program's input or export data if needed.



--------

### 4.3 Node.js versions compatibility
✅ Do: Target supported and maintained [Node.js versions](https://nodejs.org/en/about/previous-releases).

❌ Otherwise: A code-base that tries to stay compatible with older and unsupported Node.js versions will be difficult to maintain, and lose the benefits of language and runtime features.

ℹ️ Details

Sometimes it may be necessary to specifically target older Node.js versions that are missing new ECAMScript features. For example, if you are building a Node.js CLI that is mostly geared towards DevOps or IT, they may not have an ideal Node.js environment with an up to date runtime. As a reference, Debian Stretch (oldstable) ships with [Node.js 8.11.1](https://packages.debian.org/search?suite=default&section=all&arch=any&searchon=names&keywords=nodejs).

If you do need to target old versions of Node.js such as Node.js 8, 6 or 4, all of which are End of Life, prefer to use a transpiler such as Babel to make sure the generated code is compliant with the version of the V8 JavaScript engine and the Node.js run-time shipped with those versions.

Another workaround is to provide a containerized version of the CLI to avoid old targets. See Section [(4.1) Containerize the CLI](g).

Don't dumb down your program code to use an older ECMAScript language specification that matches unmaintained or EOL Node.js versions as this will only lead to code maintenance issues.

If the CLI is invoked in an unsupported environment, attempt to detect it and exit with a descriptive error message to present a friendly and information error message. See [this example](https://github.com/lirantal/dockly/blob/42d8c09631bc5348f108a50c3ce9601851fb760b/index.js#L25) for dockly.



--------

### 4.4 Shebang autodetect the Node.js runtime
✅ Do: Use an installation-agnostic reference in your [Shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) declaration that locates the Node.js runtime automatically based on the runtime environment, such as #!/usr/bin/env node.

❌ Otherwise: Using a hard-coded Node.js runtime location such as `#!/usr/local/bin/node` is only specific to your own environment and may render the Node.js CLI inoperable in other environments where the location of Node.js is different.

ℹ️ Details

It may be an easy start to develop a Node.js CLI by running the entry point file as `node cli.js`, and later on adding `#!/usr/local/bin/node` to the top of the `cli.js` file, however the latter is still a flawed approach as that location of the node executable is not guaranteed for other users' environments.

It should be noted that specifying `#!/usr/bin/env node` as the best practice, is still making the assumption that the Node.js runtime is referenced as `node` and not `nodejs` or otherwise.




---

## 5 Testing
In this section:
- 5.1 Put no trust in locales

------

### 5.1 Put no trust in locales
✅ Do: Don't assume output text to be equivalent to a string you assert for because tests may run on systems with different locales than yours, or the English default.

❌ Otherwise: Developers will experience test failures as they test on systems with different locales than the English default.

ℹ️ Details

As you choose to test the CLI by running it and parsing output, you may be inclined to grep for specific features to ensure that they exist in the output such as properly providing examples when the CLI is ran with no arguments. e.g:

```js
const output = execSync(cli);
expect(output).to.contain("Examples:"));
```

When tests will run on locales that aren't English-based, and if your CLI argument parsing library supports auto-detection of locales and adopting to it, then tests such as this will fail, due to language conversions from `Examples` to the locale-equivalent language being set as the default locale in the system.






---

## 6 Errors
This section deals with best practices concerned with making a Node.js CLI application available to users who wish to consume it but are lacking an ideal environment for which the maintainer designed the application.

In essence, the goals of the best practices laid out in this section is to help users troubleshoot errors quickly and easily, without needing to consult documentation or source code to understand errors.

In this section:
- 6.1 Trackable errors
- 6.2 Actionable errors
- 6.3 Provide debug mode
- 6.4 Proper use of exit codes
- 6.5 Effortless bug reports


------

###  6.1 Trackable errors
✅ Do: When reporting errors, provide trackable error codes that can be looked up in the project's documentation and simplify troubleshooting the error message.

If possible, extend trackable error codes with further information so these can be easily parsed and context is clear.

❌ Otherwise: Generic error messages tend to be ambiguous and make it hard for users to search for solutions. Parsing and analyzing isn't as straight-forward, and referencing them in documentation is not as clean either.

ℹ️ Details

Ensure that, when error messages are returned, they include a reference number or specific error codes that can later be consulted. Much like HTTP status codes, so to do CLI applications require named or coded errors.

Example:

```sh
$ my-cli-tool --doSomething

Error (E4002): please provide an API token via environment variables
```

------

### 6.2 Actionable errors
✅ Do: A failing error message should tell the user what is required as a fix, rather than complaining that there is an error.

❌ Otherwise: Users facing error messages, with no hint of the action to resolve the error, may not be able to successfully use your CLI app.

ℹ️ Details

Example:
```sh
$ my-cli-tool --doSomething

Error (E4002): please provide an API token via environment variables 
```



------

###  6.3 Provide debug mode
✅ Do: Allow power users to enable more detailed information if they need to diagnose problems.

❌ Otherwise: Don't skip debugging capabilities. It will be harder to collect feedback from users, and for them to pinpoint the cause of errors.

ℹ️ Details

Use environment variables as well as command line arguments to enable extended debug verbosity levels. Where it make sense in your code, plant debug messages that aid the user, and the maintainer, to understand the program flow, inputs and outputs, and other pieces of information that make problem solving easier.

📦 Recommended packages

Reference to Open Source Node.js packages:
- debug: https://www.npmjs.com/package/debug




------

###  6.4 Proper use of exit codes
✅ Do: Terminate your program with proper exit codes that convey a semantic meaning of the error or exit status.

❌ Otherwise: An incorrect or missing exit code will impede the use of your CLI in a continuous integration flows and other command line scripting use-cases.

ℹ️ Details

Command line scripts often make use of the shell's `$?` to infer a program's status code and act upon it. This is also utilized in continuous integration (CI) flows to determine whether a step completed successfully or not.

If your CLI always terminates with no specific status code, even on errors, then the shell and other programs that rely upon it have no way of knowing this. When an error happens that results in your program's termination, you should convey this meaning. For example:

```js
try {
  // something
} catch (err) {
  // cleanup or otherwise
  // then exit with proper status code
  process.exit(1);
}
```


A short reference for exit codes:
- exit code 0 conveys a successful execution
- exit code 1 conveys a failure

You may also choose to use customized exit codes with semantics of your program, but if you do, be sure to document them properly.

Reference: A [list of exit codes used by the BASH shell](http://www.tldp.org/LDP/abs/html/exitcodes.html)




------

###  6.5 Effortless bug reports
✅ Do: Make it an effortless task to submit bug reports by providing a URL to open an issue and prepopulating the required data as much as possible. [Issue templates, like on GitHub](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository), allow to further guide the users as to which information is necessary.

❌ Otherwise: Users get frustrated searching for how to report a bug and may end up with little helpful information, or not submitting an issue at all.








---

## 7 Development
This section deals with development and maintenance best practices of building a Node.js command line application.

In this section:
- 7.1 Use a bin object
- 7.2 Use relative paths
- 7.3 Use the files field


-------

### 7.1 Use a bin object
✅ Do: Use an object to define the name of the executable and its path.

❌ Otherwise: You will end up coupling the name of the package with the executable.

ℹ️ Details

The following `package.json` shows an example of decoupling the name of the executable from the filename and its location in the project:

```json
  "bin": {
    "myCli-is-cool": "./bin/myCli.js"
  }
```

-------

###   7.2 Use relative paths
✅ Do: Use `process.cwd()` to access user input paths and use `__dirname` to access project-based paths.

❌ Otherwise: You will end up with incorrect file paths and won't be able to access files.

ℹ️ Details

You may find yourself with the need to access files within the project's files scope, or to access files that are provided from the user's input, such as log, JSON files or others. Confusing the use of `process.cwd()` or `__dirname` can lead to errors, as well as not using neither of them.

How to properly access files:
- `process.cwd()`: use it when the file path that you need to access depends on the relative location of the Node.js CLI. A good example for this is when the CLI supports file paths to create logs, such as: `myCli --outfile ../../out.json`. If myCli is installed in `/usr/local/node_modules/myCli/bin/myCli.js` then `process.cwd()` will not refer to that location, but rather to the current working directory, which is whichever the directory the user is at when the CLI was invoked.
- `__dirname`: use it when you need to access a file from within the CLI's source code and refer to a file from the relevant location of the file which the code lies in. For example, when the CLI needs to access a JSON data file in another directory: `fs.readFile(path.join(__dirname, '..', 'myDataFile.json'))`.



-------

###   7.3 Use the files field
✅ Do: Use `files` field to only include necessary files in your published packages.

❌ Otherwise: You will end up with a package that contains files that may not be needed to run your CLI application. e.g. (test files, development configurations, etc.)

ℹ️ Details

To keep the published package size small, we should only include files that are required to run our CLI application. See this [post](https://nodejs.medium.com/publishing-npm-packages-c4c615a0fc6b) for more details.

The following files field tells the npm CLI to include all the files inside the src directory except the spec files.

```json
"files": [
  "src",
  "!src/**/*.spec.js"
],
```

 



---

## 8 Analytics
This section deals with analytics collections in Node.js command line applications.

In this section:
- 8.1 Strict Opt-in Analytics

-------

### 8.1 Strict Opt-in Analytics
✅ Do: Always prompt, ask, or opt-in users in an explicit way to submit usage and product analytics to a remote location.

❌ Otherwise: You risk privacy concerns for users and surprising CLI behavior which users wouldn't expect.

ℹ️ Details

Understandably, as a maintainer of a CLI application you would want to understand better how users are using it. However, stealthly and by-default "phone home" type of behavior without asking consent from users will be frawned upon.

Guidelines:
- Let the users know which data will be collected and what are you doing with it.
- Be mindful about privacy concerns and collecting potentially personal identifyable information.
- How, where and for which period of time is data stored.

References for other CLIs which collect analytics are [Angular CLI](https://v17.angular.io/cli/analytics), and [Next.js CLI](https://nextjs.org/telemetry).

---

## 9 Versioning
Following these best practices for version information in Node.js CLI apps enhances user experience, facilitates debugging, and ensures efficient project management.

In this section:
- 9.1 Include a --version Flag
- 9.2 Use Semantic Versioning
- 9.3 Provide Version Information in a 'package.json' file
- 9.4 Display Version in Error Messages and Help Text
- 9.5 Backward Compatibility
- 9.6 Publish Versioned Releases on npm
- 9.7 Update Your App's Version Documents



--------

###  9.1. Include a `--version` Flag
✅ Do: Use a proper flag to allow users to check the application's version easily.

❌ Otherwise: Users won't know which version they are using, making it challenging to track updates or report issues accurately.

ℹ️ Details

As the maintainer of the CLI project, you need to implement a functionality to display the program's version on standard output with the proper flag without any argument, and exit from the program(often also prints compiled-in configuration details as well). You can also use the `-V` as the short flag.

Example:
```sh
$ my-cli-tool --version

my-cli-tool version 11.0.5, build 11.0.5-0ubuntu1~22.04.1
```

--------

###  9.2. Use Semantic Versioning
✅ Do: Like other projects, use the SemVer format. It provides a clear, standardized way to convey version information, making it easier for users to understand the significance of updates.

❌ Otherwise: Users might not understand the impact of updates, leading to confusion or unexpected behavior.



--------

###  9.3. Provide Version Information in a 'package.json' file
✅ Do: Try to add version information in `package.json` file because this ensures consistency across your project and makes it easy to automate version updates.

❌ Otherwise: You and your program's users will face difficulty in tracking dependencies and managing the project's version.

ℹ️ Details

Tools like `npm` or `yarn` offer version management features that simplify handling dependencies and versioning.

> 👍 Tip
> Use the option of automating version increments that these tools bring, (e.g., npm version) to reduce the risk of human error and streamline the release process



--------

###  9.4. Display Version in Error Messages and Help Text
✅ Do: Provide the version in error messages so that users can include version information when reporting issues, aiding debugging and support.

❌ Otherwise: Debugging becomes more challenging, and users might not know which version to reference when seeking help or even when they use your program as a dependency on other programs.





--------

###  9.5. Backward Compatibility
✅ Do: Ensuring backward compatibility with older versions of your app allows users to upgrade without breaking their workflows.

❌ Otherwise: Frequent updates that break compatibility can frustrate users and hinder adoption.

ℹ️ Details

Ensure that, even when you remove any functionality or change the format of input/outputs, you come with some description for the user.

> 👍 Tip
> Use the document pages to provide an overview of features that are deprecated or soon going to be deprecated and their replacements. Display a message to your users and ask them to learn about the end of support for features by referring to the release notes.

Example:
```sh
DEPRECATED: The blah-blah-feat is deprecated and will be removed in a future release.
            Install the blah-feat component to do the blah:
            <https://url/to/docs>
```



--------

###  9.6. Publish Versioned Releases on npm
✅ Do: Publish your app on npm with version tags enabling users to easily install specific versions.

❌ Otherwise: Users can't access previous versions, which might be necessary for compatibility or troubleshooting.





--------

###  9.7. Update Your App's Version Documents
✅ Do: Inform users about changes, enhancements, and bug fixes in each version by providing clear release notes; A changelog helps users understand what has changed between versions and whether it affects their use case.

❌ Otherwise: Users won't know what to expect in new versions, which can lead to frustration or confusion. They also may struggle to assess whether they should upgrade or not.





---

## 10 Security
This section deals with security concerns when developing Node.js command line applications.

In this section:
- 10.1 Minimize Argument Injection

-------

### 10.1 Minimize Argument Injection
✅ Do: Carefully consider which command-line arguments are enabled by your CLI and which commands they are open to. If possible, avoid sensitive system tasks such as file system read/write.

❌ Otherwise: You risk attackers exploiting command-line argument flags in your CLI to facilitate attack vectors such as file read/write, command execution, and others.

ℹ️ Details

Argument injection attacks take advantage of vulnerabilities in how command-line applications parse user input. They happen when untrusted user input gets included as part of a command that the application then executes. In argument injection, attackers specially craft the input used as arguments and parameters in the command in order to carry out malicious actions or access unauthorized data.

Prior-art of security incidents in CLIs due to argument injection:

- Vulnerability in [git-interface](https://security.snyk.io/vuln/SNYK-JS-GITINTERFACE-2774028)
- Vulnerability in [git-pull-or-clone](https://security.snyk.io/vuln/SNYK-JS-GITPULLORCLONE-2434307)
- Vulnerability in [ungit](https://security.snyk.io/vuln/SNYK-JS-UNGIT-2414099)
- Vulnerability in [simple-git](https://security.snyk.io/vuln/SNYK-JS-SIMPLEGIT-2421199)

References for [Blamer npm package vulnerable to argument injection](https://www.nodejs-security.com/blog/destroyed-by-dashes-how-two-hyphens-cause-argument-injection-vulnerability-in-blamer-npm-package), and [Node.js Secure Coding: Defending Against Command Injection](https://www.nodejs-security.com/book/command-injection) book.



---

## 11 Appendix: CLI Frameworks
11.1 CLI Frameworks Table

||||
| :--- | :--- | :---: |
| `oclif` | A framework for building a command line interface. | https://github.com/oclif/oclif |
| `inquirer` | A collection of common interactive command line user interfaces. | https://github.com/SBoudrias/Inquirer.js |
| `ink` | Ink provides the same component-based UI building experience that React offers in the browser, but for command-line apps. | https://github.com/vadimdemedes/ink |
| `pastel` | Next.js-like framework for CLIs made with Ink. | https://github.com/vadimdemedes/pastel |
| `ink UI` | Collection of customizable UI components for CLIs made with Ink. | https://github.com/vadimdemedes/ink-ui |
| `blessed` | A curses-like library with a high level terminal interface API for node.js. | https://github.com/chjj/blessed |
| `prompts` | Lightweight, beautiful and user-friendly interactive prompts | https://github.com/terkelg/prompts |
| `vue-termui` | A Vue.js based terminal UI framework that allows you to build modern terminal applications with ease. | https://github.com/vue-terminal/vue-termui |
| `clack` | Effortlessly build beautiful command-line apps | https://github.com/bombshell-dev/clack/tree/main/packages/prompts |

	 
---

## 12 Appendix: CLI educational resources
- https://clig.dev/
- https://primer.style/cli/getting-started/principles

