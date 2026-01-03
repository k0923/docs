这是一份整合了之前所有讨论（包括性能陷阱、Zustand 最佳实践以及 Modal 优化方案）的完整前端开发规范文档。

这份文档可以直接分发给团队，作为 **React + TypeScript + Zustand** 技术栈的开发圣经。

---

# 前端开发规范指南 (React + TS + Zustand) v1.0

## 1. 核心设计原则 (Core Principles)

- **关注点分离 (Separation of Concerns)**：
  - **Component (View)**：只负责 UI 渲染、交互反馈、动画。
  - **Store (Logic)**：负责业务逻辑、数据状态变更、API 请求。
- **状态下沉 (State Push-down)**：
  - **谁使用，谁订阅**。严禁在父组件订阅它本身不需要的数据，仅仅是为了传给子组件（Prop Drilling）。
- **原子化订阅 (Atomic Subscription)**：
  - 组件应只订阅它渲染所需的最小数据切片，避免因无关数据变化导致重渲染。

---

## 2. 目录结构 (Directory Structure)

采用 **"Component-based"** 结构，实现高内聚。

```text
src/
  ├── components/              # 通用/业务组件
  │   └── UserModal/           # 文件夹即组件
  │       ├── index.tsx        # 统一导出口
  │       ├── UserModal.tsx    # 组件核心
  │       ├── hooks.ts         # 组件独有的逻辑钩子
  │       └── types.ts         # 组件 Props 定义
  ├── store/
  │   ├── useShopStore.ts      # 业务领域 Store
  │   └── useUIStore.ts        # 全局 UI 状态 (如 Modal, Sidebar)
  └── types/
      └── domain.ts            # 全局领域模型
```

---

## 3. Zustand 状态管理规范

### 3.1 Store 设计：Action 即业务

Store 不应只是 `setSomething` 的集合，而应包含业务意图。

- ❌ **反模式 (组件内写逻辑)**:
  ```typescript
  // Component.tsx
  const handleAdd = item => {
    const newList = [...list, item] // 逻辑泄露到组件
    setList(newList)
  }
  ```
- ✅ **最佳实践 (Store 内封装)**:
  ```typescript
  // Store.ts
  actions: {
    addItem: item =>
      set(state => {
        // 包含去重、库存检查、API同步等完整业务闭环
        return { list: [...state.list, item] }
      })
  }
  ```

### 3.2 渲染性能：拒绝 "巨型父组件"

**这是本规范最强调的性能红线。**

- **场景**：`Layout` 包含 `Header` 和 `ProductList`。
- **❌ 错误做法**：`Layout` 订阅了 `cartCount` 和 `products`，然后传给子组件。
  - _后果_：购物车数字一变，整个页面（包括商品列表）全部重绘。
- **✅ 正确做法**：`Layout` 不订阅任何数据。`Header` 自己订阅 `cartCount`，`ProductList` 自己订阅 `products`。

### 3.3 数据读取

- **使用 `useShallow`**：当订阅返回对象时，必须使用。
  ```typescript
  const { loading, data } = useStore(
    useShallow(s => ({
      loading: s.loading,
      data: s.data,
    }))
  )
  ```

---

## 4. Q&A 与 常见场景解决方案

针对开发中常见的问题，特别是性能与交互冲突的场景，请参考以下方案。

### Q1: Modal/Dialog 组件通常需要 `visible` 属性，导致父组件维护 State 并频繁重渲染，怎么解决？

**背景**：
通常写法 `const [visible, setVisible] = useState(false)`。当点击打开时，父组件 State 变化，导致父组件及其所有子节点重渲染。对于复杂页面（如大数据表格），这会造成明显卡顿。

**推荐方案 A：Store 托管 UI 状态 (Zustand 最佳实践)**
将“弹窗是否显示”的状态移入 Store（可以是全局 UI Store，也可以是按 Feature 划分的 Store），父组件只负责“触发”，不负责“持有状态”。

> 备注：以 Semi UI 为例，`Modal` 使用 `visible` 字段，但思想一致。

1.  **定义 Store**:
    ```typescript
    // store/useUIStore.ts
    export const useUIStore = create(set => ({
      modals: { login: false, editUser: false },
      modalData: {}, // 存放传递给弹窗的数据
      actions: {
        openModal: (key, data) =>
          set(s => ({
            modals: { ...s.modals, [key]: true },
            modalData: { ...s.modalData, [key]: data },
          })),
        closeModal: key =>
          set(s => ({
            modals: { ...s.modals, [key]: false },
          })),
      },
    }))
    ```
2.  **父组件 (0 渲染成本)**:
    ```typescript
    const Parent = () => {
      // 这里的 openModal 是稳定函数，点击不会触发 Parent 重渲染
      const openModal = useUIStore(s => s.actions.openModal)
      return <button onClick={() => openModal('editUser', { id: 1 })}>编辑</button>
    }
    ```
3.  **Modal 组件 (自管理)**:
    ```typescript
    const EditUserModal = () => {
      const isOpen = useUIStore(s => s.modals['editUser']);
      const close = useUIStore(s => s.actions.closeModal);
      // 只有这个 Modal 组件会重渲染
      return <Dialog open={isOpen} onResult={close} ... />;
    };
    ```

**实战示例：按 Feature Store 管理（以数据集创建弹窗为例）**

```typescript
// features/dataset/stores/useDatasetStore.ts
export const useDatasetStore = create(set => ({
  items: [],
  createModalOpen: false,
  openCreateModal: () => set({ createModalOpen: true }),
  closeCreateModal: () => set({ createModalOpen: false }),
  createDataset: ({ name, schema }) =>
    set(state => ({
      items: [
        ...state.items,
        { id: nanoid(), name, schema, creator: 'local', created_at: Date.now(), updated_at: Date.now() },
      ],
    })),
}))

// pages/dataset/DatasetListPage.tsx
const openCreateModal = useDatasetStore(s => s.openCreateModal)
return <Button onClick={openCreateModal}>新建数据集</Button>

// features/dataset/components/DatasetFormModal.tsx
const visible = useDatasetStore(s => s.createModalOpen)
const close = useDatasetStore(s => s.closeCreateModal)
return <Modal visible={visible} onCancel={close} />
```

**备选方案 B：命令式句柄 (useImperativeHandle)**
如果不希望使用全局 Store，可以使用 `Ref` 调用子组件方法。

```typescript
// 父组件
const modalRef = useRef<ModalRef>(null)
// 调用 ref 方法不会触发父组件重渲染
const handleOpen = () => modalRef.current?.open()
return <MyModal ref={modalRef} />
```

---

### Q2: 什么时候该用 Props，什么时候该连 Store？

为了防止矫枉过正（所有组件都连 Store），请遵循以下 **决策矩阵**：

| 组件类型                                | 数据源推荐   | 说明                                                |
| :-------------------------------------- | :----------- | :-------------------------------------------------- |
| **基础 UI 组件** (Button, Input, Card)  | **Props**    | 必须保持纯净，不能依赖 Store，以便跨项目复用。      |
| **业务容器** (ProductList, UserProfile) | **Store**    | 涉及业务逻辑，直接订阅 Store 以避免 Prop Drilling。 |
| **全局挂件** (CartIcon, Notification)   | **Store**    | 独立订阅，随处安放，不依赖父级传递。                |
| **临时状态** (Accordion 展开/收起)      | **useState** | 状态只属于组件内部 UI 表现，无需进入 Store。        |

---

### Q3: 如何处理表单数据？应该放在 Store 吗？

**原则**：**输入过程中的临时状态（Keystrokes）不要放入全局 Store。**

- **❌ 错误**：`<input onChange={e => setGlobalStore(e.target.value)} />`
  - _后果_：每次敲击键盘都会导致订阅该 Store 的所有组件检查更新，性能极差。
- **✅ 正确**：
  1.  使用组件内部 `useState` 或 `useRef` 管理输入。
  2.  或者使用 `React Hook Form` 管理表单状态。
  3.  只有在用户点击 **"提交"** 或 **"保存"** 的瞬间，才将最终结果同步到 Zustand Store。

---

## 5. 代码自查清单 (Checklist)

在提交代码（PR）前，请对照以下几点：

1.  [ ] **State 下沉检查**：父组件是否订阅了它不需要的数据？
2.  [ ] **Modal 检查**：打开弹窗是否导致了父组件重渲染？（推荐使用 Store 控制或 Ref）
3.  [ ] **性能检查**：多字段订阅是否使用了 `useShallow`？
4.  [ ] **逻辑检查**：组件内是否包含了过多的业务计算逻辑？（应移入 Store actions）
5.  [ ] **类型安全**：是否消灭了所有的 `any`？
