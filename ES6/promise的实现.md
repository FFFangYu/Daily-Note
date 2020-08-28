# Promise的实现（2020.8.14-）

首先，需要明确promise对象代表一个异步操作，有三种状态（pending、fulfilled、rejected），并且一旦状态发生改变，就不会再变。

下面看一下，Promise里面需要实现的函数：

- 构造函数
  - promise的状态
  - promise的值
  - 回调函数对象存放数组`[{OnResolved,OnRejected}]`
  - 有一个执行器executor,在构造函数内执行try
- 实例方法：
  - then
  - catch
- 静态方法
  - resolve
  - reject
  - all
  - race

在手撕这些方法的时候，需要考虑到怎么使用，也就是说手撕的这个方法怎么使用，参数是什么，返回值是什么。

## constructor

```js
new Promise((resolve,reject) => {
    setTimeout(()=>{
        resolve(1)
        // reject(2)
    },100)
}).then(
    value => {
        console.log('OnResolved',value)
    },
    reason => {
        console.log('OnRejected',reason)
    }
)
```

**明确几点：**

- 参数是一个执行器，在执行的时候，成功使用resolve方法，失败使用reject方法。通过try...catch来实现。
- 一个Promise对象有三种状态，所以需要一个status保存现在的状态，用data来保存执行的结果（值），另外需要考虑回调队列（❓这里我一直不太理解----✔在then的方法中会将两个方法存放到回调队列中），所以用一个数组来保存，里面的用对象的形式{OnResolved,OnRejected}
- 执行器当中会有两个方法，就是resolve（value）和reject（reason），在执行这两个方法的时候就会执行回调队列中存放的方法
  - 两个方法的执行过程可以总结为：
    - 判断当前对象的状态，不是pending就直接返回
    - 保存结果，就是传进来的参数
    - 查看回调队列中有没有值，有的话就执行响应的方法，resolve对应OnResolved，reject对应OnRejected

**代码**

```js
class Promise{
    constructor(executor){
        this.status = 'pending'
        this.data = undefined
        this.callbacks = []
        let resolve = (value) => {
            if(this.status !== 'pending') return
            setTimeout(() => { // setTimeout模拟异步，别忘记❗
                this.status = 'resolved'
                this.data = value
                if(this.callbacks.length>0){
                    this.callbacks.forEach(callback => {
                        callback.OnResolved(value)
                    })
                }
            })
        }
        let reject = (reason) => {
            if(this.status !== 'pending') return
            setTimeout(() => {
                this.status = 'rejected'
                this.data = reason
                if(this.callbacks.length>0){
                    this.callbacks.forEach(callback => {
                        callback.OnRejected(reason)
                    })
                }
            })
        }
        try{
            executor(resolve,reject)
        }catch(err){
            reject(err)
        }
    }
}
```



## static all方法

```js
Promise.all(promises)
```

**明确:**

- 该方法是静态方法，可以直接通过Promise来调用
- 参数promises是一个Promise对象数组
- 返回结果是一个新的Promise对象
- 中间过程，只有promises数组所有promise都成功才成功，所有成功的返回值组成一个数组传递给返回新Promise对象的回调函数；有一个失败就失败，将失败的结果返回给新Promise对象的回调函数。

**代码**

```js
static all(promises){
    let res = new Array(promises.length)
    let count = 0
    return new Promise((resolve,reject) => {
        promises.forEach( (p,index) => {
            p.then( // 用then方法来获取结果，里面是成功和失败的回调
                value => {
                    count++
                    res[index] = value
                    // 只有全部promise都成功才算成功，将成功结果传给新Promise的成功回调
                    if(count === promises.length){
                        resolve(res)
                    }
                },
                reason => {
                    reject(reason)
                }
            )
        })
    })
}
```



# 一些题目

### 声网

在做笔试的时候遇到一道比较有意思的题目。

给你一个peomise还有一个时间，实现函数，一旦没有在这个时间内完成就抛出错误的promise

```js
function withTimeout(promise,timeout){
    // 使用race返回现有结果的那个promise
    return Promise.race([
        promise,
        new Promise((resolve,reject) => {
            // 一旦超过时间，就返回一个失败的Promise
            setTimeout(()=>{
                reject(new Error('TIMEOUT'))
            },timeout)
        })
    ])
}
```

