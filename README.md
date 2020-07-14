# 简答题

## 1、当我们点击按钮的时候动态给 data 增加的成员是否是响应式数据，如果不是的话，如果把新增成员设置成响应式数据，它的内部原理是什么。

``` javascript
let vm = new Vue({
    el: '#el'
    data: {
        o: 'object',
        dog: {}
    },
    method: {
        clickHandler() {
            // 该 name 属性是否是响应式的
            this.dog.name = 'Trump'
        }
    }
})
```

   不是响应式数据，当你把一个普通的 JavaScript 对象传入 Vue 实例作为 data 选项，Vue 将遍历此对象所有的 property，并使用 Object.defineProperty 把这些 property 全部转为 getter/setter。Object.defineProperty 是 ES5 中一个无法 shim 的特性，这也就是 Vue 不支持 IE8 以及更低版本浏览器的原因。

   这些 getter/setter 对用户来说是不可见的，但是在内部它们让 Vue 能够追踪依赖，在 property 被访问和修改时通知变更

   每个组件实例都对应一个 watcher 实例，它会在组件渲染的过程中把“接触”过的数据 property 记录为依赖。之后当依赖项的 setter 触发时，会通知 watcher，从而使它关联的组件重新渲染。

   Vue 无法检测 property 的添加或移除。由于 Vue 会在初始化实例时对 property 执行 getter/setter 转化，所以 property 必须在 data 对象上存在才能让 Vue 将它转换为响应式的。例如：

``` javascript
    var vm = new Vue({
        data: {
            a: 1
        }
    })

    // `vm.a` 是响应式的

    vm.b = 2
    // `vm.b` 是非响应式的
```

对于已经创建的实例，Vue 不允许动态添加根级别的响应式 property。但是，可以使用 Vue.set(object, propertyName, value) 方法向嵌套对象添加响应式 property。例如，对于：

``` javascript
Vue.set(vm.someObject, 'b', 2)
```

您还可以使用 vm.$set 实例方法，这也是全局 Vue.set 方法的别名：

``` javascript
this.$set(this.someObject, 'b', 2)
```

有时你可能需要为已有对象赋值多个新 property，比如使用 Object.assign() 或 _.extend()。但是，这样添加到对象上的新 property 不会触发更新。在这种情况下，你应该用原对象与要混合进去的对象的 property 一起创建一个新的对象。

``` javascript
// 代替 `Object.assign(this.someObject, { a: 1, b: 2 })`
this.someObject = Object.assign({}, this.someObject, {
    a: 1,
    b: 2
})
```

## 2、请简述 Diff 算法的执行过程

答：
  patch()函数
    对比新旧vnode返回一个vnode，作为下一次对比的旧节点
    createElm()函数
    触发用户钩子函数init
    把vnode转换成DOM对象，存储到vnode.elm中（注意：没有渲染到页面）	
    sel是！创建注释节点
    sel不为空：创建对应的DOM对象；触发模块的钩子函数create；创建所有节点对应的DOM对象；触发用户的钩子函数create；如果vnode有insert钩子函数，追加到队列
    sel为空：穿件文本节点
    返回vnode.elm
removeVnode()批量删除节点
addVnode()批量添加节点
patchVnode()
    触发prepatch钩子函数
    触发update钩子函数
    新节点有text属性，且不等于旧节点的text属性
        如果老节点有children，移除老节点children对应的DOM元素
        设置新节点对应DOM元素的textContent
    新老节点都有children，且不相等
        调用updateChildren()
        对比节点，并且更新子节点的差异
    只有新节点有children属性
        如果老节点有text属性，清空对应DOM元素的textContent
        添加所有子节点
    只有老节点有children属性
        移除所有的老节点
    只有老节点有text属性
        清空对用DOM元素的textContent
    触发postpatch钩子函数
uodateChildren()
功能
    diff算法的核心，对比新旧节点的children，更新DOM
    执行过程
    要对比两颗树的差异，我们可以取第一颗树的每一个节点一次和第二颗树的每一个节点比较，但是这样复杂度为O(n^3)
    在DOM操作的时候我们很少很少会把一个父节点移动或更新到某一个子节点
    因此只需要找同级别的子节点一次比较，然后在找下一级别的节点比较，这样算法的时间复杂度为0(n)
    在进行同一级别节点比较的时候，首先会对新老节点数组的开始和结尾节点这只标记索引，遍历的过程中移动索引
    在对开始和结束节点比较的时候，总共有四种情况
        oldstartVnode/newStartVnode（旧开始节点/新开始节点）
        oldEndVnode/newEndVnode （旧结束节点/新结束节点）
        oldStartVnode/newEndVnode （旧开始节点/新结束节点）
        oldEndVnode/newStartVnode（旧结束节点/新开始节点）
    开始节点和结束节点比较，这两种情况类似
    oldstartVnode/newStartVnode（旧开始节点/新开始节点）
    oldEndVnode/newEndVnode （旧结束节点/新结束节点）
    如果oldstartVnode和newStartVnode是sameVnode（key和sel相同）
        调用patchVnode对比和更新节点
        把旧节点和新开始索引往后移动oldStartIdx++/oldStartIdx++
    oldStartVnode/newEndVnode相同
        调用patchVnode对比更新节点
        把oldStartVnode对用的DOM元素，移动到右边
        更新索引
    oldEndVnode/newStartVnode（旧结束节点/新开始节点）相同
    调用patchVnode对比更新节点
    把oldEndVnode对用的DOM元素，移动到左边
    更新索引
    如果不是以上四种情况
        遍历新节点，使用newStartVnode的key在老节点数组中找到相同节点
        如果没有找到，说明newStartVnode是新节点
            创建新节点对应的DOM元素，插入到DOM树中
    如果找到了
        判断新节点和找到的老节点的sel选择器是否相同
        如果不同，说明节点被修改了
            重新创建对应的DOM元素，插入到DOM树中
        如果相同，把elmTOMove对应的DOM元素，移动到左边
    循环结束
        当老节点的所有子节点先遍历完（oldStartIdx>oldEndIdx），循环结束
        新节点的所有子节点先遍历完（newStartIdx>oldStartIdx），循环结束
    如果老节点的数组先遍历完，说明新节点有剩余，把剩余节点批量插入到右边
    如果新节点的数组先遍历完，说明老节点有剩余，把剩余节点批量删除

``` javascript
updateChildren(parentElm, oldCh, newCh) {

    let oldStartIdx = 0,
        newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx
    let idxInOld
    let elmToMove
    let before
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (oldStartVnode == null) { // 对于vnode.key的比较，会把oldVnode = null
            oldStartVnode = oldCh[++oldStartIdx]
        } else if (oldEndVnode == null) {
            oldEndVnode = oldCh[--oldEndIdx]
        } else if (newStartVnode == null) {
            newStartVnode = newCh[++newStartIdx]
        } else if (newEndVnode == null) {
            newEndVnode = newCh[--newEndIdx]
        } else if (sameVnode(oldStartVnode, newStartVnode)) {
            patchVnode(oldStartVnode, newStartVnode)
            oldStartVnode = oldCh[++oldStartIdx]
            newStartVnode = newCh[++newStartIdx]
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
            patchVnode(oldEndVnode, newEndVnode)
            oldEndVnode = oldCh[--oldEndIdx]
            newEndVnode = newCh[--newEndIdx]
        } else if (sameVnode(oldStartVnode, newEndVnode)) {
            patchVnode(oldStartVnode, newEndVnode)
            api.insertBefore(parentElm, oldStartVnode.el, api.nextSibling(oldEndVnode.el))
            oldStartVnode = oldCh[++oldStartIdx]
            newEndVnode = newCh[--newEndIdx]
        } else if (sameVnode(oldEndVnode, newStartVnode)) {
            patchVnode(oldEndVnode, newStartVnode)
            api.insertBefore(parentElm, oldEndVnode.el, oldStartVnode.el)
            oldEndVnode = oldCh[--oldEndIdx]
            newStartVnode = newCh[++newStartIdx]
        } else {
            // 使用key时的比较
            if (oldKeyToIdx === undefined) {
                oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx) // 有key生成index表
            }
            idxInOld = oldKeyToIdx[newStartVnode.key]
            if (!idxInOld) {
                api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                newStartVnode = newCh[++newStartIdx]
            } else {
                elmToMove = oldCh[idxInOld]
                if (elmToMove.sel !== newStartVnode.sel) {
                    api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                } else {
                    patchVnode(elmToMove, newStartVnode)
                    oldCh[idxInOld] = null
                    api.insertBefore(parentElm, elmToMove.el, oldStartVnode.el)
                }
                newStartVnode = newCh[++newStartIdx]
            }
        }
    }
    if (oldStartIdx > oldEndIdx) {
        before = newCh[newEndIdx + 1] == null ? null : newCh[newEndIdx + 1].el
        addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx)
    } else if (newStartIdx > newEndIdx) {
        removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
    }

}
```

    首先介绍下这个函数中的变量定义：

        (oldStartIdx = 0）：oldVnode 的 startIdx, 初始值为 0
        (newStartIdx = 0）：vnode 的 startIdx, 初始值为 0
        (oldEndIdx = oldCh.length - 1）：oldVnode 的 endIdx, 初始值为 oldCh.length - 1
        (oldStartVnode = oldCh[0]）：oldVnode 的初始开始节点
        (oldEndVnode = oldCh[oldEndIdx]）：oldVnode 的初始结束节点
        (newEndIdx = newCh.length - 1）：vnode 的 endIdx, 初始值为 newCh.length - 1
        (newStartVnode = newCh[0]）：vnode 的初始开始节点
        (newEndVnode = newCh[newEndIdx]）：vnode 的初始结束节点

    当 oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx 时，执行如下循环判断：

        1、oldStartVnode 为 null，则 oldStartVnode 等于 oldCh 的下一个子节点，即 oldStartVnode 的下一个兄弟节点
        2、oldEndVnode 为 null, 则 oldEndVnode 等于 oldCh 的相对于 oldEndVnode 上一个子节点，即 oldEndVnode 的上一个兄弟节点
        3、newStartVnode 为 null，则 newStartVnode 等于 newCh 的下一个子节点，即 newStartVnode 的下一个兄弟节点
        4、newEndVnode 为 null, 则 newEndVnode 等于 newCh 的相对于 newEndVnode 上一个子节点，即 newEndVnode 的上一个兄弟节点
        5、oldEndVnode 和 newEndVnode 为相同节点则执行 patchVnode(oldStartVnode, newStartVnode)，执行完后 oldStartVnode 为此节点的下一个兄弟节点，newStartVnode 为此节点的下一个兄弟节点
        6、oldEndVnode 和 newEndVnode 为相同节点则执行 patchVnode(oldEndVnode, newEndVnode)，执行完后 oldEndVnode 为此节点的上一个兄弟节点，newEndVnode 为此节点的上一个兄弟节点
        7、oldStartVnode 和 newEndVnode 为相同节点则执行 patchVnode(oldStartVnode, newEndVnode)，执行完后 oldStartVnode 为此节点的下一个兄弟节点，newEndVnode 为此节点的上一个兄弟节点
        8、oldEndVnode 和 newStartVnode 为相同节点则执行 patchVnode(oldEndVnode, newStartVnode)，执行完后 oldEndVnode 为此节点的上一个兄弟节点，newStartVnode 为此节点的下一个兄弟节点
        9、使用 key 时的比较：
        oldKeyToIdx为未定义时，由 key 生成 index 表，具体实现为 createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)，createKeyToOldIdx 的代码如下：

``` javascript
function createKeyToOldIdx(children: Array < VNode > , beginIdx: number, endIdx: number): KeyToIndexMap {
    let i: number, map: KeyToIndexMap = {},
        key: Key | undefined, ch;
    for (i = beginIdx; i <= endIdx; ++i) {

        ch = children[i];
        if (ch != null) {
            key = ch.key;
            if (key !== undefined) map[key] = i;
        }

    }
    return map;
}
```

    createKeyToOldIdx 方法，用以将 oldCh 中的 key 属性作为键，而对应的节点的索引作为值。然后再判断在 newStartVnode 的属性中是否有 key，且是否在 oldKeyToIndx 中找到对应的节点。
        如果不存在这个 key，那么就将这个 newStartVnode 作为新的节点创建且插入到原有的 root 的子节点中，然后将 newStartVnode 替换为此节点的下一个兄弟节点。
        如果存在这个key，那么就取出 oldCh 中的存在这个 key 的 vnode，然后再进行 diff 的过程，并将 newStartVnode 替换为此节点的下一个兄弟节点。
    当上述 9 个判断执行完后，oldStartIdx 大于 oldEndIdx，则将 vnode 中多余的节点根据 newStartIdx 插入到 dom 中去；newStartIdx 大于 newEndIdx，则将 dom 中在区间 【oldStartIdx， oldEndIdx】的元素节点删除
