# Webpack 工作流程

## Webpack 执行流程

1. 初始化 Compiler : Webpack ( config )得到 Compiler 对象
2. 开始编译 : 调用 Compiler 对象 run 方法开始执行编译
3. 确定入口 : 根据配置中的 entry 找出所有的入口文件。
4. 编译模块: 从入口文件出发,调用所有配置的 Loader 对模块进行编译,再找出该模块依赖的模块,递归直到所有模块被加载进来
5. 完成模块编译 : 在经过第4步使用 Loader 编译完所有模块后,得到了每个模块被编译后的最终内容以及它们之间的依赖关系。
6. 输出资源 : 根据入口和模块之间的依赖关系,组装成一个个包含多个模块的 Chunk ,再把每个 Chunk 转换成一个单独的文件加入到输出列表。(注意:这步是可以修改输出内容的最后机会)
7. 输出完成 : 在确定好输出内容后,根据配置确定输出的路径和文件名,把文件内容写入到文件系统

## 目录结构

|- bin

|- lib

|- package.json

|- yarn.lock

## bin目录(gh-pack.js)

```js
#! /usr/bin/env node

// 1)需要找到当前执行命的路径 拿到webpack.config.js
let path = require("path");

// config 配置文件
let config = require(path.resolve("webpack.config.js"));

let Compiler = require("../lib/Compiler");

let compiler = new Compiler(config);

compiler.hooks.entryOption.call();
// 标识 运行编译

compiler.run();

```



## lib目录（Compiler.js）

```js
const fs = require("fs");
const path = require("path");
let babylon = require("babylon"); // 主要把源码转换成ast
let t = require("@babel/types"); // 替换节点
let traverse = require("@babel/traverse").default; // 遍历节点
let generator = require("@babel/generator").default; // 生成新的节点
const { tsThisType } = require("@babel/types");
const { SyncHook } = require("tapable");
let ejs = require("ejs");
class Compiler {
	constructor(config) {
		this.config = config;
		// 需要保存入口文件的路径
		this.entryId; // '。、src/index.js'
		// 需要保存所有的模块依赖
		this.modules = {};
		this.entry = config.entry;
		// 工作路径
		this.root = process.cwd();

		this.hooks = {
			entryOption: new SyncHook(),
			compile: new SyncHook(),
			afterCompile: new SyncHook(),
			afterPlugins: new SyncHook(),
			run: new SyncHook(),
			emit: new SyncHook(),
			done: new SyncHook(),
		};

		let plugins = this.config.plugins;
		if (Array.isArray(plugins)) {
			plugins.forEach((plugin) => {
				plugin.apply(this);
			});
		}
		this.hooks.afterPlugins.call();
	}

	getSource(modulePath) {
		let rules = this.config.module.rules;
		let content = fs.readFileSync(modulePath, "utf8");
		for (let i = 0; i < rules.length; i++) {
			let rule = rules[i];
			let { test, use } = rule;
			let len = use.length - 1;
			if (test.test(modulePath)) {
				// 获取对应的loader函数
				function normalLoader() {
					let loader = require(use[len--]);
					content = loader(content);
					if (len >= 0) {
						normalLoader();
					}
				}
				normalLoader();
			}
		}
		return content;
	}

	// 解析源码
	parse(source, parentPath) {
		let ast = babylon.parse(source);
		let dependencies = []; // 依赖的数组
		// console.log(ast);
		traverse(ast, {
			CallExpression(p) {
				let node = p.node; // 对应的节点
				if (node.callee.name === "require") {
					node.callee.name = "__webpack_require__";
					let moduleName = node.arguments[0].value; // 取到的是模块的引用名字
					moduleName = moduleName + (path.extname(moduleName) ? "" : ".js");
					moduleName = "./" + path.join(parentPath, moduleName);
					dependencies.push(moduleName);
					node.arguments = [t.stringLiteral(moduleName)];
				}
			},
		});
		let sourceCode = generator(ast).code;
		return { sourceCode, dependencies };
	}

	// 构建模块
	buildModule(modulePath, isEntry) {
		// 拿到模块的内容
		let source = this.getSource(modulePath);
		// 模块id modulePath = modulePath  - this.root
		let moduleName = "./" + path.relative(this.root, modulePath);
		if (isEntry) {
			this.entryId = moduleName; // 保存入口的名字
		}
		// 解析需要把source源码进行改造 返回一个依赖列表
		let { sourceCode, dependencies } = this.parse(source, path.dirname(moduleName));
		console.log(sourceCode, dependencies);
		// 把相对路径和模块中的内容 对应起来
		this.modules[moduleName] = sourceCode;

		dependencies.forEach((dep) => {
			// 父模块的附属品
			this.buildModule(path.join(this.root, dep), false);
		});
	}

	emitFile() {
		// 用数据渲染 我们的模板
		// 输出路径
		let main = path.join(this.config.output.path, this.config.output.filename);
		// 模板路径
		let templateStr = this.getSource(path.join(__dirname, "main.ejs"));
		let code = ejs.render(templateStr, { entryId: this.entryId, modules: this.modules });
		this.assets = {};
		// 资源中 路径对应的代码
		this.assets[main] = code;
		fs.writeFileSync(main, this.assets[main]);
	}

	run() {
		// 执行
		this.hooks.run.call();
		this.buildModule(path.resolve(this.root, this.entry), true);
		console.log(this.modules, tsThisType.entryId);
		// 发射一个文件 打包后的文件
		this.emitFile();
		this.hooks.emit.call();
		this.hooks.done.call();
	}
}

module.exports = Compiler;

```



## lib目录(main.ejs)

```js
(function (modules) { 
    var installedModules = {};
    function __webpack_require__(moduleId) { 
        if (installedModules[moduleId]) { 
            return installedModules[moduleId].exports; 
        }
        var module = (installedModules[moduleId] = { 
            i: moduleId, 
            l: false, 
            exports: {}
        });
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__); 
        module.l = true; 
        return module.exports; 
    } 
    return __webpack_require__((__webpack_require__.s = "<%-entryId%>"));
 })
 ({ 
    <%for(let key in modules){%> 
        "<%-key%>": 
        (function (module, exports, __webpack_require__) {
            eval(`<%-modules[key]%>`); 
        }), 
    <%}%> 
});
```

