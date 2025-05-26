# package.json简要说明

：：： info

本文是对package.json中重要字段的概念进行的回顾

：：：

## 1.scripts

`scripts`指定了运行脚本命令的npm命令行缩写,以下设置指定了`npm run start`、`npm run test`时，所要执行的命令

```
"scripts":{
  "start":"node index.js",
  "test":"tap test/*.js"
}
```

## 2.dependencies、devDependencies字段

`dependencies`字段指定了项目运行依赖的模块，`devDependencies`字段指定了项目开发所需要的模块每个对象都由模块名+版本要求组成，表示依赖的模块以及版本范围

```
{
  "devDependencies": {
    "browserify": "~13.0.0",
    "karma-browserify": "~5.0.1"
  }
}
```

版本要求分为以下几种：

- 指定版本：如`1.1.1`，遵循”大、次、小版本“的格式规定，安装指定版本
- 波浪号版本：如`~1.2.3`，表示安装1.2.x的最新版本(不改变大和次版本号)
- 插入号版本：如`^1.2.3`，表示安装1.x.x的最新版本(不改变大版本号)
- latest:安装最新版本

使用`npm init`可以初始化package.json文件，使用`npm install/npm install xxx --save-dev`可以安装全部模块或者安装某个模块并指定为开发模块

## 3.peerDependencies

`peerDependencies`字段是用来指定一个模块所依赖的模块的版本

```
{
  "name": "chai-as-promised",
  "peerDependencies": {
    "chai": "1.x"
  }
}
```

安装`chai-as-promised`模块时，依赖模块`chai`必须一起安装，而且`chai`必须是1.x，否则会报错

## 4.bin字段

`bin`字段用来指定各个内部命令对应可执行文件位置

```
"bin": {
  "someTool": "./bin/someTool.js"
}
//原来的
scripts: {  
  start: './node_modules/someTool/someTool.js build'
}
//bin指定后的
scripts: {  
  start: 'someTool build'
```

总结来说：bin字段就是指定为文件路径提供一个简要命名，方便scripts中指定对应的npm运行命令

## 5.其他字段

`main`字段指定项目加载的入口文件；

`config`字段用于添加项目的环境变量

```
//这是一个package.json文件
{
  "name" : "foo",
  "config" : { "port" : "8080" },
  "scripts" : { "start" : "node server.js" }
}
```

```
//这是server.js脚本 (process.env.npm_package_config_port就是设置的环境变量取的是8080)
http
  .createServer(...)
  .listen(process.env.npm_package_config_port)
```

运行`npm run start`命令时，这个脚本就可以得到值。

`browser`字段指定浏览器使用的版本(browserify这样的浏览器打包工具就知道打包哪个文件)

`style`字段指定浏览器使用的样式文件(parcelify这样的浏览器打包工具就知道打包哪个样式文件)

`engines`字段模块运行的环境

```
{ "engines" : { "node" : ">=0.10.3 <0.12" } }
```

