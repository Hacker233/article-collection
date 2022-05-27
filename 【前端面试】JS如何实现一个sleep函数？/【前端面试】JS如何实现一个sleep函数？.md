# 前言

我们都是 JavaScript 是一个单线程语言，单线程有它的好处也有它的坏处。在我们熟知的如 Java、C++等语言中，都提供了一个叫做 Sleep 的内置函数。这个函数的作用就和它的名字一样：**睡眠**。

假设我们有这样一个场景：我们需要在项目运行起来后的十分钟之后去执行一段代码，这段代码可以是符合你业务场景的任何代码，比如查看内存占用多少等等。

在 Java 这类语言中，可以直接使用 Sleep 这个内置函数实现这个需求，Sleep 函数会让出或者停下当前线程，让其它程序先执行，到底指定时间后在继续执行。

然而我们的 JavaScript 没有提供 sleep 内置函数，大致就是因为单线程的原因把！所以说我们可以尝试着自己封装一个！

# 1.目标分析

既然我们要去实现一个 sleep 函数，那么我们肯定要先有一个比较实际的场景，这样才好开展工作。

假设我们有如下一段代码：

```javascript
<script>
  function fnA() {
    console.log('A');
  }
  function fnB() {
    console.log('B');
  }
  function fnC() {
    console.log('C');
  }

  // 实现目标
  fnA(); // 1 秒后打印
  fnB(); // 2 秒后打印
  fnC(); // 3 秒后打印
</script>
```
上段代码非常简单，我们的目的就是为了让它们几个分别间隔 1 秒打印，需求非常简单，实现起来也很容易，可能有些小伙伴直接想到了 setTimeout。
确实，setTimeout 可以实现我们的需求，比如下面的代码：

```javascript
setTimeout(fnA, 1000);
setTimeout(fnB, 2000);
setTimeout(fnC, 3000);
```
定时器确实可以满足我们的需求，但是如果项目中到处些定时器的可能会让人很疑惑，所以我们有必要进行封装，写一个自己的 sleep 函数，大家多看几种实现方式应该就会豁然开朗了。
# 2.setTimeout 封装

这是大家最容易想到也是最暴力的一种方式，毕竟一提到延时执行大家都会想到定时器，所我们直接利用 setTimeout 的这个特性来实现我们的 sleep 函数。

**代码如下：**

```javascript
<script>
  function fnA() {
    console.log('A');
  }
  function fnB() {
    console.log('B');
  }
  function fnC() {
    console.log('C');
  }
  // sleep 函数
  function sleep(fun, time) {
    setTimeout(() => {
      fun();
    }, time);
  }
  sleep(fnA, 1000); // 1 秒后输出 A
  sleep(fnB, 2000); // 2 秒后输出 B
  sleep(fnC, 3000); // 3 秒后输出 C
</script>
```
这是最原始的一种方式，其实本质就是定时器，只不过我们封装成一个函数罢了。
这种实现方式有如下优缺点：

**优点：**

简单易实现，兼容性好，毕竟只是用了 setTimeout，而且非常好理解。

**缺点：**

我们需要传入回调函数的方式进去，如果函数里面有多回调函数可能不太好理解。另外一点就是它不会阻塞同步任务，比如下面代码的输出结果：

```javascript
sleep(fnA, 1000);
console.log('E');
sleep(fnB, 2000);
console.log('G');
sleep(fnC, 3000);
```
**输出结果：**
![图片](data:image/png;base64,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)


在有些场景下我们可能需要 sleep 函数会阻塞代码，依次执行，这个时候这种封装就满足不了了。

# 3.Promise 封装

promise 是 ES6 提出的一种异步解决方案，它和 setTimeout 一样，都可以实现异步，区别在于 promise 解决了回调函数的问题，它可以实现链式调用，我们可以接触 promise 来实现 sleep 函数。

**代码如下：**

```javascript
<script>
  function fnA() {
    console.log('A');
  }
  function fnB() {
    console.log('B');
  }
  function fnC() {
    console.log('C');
  }

  // sleep 函数--Promise 版本
  function sleep(time) {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve();
      }, time);
    });
  }
  sleep(1000).then(fnA); // 1 秒后输出 A
  sleep(2000).then(fnB); // 2 秒后输出 B
  sleep(3000).then(fnC); // 3 秒后输出 C
</script>
```
上段代码中利用 promise 实现了 sleep 函数，其实是 promise 和 setTimeout 的结合，不过上段代码中我们可以进行链式调用了，不必再往 sleep 函数中传入回调函数了。
**优点：**

不用再传入回调函数，采用链式调用。

**缺点：**

仍未解决阻塞问题，依然会先执行同步任务，**代码如下：**

```javascript
sleep(1000).then(fnA); // 1 秒后输出 A
console.log('E');
sleep(2000).then(fnB); // 2 秒后输出 B
console.log('G');
sleep(3000).then(fnC); // 3 秒后输出 C
```
**输出结果：**
![图片](data:image/png;base64,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)


# 4.async/await

前面两个封装中我们一直提及阻塞问题，那么既然我们使用了 promise，我们就很有必要将 async/await 拿出来使用，它们可以完美的阻塞我们的代码，然我们的代码依次执行。

**代码如下：**

```javascript
<script>
  function fnA() {
    console.log('A');
  }
  function fnB() {
    console.log('B');
  }
  function fnC() {
    console.log('C');
  }
  // sleep 函数--Promise 版本
  function sleep(time) {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve();
      }, time);
    });
  }
  async function sleepTest() {
    fnA();                // 输出 A
    await sleep(1000);    // 睡眠 1 秒
    console.log('E');     // 输出 E
    fnB();                // 输出 B
    await sleep(1000);    // 睡眠 1 秒
    fnC();                // 输出 C
    await sleep(1000);    // 睡眠 1 秒
    console.log('G');     // 输出 G
  }
  sleepTest();
</script>
```
**输出结果：**
![图片](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAX8AAADnCAYAAAD2Blb9AAAJ60lEQVR4Ae3BX2zV932A4ZePDznhj2ztwiqZiDobXE12hnRMieIIabayhIhB0GgN7UKjrSlMYem6MFWjlmjXIHlpqiUXTSOtbpBCszWxVU8pSRZIkemEzBRSW6W1p2Jsa8JqmHyRHSAmJ8hjpC2ZBwcWSzD8O9/3eeYVz5y9gCQpKblfHB9FkpSWecUzZy9UL17E0YFjrGpegSSp8gWSpOQEkqTkBJKk5MR/vP1LJElpCSRJyQkkSckJJEnJCSRJyQkkSckJJEnJCSRJyQkkSckJKsDQt9qoq2+grn4Te/8dSdL/Ici66WH6eiZo2vYwGxik+9AokqRrC7Ju8Ic8M9HIpvbttK2Fod4jnECSdC1Bxg386AVKS9ewclk1d7e2ws966B9FknQNQZZND/L6CyVub2+lCahdfR9tDNN9eBRJ0tUFWTa4n71Tee5vaeRXlrTQWoCh5/YzhCTpaoLMKtH/6guUFm7h/gK/sZS7H2iEiR76hpEkXUWQVaUB9veUyLf/Pk1VfGj56naamKD7x8NIksrLkVGlowd4aQpKzz/E7z7Plb5/iKFHGmlCknS5IJNK/ORgDyWuYaKHvmEkSWUEWVQaYH9PCdZ/m5+OjTA+NsL42AjjYyOMj40w/vJj3M4E3QeHkSRdKcig0pED7J2CDWvuopoyGlu4fyGc7DnE0DSSpMsEmVOi/+ALwH20tVRTVlWB+7fkYaKHvmEkSZeZd/z48QsNDQ0cHTjGquYVSJIqXyBJSk4gSUpOIElKTiBJSk4gSUpOIElKTiBJSk4gSUpOIElKTo4ZisUikqTKl2OGmpoaJEmVL5AkJSeQJCUnkCQlJ5AkJSeQJCUnkCQlJ5AkJSeQJCUnPnbbbyNJSksgSUpOIElKTo7MGqSzfhNdXMW2bsZ3FpAkXSmQJCUnR9a1dvLmnnZqkSR9VIEkKTmBJCk5ObLuUAd31ndwubYn+9nz6VokSVcKJEnJyZF1rZ28uaedWiRJH1UgSUpOIElKTiBJSk6OrDvUwZ31HVxhWzfjOwtIkq4USJKSkyOzCnSMjdCBJGm2AklScgJJUnICSVJyAklScgJJUnICSVJyAklScgJJUnJyzFAsFpEkVb4cM9TU1CBJqnyBJCk5gSQpOYEkKTmBJCk5gSQpOYEkKTmBJCk5gSQpOYEkKTmBJCk5gSQpOTmy6q0nqNv0HP/Lwlqamu9iw7avsHV1LZKk8oJKMjXJ0OF9dD7Uxl++dhpJUnlBxm3tHmF8bITxsRHGR37Ov3RtYTklXt7/r5xGklROUEmq8tx+zwO0IUm6lhyVZGqCgeefZS95Nqy5i2okSeXkyLiuTQ10MUNtC4/u7eOvVlcjSSovqDSTR3jmS3/D3uMlJEnlBRm3tXuE8bERxsdGGP+3ft7o2k5z6QBf+8JzDE0jSSojqCT5Wpbf8xjf+HIjTPTQ9wskSWUEFaua6jySpDKCSjJd4uThp/nrbw7D0jWs/B0kSWXkyLiuTQ10cZmFjTz61MM0VSFJKiNHBckvaaTtgS1s/ZN2mpcgSbqKHFn1yZ2Mj+1EkjR7gSQpOYEkKTmBJCk5gSQpOYEkKTmBJCk5gSQpOYEkKTmBJCk5OWYoFotIkipfjhlqamqQJFW+QJKUnECSlJxAkpScQJKUnECSlJxAkpScQJKUnECSlJxAkpScQJKUnECSlJwg66Yn6f/ODh68dyV19Q3U1a/k3od20HV4EklSeTmybGIfj2zYwevvMMNpThzeR+fhM1T3d7F5CZKky+TIqtIgf/enO3j9HVj+6a/xjT9vp/njeT5Qmhyl/8UeTiNJKidHRp0+8F2eGYX8+qf4x79dT20VH8rXLqPtizuRJJUXZNIkfT88ADTS8Rfrqa1CkjQLQSZNMHSQi1q4YxmSpFkKJEnJCSRJyQkyqZqlv8dFR/j5KJKkWQoyaRkr/2ApMMx3v3+EEpKk2Qgyqql9O23AyT1b+VRHD0OnSlxSOjVK37ee4OVTSJLKmFc8c/ZC9eJFHB04xqrmFWTJydc6+PyjPZygnFae6O9i8xIkSZcJMuz2tZ280b+Xrz94H01L8vzKwlqa1jzME92dbF6CJKmMecUzZy9UL17E0YFjrGpegSSp8gWSpOQEkqTkBJKk5ASSpOQEkqTkBJKk5ASSpOQEkqTkBJKk5OSYoVgsIkmqfDlmqKmpQZJU+QJJUnICSVJyAklScgJJUnICSVJyAklScgJJUnICSVJyAklScgJJUnICSVJygqw7PUrfd3bw4L0rqatvoK7+Du7ctIOug6OcnkaSVEaODCv97Fn++LNPMzDFDCUm39pH51v7mOweoeOTSJIukyOrTu3jkc8+zcBUnqbP7OTxP2un+eN5PlCaHKX/n77NiSokSWXkyKQS/X//FfqmYPm27/GDnQXy/I987TLatj1FG5KkcoIsKh1h//MlWLiFrz9WII8kaTaCLJqY4CdcdE8Ld+T5tVM9fL6+gbr6BurqG6irb6DzLSRJZQRZ9M4EQ1x0Wy3VSJJmK8iiqjx5Lnq3SInfWNLOnrERxsdGePPJViRJVxdk0dLl3M1FPx7mxDSSpFkKsqi2QGsBmHiWrtcmkSTNTpBJS9n8xS3kKfHylz7Hzn8Y5OQUHyq+W0KSdHU5MirfupMffHmIT31zkJd2beKlXUiSPqIgs/I0PdLNm2900fFHLSz/LX5tYS1Nq9fz6JPdbC0gSSpjXvHM2QvVixdxdOAYq5pXIEmqfIEkKTmBJCk5gSQpOYEkKTmBJCk5gSQpOYEkKTk5ZigWi0iSKl+OGWpqapAkVb5AkpScQJKUnECSlJxAkpScQJKUnECSlJxAkpScQJKUnECSlJxAkpSc4AZ46rX3OP72NJKkuSm4zl7sf5+XjrzP9j1THH97GknS3BNcZ8dPTfOBM+9d4HPPvssrg+eRJM0twXX21Y0L+MPCfC7Z3XuOVwbPI0maO4Ib4KsbF7C55RYu2d17jlcGzyNJmhuCG2TH2lvZtXEBl+zuPcfT//wekqSbL7iB1hXms2vjAi55sf99dveeQ5J0cwX/zy4gSbrZctxArwyeZ3fvOS7Z3HILO9beiiTp5spxgzzee45XB89zya6NC1hXmI8k6eYLboDHe8/x6uB5Ltm1cQHrCvORJM0NwXX2eO85Xh08zyW7Ni5gXWE+kqS5I7jOPrGkig8svnUe39u+iHWF+UiS5pYc19ln7r6FX/7nf7GuMJ9P3FaFJGnuyXED7Fh7K5KkuSuQJCUnkCQlJ5AkJSeQJCUnkCQlJ5AkJSeQJCUnkCQlJ5AkJSeQJCUnkCQlJ5AkJSeQJCUnkCQlJ5AkJSeQJCUnkCQlJ5AkJSeQJCUnkCQlJ5AkJSeQJCUnkCQlJ5AkJSeQJCXnvwENzS1khIAqjwAAAABJRU5ErkJggg==)


上段代码中我们封装的 sleep 函数并没有改变，只是我们调用 sleep 函数的使用采用了 async/await 的方式调用，这就很好的解决了我们程序没有阻塞的额问题了。

# 总结

实现 sleep 函数其实非常简单，主要是理解 JavaScript 中异步执行情况。虽然上面的代码中使用 await sleep()的方式看起来最像一个真正的 sleep 函数，但是凡事都有两面性，比如我们有些场景只是需要一定时间后执行某个函数，不想阻塞代码的执行，这个时候 setTimeout 和 promise 都是非常好的选择，但有时候我们就是需要阻塞代码的执行，比如后面的代码用到了前面这个函数的返回结果，这个时候 async/await 就是一个很好的选择了。

