<!-- 主页 -->

## TypeScript 安装

```shell
npm install -g typescript
查看版本
tsc -v
编译成js文件
tsc demo.ts
初始化tsconfig.json
tsc -init
安装ts-node,可以直接使用ts-node运行ts文件
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
## 数组类型注解方法

```typescript
const numberArr : number[] = [1,2,3]
const strArr : string[] = ['aaa', 'bbb']
const undefinedArr : undefined[] = [undefined, undefined]
const arr : (number | string)[] = [1, 'string', 3]

类型别名
type lady = {name:string, age: number}
或者使用class也行
class lady {
    name: string,
    age: number
}
const xiedajiao : lady[] = [
    {name:'小明', age: 18},
    {name:'小花', age: 19}
]
```
## 接口

```typescript
interface Girl {
    name:string;
    //age此属性不是必须的
    age ?: number;
    //可传入任意属性，任意值
    [propname:string]: any;
	//方法返回类型string
	say(): string;
}
class Xiaojiejie implements Girl{
    name = '小明';
    age = 18;
    sex = '女';
	say() {
		return '欢迎光临'
	};
}
const girl = new Xiaojiejie()
const getGirl = (girl:Girl)=> {
    console.log(girl.name + ':' + girl.sex)
    console.log(girl.say())
}
getGirl(girl)
```

## 枚举类型

```typescript
enum Status {
    dog,
    cat,
    pig
}
不给值，默认从下标0开始
enum Status {
    dog = 'a',
    cat = 2,
    pig = 3
}
function getStauts(status : any) {
    if (status === Status.dog) {
        console.log("dog")
    } else if (status === Status.cat) {
        console.log("cat")
    } else if (status === Status.pig) {
        console.log("pig")
    }
}
getStauts('a')
```
## 泛型

```typescript
function join<T>(first:T, second:T) {
    return `${first}${second}`
}
console.log(join<string>('a', 'b'))

//泛型中数组的使用
function myFun<T>(param: Array<T>) {
    return param
}
console.log(myFun<string>(['aa', 'bb']))
```