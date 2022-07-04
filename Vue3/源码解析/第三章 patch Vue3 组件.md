# Patch Vue3 组件
## 前言
第二章 执行流程已经介绍了从 createApp 到 patchFn 的逻辑。本章接着执行流程讲 Vue3 组件的 diff 算法。
## 源码解析
processComponent: 
- 若不存在旧节点，调用 mountComponent 函数。
- 若存在旧节点，调用 updateComponent 函数。
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 1153
 */

const processComponent = (
    n1: VNode | null,
    n2: VNode,
    container: RendererElement,
    anchor: RendererNode | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    slotScopeIds: string[] | null,
    optimized: boolean
) => {
    n2.slotScopeIds = slotScopeIds
    if(n1 == null) {
        if(n2.shapeFlag & ShapeFlags.COMPONENT_KEPT_ALIVE) {
            ;(parentComponent!.ctx as KeepAliveContext).activate(
                n2,
                container,
                anchor,
                isSVG,
                optimized
            )
        } else {
            mountComponent(
                n2,
                container,
                anchor,
                parentComponent,
                parentSuspense,
                isSVG,
                optimized
            )
        }
    } else {
        updateComponent(n1, n2, optimized)
    }
}
~~~
### 挂载阶段
mountComponent:
- 创建 instance 实例。
- 调用 setupRenderEffect 函数。
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 1190
 */
const mountComponent: MountComponentFn = (
    initialVNode,
    container,
    anchor,
    parentComponent,
    parentSuspense,
    isSVG,
    optimized
) => {
    const compatMountInstance = __COMPAT__ && initialVNode.isCompatRoot && initialVNode.component
    const  instance: ComponentInternalInstance = compatMountInstance || (initialVNode.component = createComponentInstance(
        initialVNode,
        parentComponent,
        parentSuspense
    ))
    
    /* ... */

    // linenum : 1250
    setupRenderEffect(
        instance,
        initialVNode,
        container,
        anchor,
        parentSuspense,
        isSVG,
        optimized
    )
}
~~~
setupRenderEffect：
- 创建 componentUpdateFn 内部函数。
- 将 componentUpdateFn 包装为 update 函数，并赋值给 instance.update。
- 调用 update

componentUpdateFn:
#### 实例未挂载
* 调用 renderComponentRoot 函数。
* 调用 patch 函数 patch 子树。
* 调用 hooks 钩子。
* 标记实例未已挂载。
#### 实例已挂载
* 调用 renderComponentRoot 获取新子树。
* 调用 patch 函数 patch 旧子树和新子树。
* 调用 hooks 钩子。

~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 1300
 */
const setupRenderEffect: SetupRenderEffectFn = (
    instance,
    initialVNode,
    container,
    anchor,
    parentSuspense,
    isSVG,
    optimized
) => {
    const componentUpdateFn = () => {
        if(!instance.isMounted) {
            let vnodeHook: VNodeHook | null | undefined
            const {el, props} = initialVNode
            const {bm, m, parent} = instance
            const isAsyncWrapperVNode = isAsyncWrapper(intialVNode)

            toggleRecurse(instance, false)
            if(bm) {
                invokeArrayFns(bm)
            }
                
            /* ... */

            toggleRecurse(instance, true)
            if(el && hydrateNode) {
                
                /* ... */

            } else {
                
                /* ... */

                const subTree = ((instance.subTree) = renderComponentRoot(instance))
                
                /* ... */

                patch(
                    null,
                    subTree,
                    container,
                    anchor,
                    instance,
                    parentSuspense,
                    isSVG
                )
                if(__DEV__) {
                    endMeasure(instance, 'patch')
                }
                initialVNode.el = subTree.el
            }
            if(m) {
                queuePostRenderEffect(m, parentSuspense)
            }

            if(!isAsyncWrapperVNode && 
                (vnodeHook = props && props.onVnodeMounted)
            ){
                const scopedInitialVNode = initialVNode
                queuePostRenderEffect(
                    ()=>invokeVNodeHook(vnodeHook!, parent, scopedInitialVNode),
                    parentSuspense
                )
            }

            instance.isMounted = true
            initialVNode = container = anchore = null as any
        } else {
            let {next, bu, u, parent, vnode} = instance
            let originNext = next
            let vnodeHook: VNodeHook | null | undefined
                
                /* ... */

            toggleRecurse(intance, false)
            if(next) {
                next.el = vnode.el
                updateComponentPreRender(instance, next, optimized)
            } else {
                next = vnode
            }

            if(bu) {
                invokeArrayFns(bu)
            }

            if((vnodeHook = next.props && next.props.onVnodeBeforeUpdate)) {
                invokeVNodeHook()
            }
                
            /* ... */

            toggleRecurse(instance, true)
                
            /* ... */

            const nextTree = renderComponentRoot(instance)
                
            /* ... */

            const prevTree = instance.subTree
            instance.subTree = nextTree
                
            /* ... */

            patch(
                prevTree,
                nextTree,
                hostParentNode(prevTree.el!)!,
                getNextHostNode(prevTree),
                instance,
                parentSuspense,
                isSVG
            )
                
            /* ... */

            next.el = nextTree.el
            if(originNext === null) {
                updateHOCHostEl(instance, nextTree.el)
            }
            if(u) {
                queuePostRenderEffect(u, parentSuspense)
            }
            if((vnodeHook = next.props && next.props.onVnodeUpdated)){
                queuePostRenderEffect(
                    ()=>invokeVNodeHook(vnodeHook!, parent, next!, vnode),
                    parentSuspense
                )
            }
                            
            /* ... */

        }
    }
    const effect = (instance.effect = new ReactiveEffect(
        componentUpdateFn,
        () => queueJob(update),
        instance.scope
    ))
    const update: SchedulerJob = (instance.update = () => effect.run())
    update.id = instance.uid
    toggleRecurse(instance, true)
    update()
}
~~~
### 更新阶段
updateComponent:
* 通过 shouldUpdateComponent 函数判断是否需要更新。
* 若需要更新，调用 instance.update 方法。
* 若不需要更新，将旧节点替换为新节点。
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 1266
 */

const updateComponent = (n1: VNode, n2: VNode, optimized: boolean) => {
    const instance = (n2.component = n1.component)!
    if(shouldUpdateComponent(n1, n2, optimized)){
        if(
            __FEATURE_SUSPENSE__ &&
            instance.asyncDep &&
            !instance.asyncResolved
        ) {
            if(__DEV__) {
                pushWarningContext(n2)
            }
            updateComponentPreRender(instance, n2, optimized)
            if(__DEV__) {
                popWarningContext()
            }
            return
        } else {
            instance.next = n2
            invalidateJob(instance.update)
            instance.update()
        }
    } else {
        n2.el = n1.el
        instance.vnode = n2
    } 
}
~~~