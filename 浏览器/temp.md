**`window.requestAnimationFrame()`**  要求浏览器在下次重绘之前调用指定的回调函数



当执行栈为空时，requestAnimationFrame()会不断执行

当执行栈执行很长的js任务时，requestAnimationFrame()会中断，等js执行完后再执行



当前宏任务执行完毕后，会执行微任务直到清空微任务队列，执行GUI渲染，然后开始下一个宏任务



经测试，多次onclick执行一个长js任务时，多次任务之间没有执行渲染（requestAnimationFrame()未执行），因此onclick是微任务？（onclick与主js进程之间也没渲染）

作为对比，setTimeout之间会执行渲染（requestAnimationFrame()有执行）

```js
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<style>
  @keyframes myfirst {
    from {
      transform: rotate(0deg);
    }

    to {
      transform: rotate(360deg);
    }
  }

  .c1 {
    width: 500px;
    height: 100px;
    background-color: antiquewhite;
  }

  .c2 {
    width: 100px;
    height: 100px;
    background-color: rgb(175, 117, 40);
  }

  .c3 {
    width: 100px;
    height: 100px;
    background-color: rgb(245, 146, 17);
    animation:myfirst 1s linear 3;
    /* animation-iteration-count: infinite; */
  }

  .c4 {
    width: 100px;
    height: 100px;
    background-color: rgb(177, 245, 17);
  }

  .changeColor {
    background-color: aquamarine;
  }
</style>

<body>
  <div class="c1">
    点击开始长js任务
  </div>
  <div class="c2">点击进行重绘</div>
  <div class="c3">我是动画</div>
  <div class="c4">开始requestAnimationFrame</div>
</body>

<script>
  function myLongTask(from) {
    if (from) {
      console.log(`from ${from}`)
    }
    console.log('beginTime', new Date());
    let i = 100000;
    while (i > 0) {
      let j = 100000;
      while (j > 0) {
        j -= 1;
      }
      i -= 1;
    }
    console.log('endTime', new Date());
  }
  
  const elem1 = document.querySelector('.c1');
  elem1.addEventListener('click', function (e) {
    elem1.style.backgroundColor = 'yellow';
    myLongTask('click');
    setTimeout(myLongTask, 0, 'setTimeout');
  });

  const elem2 = document.querySelector('.c2');
  elem2.addEventListener('click', function (e) {
    elem2.className += ' changeColor';
    // if (elem2.style.backgroundColor === 'grey') {
    //   elem2.style.backgroundColor = 'yellow';
    // } else {
    //   elem2.style.height = '300px';
    //   elem2.style.backgroundColor = 'grey';
    // }
  });

  const elem4 = document.querySelector('.c4');
  function runAni() {
    console.log('requestAnimationFrame execute')
    window.requestAnimationFrame(runAni);
  }
  elem4.addEventListener('click', function (e) {
    runAni();
  });
  
</script>
</html>
```

