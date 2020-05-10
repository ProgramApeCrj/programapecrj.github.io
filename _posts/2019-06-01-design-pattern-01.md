---
layout: default_post
title : 工作中一次代码优化及对设计模式的思考
category: 设计模式
---


前言：虽然之前看过一些设计模式的书或者专栏，但也只是依样画瓢，没有体验过为什么要有设计模式的思维。经历过一次工作中对代码的优化后，终于明白了一丢丢。下面把思路再复现一下。

#### 导航
1. 场景描述
2. 暴力写法
3. 依样画瓢
4. 进行优化
5. 考虑演进
6. 再一次封装
7. 总结


<br>
# 1. 场景描述

在开发中，很多时候要收集小程序用户的一些业务上报，比如足迹，点赞，观看时长等等。通过上报后，借助 kafka 来进行消费。

现在的场景是这样的，业务分为多种，会和前端约定相应的key。比如
- 10001 代表足迹
- 10002 代表点赞
- 10003 代表支付
- 。。。

大致的格式就是 {key："10001", "content":".....", xx : "..."}

要求对不同 key 都编写对应的一个处理过程，或入库，或做累加等等。。规则也各不相同。

# 2. 暴力写法

其实上面的场景很简单，上来不用思考就可以写了：

写三个办法分别处理不同的 key

```
func handle10001(msg Msg) {
   // todo 处理逻辑
}

func handle10002 (msg Msg) {
   // todo 处理逻辑
}

func handle10003 (msg Msg) {
   // todo 处理逻辑
}

```
然后消费数据的时候直接调用

```
func cus(msg Msg){
   if msg.key == "10001"{
      handle10001(msg)
   }else if msg.key == "10002"{
      handle10002(msg)
   }else if msg.key == "10003"{
      handle10003(msg)
   }
}


```

<br>
# 3. 依样画瓢

暴力写法其实也可以，也挑不出什么毛病。但是这段时间我可是在看设计模式，怎么也得写个工厂办法之类的把，于是变成如下这样：

定义了处理数据的接口
```
type msgHandle interface{
   handle(msg Msg)
}

```

每个需要处理 key 的程序都需要实现该接口(鸭子类型)
```
type handle10001 struct {

}

func(h handle10001) handle(msg Msg){
   // todo 处理逻辑
}

type handle10002 struct {

}

func(h handle10002) handle(msg Msg){
   // todo 处理逻辑
}

type handle10003 struct {

}

func(h handle10003) handle(msg Msg){
   // todo 处理逻辑
}

```

因此处理的逻辑变成了
```
func cus(msg Msg){

   // 这段封装起来就是一个工厂
   var msgh msgHandle
   if msg.key == "10001"{
      msgh = handle10001{}
   }else if msg.key == "10002"{
      msgh = handle10002{}
   }else if msg.key == "10003"{
      msgh = handle10003{}
   }

   // 真正处理过程
   msgh.handle(msg)
}

```

<br>
# 4. 进行优化

上面就是学了设计模式的人的通病，有了锤子，啥都是钉子。那样写虽然看起来好像不错，实际上和之前的并没有太大区别。

不过，工厂模式其实可以通过 map 去优化，优化后如下：

```
mmp := make(map[string]msgHandle)
mmp["10001"] = handle10001{}
mmp["10002"] = handle10002{}
mmp["10003"] = handle10003{}

```

调用程序就显得更加简单直观，增加其他的 key 只需要修改 map, 然后编写对应的结构体  

```
func cus(msg Msg){
   msgh, _ := mmp[msg.key]
   msgh.handle(msg)
}
```

<br>
# 5. 考虑演进

就这样我写了处理 10001 的代码。这时候，同事小峰对我说，他也要处理10001, 而且和我的处理方式不同。我就跟他说，那你封装个办法，写在我下面把，代码如下：

```
type handle10001 struct {

}

func(h handle10001) handle(msg Msg){
   // todo 我的处理逻辑

   // 小峰的
   xiaofengHandle(msg Msgs)
}


func xiaofengHandle(msg Msg){

}

```

相信这段代码很容易看出弊端，现在只有小峰，以后还有小志，小源，他们都要处理 10001，而且处理的方式完全不一样，怎么办？

一旦人多起来，办法就变得臃肿，而且我每次要改自己的代码，都要看有没有动到别人的代码，增加了心智负担。另外，如果我 panic 了，下面的代码都执行不到了，我怕是要被人群殴。

这时候我突然想到，可以用发布订阅的设计模式，我们在代码层面，不同的 key 作为一个事件，别人都来订阅，消费该 key 的时候通知所有的订阅者。

```
type handle10001 struct {
   observerList []observer
}

func(h handle10001) addObserver(ob observer) {
   h.observerList = append(h.observerList, ob)
}

func(h handle10001) handle(msg Msg){
   for _,o := range h.observerList {
      o.update(msg)
   }
}

type observer interface{
   update(msg Msg)
}

type my10001observer struct {
}

func(my my10001observer) update(msg Msg)  {
   // 我对100001的处理
}

type xiaoFeng10001observer struct {
}

func(xiaofeng xiaoFeng10001observer) update(msg Msg)  {
   // 小峰对100001的处理
}

```

执行如下
```
// 这里的初始化可以放在 init 里
obs := handle10001{[]observer{}} 
obs.addObserver(my10001observer{}) // 我的
obs.addObserver(xiaoFeng10001observer{}) // 小峰的
mmp["10001"] = obs

// 10002
obs2 := handle10002{[]observer{}} 
obs2.addObserver(my10002observer{})
mmp["10002"] = obs2


// 办法
func cus(msg Msg){
   obs, _ := mmp[msg.key]
   obs.handle(msg)// 通知要处理的人
}

```

handle 办法这样写还可以避免两人的 pannic 互相影响，如果不进行上面的封装，这点根本做不到。好处已经显现出来了。

```
func handleWithCoverPanic(obs observer, msg Msg) {
   defer func() {
      if err := recover(); err != nil {
         // 处理 panic
      }
   }()
   obs.update(msg)
}

func(h handle10001) handle(msg Msg){
   for _,o := range h.observerList {
      handleWithCoverPanic(o,msg)
   }
}

```

<br>
# 6. 再一次封装
其实上面的代码已经符合开放封闭原则了，增加新的 key 只需要实现新的   msgHandle，然后注册到 map 里。别人想处理同样的 key 也只需要实现新的 Observer，然后登记到对应的 msgHandle 上。不同的人不需要动到其他人的代码。

但是仔细看了好像创建了过多的结构体。实际上，go 语言可以能直接把函数当参数的，为啥我要这么麻烦创建结构体呢？不要被别的语言所束缚：

```
type msgHandle2 struct {
   Methods map[string][]func(msg Msg) // key 是要消费的 key, val 是一个包含多个办法的数组
}

func (mgsh *msgHandle2) AddMethod(key string, fn func(msg Msg)) {
   mgsh.Methods[key] = append(mgsh.Methods[key], fn)
}

func (mgsh *msgHandle2) handle(key string, msg Msg) {

   for _, v := range mgsh.Methods[key] {
      handleWithRecover(v, msg)
   }
}

func handleWithRecover(fn func(msg Msg), msg Msg) {
   defer func() {
      if err := recover(); err != nil {
      
      }
   }()
   fn(msg)
}

// 我的处理
func myHandle10001(msg Msg) {
   
}

// 小峰的处理
func xiaoFengHandle10001(msg Msg) {

}
var msgh2 msgHandle2

func init() {
   //初始化
   msgh2 = msgHandle2{make(map[string][]func(msg Msg))}

   // 10001
   msgh2.AddMethod("10001", myHandle10001) // 我对 10001 的处理
   msgh2.AddMethod("10001", xiaoFengHandle10001) // 小峰对 10001 的处理

   // 10002 
   msgh2.AddMethod("10002", myHandle10002) // 我对 10002 的处理

   // 10003  
   ....

   // 增加新的 key 和 办法，不需要改上面的。
   // 有一天我不想处理 10001，只需要注释掉我的那行代码
   // 这看起来有点像 gin 框架的路由注册
}


```
这里稍微解释下，上面其实是定义一个了一个这样的 map。"10001" => ["办法1"，"办法2"], "10002" => ["办法3"，"办法4"]，根据 key 定位到数组，遍历执行数组里的办法。

处理如下，只剩下一句话
```
func cus(msg Msg){
  msgh2.handle(msg.key, msg)
}

```

你会发现，最终你并不能说我究竟用了哪几种设计模式，但是演进的过程却是参考了设计模式的想法。不同语言对于设计模式的代码诠释也不一定一样。


<br>
# 7. 总结
一点小总结

- 写代码从最简单入手，暴力的写法满足的话，可以不用过度优化。
- 进行迭代的时候，发现有问题了，可以开始尝试封装，不然你不知道能演变成什么样子。
- 代码是演进的，不是一蹴而就，别人的框架亦是如此。
- 不要拘泥于语言或者设计模式，不同语言可能写出来不是很一样，但是核心思想也都是设计模式说的开放封闭原则。用大白话说，就是增加新的功能，不需要有看别人的代码，不需要改别人的代码，不需要有过多的心智负担。

