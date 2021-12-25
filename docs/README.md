<!-- 主页 -->

## TypeScript 安装

```shell
npm install -g typescript
查看版本
tsc -v
安装ts-node
npm install -g ts-node
```

## 基础静态类型

```typescript
number,string,null,undefinde,boolean,void,symbol
const count: number = 100;
const myName: string = '小明';
```

## 对象静态类型

```typescript
对象类型
const mm() {
	name: string,
	age: number
} = {
	name: '大脚',
	age: 18
}
数组类型
const mms = string[] = ['谢大脚', '刘颖', '123']
类类型
class person{}
const dajie = new person()
函数类型
const xiedajiao :()=>string =()=>{
	retrun '大脚'
}
```

## 函数返回类型

```typescript
function getTotal(one : number, two : number) : number {
    retrun one + two
}
const total = getTotal(1, 2)
```

