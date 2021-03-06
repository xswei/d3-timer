# d3-timer

这个模块提供了一个高效的队列，能管理上千并发动画同时保证与并发或分段动画一致的同步时序。在内部使用 [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) 进行 [fluid animation](https://en.wikipedia.org/wiki/Fluid_animation)(如果支持的话)，否则切换使用 [setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setTimeout)来实现。

## Installing

NPM 安装: `npm install d3-timer`. 此外还可以下载 [latest release](https://github.com/d3/d3-timer/releases/latest)。也可以直接从 [d3js.org](https://d3js.org) 加载 [standalone library(单独的标准库)](https://d3js.org/d3-timer.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分加载。支持 AMD, CommonJS 以及基础的标签引入形式，如果使用标签引入会暴露一个 `d3` 全局变量:

```html
<script src="https://d3js.org/d3-timer.v1.min.js"></script>
<script>

var timer = d3.timer(callback);

</script>
```

[在浏览器中测试 d3-timer.](https://tonicdev.com/npm/d3-timer)

## API Reference

<a name="now" href="#now">#</a> d3.<b>now</b>() [<>](https://github.com/d3/d3-timer/blob/master/src/timer.js#L15 "Source")

返回由 [performance.now](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now) 定义的当前时间(如果支持的话), 否则返回 [Date.now](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Date/now)。当前时间在第一帧时被更新，在每一帧过程中保持不变，同一帧期间调用任何定时器都会被同步执行。如果在帧外部调用此方法(比如响应用户事件),则计算当前时间然后固定到下一帧以确保事件处理期间的时间一致。

<a name="timer" href="#timer">#</a> d3.<b>timer</b>(<i>callback</i>[, <i>delay</i>[, <i>time</i>]]) [<>](https://github.com/d3/d3-timer/blob/master/src/timer.js#L52 "Source")

定义一个新的定时器，然后重复执行指定的 *callback* 直到定时器被 [stopped](#timer_stop)。可选的数值类型 *delay* 单位为毫秒可以用来指定调用 *callback* 的延迟时间，如果没有指定 *delay* 则默认为 0。延迟时间相对于指定的 *time* (ms)，如果没有指定 *time* 则默认为 [now](#now).

*callback* 参数为 *elapsed* 以表示自从定时器被激活后过去的时间(表面上的)。例如:

```js
var t = d3.timer(function(elapsed) {
  console.log(elapsed);
  if (elapsed > 200) t.stop();
}, 150);
```

输出大概如下(不同的环境不一样):

```
3
25
48
65
85
106
125
146
167
189
209
```

(确切的值取决于JavaScript运行时以及计算机的具体任务.) 注意的是第一次打印的 *elapsed* 时间为3ms: 表示定时器启动时过去的时间，而不是定时器被定义的时间。由于指定了延时，因此定时器在定义150ms后执行。*elapsed* 时间可能小于真实的 *elapsed* 时间，比如当页面处于未激活也就是 [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) 暂停执行时会出现这种情况，页面非激活状态时，*elapsed* 表示的时间会被暂时冻结。 

如果 [timer](#timer) 在其他的定时器内部被调用，则新的定时器回调会在当前帧结束时立即调用(当然要符合指定的 *delay* 和 *time*)，而不是等到下一帧。在同一帧内部，定时器回调会安装先后次序调用，而不管其开始时间。

<a name="timer_restart" href="#timer_restart">#</a> <i>timer</i>.<b>restart</b>(<i>callback</i>[, <i>delay</i>[, <i>time</i>]]) [<>](https://github.com/d3/d3-timer/blob/master/src/timer.js#L31 "Source")

指定 *callback* 并重新启动定时器，也可以指定 *delay* 和 *time* 选项。等价于停止之前的定时器并重新定义一个新的定时器，尽管此定时器保留了原始调用的优先级。

<a name="timer_stop" href="#timer_stop">#</a> <i>timer</i>.<b>stop</b>() [<>](https://github.com/d3/d3-timer/blob/master/src/timer.js#L43 "Source")

停止定时器并阻止后续的回调执行。如果定时器已经停止的话则这个方法没有任何影响。

<a name="timerFlush" href="#timerFlush">#</a> d3.<b>timerFlush</b>() [<>](https://github.com/d3/d3-timer/blob/master/src/timer.js#L58 "Source")

立刻调用符合条件的定时器回调。要注意的是定时器会在下一帧执行，因此会有不大于 17ms 的延迟。这可能会导致页面闪烁，因为浏览器被渲染两次: 一次是在第一次事件循环结束时，另一次是在定时器回调第一次执行时。可以通过在第一个事件循环结束时刷新定时器队列，你可以启用一个零延迟的定时器以避免闪烁。

<a name="timeout" href="#timeout">#</a> d3.<b>timeout</b>(<i>callback</i>[, <i>delay</i>[, <i>time</i>]]) [<>](https://github.com/d3/d3-timer/blob/master/src/timeout.js "Source")

与 [timer](#timer) 类似，但是在执行完第一次回调之后会自动 [stops](#timer_stop)。可以作为 [setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setTimeout) 的替换，因为这个方法不会在页面处于非活动状态时运行(内部使用requestAnimationFrame实现)。*callback* 参数为 *elapsed* (定义定时器到回调执行之间的时间)。

<a name="interval" href="#interval">#</a> d3.<b>interval</b>(<i>callback</i>[, <i>delay</i>[, <i>time</i>]]) [<>](https://github.com/d3/d3-timer/blob/master/src/interval.js "Source")

与 [timer](#timer) 类似，但是 *delay* 在此处的含义为每执行完一次 *callback* 后下一次 *callback* 会在 *delay* 后执行，如果没有指定 *delay* 则功能与 [timer](#timer) 一样。这个方法可以用来替换 [setInterval](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval) 因为基于 *requestAnimationFrame* 实现，不会在后台执行。*callback* 的参数为从定义定时器开始到当前回调执行所经历的时间间隔大小。
