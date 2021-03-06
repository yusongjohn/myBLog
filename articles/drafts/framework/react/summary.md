主要流程
- ReactDom.render -> 创建root_fiber & 估算expirationTime（同步
- scheduleRootUpdate: 创建更新对象{element}并添加到fiber.updateQueue 
- scheduleWork: scheduleWorkToRoot & requestWork(addRootToSchedule & performSyncWork -> performWork)
- while(){performWorkOnRoot & findHighestPriorityRoot}
- performWorkOnRoot
    - renderRoot -> workLoop -> performUnitOfWork
        - beginWork -> updateXxxx ( -> reconcile)
            - updateContextProvider: pushProvider & 设置context._currentValue & propagateContextChange
            - updateClassComponent: 类组件的创建更新 & readContext(添加context依赖)
            - updateContextConsumer: readContext & render(value) [context.provider的孩子是个函数
            - mountIndeterminateComponent & updateFunctionComponent -> renderWithHooks
        - completeUnitOfWork
            - 副作用链表
            - completeWork
                - popProvider/Context等 
                - HostComponet: dom节点创建 & differProperties & setInitialProperties(:事件监听)
                - Suspense组件？
    - commitRoot -> commitRoot
        - commitBeforeMutationLifeCycles: getSnapshotBeforeUpdate
        - commitAllHostEffects
            - Update: commitWork -> commitHookEffectList:unmout（执行useLayoutEffect的destroy
            - <span>commitDetachRef</span>
        - commitAllLifeCycles
            - commitLifeCycles -> commitHookEffectList:mount（执行useLayoutEffect的create
            - commitAttachRef
        - 生成异步任务执行useEffect -> commitPassiveEffects -> commitPassiveHookEffects(destroy & create)

hooks原理
- mount & update 阶段的执行逻辑不一致，
- mount阶段主要是生成一个初始的值并保存，
    - 在renderWithHooks中会执行函数组件然后执行函数组件中的hooks，每个hook都会生成一个hook对象，这些hook组成一个链表，函数组件对应的fiber提供一个属性memoizedState指向了这个链表的第一个节点
- 插入下setState的逻辑，useState生成hook对象有个queue属性，指向了一个更新对象，该更新对象就是包含了待更新的data数据（就hook.queue就像是类组件中setState会将返回的数据保存在fiber.updateQueue中    
- update阶段
    - 当函数组件更新时，会执行renderWithHooks，从而执行函数组件的函数体，进而执行内部的hooks（hooks的执行顺序对应了fiber.memoizedState执行hook对象链表的顺序
