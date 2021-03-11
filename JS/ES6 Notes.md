# ES6 Nodes

[TOC]

## Node.js

#### Install

##### Download

https://nodejs.org/zh-cn/

##### After install

Run `Set-ExecutionPolicy unrestricted` in the administrator Powershell to Enable npm command like "babel-node".

##### Enable ES6

New a `.babelrc` in the project root and add content follow.

```json
{
    "presets": ["es2015"]
}
```

install babel and run scripts with command `babel-node`

```bash
$ npm install babel-preset-es2015 --save-dev
$ npm install babel-cli -g
$ babel-node main
```

#### npm

```bash
$ npm install webpack
$ npm install webpack-cli
$ npm install jquery -g
$ npm install
```

