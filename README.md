# TSLint Development Environment
I was tired to always `tsc lint-rule` and check if it compiled, then check if the rule worked as I wanted, not being able to debug the rule itself, so I put some time to setup this small project.

It helps me do the following:
* Quickly run new rules on an existing project
* No additional dependencies on the new project
* Debugging the execution of the rule in typescript with node --inspect
* It is not required to add the rule to the base repository while testing

## Setup
* clone this repository next to your actual project
* `npm i` on this repository
* [code the new rule](https://palantir.github.io/tslint/develop/custom-rules/) e. g. as `src/someAwesomeRule.ts`
* test-run & debug the rule
* build the rule and include it in your project

There are several possibilities to run the rule, e. g. using a file watcher like nodemon to automatically rerun when the files change, I prefer the following setup to debug from VSCode
### VSCode `launch.json`
* Open new rule file, without leaving current workspace (e. g. `../tslint-devenv/src/someAwesomeRule.ts`)
* Add the following to `launch.json`:
  ```
  {
    "type": "node",
    "request": "launch",
    "name": "tslint-env",
    "port": 9299,
    "cwd": "${workspaceFolder}/../tslint-devenv",
    "runtimeArgs": [
      "--inspect-brk=9299",
      "-r",
      "ts-node/register",
    ],
    "args": [
      "-p",
      "${workspaceFolder}/tsconfig.json",
      "-r",
      "src",
    ],
    "program": "${workspaceFolder}/../tslint-devenv/node_modules/.bin/tslint",
    "sourceMaps": true,
    "console": "integratedTerminal"
  }
  ```
### Call every time from terminal
Another possibility would be to run the command using nodemon. The tricky part is to pass in `-p path/to/your/project` and `-r src-folder/with/the/rule` to tslint while also calling `node --inspect-brk -r ts-node/register` to enable debugging, so at least in this command both the repositories have to be referenced. A full one-time command could look like this:  
__Calling from `tslint-devenv`__
```
node --inspect-brk -r ts-node-register node_modules/.bin/tslint -r src -p ../my-project/tsconfig.json
```
__Calling from your project__
```
node --inspect-brk -r ts-node-register ../tslint-devenv/node_modules/.bin/tslint -r ../tslint-devenv/src -p tsconfig.json
```

### Use the rules in your project
Once the rule is finished, run `npm run build` and copy the compiled JavaScript file from `dist` to your project's rule folder. Do not forget to add the new rule name and the folder containing your custom rules to your `tslint.json`.
