```
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class MyPromise {
  constructor(executor) {
    try {
      executor(this.resolve, this.reject);
    } catch (e) {
      this.reject(e);
    }
  }

  // 储存状态的变量，初始值是 pending
  state = PENDING;
  // 成功之后的值
  value = null;
  // 失败之后的原因
  reason = null;

  // 存储成功、失败回调函数
  // 用数组存储，因为可能对同一个promise调用多次then，每次都传入不同的回调函数
  onFulfilledCallbacks = [];
  onRejectedCallbacks = [];

  // resolve和reject为什么要用箭头函数？
  // 如果直接调用的话，普通函数this指向的是window或者undefined
  // 用箭头函数就可以让this指向当前实例对象（在箭头函数中，this引用的是定义箭头函数的上下文）

  // 这两个函数是MyPromise传到执行器里的
  resolve = value => {
    if (this.state === PENDING) {
      this.state = FULFILLED;
      this.value = value;

      // 如果之前有保存传递进then的回调方法，就直接执行它们
      while (this.onFulfilledCallbacks.length > 0) {
        this.onFulfilledCallbacks.shift()(value);
      }
    }
  };

  reject = reason => {
    if (this.state === PENDING) {
      this.state = REJECTED;
      this.reason = reason;

      while (this.onRejectedCallbacks.length > 0) {
        this.onRejectedCallbacks.shift()(reason);
      }
    }
  };

  // then不需要箭头函数，因为then调用时就是Promise.prototype.then的调用
  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw new reason };

    // 1、要实现then的链式调用，就要让then在内部返回一个新的Promise(Promise2)实例（then()在状态改变时执行，但其返回值在定义时就已经返回了，即返回一个pending状态的Promise2）
    // 2、Promise2的状态和最外层Promise（Promise1）的状态是不相关的，唯一确定的是只有Promise1落定后，Promise2才会落定
    // 3、Promise2内部逻辑的主要工作，就是设定其状态变化时机，也就是在正确的时候调用Promise2的resolve和reject方法
    // 3.1、所谓正确的时候，即：当then1返回普通值res时，调用Promise2的resolve(res)
    // 3.2、当then1返回promise时（和子promise不是同一个，称其为promise3）时，设置（也算调用，只是不一定会用）promise3的then方法，并传入promise2的resolve和reject，作为then3的参数
    // 3.2.1、上条意思是：当promise3落定时，触发promise2的resolve或reject，从而触发promise2的then方法，也就是链式调用的then方法
    // 以上就是链式调用原理

    // 在箭头函数中，this引用的是定义箭头函数的上下文, 即外部的MyPromise实例，而非innerPromise
    const resPromise = new MyPromise((resolve, reject) => {
      // 这是执行器的内容，会在then方法调用时作为同步函数内容立即执行

      // 执行then时，如果promise已经是落定状态，就直接执行onFulfilled，否则就保存回调函数
      if (this.state === FULFILLED) {

        // 获取onFulfilled的返回值，其将作为promise2的返回值
        // 若为promise类型，就原样传递，如果是pending状态则then2不会执行，否则会执行
        // 若为普通值，就会转化为promise，并且会自动调用resolve(thenRes)(就算是返回promise，也会被resolve包裹的，只是没什么用)
        queueMicrotask(() => {
          try {
            // 创建微任务，防止resPromise尚未创建，值为undefined
            const thenRes = onFulfilled(this.value);
            handleSubPromise(resPromise, thenRes, resolve, reject);
          } catch (e) {
            // 注意，前一个then中抛出的异常，被捕捉后，要在下一个then中的onRejected才会有相关展示，异常内容会作为参数传入onRejected
            reject(e);
          }
        })
      } else if (this.state === REJECTED) {
        queueMicrotask(() => {
          try {
            // 创建微任务，防止resPromise尚未创建，值为undefined
            const thenRes = onRejected(this.reason);
            handleSubPromise(resPromise, thenRes, resolve, reject);
          } catch (e) {
            reject(e);
          }
        })
      } else if (this.state === PENDING) {
        // 当状态尚未改变时调用then，将成功回调和失败回调存储起来
        // 等到执行resolve或者reject方法时，就可直接调用存储起来的回调
        this.onFulfilledCallbacks.push(() => {
          try {
            // 创建微任务，防止resPromise尚未创建，值为undefined
            const thenRes = onFulfilled(this.value);
            handleSubPromise(resPromise, thenRes, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
        this.onRejectedCallbacks.push(() => {
          try {
            // 创建微任务，防止resPromise尚未创建，值为undefined
            const thenRes = onRejected(this.reason);
            handleSubPromise(resPromise, thenRes, resolve, reject);
          } catch (e) {
            reject(e);
          }
        });
      }
    });

    return resPromise;
  }

  static resolve (value) {
    return new MyPromise((resolve, reject) => {
      resolve(value);
    })
  }

  static reject (reason) {
    return new MyPromise((resolve, reject) => {
      reject(reason);
    })
  }
}

function handleSubPromise(resPromise, thenRes, resolve, reject) {
  // 判断是否是循环引用
  if (resPromise === thenRes) {
    return reject(new TypeError('Chaining cycle detected for promise #<Promise>'))
  }
  // 判断x是promise还是普通值
  if (thenRes instanceof MyPromise) {
    thenRes.then(resolve, reject);
  } else {
    resolve(thenRes);
  }
}

export default MyPromise;

```

