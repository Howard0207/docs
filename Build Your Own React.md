# Build Your Own React

## step 1 (创建元素)

```js
function createElement(type, props, ...children) {
	return {
		type,
		props: {
			...props,
			children: children.map((child) => (typeof child === "object" ? child : createTextElement(child))),
		},
	};
}

function createTextElement(text) {
	return {
		type: "TEXT_ELEMENT",
		props: {
			nodeValue: text,
			children: [],
		},
	};
}

/** @jsx Didact.createElement */
const element = (
	<div id="foo">
		<a>bar</a>
		<b />
	</div>
);

const Didact = {
	createElement,
};


```





## step 2 (实现render)

```js
// 实现render

function createElement(type, props, ...children) {
	return {
		type,
		props: {
			...props,
			children: children.map((child) => (typeof child === "object" ? child : createTextElement(child))),
		},
	};
}

function createTextElement(text) {
	return {
		type: "TEXT_ELEMENT",
		props: {
			nodeValue: text,
			children: [],
		},
	};
}

function render(element, container) {
	// TODO create dom nodes
	const dom = element.type === "TEXT_ELEMENT" ? document.createTextNode("") : document.createElement(element.type);

	// 为当前节点添加属性
	const isProperty = (key) => key !== "children";
	Object.keys(element.props)
		.filter(isProperty)
		.forEach((name) => {
			dom[name] = element.props[name];
		});

	// 递归处理子元素
	element.props.children.forEach((child) => render(child, dom));
	container.appendChild(dom);
}

const Didact = {
	createElement,
	render,
};

/** @jsx Didact.createElement */
const element = (
	<div id="foo">
		<a>bar</a>
		<b />
	</div>
);

const container = document.getElementById("root");
Didact.render(element, container);

```





## step 3（添加Concurrent Mode 并发模式）

```js
// Concurrent Mode 并发模式

function createElement(type, props, ...children) {
	return {
		type,
		props: {
			...props,
			children: children.map((child) => (typeof child === "object" ? child : createTextElement(child))),
		},
	};
}

function createTextElement(text) {
	return {
		type: "TEXT_ELEMENT",
		props: {
			nodeValue: text,
			children: [],
		},
	};
}

function render(element, container) {
	// TODO create dom nodes
	const dom = element.type === "TEXT_ELEMENT" ? document.createTextNode("") : document.createElement(element.type);

	// 为当前节点添加属性
	const isProperty = (key) => key !== "children";
	Object.keys(element.props)
		.filter(isProperty)
		.forEach((name) => {
			dom[name] = element.props[name];
		});

	// 递归处理子元素
	element.props.children.forEach((child) => render(child, dom));
	container.appendChild(dom);
}

let nextUnitOfWork = null;

function workLoop(deadline) {
	let shouldYield = false;
	while (nextUnitOfWork && !shouldYield) {
		nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
		shouldYield = deadline.timeRemaining() < 1;
	}
	requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(nextUnitOfWork) {
	// TODO
}

const Didact = {
	createElement,
	render,
};

/** @jsx Didact.createElement */
const element = (
	<div id="foo">
		<a>bar</a>
		<b />
	</div>
);

const container = document.getElementById("root");
Didact.render(element, container);

```





## step 4 (添加Fiber)

```js
// Fibers
/**
 * In the render we’ll create the root fiber and set it as the nextUnitOfWork.
 * The rest of the work will happen on the performUnitOfWork function, there we will do three things for each fiber:
 * 1. add the element to the DOM
 * 2. create the fibers for the element’s children
 * 3. select the next unit of work
 */

function createElement(type, props, ...children) {
	return {
		type,
		props: {
			...props,
			children: children.map((child) => (typeof child === "object" ? child : createTextElement(child))),
		},
	};
}

function createTextElement(text) {
	return {
		type: "TEXT_ELEMENT",
		props: {
			nodeValue: text,
			children: [],
		},
	};
}

function createDom(element) {
	const dom = element.type === "TEXT_ELEMENT" ? document.createTextNode("") : document.createElement(element.type);
	// 为当前节点添加属性
	const isProperty = (key) => key !== "children";
	Object.keys(element.props)
		.filter(isProperty)
		.forEach((name) => {
			dom[name] = element.props[name];
		});
	return dom;
}

function render(element, container) {
	nextUnitOfWork = {
		dom: container,
		props: {
			children: [element],
		},
	};
}

let nextUnitOfWork = null;

function workLoop(deadline) {
	let shouldYield = false;
	while (nextUnitOfWork && !shouldYield) {
		nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
		shouldYield = deadline.timeRemaining() < 1;
	}
	requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(fiber) {
	// TODO
	if (!fiber.dom) {
		fiber.dom = createDom(fiber);
	}
	if (fiber.parent) {
		fiber.parent.dom.appendChild(fiber.dom);
	}

	const elements = fiber.props.children;
	let index = 0;
	let prevSibling = null;
	// 同层元素遍历, append
	while (index < elements.length) {
		const element = elements[index];
		const newFiber = {
			type: element.type,
			props: element.props,
			parent: fiber,
			dom: null,
		};
		if (index === 0) {
			fiber.child = newFiber;
		} else {
			prevSibling.sibling = newFiber;
		}
		prevSibling = newFiber;
		index++;
	}
	if (fiber.child) {
		return fiber.child;
	}
	// dfs 向上遍历 fiber tree
	let nextFiber = fiber;
	while (nextFiber) {
		if (nextFiber.sibling) {
			return nextFiber.sibling;
		}
		nextFiber = nextFiber.parent;
	}
}

const Didact = {
	createElement,
	render,
};

/** @jsx Didact.createElement */
const element = (
	<div id="foo">
		<a>bar</a>
		<b />
		<main>
			<div>123</div>
		</main>
	</div>
);

const container = document.getElementById("root");
Didact.render(element, container);

```





## step 5（添加 Commit Phases）

```js
// Render and Commit Phases
/**
 * In the render we’ll create the root fiber and set it as the nextUnitOfWork.
 * The rest of the work will happen on the performUnitOfWork function, there we will do three things for each fiber:
 * 1. add the element to the DOM
 * 2. create the fibers for the element’s children
 * 3. select the next unit of work
 */

function createElement(type, props, ...children) {
	return {
		type,
		props: {
			...props,
			children: children.map((child) => (typeof child === "object" ? child : createTextElement(child))),
		},
	};
}

function createTextElement(text) {
	return {
		type: "TEXT_ELEMENT",
		props: {
			nodeValue: text,
			children: [],
		},
	};
}

function createDom(element) {
	const dom = element.type === "TEXT_ELEMENT" ? document.createTextNode("") : document.createElement(element.type);
	// 为当前节点添加属性
	const isProperty = (key) => key !== "children";
	Object.keys(element.props)
		.filter(isProperty)
		.forEach((name) => {
			dom[name] = element.props[name];
		});
	return dom;
}

function commitRoot() {
	// TODO add nodes to dom
	commitWork(wipRoot.child);
	wipRoot = null;
}

// 这是一个dfs过程
function commitWork(fiber) {
	if (!fiber) {
		return;
	}
	const domParent = fiber.parent.dom;
	domParent.appendChild(fiber.dom);
	commitWork(fiber.child);
	commitWork(fiber.sibling);
}

function render(element, container) {
	wipRoot = {
		dom: container,
		props: {
			children: [element],
		},
	};
	nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let wipRoot = null;

function workLoop(deadline) {
	let shouldYield = false;
	while (nextUnitOfWork && !shouldYield) {
		nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
		shouldYield = deadline.timeRemaining() < 1;
	}
	if (!nextUnitOfWork && wipRoot) {
		// 统一提交变化
		commitRoot();
	}
	requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(fiber) {
	// TODO
	if (!fiber.dom) {
		fiber.dom = createDom(fiber);
	}
	// 以前在此处appendChild
	const elements = fiber.props.children;
	let index = 0;
	let prevSibling = null;
	// 给子节点添加 dom, parent 属性
	while (index < elements.length) {
		const element = elements[index];
		const newFiber = {
			type: element.type,
			props: element.props,
			parent: fiber,
			dom: null,
		};
		if (index === 0) {
			fiber.child = newFiber;
		} else {
			prevSibling.sibling = newFiber;
		}
		prevSibling = newFiber;
		index++;
	}
	if (fiber.child) {
		return fiber.child;
	}
	// dfs 遍历fiber tree
	let nextFiber = fiber;
	while (nextFiber) {
		if (nextFiber.sibling) {
			return nextFiber.sibling;
		}
		nextFiber = nextFiber.parent;
	}
}

const Didact = {
	createElement,
	render,
};

/** @jsx Didact.createElement */
const element = (
	<div id="foo">
		<a>bar</a>
		<b />
	</div>
);

const container = document.getElementById("root");
Didact.render(element, container);

```





## step 6 (Reconciliation 协调)

```js
// Reconciliation 协调

// 更新 删除节点 这里面有dom diff

function createElement(type, props, ...children) {
	return {
		type,
		props: {
			...props,
			children: children.map((child) => (typeof child === "object" ? child : createTextElement(child))),
		},
	};
}

function createTextElement(text) {
	return {
		type: "TEXT_ELEMENT",
		props: {
			nodeValue: text,
			children: [],
		},
	};
}

function createDom(fiber) {
	const dom = fiber.type === "TEXT_ELEMENT" ? document.createTextNode("") : document.createElement(fiber.type);
	updateDom(dom, {}, fiber.props);
	return dom;
}

const isEvent = (key) => key.startsWith("on");
const isProperty = (key) => key !== "children" && !isEvent(key);
const isNew = (prev, next) => (key) => prev[key] !== next[key];
const isGone = (prev, next) => (key) => !(key in next);

// 更新节点属性
function updateDom(dom, prevProps, nextProps) {
	// Remove old or changed event listeners
	Object.keys(prevProps)
		.filter(isEvent)
		.filter((key) => !(key in nextProps) || isNew(prevProps, nextProps)(key))
		.forEach((name) => {
			const eventType = name.toLowerCase().substring(2);
			dom.removeEventListener(eventType, prevProps[name]);
		});
	// 删除不再使用的属性
	Object.keys(prevProps)
		.filter(isProperty)
		.filter(isGone(prevProps, nextProps))
		.forEach((name) => {
			dom[name] = "";
		});
	// 添加新属性和修改改变的属性
	Object.keys(nextProps)
		.filter(isProperty)
		.filter(isNew(prevProps, nextProps))
		.forEach((name) => {
			dom[name] = nextProps[name];
		});
	// 添加监听器
	Object.keys(nextProps)
		.filter(isEvent)
		.filter(isNew(prevProps, nextProps))
		.forEach((name) => {
			const eventType = name.toLowerCase().substring(2);
			dom.addEventListener(eventType, nextProps[name]);
		});
}

function commitRoot() {
	deletions.forEach(commitWork);
	// TODO add nodes to dom
	commitWork(wipRoot.child);
	currentRoot = wipRoot;
	wipRoot = null;
}

function commitWork(fiber) {
	if (!fiber) {
		return;
	}
	const domParent = fiber.parent.dom;
	if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
		domParent.appendChild(fiber.dom);
	} else if (fiber.effectTag === "UPDATE" && fiber.dom !== null) {
		updateDom(fiber.dom, fiber.alternate.props, fiber.props);
	} else if (fiber.effectTag === "DELETION") {
		domParent.removeChild(fiber.dom);
	}
	commitWork(fiber.child);
	commitWork(fiber.sibling);
}

function reconcileChildren(wipFiber, elements) {
	let index = 0;
	let oldFiber = wipFiber.alternate && wipFiber.alternate.child;
	let prevSibling = null;
	while (index < elements.length || oldFiber !== null) {
		const element = elements[index];
		let newFiber = null;
		const sameType = oldFiber && element && element.type === oldFiber.type;

		if (sameType) {
			//TODO update the node
			newFiber = {
				type: oldFiber.type,
				props: element.props,
				dom: oldFiber.dom,
				parent: wipFiber,
				alternate: oldFiber,
				effectTag: "UPDATE",
			};
		}
		if (element && !sameType) {
			// TODO add this node
			newFiber = {
				type: element.type,
				props: element.props,
				dom: null,
				parent: wipFiber,
				alternate: null,
				effectTag: "PLACEMENT",
			};
		}
		if (oldFiber && !sameType) {
			// TODO delete the oldFiber's node
			oldFiber.effectTag = "DELETION";
			deletions.push(oldFiber);
		}
		// TODO compare oldFiber to element
		if (oldFiber) {
			oldFiber = oldFiber.sibling;
		}

		if (index === 0) {
			wipFiber.child = newFiber;
		} else if (element) {
			prevSibling.sibling = newFiber;
		}
		prevSibling = newFiber;
		index++;
	}
}

function render(element, container) {
	wipRoot = {
		dom: container,
		props: {
			children: [element],
		},
		alternate: currentRoot, // 备用节点，为前一次提交节点
	};
	deletions = [];
	nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let currentRoot = null; // 前一次提交跟节点
let wipRoot = null;
let deletions = null;

function workLoop(deadline) {
	let shouldYield = false;
	while (nextUnitOfWork && !shouldYield) {
		nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
		shouldYield = deadline.timeRemaining() < 1;
	}
	if (!nextUnitOfWork && wipRoot) {
		commitRoot();
	}
	requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(fiber) {
	// TODO
	if (!fiber.dom) {
		fiber.dom = createDom(fiber);
	}
	const elements = fiber.props.children;

	// 当前fiberchildren层级进行修改
	reconcileChildren(fiber, elements);

	if (fiber.child) {
		return fiber.child;
	}
	// dfs 遍历fiber tree
	let nextFiber = fiber;
	while (nextFiber) {
		if (nextFiber.sibling) {
			return nextFiber.sibling;
		}
		nextFiber = nextFiber.parent;
	}
}

const Didact = {
	createElement,
	render,
};

function test() {
	console.log("====================test=================");
}

/** @jsx Didact.createElement */
const element = (
	<div id="foo">
		<a
			onClick={() => {
				console.log(123);
			}}
		>
			bar
		</a>
		<b />
	</div>
);

const container = document.getElementById("root");
Didact.render(element, container);

```





## step 7 (函数式组件)

```js
// 函数式组件
/**
 * Function components are differents in two ways
 *  1. the fiber from a function component doesn’t have a DOM node
 *  2. and the children come from running the function instead of getting them directly from the props
 */

function createElement(type, props, ...children) {
    props = typeof props === 'object' && props!==null ? props : {};
    const childrenLength = arguments.length - 2;
    if (childrenLength === 1) {
        if(Array.isArray(children[0])) {
            props.children = children[0];
        } else {
            props.children = children.map(child => typeof child === "object" ? child : createTextElement(child));
        }
    } else if (childrenLength > 1) {
        const childArray = Array(childrenLength);
        for (let i = 0; i < childrenLength; i++) {
            childArray[i] = arguments[i + 2];
        }
        const propChildren = childArray.map(child => typeof child === "object" ? child : createTextElement(child));
        props.children = propChildren;
    }
    return {
        type,
        props,
    };
}

function createTextElement(text) {
	return {
		type: "TEXT_ELEMENT",
		props: {
			nodeValue: text,
			children: [],
		},
	};
}

function createDom(fiber) {
	const dom = fiber.type === "TEXT_ELEMENT" ? document.createTextNode("") : document.createElement(fiber.type);
	updateDom(dom, {}, fiber.props);
	return dom;
}

const isEvent = (key) => key.startsWith("on");
const isProperty = (key) => key !== "children" && !isEvent(key);
const isNew = (prev, next) => (key) => prev[key] !== next[key];
const isGone = (prev, next) => (key) => !(key in next);

// 更新节点属性
function updateDom(dom, prevProps, nextProps) {
	// Remove old or changed event listeners
	Object.keys(prevProps)
		.filter(isEvent)
		.filter((key) => !(key in nextProps) || isNew(prevProps, nextProps)(key))
		.forEach((name) => {
			const eventType = name.toLowerCase().substring(2);
			dom.removeEventListener(eventType, prevProps[name]);
		});
	// 删除不再使用的属性
	Object.keys(prevProps)
		.filter(isProperty)
		.filter(isGone(prevProps, nextProps))
		.forEach((name) => {
			dom[name] = "";
		});
	// 添加新属性和修改改变的属性
	Object.keys(nextProps)
		.filter(isProperty)
		.filter(isNew(prevProps, nextProps))
		.forEach((name) => {
			dom[name] = nextProps[name];
		});
	// 添加监听器
	Object.keys(nextProps)
		.filter(isEvent)
		.filter(isNew(prevProps, nextProps))
		.forEach((name) => {
			const eventType = name.toLocaleLowerCase().substring(2);
			dom.addEventListener(eventType, nextProps[name]);
		});
}

function commitRoot() {
	deletions.forEach(commitWork);
	// TODO add nodes to dom
	commitWork(wipRoot.child);
	currentRoot = wipRoot;
	wipRoot = null;
}

function commitWork(fiber) {
	if (!fiber) {
		return;
	}
	let domParentFiber = fiber.parent;
	while (!domParentFiber.dom) {
		domParentFiber = domParentFiber.parent;
	}
	const domParent = domParentFiber.dom;

	if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
		domParent.appendChild(fiber.dom);
	} else if (fiber.effectTag === "UPDATE" && fiber.dom !== null) {
		updateDom(fiber.dom, fiber.alternate.props, fiber.props);
	} else if (fiber.effectTag === "DELETION") {
		commitDeletion(fiber, domParent);
	}
	commitWork(fiber.child);
	commitWork(fiber.sibling);
}

function commitDeletion(fiber, domParent) {
	if (fiber.dom) {
		domParent.removeChild(fiber.dom);
	} else {
		commitDeletion(fiber.child, domParent);
	}
}

// dom diff
function reconcileChildren(wipFiber, elements) {
	let index = 0;
	let oldFiber = wipFiber.alternate && wipFiber.alternate.child;
	let prevSibling = null;
	while (index < elements.length || oldFiber !== null) {
		const element = elements[index];
		let newFiber = null;
		const sameType = oldFiber && element && element.type === oldFiber.type;

		if (sameType) {
			//TODO update the node
			newFiber = {
				type: oldFiber.type,
				props: element.props,
				dom: oldFiber.dom,
				parent: wipFiber,
				alternate: oldFiber,
				effectTag: "UPDATE",
			};
		}
		if (element && !sameType) {
			// TODO add this node
			newFiber = {
				type: element.type,
				props: element.props,
				dom: null,
				parent: wipFiber,
				alternate: null,
				effectTag: "PLACEMENT",
			};
		}
		if (oldFiber && !sameType) {
			// TODO delete the oldFiber's node
			oldFiber.effectTag = "DELETION";
			deletions.push(oldFiber);
		}
		// TODO compare oldFiber to element
		if (oldFiber) {
			oldFiber = oldFiber.sibling;
		}

		if (index === 0) {
			wipFiber.child = newFiber;
		} else if (element) {
			prevSibling.sibling = newFiber;
		}
		prevSibling = newFiber;
		index++;
	}
}

function render(element, container) {
	wipRoot = {
		dom: container,
		props: {
			children: [element],
		},
		alternate: currentRoot,
	};
	deletions = [];
	nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let currentRoot = null;
let wipRoot = null;
let deletions = null;

function workLoop(deadline) {
	let shouldYield = false;
	while (nextUnitOfWork && !shouldYield) {
		nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
		shouldYield = deadline.timeRemaining() < 1;
	}
	if (!nextUnitOfWork && wipRoot) {
		commitRoot();
	}
	requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(fiber) {
	// TODO
	const isFunctionComponent = fiber.type instanceof Function;
	if (isFunctionComponent) {
		updateFunctionComponent(fiber);
	} else {
		updateHostComponent(fiber);
	}

	if (fiber.child) {
		return fiber.child;
	}
	// dfs 遍历fiber tree
	let nextFiber = fiber;
	while (nextFiber) {
		if (nextFiber.sibling) {
			return nextFiber.sibling;
		}
		nextFiber = nextFiber.parent;
	}
}

function updateFunctionComponent(fiber) {
	// TODO
	const children = [fiber.type(fiber.props)];
	reconcileChildren(fiber, children);
}

function updateHostComponent(fiber) {
	if (!fiber.dom) {
		fiber.dom = createDom(fiber);
	}
	reconcileChildren(fiber, fiber.props.children);
}

const Didact = {
	createElement,
	render,
};

function test() {
	console.log("====================test=================");
}

// /** @jsx Didact.createElement */
// const element = (
// 	<div id="foo">
// 		<a
// 			onClick={() => {
// 				console.log(123);
// 			}}
// 		>
// 			bar
// 		</a>
// 		<b />
// 	</div>
// );

/** @jsx Didact.createElement */
function App(props) {
	return <h1>Hi {props.name}</h1>;
}
const container = document.getElementById("root");
Didact.render(<App name="foo" />, container);

// const container = document.getElementById("root");
// Didact.render(element, container);

```





## step 8 (函数式组件)

```js
// 函数式组件
/**
 * Function components are differents in two ways
 *  1. the fiber from a function component doesn’t have a DOM node
 *  2. and the children come from running the function instead of getting them directly from the props
 */

function createElement(type, props, ...children) {
    props = typeof props === 'object' && props!==null ? props : {};
    const childrenLength = arguments.length - 2;
    if (childrenLength === 1) {
        if(Array.isArray(children[0])) {
            props.children = children[0];
        } else {
            props.children = children.map(child => typeof child === "object" ? child : createTextElement(child));
        }
    } else if (childrenLength > 1) {
        const childArray = Array(childrenLength);
        for (let i = 0; i < childrenLength; i++) {
            childArray[i] = arguments[i + 2];
        }
        const propChildren = childArray.map(child => typeof child === "object" ? child : createTextElement(child));
        props.children = propChildren;
    }
    return {
        type,
        props,
    };
}

function createTextElement(text) {
	return {
		type: "TEXT_ELEMENT",
		props: {
			nodeValue: text,
			children: [],
		},
	};
}

function createDom(fiber) {
	const dom = fiber.type === "TEXT_ELEMENT" ? document.createTextNode("") : document.createElement(fiber.type);
	// 为当前节点添加属性
	updateDom(dom, {}, fiber.props);
	return dom;
}

const isEvent = (key) => key.startsWith("on");
const isProperty = (key) => key !== "children" && !isEvent(key);
const isNew = (prev, next) => (key) => prev[key] !== next[key];
const isGone = (prev, next) => (key) => !(key in next);

// 更新节点属性
function updateDom(dom, prevProps, nextProps) {
	// Remove old or changed event listeners
	Object.keys(prevProps)
		.filter(isEvent)
		.filter((key) => !(key in nextProps) || isNew(prevProps, nextProps)(key))
		.forEach((name) => {
			const eventType = name.toLowerCase().substring(2);
			dom.removeEventListener(eventType, prevProps[name]);
		});

	// 删除不再使用的属性
	Object.keys(prevProps)
		.filter(isProperty)
		.filter(isGone(prevProps, nextProps))
		.forEach((name) => {
			dom[name] = "";
		});

	// 添加新属性和修改改变的属性
	Object.keys(nextProps)
		.filter(isProperty)
		.filter(isNew(prevProps, nextProps))
		.forEach((name) => {
			dom[name] = nextProps[name];
		});

	// 添加监听器
	Object.keys(nextProps)
		.filter(isEvent)
		.filter(isNew(prevProps, nextProps))
		.forEach((name) => {
			const eventType = name.toLowerCase().substring(2);
			dom.addEventListener(eventType, nextProps[name]);
		});
}

function commitRoot() {
	deletions.forEach(commitWork);
	// TODO add nodes to dom
	commitWork(wipRoot.child);
	currentRoot = wipRoot;
	wipRoot = null;
}

function commitWork(fiber) {
	if (!fiber) {
		return;
	}
	let domParentFiber = fiber.parent;
	while (!domParentFiber.dom) {
		domParentFiber = domParentFiber.parent;
	}
	const domParent = domParentFiber.dom;

	if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
		domParent.appendChild(fiber.dom);
	} else if (fiber.effectTag === "UPDATE" && fiber.dom !== null) {
		updateDom(fiber.dom, fiber.alternate.props, fiber.props);
	} else if (fiber.effectTag === "DELETION") {
		commitDeletion(fiber, domParent);
	}

	commitWork(fiber.child);
	commitWork(fiber.sibling);
}

function commitDeletion(fiber, domParent) {
	if (fiber.dom) {
		domParent.removeChild(fiber.dom);
	} else {
		commitDeletion(fiber.child, domParent);
	}
}

// dom diff
function reconcileChildren(wipFiber, elements) {
	let index = 0;
	let oldFiber = wipFiber.alternate && wipFiber.alternate.child;
	let prevSibling = null;
	while (index < elements.length || (oldFiber !== null && oldFiber !== undefined)) {
		const element = elements[index];
		let newFiber = null;
		const sameType = oldFiber && element && element.type === oldFiber.type;

		if (sameType) {
			//TODO update the node
			newFiber = {
				type: oldFiber.type,
				props: element.props,
				dom: oldFiber.dom,
				parent: wipFiber,
				alternate: oldFiber,
				effectTag: "UPDATE",
			};
		}
		if (element && !sameType) {
			// TODO add this node
			newFiber = {
				type: element.type,
				props: element.props,
				dom: null,
				parent: wipFiber,
				alternate: null,
				effectTag: "PLACEMENT",
			};
		}
		if (oldFiber && !sameType) {
			// TODO delete the oldFiber's node
			oldFiber.effectTag = "DELETION";
			deletions.push(oldFiber);
		}
		// TODO compare oldFiber to element
		if (oldFiber) {
			oldFiber = oldFiber.sibling;
		}

		if (index === 0) {
			wipFiber.child = newFiber;
		} else if (element) {
			prevSibling.sibling = newFiber;
		}
		prevSibling = newFiber;
		index++;
	}
}

function render(element, container) {
	wipRoot = {
		dom: container,
		props: {
			children: [element],
		},
		alternate: currentRoot,
	};
	deletions = [];
	nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let currentRoot = null;
let wipRoot = null;
let deletions = null;

function workLoop(deadline) {
	let shouldYield = false;
	while (nextUnitOfWork && !shouldYield) {
		nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
		shouldYield = deadline.timeRemaining() < 1;
	}
	if (!nextUnitOfWork && wipRoot) {
		commitRoot();
	}
	requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(fiber) {
	// TODO
	const isFunctionComponent = fiber.type instanceof Function;
	if (isFunctionComponent) {
		updateFunctionComponent(fiber);
	} else {
		updateHostComponent(fiber);
	}

	if (fiber.child) {
		return fiber.child;
	}
	// dfs 遍历fiber tree
	let nextFiber = fiber;
	while (nextFiber) {
		if (nextFiber.sibling) {
			return nextFiber.sibling;
		}
		nextFiber = nextFiber.parent;
	}
}

let wipFiber = null;
let hookIndex = null;

// 更新函数式组件
function updateFunctionComponent(fiber) {
	// TODO
	wipFiber = fiber;
	hookIndex = 0;
	wipFiber.hooks = [];
	const children = [fiber.type(fiber.props)];
	reconcileChildren(fiber, children);
}

// 更新普通fiber
function updateHostComponent(fiber) {
	if (!fiber.dom) {
		fiber.dom = createDom(fiber);
	}
	reconcileChildren(fiber, fiber.props.children);
}

function useState(initial) {
	// TODO
	const oldHook = wipFiber.alternate && wipFiber.alternate.hooks && wipFiber.alternate.hooks[hookIndex];
	const hook = {
		state: oldHook ? oldHook.state : initial,
		queue: [],
	};

	const actions = oldHook ? oldHook.queue : [];
	actions.forEach((action) => {
		hook.state = action(hook.state);
	});

	const setState = (action) => {
		hook.queue.push(action);
		wipRoot = {
			dom: currentRoot.dom,
			props: currentRoot.props,
			alternate: currentRoot,
		};
		nextUnitOfWork = wipRoot;
		deletions = [];
	};

	wipFiber.hooks.push(hook);
	hookIndex++;
	return [hook.state, setState];
}

const Didact = {
	createElement,
	render,
	useState,
};

function test() {
	console.log("====================test=================");
}

// /** @jsx Didact.createElement */
// const element = (
// 	<div id="foo">
// 		<a
// 			onClick={() => {
// 				console.log(123);
// 			}}
// 		>
// 			bar
// 		</a>
// 		<b />
// 	</div>
// );

/** @jsx Didact.createElement */
function Counter() {
	const [state, setState] = Didact.useState(1);
	return <h1 onClick={() => setState((c) => c + 1)}>Count: {state}</h1>;
}

const container = document.getElementById("root");
Didact.render(<Counter />, container);

// const container = document.getElementById("root");
// Didact.render(element, container);

```

