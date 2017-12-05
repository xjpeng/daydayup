# ES6 export and import

## export 模块导出


### 语法
    export { name1, name2, …, nameN };
	export { variable1 as name1, variable2 as name2, …, nameN };
	export let name1, name2, …, nameN; // also var
	export let name1 = …, name2 = …, …, nameN; // also var, const
	
	export default expression;
	export default function (…) { … } // also class, function*
	export default function name1(…) { … } // also class, function*
	export { name1 as default, … };
	
	export * from …;
	export { name1, name2, …, nameN } from …;
	export { import1 as name1, import2 as name2, …, nameN } from …;

### 例子
	//方式一
	export var foo ='bar';
	export let foo1 = 'bar1';
	export cont foo2 = 'bar2';
    export let obj = {k1:1,k2:2};

	export function fun(p1,p2){
	  return p1*p2;
	};
	export function fun2(p1,p2){
	  return p1+p2;
	};
	
	//方式二
	let foo = 'bar';
	let foo1 = 'bar1';
	let	foo2 ='bar2';
	let obj	= {k1:1,k2:2};
	
	let f1 =  function fun(p1,p2){
	  return p1*p2;
	};
	let f2= function fun2(p1,p2){
	  return p1+p2;
	};
	export {foo,foo1,foo2,obj,f1,f2};
	
	//两种方式等价的
	//使用时候
    import {foo} from 'xx.js'


### export default 默认输出

	//默认输出只能输出对象和函数

	//输出函数
	export default function(){};

	//输出对象
	let f=function(){};
	let o = 100;
	export default {f,o};
每个module文件最多能包含一个export default输出

## import 模块导入

### 语法
    import defaultExport from "module-name";
	import * as name from "module-name";
	import { export } from "module-name";
	import { export as alias } from "module-name";
	import { export1 , export2 } from "module-name";
	import { export1 , export2 as alias2 , [...] } from "module-name";
	import defaultExport, { export [ , [...] ] } from "module-name";
	import defaultExport, * as name from "module-name";
	import "module-name";

### 例子
    //module.js
	let foo = 'bar';
	let foo1 = 'bar1';
	let	foo2 ='bar2';
	let obj	= {k1:1,k2:2};
	
	let f1 =  function fun(p1,p2){
	  return p1*p2;
	};
	let f2= function fun2(p1,p2){
	  return p1+p2;
	};
	export {foo,foo1,foo2,obj,f1,f2};
	
	export function f3(p){
		console.log(p);
	};
	
	export default  function (){
		console.log('default');
	};
	    
    //index.js
	import defaultfun,{foo,f1,f3} from './module.js'
    console.log(defaultfun,foo,f1,f3);
    //defaultfun 默认输出
	//foo 输出bar
	//f1 输出module.js里面的函数 f1
	//f3 输出module.js里面的函数 f3