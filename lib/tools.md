# tools 封装的库
将常用的函数进行封装成库, 以及达到代码高度复用的作用.


## `npm`包封装步骤

### 注册账号
需要在 `npmjs.org` 网站注册一个账号. 将封装好的包发布到 `npmjs.org` 上, 用可以使用 npm install 命令将包安装到项目中.


### npm 初始项目
需要使用 `npm` 命令初始化一个项目. 首先, 需要创建项目目录, 在项目目录下进行初始化.

```sh
mkdir ubtore-tools
cd ubtore-tools
```

### 使用 npm init 命令

会创建一个`package.json`文件，这个有很多作用，如：

 * 指定包/项目名、版本号
 * 指定程序入口点
 * 指定包`npm`脚本信息
 * 为项目所依赖的包提供文档
 * 根据`npm`的语义规则为依赖包指定所需的版本

```sh
npm init
```

 输入命令后，npm会通过命令行问答的方式来初始化并创建package.json文件。
 ```sh
 This utility will walk you through creating a package.json file.
 It only covers the most common items, and tries to guess sensible defaults.

 See `npm help json` for definitive documentation on these fields
 and exactly what they do.

 Use `npm install <pkg> --save` afterwards to install a package and
 save it as a dependency in the package.json file.

 Press ^C at any time to quit.
 name: (npm) my-package
 version: (1.0.0)
 description: 我的第一个包
 entry point: (index.js)
 test command:
 git repository:
 keywords:
 author:
 license: (ISC)
 About to write to /Users/liuht/code/npm/package.json:

 {
   "name": "my-package",
   "version": "1.0.0",
   "description": "",
   "main": "index.js",
   "scripts": {
     "test": "echo \"Error: no test specified\" && exit 1"
   },
   "author": "",
   "license": "ISC"
 }

 Is this ok? (yes)
 ```



### 编写代码
编写需要封装的工具代码

## 发布 npm package
将编写好的 `npm package` 发布到 `npmjs.org` 平台上.


### 登录
需要用命令行, 登录到 `npmjs.org` 平上. 这块需要用到我们刚才注册的账号号.

```sh
npm login
-------------
Username:                    // 注册时的用户名
Password:                    // 注册时的密码
Email: (this IS public)      // 注册时的邮箱
```


### 发布
需要使用 `npm publish` 命令去发布包到 `npmjs.org`.

```sh
npm publish ./
```


## 注意

 1. 如果使用`ES6`语法需要使用`babel`.
 2. 如果每次更新时, 需要去修改版本, 如果不修改在发布的时, 会出错.
 
