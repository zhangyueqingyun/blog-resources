# Patch HTML 节点
processElement 
- 当源节点不存在时，直接挂载新节点。
- 当源节点存在时，调用 patchElement 函数
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 573
 */

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
mountElement
#### vnode.el 已存在
- 调用 hostCloneNode 克隆元素并挂载在 vnode.el 上
#### vnode.el 不存在
- 调用 hostCreateElement 创建元素并挂载在 vnode.el 上。
- 若为文本类型，调用 hostSetElementText 函数。
- 若为后代节点的数组类型，调用 mountChildren 函数。
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 609
 */
const mountElement = (
    vnode: VNode,
    container: RendererElement,
    anchor: RendererNode | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    slotScopeIds: string[] | null,
    optimized: boolean
) {
    let el: RendererElement
    let vnodeHook: VNodeHook | undefined | null
    const {type, props, shapeFlag, transition, patchFlag, dirs} = vnode
    if(!__DEV__ && vnode.el && hostCloneNode !== undefined && patchFlag === PatchFlags.HOSTED) {
        el = vnode.el = hostCloneNode(vnode.el)
    } else {
        el = vnode.el = hostCreateElement(
            vnode.type as string,
            isSVG,
            props && props.is,
            props
        )

        if(shapeFlag & ShapeFlags.TEXT_CHILDREN) {
            hostSetElementText(el, vnode.children as string)
        } else if (shapeFlag & ShapeFlags.ARRAY_CHILDREN){
            mountChildren(
                vnode.children as VNodeArrayChildren,
                el,
                null,
                parentComponent,
                parentSuspense,
                isSVG && type !== 'foreignObject',
                slotScopeIds,
                optimized
            )
        }

        if(dirs) {
            invokeDirectiveHook(vnode, null, parentComponent, 'created')
        }

        if(props) {
            for(const key in props) {
                if(key !== 'value' && !isReservedProps(key)) {
                    hostPatchProp(
                        el,
                        key,
                        null,
                        props[key],
                        isSVG,
                        vnode.children as VNode[],
                        parentComponent,
                        parentSuspense,
                        unmountChildren
                    )
                }
            }

            if('value' in props) {
                hostPatchProp(el, 'value', null, props.value)
            }
            if((vnodeHook = props.onVnodeBeforeMount)){
                invokeVNodeHook(vnodeHook, parentComponent, vnode)
            }
        }

        setScopeId(el, vnode, vnode.scopeId, slotScopeIds, parentComponent)
    }

    /* ... */

    hostInsert(el, container, anchor)

    /* ... */
}
~~~
patchElement
- 触发钩子
- 更新属性
- 更新子节点
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 1153
 */
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