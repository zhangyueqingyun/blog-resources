# Patch HTML 节点
processElement 
- 当源节点不存在时，直接挂载新节点。
- 当源节点存在时，调用 patchElement 函数
~~~typescript
const processElement = (
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
    isSVG = isSVG || (n2.type as string) === 'svg'
    if(n1 == null) {
        mountElement(
            n2,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
        )
    } else {
        patchElement(
            n1,
            n2,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
        )
    }
}
~~~
patchElement
- 触发钩子
- 更新属性
- 更新子节点
~~~typescript
const patchElement = (
    n1: VNode,
    n2: VNode,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    slotScopeIds: string[] | null,
    optimized: boolean
) => {
    const el = (n2.el = n1.el)
    let {patchFlag, dynamicChildren, dirs} = n2
    patchFlag |= n1.patchFlag & PatchFlags.FULL_PROPS
    const oldProps = n1.props || EMPTY_OBJ  
    const newProps = n2.props || EMPTY_OBJ
    let vnodeHook: VNodeHook | undefined | null

    parentComponent && toggleRecurse(parentComponent, false)
    if((vnodeHook = newProps.onVnodeBeforeUpdate)) {
        invokeVNodeHook(vnodeHook, parentComponent, n2, n1)
    }
    if(dirs) {
        invokeDirectiveHook(n2, n1, parentComponent, 'beforeUpdate')
    }
    parentComponent && toggleRecurse(parentComponent, true)

    const areChildrenSVG = isSVG && n2.type !== 'foreignObject'
    if(dynamicChildren) {
        patchBlockChildren(
            n1.dynamicChildren!,
            dynamicChildren,
            el,
            parentComponent,
            parentSuspense,
            areChildrenSVG,
            slotScopeIds
        )
    }
}
~~~

(今天先到这里，下次追更)