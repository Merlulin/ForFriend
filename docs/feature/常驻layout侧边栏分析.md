# 常驻 layout 侧边栏分析

## 个人分析

本质上是layout作为整个后台项目的固定常驻内容，那么肯定作为App首页需要作为Router-view导入的，那么页面内的具体内容就是作为这个常驻内容的子容器，所以layout本质上一定包含了AppMain的子组件存在子Router-view，在router配置中通过下属方式配置，children模板会在layout内的router-view中渲染。
所以layout才是最主要的入口界面！
``` js
{
    path: "/",
    component: Layouts,
    redirect: "/dashboard",
    children: [
      {
        path: "dashboard",
        component: () => import("@/pages/dashboard/index.vue"),
        name: "Dashboard",
        meta: {
          title: "首页",
          svgIcon: "dashboard",
          affix: true
        }
      }
    ]
  },
```

## 结论

`layouts/index.vue` 会常驻在页面中，是因为它被配置为多个一级路由的父组件，子页面内容通过嵌套路由的 `<router-view>` 渲染到布局内部。路由切换时只替换子组件，布局组件本身不卸载，因此侧边栏等结构保持常驻。

## 关键实现位置

- 路由中引入布局组件并作为多个父路由的 `component`：
  - [src/router/index.ts](src/router/index.ts#L1-L40)
- 布局入口，选择布局模式（左/顶/混合）：
  - [src/layouts/index.vue](src/layouts/index.vue#L1-L40)
- 左侧布局中引入 `AppMain` 作为页面内容容器：
  - [src/layouts/modes/LeftMode.vue](src/layouts/modes/LeftMode.vue#L20-L60)
- `AppMain` 内部通过 `<router-view>` 渲染具体页面，并配合 `keep-alive`：
  - [src/layouts/components/AppMain/index.vue](src/layouts/components/AppMain/index.vue#L8-L30)

## 运行流程说明

1. 路由声明中，多个父路由的 `component` 指向 `Layouts`，意味着这些路由共享同一布局壳。
2. `layouts/index.vue` 根据布局模式渲染不同布局组件（左侧/顶部/混合）。
3. 以左侧布局为例，`LeftMode.vue` 中包含侧边栏、导航栏以及页面主内容区域。
4. 页面主内容区域由 `AppMain` 提供，`AppMain` 内部的 `<router-view>` 会渲染子路由组件。
5. 当路由切换时，只有 `<router-view>` 的内容替换，父布局组件不变，因此侧边栏与顶栏持续存在。

## 涉及的 Vue 知识点

- 嵌套路由：父路由组件保持，子路由渲染到父组件的 `<router-view>`。
- 动态组件：`<component :is="Component">` 依据路由渲染具体页面。
- 组件缓存：`<keep-alive>` 可缓存符合条件的页面组件，避免重复销毁/创建。
- 页面切换过渡：`<transition>` 为路由切换提供过渡动画。

## 小结

布局常驻的本质是“父子路由 + `<router-view>`”的组合设计。布局承担壳层结构，页面内容由子路由渲染。这样可以保证侧边栏、顶部栏等公共区域在全站切换时保持稳定，提升用户体验与结构复用性。
