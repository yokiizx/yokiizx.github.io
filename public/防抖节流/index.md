# 防抖节流


诸如 scroll 之类的高频事件，或者恶意疯狂点击，往往需要防抖节流来帮我们做一些优化。

## 防抖

如果事件再设定事件内再次触发，就取消之前的一次事件。

```js
function debounce(fn, wait) {
    let timeout = null
    return function () {
        if (timeout) clearTimeout(timeout)
        const args = [...arguments]
        timeout = setTimeout(() => {
            fn.apply(this, args)
        }, wait)
    }
}
```

是否立即先执行一次版：

```js
/**
 * @description:        防抖函数
 * @param  {Function}:  fn        - 需要添加防抖的函数
 * @param  {Number}:    wait      - 延迟执行时间
 * @param  {Boolean}:   immediate - true,立即执行,wait内不能再次执行;false,wait后执行
 * @return {Function}:  function  - 返回防抖后的函数
 * @author: yk
 */
function debounce(fn, time, immediate) {
    let timeout
    return (...args) => {
        if (timeout) clearTimeout(timeout)
        if (immediate) {
            if (!timeout) fn.apply(this, args)
            timeout = setTimeout(() => {
                timeout = null
            }, time)
        } else {
            timeout = setTimeout(() => {
                fn.apply(this, args)
            }, time)
        }
    }
}
```

## 节流

事件高频触发，让事件根据设定的时间，按照一定的频率触发。

```js
// timeout 版本
function throttle(fn, time) {
    let timeout
    return (...args) => {
        if (timeout) return
        /** 若是想先执行把 fn.apply 提到这里即可 */
        timeout = setTimeout(() => {
            fn.apply(this, args)
            timeout = null
            clearTimeout(timeout)
        }, time)
    }
}
```

```js
// 时间戳版本
function throttle(fn, time) {
    /** 若想先执行，把 prev 设为 0 即可 */
    let prev = new Date()
    return (...args) => {
        if (new Date() - prev > time) {
            fn.apply(this, args)
            prev = new Date()
        }
    }
}
```

