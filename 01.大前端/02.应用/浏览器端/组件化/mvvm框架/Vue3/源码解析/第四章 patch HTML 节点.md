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
## 挂载阶段
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
hostCreateElement 的实现在 'runtime-dom' 包中的 nodeOpts 文件中，主要功能是创建 HTML 节点并返回。
~~~typescript
/**
 * module   : runtime-dom
 * filepath : src/nodeOps.ts
 * linenum  : 9
 */
export const nodeOps: Omit<RendererOptions<Node, Element>, 'patchProp'> = {
    createElement: (tag, isSVG, is, props): Element => {
        const el = isSVG 
            ? doc.createElementNS(svgNS, tag)
            : doc.createElement(tag, is ? {is} : undefined)

        if(tag === 'select' && props && props.multiple != null) {
            ;(el as HTMLSelectElement).setAttribute('multiple', props.multiple)
        }

        return el
    }
} 
~~~
mountChildren:
- 遍历子节点并对每个子节点执行 patch 方法。
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 771
 */
const mountChildren: MountChildren = (
    children,
    container,
    anchor,
    parentComponent,
    parentSuspense,
    isSVG,
    slotScopeIds,
    optimized,
    start = 0
) => {
    for (let i = start; i < children.length; i++) {
        const child = (children[i] = optimized
            ? cloneIfMounted(children[i] as VNode)
            : normalizeVNode(children[i]))
        
        patch(
            null,
            child,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
        )
    }
}
~~~
## 更新阶段
patchElement
- 触发钩子
- 更新属性
- 更新子节点
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 800
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
    const el = (n2.el = n1.el!)
    let { patchFlag, dynamicChildren, dirs } = n2
    patchFlag |= n1.patchFlag & PatchFlags.FULL_PROPS
    const oldProps = n1.props || EMPTY_OBJ
    const newProps = n2.props || EMPTY_OBJ
    let vnodeHook: VNodeHook | undefined | null
    
    parentComponent && toggleRecurse(parentComponent, false)
    if((vnodeHook = newProps.onVnodeBeforeUpdate)) {
        invokeVNodeHook(vnodeHook, parentComponent, n2, n1)
    }
    parentComponent && toggleRecurse(parentComponent, true)

    /* ... */

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

        /* ... */

    } else if (!optimized) {
        patchChildren(
            n1,
            n2,
            el,
            null,
            parentComponent,
            parentSuspense,
            areChildrenSVG,
            slotScopeIds,
            false
        )
    }

    if(patchFlag > 0) {
        if(patchFlag & PatchFlags.FULL_PROPS) {
            patchProps(
                el,
                n2,
                oldProps,
                newProps,
                parentComponent,
                parentSuspense,
                isSVG
            )
        } else {
            if (patchFlag & PatchFlags.CLASS) {
                if(oldProps.class !== newProps.class) {
                    hostPatchProp(el, 'class', null, newProps.class, isSVG)
                }
            }

            if(patchFlag & PatchFlags.STYLE) {
                hostPatchProp(el, 'style', oldProps.style, newProps.style, isSVG)
            }

            if(patchFlag & PatchFlags.PROPS) {
                const propsToUpdate = n2.dynamicProps
                for(let i = 0; i < propsToUpdate.length; i++) {
                    const key = propsToUpdate[i]
                    const prev = oldProps[key]
                    const next = newProps[key]

                    if(next !== prev || key === 'value') {
                        hostPatchProp(
                            el,
                            key,
                            prev,
                            next,
                            isSVG,
                            n1.children as VNode[],
                            parentComponent,
                            parentSuspense,
                            unmountChildren
                        )
                    }
                }
            }
        }

        if (patchFlag & PatchFlags.TEXT) {
            if(n1.children !== n2.children) {
                hostSetElementText(el, n2.children as string)
            }
        }
    } else if (!optimized && dynamicChildren == null) {
        patchProps(
            el,
            n2,
            oldProps,
            newProps,
            parentComponent,
            parentSuspense,
            isSVG
        )
    }

    /* ... */

}
~~~
patchBlockChildren
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 955
 */
const patchBlockChildren: PatchBlockChildren = (
    oldChildren,
    newChildren,
    fallbackContainer,
    parentComponent,
    parentSuspense,
    isSVG,
    slotScopeIds
) => {
    for(let i = 0; i < newChildren.length; i++) {
        const oldVNode = oldChildren[i]
        const newVNode = newChildren[i]

        const container = 
            oldVNode.el && 
            (oldVNode.type === Fragment || 
                !isSameVNodeType(oldVNode, newVNode) ||
                oldVNode.shapFlag & (ShapeFlags.COMPONENT | ShapeFlags.TELEPORT)) 
                ? hostParentNode(oldValue.el) 
                : fallbackContainer
        
        patch(
            oldVNode,
            newVNode,
            container,
            null,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            true
        )
    }
}
~~~
patchProps
~~~typescript
/**
 * module   : runtime-core
 * filepath : src/renderer.ts
 * linenum  : 998
 */

const patchProps = (
    el: RendererElement,
    vnode: VNode,
    oldProps: Data,
    newProps: Data,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean
) {
    if(oldProps !== newProps) {
        for(const key in newProps) {
            if(isReservedProp(key)) continue
            const next = newProps[key]
            const prev = oldProps[key]

            if(next !== prev && key !== 'value') {
                hostPatchProp(
                    el,
                    key,
                    prev,
                    next,
                    isSVG,
                    vnode.children as VNode[],
                    parentComponent,
                    parentSuspense,
                    unmountChildren
                )
            }
        }
        if(oldProps !== EMPTY_OBJ) {
            for(const key in oldProps) {
                if(!isReservedProp(key) && !(key in newProps)) {
                    hostPatchProp(
                        el,
                        key,
                        oldProps[key],
                        null,
                        isSVG,
                        vnode.children as VNode[],
                        parentComponent,
                        parentSuspense,
                        unmountChildren
                    )
                }
            }
        }

        if('value' in newProps) {
            hostPatchProp(el, 'value', oldProps.value, newProps.value)
        }
    }
}
~~~
hostPatchProp 的实现位于 'runtime-dom' 包内的 patchProp.ts 文件内。
~~~typescript
/**
 * module   : runtime-dom
 * filepath : src/patchProp.ts
 * linenum  : 998
 */

export const patchProp: DOMRendererOptions['patchProp'] = (
    el,
    key,
    prevValue,
    nextValue,
    isSVG = false,
    prevChildren,
    parentComponent,
    parentSuspense,
    unmountChildren
) {
    if(key === 'class') {
        patchClass(el, nextValue, isSVG)
    } else if (key === 'style') {
        patchStyle(el, prevValue, nextValue)
    } else if (isOn(key)) {
        if(!isModelListener(key)){
            patchEvent(el, key, prevValue, nextValue, parentComponent)
        }
    } else if (
        key[0] === '.'
            ? ((key = key.slice(1)), true)
            : key[0] === '^'
            ? ((key = key.slice(1)), false)
            : shouldSetAsProp(el, key, nextValue, isSVG)
    ) {
        patchDOMProp(
            el,
            key,
            nextValue,
            prevChildren,
            parentComponent,
            parentSuspense,
            unmountChildren
        )
    } else {
        if (key === 'true-value') {
            ;(el as any)._trueValue = nextValue
        } else if (key === 'false-value') {
            ;(el as any)._falseValue = nextValue
        }
        patchAttr(el, key, nextValue, isSVG, parentComponent)
    }
}
~~~
patchChildren 
~~~typescript
/**
 * module   : runtime-dom
 * filepath : src/patchProp.ts
 * linenum  : 1597
 */

const patchChildren: PatchChildren = (
    n1,
    n2,
    container,
    anchor,
    parentComponent,
    parentSuspense,
    isSVG,
    slotScopeIds,
    optimized = false
) => {
    const c1 = n1 && n1.children
    const prevShapFlag = n1 ? n1.shapeFlag : 0
    const c2 = n2.children
    const { patchFlag, shapeFlag } = n2
    if(patchFlag > 0) {
        if(patchFlag & PatchFlag.KEYED_FRAGMENT) {
            patchKeyedChildren(
                c1 as VNode[],
                c2 as VNodeArrayChildren,
                container,
                anchor,
                parentComponent,
                parentSuspense,
                isSVG,
                slotScopeIds,
                optimzied
            )
            return
        } else if (patchFlag & PatchFlags.UNKEYED_FRAGMENT) {
            patchUnkeyedChildren(
                c1 as VNode[],
                c2 as VNodeArrayChildren,
                container,
                anchor,
                parentComponent,
                parentSuspense,
                isSVG,
                slotScopeIds,
                optimized
            )
            return 
        }
    }
    if(shapeFlag & ShapeFlags.TEXT_CHILDREN) {
        if(prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) {
            unmountChildren(c1 as VNOde[], parentComponent, parentSuspense)
        }
        if(c2 !== c1) {
            hostSetElementText(container, c2 as string)
        }
    } else {
        if (prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) {
            if(shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
                patchKeyedChildren(
                    c1 as VNode[],
                    c2 as VNodeArrayChildren,
                    container,
                    anchor,
                    parentComponent,
                    parentSuspense,
                    isSVG,
                    slotScopeIds,
                    optimized
                )
            } else {
                unmountChildren(c1 as VNode[], parentComponent, parentSuspense, true)
            }
        } else {
            if (prevShapeFlag & ShapeFlags.TEXT_CHILDREN) 
            {
                hostSetElementText(container, '')
            }
            if (shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
                mountChildren(
                    c2 as VNodeArrayChildren,
                    container,
                    anchor,
                    parentComponent,
                    parentSuspense,
                    isSVG,
                    slotScopeIds,
                    optimzied
                )
            }
        }
    }
}
~~~
patchUnkeyedChildren
~~~typescript
/**
 * module   : runtime-dom
 * filepath : src/patchProp.ts
 * linenum  : 1699
 */

const patchUnkeyedChildren = (
    c1: VNode[],
    c2: VNodeArrayChildren,
    container: RendererElement,
    anchor: RendererNode | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    slotScopeIds: string[] | null,
    optimized: boolean
) {
    c1 = c1 || EMPTY_ARR
    c2 = c2 || EMPTY_ARR
    const oldLength = c1.length
    const newLength = c2.length
    const commonLength = Math.min(oldLength, newLength)
    let i 
    for(i = 0; i < commonLength; i++) {
        const nextChild = (c2[i] = optimized)
            ? cloneIfMounted(c2[i] as VNode)
            : normalizeVNode(c2[i])

        patch(
            c1[i],
            nextChild,
            container,
            null,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
        )
    }
    if(oldLength > newLength) {
        unmountChildren(
            c1,
            parentComponent,
            parentSuspense,
            true,
            false,
            commonLength
        )
    } else {
        mountChildren(
            c2,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized,
            commonLength
        )
    }
}
~~~
patchKeyedChildren
~~~typescript
const patchKeyedChildren = (
    c1: VNode[],
    c2: VNodeArrayChildren,
    container: RendererElement,
    parentAnchor: RendererNode | null,
    parentComponent: ComponentInternalInstance | null,
    parentSuspense: SuspenseBoundary | null,
    isSVG: boolean,
    slotScopeIds: string[] | null,
    optimized: boolean
) => {
    let i = 0
    const l2 = c2.length
    let e1 = c1.length - 1
    let e2 = l2 - 1

    while (i <= e1 && i <= e2) {
        const n1 = c1[i]
        const n2 = (c2[i] = optimized 
            ? cloneIfMounted(c2[i] as VNode)
            : normalizeVNode(c2[i])
        )

        if(isSameVNodeType(n1, n2)){
            patch(
                n1,
                n2,
                container,
                null,
                parentComponent,
                parentSuspense,
                isSVG,
                slotScopeIds,
                optimized
            )
        } else {
            break
        }
        i++
    }

    while (i <= e1 && i <= e2) {
        const n1 = c1[e1]
        const n2 = (c2[e2] = optimized
            ? cloneIfMounted(c2[e2] as VNode)
            : normalizeVNode(c2[e2]))
        if(isSameVNodeType(n1, n2)) {
            patch(
                n1,
                n2,
                container,
                null,
                parentComponent,
                parentSuspense,
                isSVG,
                slotScopeIds,
                optimized
            )
        } else {
            break
        }

        e1--
        e2--
    }

    if(i > e1) {
        if(i <= e2) {
            const nextPos = e2 + 1
            const anchor = nextPos < l2 ? (c2[nextPos] as VNode).el : parentAnchor

            while(i <= e2) {
                patch(
                    null,
                    (c2[i] = optimized
                        ? cloneIfMounted(c2[i] as VNode)
                        : normalizeVNode(c2[i])
                    ),
                    container,
                    anchor,
                    parentComponent,
                    parentSuspense,
                    isSVG,
                    slotScopeIds,
                    optimized
                )
                i++
            }
        }
    }
    else if(i > e2) {
        while(i <= e1) {
            unmount(c1[i], parentComponent, parentSuspense, true)
            i++
        }
    }
    else {
        const s1 = i
        const s2 = i

        const keyTyNewIndexMap: Map<string | number | symbol, number> = new Map()
        for(i = s2; i <= e2; i++){
            const nextChild = (c2[i] = optimized 
                ? cloneIfMounted(c2[i] as VNode)
                : normalizeVNode(c2[i])
            )

            if(nextChild.key != null) {

                /* ... */

                keyToNewIndexMap.set(nextChild.key, i)
            }
        }
    }

    let j
    let patched = 0
    const toBeaptched = e2 - s2 + 1
    let moved = false
    let maxNewIndexSoFar = 0
    const newIndexToOldIndexMap = new Array(toBePatched)
    for(i = 0; i <= e1; i++){
        const prevChild = c1[i]
        if(patched >= toBePatched) {
            unmount(prevChild, parentComponent, parentSuspense, true)
            continue
        }

        let newIndex
        if(prevChild.key != null) {
            newIndex = keyToNewIndexMap.get(prevChild.key)
        } else {
            for(j = s2; j <= e2; j++) {
                if (
                    newIndexToOldIndexMap[j-s2] === 0 &&
                    isSameVNodeType(prevChild, c2[j] as VNode)
                ) {
                    newIndex = j
                    break
                }
            }
        }

        if(newIndex === undefined) {
            unmount(prevChild, parentComponent, parentSuspense, true)
        } else {
            newIndexToOldIndexMap[newIndex - s2] = i + 1

            if(newIndex >= maxNewIndexSoFar) {
                maxNewIndexSoFar = newIndex
            } else {
                moved = true
            }
            patch(
                prevChild,
                c2[newIndex] as VNode,
                container,
                null,
                parentComponent,
                parentSuspense,
                isSVG,
                slotScopeIds,
                optimized
            )
            patched++
        }
    }

    const increasingNewIndexSequence = moved 
        ? getSequence(newIndexToOldIndexMap)
        : EMPTY_ARR
    j = increasingNewIndexSequence.length - 1
    for(i = toBePatched - 1; i >= 0; i--) {
        const nextIndex = s2 + i
        const nextChild = c2[nextIndex] as VNode
        const anchor = 
            nextIndex + 1 < l2 ? (
                c2[nextIndex + 1] as VNode
            ).el : parentAnchor
        
        if(newIndexToOldIndexMap[i] === 0) {
            patch(
                null,
                nextChild,
                container,
                anchor,
                parentComponent,
                parentSuspense,
                isSVG,
                slotScopeIds,
                optimized
            )
        } else if (moved) {
            if(j < 0 || i !== increasingNewIndexSequence[j]) {
                move(nextChild, container, anchor, MoveType.REORDER)
            } else {
                j--
            }
        }
    }
}
~~~
(未完待续)