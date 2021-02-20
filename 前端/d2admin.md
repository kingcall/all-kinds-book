[D2Admin基本使用](https://www.cnblogs.com/izbw/p/11077815.html)



目录

- d2-admin基本使用
  - 1 安装
    - [1.1 全局安装 d2-admin](https://www.cnblogs.com/izbw/p/11077815.html#11-全局安装-d2-admin)
    - [1.2 创建项目](https://www.cnblogs.com/izbw/p/11077815.html#12-创建项目)
    - [1.3 启动项目](https://www.cnblogs.com/izbw/p/11077815.html#13-启动项目)
    - [1.4 注意事项](https://www.cnblogs.com/izbw/p/11077815.html#14-注意事项)
  - 2 修改框架 title 和 logo
    - [2.1 修改 title](https://www.cnblogs.com/izbw/p/11077815.html#21-修改-title)
    - [2.2 修改 logo](https://www.cnblogs.com/izbw/p/11077815.html#22-修改-logo)
  - 3 图表库
    - [3.1 常用的图表库](https://www.cnblogs.com/izbw/p/11077815.html#31-常用的图表库)
    - [3.2 安装v-charts](https://www.cnblogs.com/izbw/p/11077815.html#32-安装v-charts)
    - [3.3 导入项目](https://www.cnblogs.com/izbw/p/11077815.html#33-导入项目)
    - [3.4 简单举例](https://www.cnblogs.com/izbw/p/11077815.html#34-简单举例)
    - [3.5 注意事项](https://www.cnblogs.com/izbw/p/11077815.html#35-注意事项)
  - 4 CURD插件（表格）
    - [4.1 安装](https://www.cnblogs.com/izbw/p/11077815.html#41-安装)
    - [4.2 导入项目](https://www.cnblogs.com/izbw/p/11077815.html#42-导入项目)
    - [4.3 图表使用](https://www.cnblogs.com/izbw/p/11077815.html#43-图表使用)
    - [4.4 注意事项](https://www.cnblogs.com/izbw/p/11077815.html#44-注意事项)
  - 5 定义数据API
    - [5.1 第一步，使用mockjs创建数据](https://www.cnblogs.com/izbw/p/11077815.html#51-第一步，使用mockjs创建数据)
    - [5.2 第二步，创建API接口，共VUE项目调用](https://www.cnblogs.com/izbw/p/11077815.html#52-第二步，创建api接口，共vue项目调用)
    - [5.3 第三步，在 VUEX 中创建API，调用第二步中创建API接口，对数据进行操作](https://www.cnblogs.com/izbw/p/11077815.html#53-第三步，在-vuex-中创建api，调用第二步中创建api接口，对数据进行操作)
    - [5.4 第四步，在 view 中调用 VUEX 中的模块中的方法实现具体功能。](https://www.cnblogs.com/izbw/p/11077815.html#54-第四步，在-view-中调用-vuex-中的模块中的方法实现具体功能。)
  - 6 权限控制
    - [6.1 第一步，在 mock 中添加用户信息，以及不同的用户对应的不同的权限，并保存至本地的sessionStorage中。](https://www.cnblogs.com/izbw/p/11077815.html#61-第一步，在-mock-中添加用户信息，以及不同的用户对应的不同的权限，并保存至本地的sessionstorage中。)
    - [6.2 第二步，在 src/API 中定义调用 mock 中 API 的接口，取到用户信息](https://www.cnblogs.com/izbw/p/11077815.html#62-第二步，在-srcapi-中定义调用-mock-中-api-的接口，取到用户信息)
    - [6.3 第三步，在 router/index.js 中添加钩子函数。](https://www.cnblogs.com/izbw/p/11077815.html#63-第三步，在-routerindexjs-中添加钩子函数。)
    - [6.4 第四步，在 store 中根据权限处理数据](https://www.cnblogs.com/izbw/p/11077815.html#64-第四步，在-store-中根据权限处理数据)
    - [6.5 第五步，渲染页面后，即可看到当前用户具体有哪些操作权限](https://www.cnblogs.com/izbw/p/11077815.html#65-第五步，渲染页面后，即可看到当前用户具体有哪些操作权限)



## d2-admin基本使用

> 官方演示：https://d2admin.fairyever.com/#/index
> 官方文档：https://doc.d2admin.fairyever.com/zh/

## 1 安装

### 1.1 全局安装 d2-admin

```
npm install -g @d2-admin/d2-admin-cli
```

### 1.2 创建项目

```
d2 create 项目名称
```

或者

```
d2 c 项目名称
```

之后选择 **简化版**

### 1.3 启动项目

进入项目文件夹

```
npm i
npm run serve
```

### 1.4 注意事项

- `d2-container`是 D2Admin 构建页面最重要的组件,在模板页面中记得要用该标签包裹，该标签针对不样式的页面内置不同的类型，详见[官方文档](https://doc.d2admin.fairyever.com/zh/sys-components/container.html#参数)

## 2 修改框架 title 和 logo

### 2.1 修改 title

```html
// .env.development

# 开发环境

# 页面 title 前缀
VUE_APP_TITLE=ZAdmin
```

修改完成后重启项目即生效

### 2.2 修改 logo

**首页LOGL：**
替换 .\public\image\theme\d2\logo\all.png

**网页 ico 小图标：**
替换 .\public\icon.ico

## 3 图表库

### 3.1 常用的图表库

- 百度图表库：echarts (此框架不方便)
- 饿了么图表库：v-charts （用这个）

> v-charts 官方文档：https://v-charts.js.org/#/

### 3.2 安装v-charts

```
npm i v-charts echarts -S
```

### 3.3 导入项目

```javascript
// main.js
import Vue from 'vue'
import VCharts from 'v-charts'
import App from './App.vue'

Vue.use(VCharts)

new Vue({
  el: '#app',
  render: h => h(App)
})
```

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142007718-968665986.png)

### 3.4 简单举例

以折线图为例，其他类型详见官方文档。

为了美观，将其包含在elementUI的Card组件中。

```javascript
<template>
  <el-card>
    <ve-line :data="chartData"></ve-line>
  </el-card>
</template>

<script>
  export default {
    data: function () {
      return {
        chartData: {
          columns: ['日期', '访问用户', '下单用户', '下单率'],
          rows: [
            { '日期': '1/1', '访问用户': 1393, '下单用户': 1093, '下单率': 0.32 },
            { '日期': '1/2', '访问用户': 3530, '下单用户': 3230, '下单率': 0.26 },
            { '日期': '1/3', '访问用户': 2923, '下单用户': 2623, '下单率': 0.76 },
            { '日期': '1/4', '访问用户': 1723, '下单用户': 1423, '下单率': 0.49 },
            { '日期': '1/5', '访问用户': 3792, '下单用户': 3492, '下单率': 0.323 },
            { '日期': '1/6', '访问用户': 4593, '下单用户': 4293, '下单率': 0.78 }
          ]
        }
      }
    }
  }
</script>
```

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142122635-1777951620.png)

### 3.5 注意事项

- 将多个图表分别放置在tab标签页时，切换标签页后下个图表可能会等待很久才会出现，是因为收到 elementUI 中Tabs标签页的 `lazy` 属性的影响（初始化时有个渲染的过程），如果没有开启延迟渲染，会只渲染第一个标签页内容，因此需要将 `lazy` 设置为 `true`

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142141283-1672505998.png)

从第二 tbgs 标签页起，将`lazy` 属性设置为 `true`

```html
<el-tabs v-model="activeName" @tab-click="handleClick">
    <el-tab-pane label="用户管理" name="first">
        <ve-line :data="chartData"></ve-line>      
    </el-tab-pane>
    <el-tab-pane :lazy="true" label="配置管理" name="second">
        <ve-histogram :data="chartDataHis"></ve-histogram>      
    </el-tab-pane>
    <el-tab-pane :lazy="true" label="角色管理" name="third">角色管理</el-tab-pane>
    <el-tab-pane :lazy="true" label="定时任务补偿" name="fourth">定时任务补偿</el-tab-pane>
</el-tabs>
```

## 4 CURD插件（表格）

D2 CURD是一个基于Vue.js 和 Element UI的表格组件，封装了常用的表格操作，使用该组件可以快速在页面上处理表格数据。
详见[官方文档](https://doc.d2admin.fairyever.com/zh/ecosystem-d2-crud/)

### 4.1 安装

```
npm i element-ui @d2-projects/d2-crud -S
```

### 4.2 导入项目

```javascript
import Vue from 'vue'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
import D2Crud from '@d2-projects/d2-crud'

Vue.use(ElementUI)
Vue.use(D2Crud)

new Vue({
  el: '#app',
  render: h => h(App)
})
```

### 4.3 图表使用

此处为带有新增和分页功能的表格，但CURD2.x中分页功能的数据需要从后台获取，因此这里只简单演示了表格的样式。

`columns`: 表格的列属性
`data`: 表格的数据
`add-title`: 弹出新增模态框的标题
`pagination`: 开启分页功能
`loading`: 加载数据时会有加载中的样式
`slot="header"`： 表格的头部信息，自定义样式（如：标题，按钮，输入框等）可以显示在表格的顶部（2.x新增，更好的替代了1.x中的 `title` 属性）
`BusinessTable1List`: 后台数据API，获取后台数据，这里只是页面展示，采用固定的数据，未使用API接口。

```html
<template>
<d2-container>
  <el-card>
    <d2-crud
      ref="d2Crud"
      :columns="columns"
      :data="data"
      add-title="添加数据"
      :add-template="addTemplate"
      :form-options="formOptions"
      :pagination="pagination"
      :loading="loading"
      @pagination-current-change="paginationCurrentChange"
      @dialog-open="handleDialogOpen"
      @row-add="handleRowAdd"
      @dialog-cancel="handleDialogCancel">
      <el-button slot="header" style="margin-bottom: 5px" @click="addRow"><i class="fa fa-plus" aria-hidden="true"></i> 新增</el-button>
      <el-button slot="header" style="margin-bottom: 5px" @click="addRowWithNewTemplate">使用自定义模板新增</el-button>
      <el-input slot="header" style="margin-bottom: 5px" placeholder="商品名称" suffix-icon="el-icon-search"> </el-input>
      <el-input slot="header" style="margin-bottom: 5px" placeholder="最低价格" suffix-icon="el-icon-caret-bottom"> </el-input>
      <el-input slot="header" style="margin-bottom: 5px" placeholder="最高价格" suffix-icon="el-icon-caret-top"> </el-input>
      <el-button slot="header" style="margin-bottom: 5px"><i class="el-icon-search"></i> 搜索</el-button>
    </d2-crud>
  </el-card>
</d2-container>
</template>

<script>
// import { BusinessTable1List } from '@api/demo.business.table.1'
export default {
  data () {
    return {
      columns: [
        {
          title: '日期',
          key: 'date'
        },
        {
          title: '姓名',
          key: 'name'
        },
        {
          title: '地址',
          key: 'address'
        }
      ],
      data: [
          {
            date: '2016-05-02',
            name: '王小虎',
            address: '上海市普陀区金沙江路 1518 弄'
          },
          {
            date: '2016-05-04',
            name: '王小虎',
            address: '上海市普陀区金沙江路 1517 弄'
          },
          {
            date: '2016-05-01',
            name: '王小虎',
            address: '上海市普陀区金沙江路 1519 弄'
          },
          {
            date: '2016-05-03',
            name: '王小虎',
            address: '上海市普陀区金沙江路 1516 弄'
          }
      ],
      addTemplate: {
        date: {
          title: '日期',
          value: '2016-05-05'
        },
        name: {
          title: '姓名',
          value: '王小虎'
        },
        address: {
          title: '地址',
          value: '上海市普陀区金沙江路 1520 弄'
        }
      },
      formOptions: {
        labelWidth: '80px',
        labelPosition: 'left',
        saveLoading: false
      },
      loading: false,
      pagination: {
        currentPage: 1,
        pageSize: 5,
        total: 100
      }
    }
  },
  mounted () {
    this.fetchData()
  },
  methods: {
    handleDialogOpen ({ mode }) {
      this.$message({
        message: '打开模态框，模式为：' + mode,
        type: 'success'
      })
    },
    // 普通的新增
    addRow () {
      this.$refs.d2Crud.showDialog({
        mode: 'add'
      })
    },
    // 传入自定义模板的新增
    addRowWithNewTemplate () {
      this.$refs.d2Crud.showDialog({
        mode: 'add',
        template: {
          name: {
            title: '姓名',
            value: ''
          },
          value1: {
            title: '新属性1',
            value: ''
          },
          value2: {
            title: '新属性2',
            value: ''
          }
        }
      })
    },
    handleRowAdd (row, done) {
      this.formOptions.saveLoading = true
      setTimeout(() => {
        console.log(row)
        this.$message({
          message: '保存成功',
          type: 'success'
        });

        // done可以传入一个对象来修改提交的某个字段
        done({
          address: '我是通过done事件传入的数据！'
        })
        this.formOptions.saveLoading = false
      }, 300)
    },
    handleDialogCancel (done) {
      this.$message({
        message: '取消保存',
        type: 'warning'
      });
      done()
    },
    paginationCurrentChange (currentPage) {
      this.pagination.currentPage = currentPage
      this.fetchData()
    },
    fetchData () {
      this.loading = true
      BusinessTable1List({
        ...this.pagination
      }).then(res => {
        this.data = res.list
        this.pagination.total = res.page.total
        this.loading = false
      }).catch(err => {
        console.log('err', err)
        this.loading = false
      })
    }
  }
}
</script>

<style scoped>
  .el-input {
    width: 200px;
    margin-right: 10px;
  }
</style>
```

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142212958-406133378.png)

### 4.4 注意事项

- 课程中使用CURD组件为1.x，最新版本为2.x，变化较大，尤其表现在分页和添加功能上，如在1.x中分页是完全基于前端，而2.x版本的CURD是基于后端的，其需要后端提供数据接口，对返回的数据进行分页展现。详见[官方文档](https://doc.d2admin.fairyever.com/zh/ecosystem-d2-crud/)。关于如何从定义后台API并使用mockjs模拟生成后台数据，请看下一章

## 5 定义数据API

- mock （模拟后端，设计API）
- api （封装axios调用api，对接后端和vuex）
- view （使用vuex处理数据）

两个概念：

- 模块化
- 命名空间

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142246239-922609799.png)
![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142257752-96853611.png)

### 5.1 第一步，使用mockjs创建数据

> 注意：创建调用API的URL时，需带上前缀 `/api/xxx`

```javascript
// /mock/api/product.js
const Mock = require('mockjs')
const Qs = require('qs')
const Random = Mock.Random


const titleList = ['男士上衣', 'T恤', '衬衫', '牛仔裤', '皮衣', '短裙', '女士衬衫', '长裙', '羽绒服', '秋裤', '军大衣'];

const getProductList = function (params) {
  // 获取全部商品列表或者分页商品列表
  let products = []
  for (let i = 0; i < 100; i++) {
    let product = {
      id: i + 1,
      title: titleList[Math.floor(Math.random()*titleList.length)],
      price: Random.float(10, 100).toFixed(2),
      stock: Random.integer(10, 100),
      saleCount: Random.integer(0, 90),
      isSale: Random.integer(0, 1),
      createTime: Random.datetime(),
      imgUrl: Random.dataImage('60x60', 'ZAdmin-' + (i + 1)),
      showEditButton: true,
      showRemoveButton: true
    }
    products.push(product)
  }

  let total = products.length;
  if (params.body && products.length) {
    let pageInfo = JSON.parse(Qs.parse(params).body);
    let start = (pageInfo.currentPage - 1) * pageInfo.pageSize;
    let end = pageInfo.currentPage * pageInfo.pageSize;
    products = products.slice(start, end);
    console.log(`start: ${start}  end: ${end}  products: ${products}  total: ${total}`);
  }

  return { products, total }
}

// 通过post请求，使用参数的token判断是否登录，并通过参数判断获取全部评论列表或者获取分页评论列表
const getProductComments = function (params) {
  let data = JSON.parse(Qs.parse(params).body);
  console.log('api-comments-params: ', data);
  if (!data.token) {
    return {
      status: 401,
      msg: 'token错误，需要登录',
      data: {}
    }
  }

  let comments = []
  for (let i = 0; i < 120; i++) {
    let comment = {
      id: i + 1,
      name: Random.cname(),
      itemScore: Random.integer(1, 5),
      serviceScore: Random.integer(1, 5),
      content: Random.ctitle(10, 50),
      createTime: Random.datetime(),
      showEditButton: true,
      showRemoveButton: true
    }
    comments.push(comment)
  }

  let total = comments.length;

  if (data.page) {
    let pageInfo = data.page;
    let start = (pageInfo.currentPage - 1) * pageInfo.pageSize;
    let end = pageInfo.currentPage * pageInfo.pageSize;
    comments = comments.slice(start, end);
    console.log(`currentPage: ${pageInfo.currentPage} start: ${start}  end: ${end}  comments: ${comments}  total: ${total}`);
  }

  return {
    status: 0,
    msg: '成功',
    data: comments,
    total: total
  }
}


// 创建API的URL，让vue通过URL获取数据
Mock.mock('/api/products', 'get', getProductList); // 获取所有商品
Mock.mock('/api/products', 'post', getProductList); // 获取分页商品数据
Mock.mock('/api/productComments', 'post', getProductComments)
```

### 5.2 第二步，创建API接口，共VUE项目调用

> 注意：
> 引用 plugin/axios 中的 `request` 方法发起请求
> 发起请求的URL，不需要带 `/api/` 前缀

```javascript
// /src/api/pruduct.js
import request from '@/plugin/axios'

export function getProducts (data) {
  return request({
    url: '/products',
    method: 'get',
    data
  })
}

export function getPageProducts (data) {
  return request({
    url: '/products',
    method: 'post',
    data
  })
}

export function getProductComments (data) {
  return request({
    url: '/productComments',
    method: 'post',
    data
  })
}
```

### 5.3 第三步，在 VUEX 中创建API，调用第二步中创建API接口，对数据进行操作

> 注意：
>
> - 在 /src/store/modules/ 下创建操作对象的模块，如 product，其包括了 modules 文件夹和 index.js 文件，index.js 文件复制 \src\store\modules\d2admin\index.js即可
> - 在modules文件夹下创建 product.js 文件用于调用第二步中创建的 API 接口，并对收到的数据进行操作，文件结构与 VUE 项目中store.js相同。
> - 在 product.js 文件中声明 `namespaced: true`，加了命名空间操作，将该文件直接加在了product文件夹下，被调用时，中间省去了一层 modules，为 product/product
> - 调用 API 时，需要先导入提前定义好的 API
> - 修改 \src\store\index.js 文件，将该模块导出，使 view 能够调用

```javascript
// \src\store\modules\product\modules\product.js
import { getProducts, getPageProducts, getProductComments } from '@/api/product'


export default {
  namespaced: true,
  actions: {
    getProducts ({ commit }, payload) {
      commit('getProducts', payload)
    },

    getPageProducts ({ commit }, payload) {
      commit('getPageProducts', payload)
    },

    getComments ({ commit }, payload) {
      commit('getComments', payload)
    }
  },

  mutations: {
    getProducts (state, payload) {
      state.loading = true;
      getProducts(payload).then(res => {
        console.log('getProducts:  ', res);
        state.loading = false;
        state.products = res.products;
      }).catch( err => {
        console.log('err', err)
        state.loading = false
      })
    },

    getComments (state, payload) {
      // 获取全部评价信息或者分页评价信息
      state.loading = true;
      console.log(payload);
      getProductComments(payload).then(res => {
        console.log('getComments:  ', res);
        if (res.status == 0) {
          state.loading = false;
          state.total = res.total;
          if (payload.page) {
            state.pageComments = res.data;
          } else {
            state.comments = res.data;
          }
        }
      }).catch( err => {
        console.log('err', err)
        state.loading = false
      })
    },

    getPageProducts (state, payload) {
      console.log('page_params: ', payload);
      state.loading = true;
      getPageProducts(payload).then(res => {
        console.log('res:  ', res);
        state.loading = false;
        state.total = res.total;
        state.pageProducts = res.products;
      }).catch( err => {
        console.log('err', err)
        state.loading = false
      })
    }
  },

  state: {
    products: [],
    pageProducts: [],
    comments: [],
    pageComments: [],
    loading: false,
    total: 0
  },

  getters: {
    products: (state) => { return state.products },
    pageProducts: (state) => { return state.pageProducts },
    comments: (state) => { return state.comments },
    pageComments: (state) => { return state.pageComments },
    loading: (state) => { return state.loading },
    total: (state) => { return state.total }
  }
}
// \src\store\index.js

import Vue from 'vue'
import Vuex from 'vuex'

import d2admin from './modules/d2admin'
import product from './modules/product'

Vue.use(Vuex)

export default new Vuex.Store({
  modules: {
    d2admin,
    product
  }
})
```

### 5.4 第四步，在 view 中调用 VUEX 中的模块中的方法实现具体功能。

> 注意：
>
> - 调用vuex中的方法以及从vuex中取数据时，都需加上命名空间 `product/product`
> - 使用mockjs 生成的图片为base64 格式，需要新建组件转换图片进行显示，view使用时需导入该组件
> - D2-admin框架自带很多工具集，其中 src/libs/util.js 中`cookies.get('token')` 可以获取当前用户的token
> - CURD2.x 分页功能放弃纯前端分页方法，采用后端传入分页好的数据（切片处理）进行分页

自定义组件，用于显示图片：

```html
// \src\views\product\all\MyImage.vue

<template>
    <div>
        <img :src="value" alt="商品图片">
    </div>
</template>

<script>
export default {
    props: {
        value: '' // 必须叫value
    }
}
</script>
```

基于 CURD2.x 的本地增删改查 以及 分页功能实现

```html
<template>
<d2-container>
  <template slot="header">
    <div class="flex-header">
      <div class="header-title">
        商品列表
      </div>
      <div>
        <el-input v-model="searchWords" placeholder="商品名称" suffix-icon="el-icon-search"></el-input>
        <el-input v-model="searchMinPrice" placeholder="最低价格" suffix-icon="el-icon-caret-bottom"> </el-input>
        <el-input v-model="searchMaxPrice" placeholder="最高价格" suffix-icon="el-icon-caret-top"> </el-input>
        <el-button @click="onSearch"><i class="el-icon-search"></i> 搜索</el-button>
        <el-button type="primary" @click="addRow"><i class="fa fa-plus" aria-hidden="true"></i> 添加商品</el-button>
      </div>
    </div>
  </template>
  <el-card>
    <d2-crud
      ref="d2Crud"
      :columns="columns"
      :data="filterProducts.length ? filterProducts : products"
      add-title="添加商品"
      :add-template="addTemplate"
      :rowHandle="rowHandle"
      edit-title="编辑商品信息"
      :edit-template="editTemplate"
      :form-options="formOptions"
      :loading="loading"
      :pagination="pagination"
      @pagination-current-change="paginationCurrentChange"
      @row-add="handleRowAdd"
      @row-edit="handleRowEdit"
      @row-remove="handleRowRemove"
      @dialog-cancel="handleDialogCancel">
    </d2-crud>
  </el-card>
  <template slot="footer">ZAdmin Created By <a href="">@ZBW</a> for D2-admin</template>
</d2-container>
</template>

<script>
// 导入自定义用于显示图片的组件
import MyImage from "./MyImage"

export default {
  data () {
    return {
      searchWords: '',
      searchMinPrice: '',
      searchMaxPrice: '',
      filterProducts: [],
      columns: [
        {
          title: 'ID',
          key: 'id',
          width: '40'
        },
        {
          title: '名称',
          key: 'title'
        },
        {
          title: '价格',
          key: 'price',
          width: '80'
        },
        {
          title: '库存',
          key: 'stock',
          width: '80'
        },
        {
          title: '销量',
          key: 'saleCount',
          width: '80'
        },
        {
          title: '是否上架',
          key: 'isSale',
          component: {
            name: 'el-select',
            options: [
              {
                value: 0,
                label: '否'
              },
              {
                value: 1,
                label: '是'
              }
            ]
          }
        },
        {
          title: '图片',
          key: 'imgUrl',
          width: '120',
          component: {
            name: MyImage
          }
        },
        {
          title: '创建时间',
          key: 'createTime'
        }
      ],
      addTemplate: {
        createTime: {
          title: '创建日期',
          value: '2019-06-01',
          component: {
            name: 'el-date-picker',
            span: 12
          }
        },
        isSale: {
          title: '是否上架',
          value: 0,
          component: {
            name: 'el-select',
            options: [
              {
                value: 0,
                label: '否'
              },
              {
                value: 1,
                label: '是'
              }
            ],
            span: 12
          }
        },
        title: {
          title: '名称',
          value: '',
          span: 24
        },
        price: {
          title: '价格',
          value: '',
          span: 24
        }
      },
      rowHandle: {
        columnHeader: '操作',
        edit: {
          icon: 'el-icon-edit',
          text: '编辑',
          size: 'mini',
          show (index, row) {
            if (row.showEditButton) {
              return true
            }
            return false
          },
          disabled (index, row) {
            if (row.forbidEdit) {
              return true
            }
            return false
          }
        },
        remove: {
          icon: 'el-icon-delete',
          size: 'mini',
          text: '删除',
          fixed: 'right',
          confirm: true,
          show (index, row) {
            if (row.showRemoveButton) {
              return true
            }
            return false
          }
        }
      },
      editTemplate: {
        createTime: {
          title: '创建日期',
          component: {
            name: 'el-date-picker',
            span: 12
          }
        },
        isSale: {
          title: '是否上架',
          component: {
            name: 'el-select',
            options: [
              {
                value: 0,
                label: '否'
              },
              {
                value: 1,
                label: '是'
              }
            ],
            span: 12
          }
        },
        title: {
          title: '名称',
          span: 24
        },
        price: {
          title: '价格',
          span: 24
        },
        forbidEdit: {
          title: '禁用按钮',
          value: false,
          component: {
            show: false
          }
        },
        showEditButton: {
          title: '显示按钮',
          value: true,
          component: {
            show: false
          }
        }
      },
      formOptions: {
        labelWidth: '80px',
        labelPosition: 'left',
        saveLoading: false
      },
    }
  },
  created() {
    // 调用vuex中方法时，需要加上命名空间 product/product
    this.fetchData();
    this.$store.commit('product/product/getProducts');  // 请求全部商品列表

  },
  computed: {
    all_products() {
      // 全部商品列表，用于搜索
      return this.$store.getters['product/product/products'];
    },
    products() {
      // 当前分页的商品列表
      // 取 vuex 中数据时，需要加上命名空间 product/product
      return this.$store.getters['product/product/pageProducts']
    },
    loading() {
      return this.$store.getters['product/product/loading']
    },
    pagination() {
      return {
        currentPage: 1,
        pageSize: 5,
        background: true,
        total: this.$store.getters['product/product/total']
      }
    }
  },
  
  
  methods: {
    onSearch() {
      this.filterProducts = this.all_products.filter(p => {
        if (this.searchWords){
          return p.price >= parseFloat(this.searchMinPrice) && p.price <= parseFloat(this.searchMaxPrice) && p.title.includes(this.searchWords);
        } else {
          return p.price >= parseFloat(this.searchMinPrice) && p.price <= parseFloat(this.searchMaxPrice);
        }
      })
      console.log('filterProducts: ', this.filterProducts);
    },
    addRow () {
      // 点击新增后，以“添加”模式打开模态框
      this.$refs.d2Crud.showDialog({
        mode: 'add'
      })
    },
    handleRowAdd (row, done) {
      // 点击确认添加后触发的事件，可以将数据传递到后台，保存至数据库中
      this.formOptions.saveLoading = true
      setTimeout(() => {
        // row 是表单提交的内容
        console.log(row)
        this.$message({
          message: '保存成功',
          type: 'success'
        });

        // done可以传入一个对象来修改提交的某个字段
        done({
          price: '你虽然提交了 但是我能在这修改你显示在页面的内容！'
        })
        this.formOptions.saveLoading = false
      }, 300)
    },
    handleRowEdit ({ index, row }, done) {
      // 点击确认修改后触发的事件，可以将数据传递到后台，保存至数据库中
      //  index 是当前编辑行的索引， row 是当前编辑行的数据， done 用于控制编辑成功，可以在 done() 之前加入自己的逻辑代码 
      this.formOptions.saveLoading = true
      setTimeout(() => {
        console.log(index)
        console.log(row)
        this.$message({
          message: '编辑成功',
          type: 'success'
        })

        // done可以传入一个对象来修改提交的某个字段
        // done()可以传入包含表单字段的对象来覆盖提交的数据，done(false) 可以取消编辑
        done({
          price: '你虽然在后台修改了价格，但是我能在这控制你在前台显示的内容'
        })
        this.formOptions.saveLoading = false
      }, 300)
    },
    handleRowRemove ({ index, row }, done) {
      // 与编辑类似
      setTimeout(() => {
        console.log(index)
        console.log(row)
        this.$message({
          message: '删除成功',
          type: 'success'
        })
        done()
      }, 300)
    },
    handleDialogCancel (done) {
      // 关闭模态框执行的事件，并可以自定义执行done函数
      this.$message({
        message: '取消保存',
        type: 'warning'
      });
      done()
    },
    paginationCurrentChange (currentPage) {
      // 分页页码发生改变触发的事件
      this.pagination.currentPage = currentPage
      this.fetchData()
    },
    fetchData () {
      // 点击分页按钮后，动态请求该页所需的数据
      this.$store.commit('product/product/getPageProducts', {pageSize: this.pagination.pageSize, currentPage: this.pagination.currentPage})
    }
  }
}
</script>

<style scoped>
  .flex-header {
    display: flex;
    justify-content: space-between;
    align-items:center
  }

  .header-title {
    min-width: 4rem;
  }
  .flex-header .el-input {
    width: 200px;
    margin-right: 10px;
  }
</style>
```

经测试，各功能全部正常。
![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142344999-958714308.png)

## 6 权限控制

**用户权限：**

- 使用token进行验证
- 使用sessionStorage保存用户信息（包含权限列表）
  - 登录时保存
  - 退出时删除
  - 结合路由meta信息进行判断
- meta：注意使用...meta，原来有bug，需要解析赋值(最新版中，该BUG已修复)

**控制权限的几种方式：**

- 控制查看表格，以及表格的操作按钮
- 无权限访问某页面或者页面部分内容

### 6.1 第一步，在 mock 中添加用户信息，以及不同的用户对应的不同的权限，并保存至本地的sessionStorage中。

- 在 \src\mock\api\sys.login.js 中定义用户信息，以及不同用户角色(role)对应不同页面(路由)的权限信息
- 定义获取用户信息接口，其中包括该用户对用角色的权限信息

```javascript
// src\mock\api\sys.login.js

import Mock from 'mockjs'

const userDB = [
  { username: 'admin', password: 'admin', uuid: 'admin-uuid', name: '管理员', role: 'admin' },
  { username: 'editor', password: 'editor', uuid: 'editor-uuid', name: '编辑', role: 'editor' },
  { username: 'user1', password: 'user1', uuid: 'user1-uuid', name: '用户1', role: 'user' }
]

// 为不同的用户角色(role)划分不同的权限

const permissions = {
  admin: [
    {
      path: '/index',
      operations: ['modify', 'delete', 'add']
    },
    {
      path: '/product',
      operations: ['modify', 'delete', 'add']
    },
    {
      path: '/order',
      operations: ['modify', 'delete', 'add']
    },
    {
      path: '/permission',
      operations: ['modify', 'delete', 'add']
    },
    {
      path: '/users',
      operations: ['modify', 'delete', 'add']
    },
    {
      path: '/menu',
      operations: ['modify', 'delete', 'add']
    },
    {
      path: '/stuff',
      operations: ['modify', 'delete', 'add']
    },
    {
      path: '/data-charts',
      operations: ['modify', 'delete', 'add']
    },
    {
      path: '/log'
    }
  ],
  editor: [{
    path: '/index',
    operations: ['modify', 'add']
    },
    {
      path: '/product',
      operations: ['modify', 'add']
    },
    {
      path: '/permission',
      operations: ['modify']
    },
    {
      path: '/order',
      operations: ['modify', 'add']
    }
  ],
  user: [{
    path: '/index',
    operations: ['add']
    },
    {
      path: '/product',
      operations: ['add']
    },
    {
      path: '/order',
      operations: ['add']
    }
  ]
}

Mock.mock('/api/userInfo', 'post', ({ body }) => {
  const bodyObj = JSON.parse(body)
  if (!bodyObj.token) {
    return {
      status: 401,
      msg: 'token错误，需要登录',
      data: {}
    }
  }
  const user = userDB.find(e => e.uuid === bodyObj.uuid)
  if (user) {
    return {
      status: 0,
      msg: '成功',
      data: {
        username: user.username,
        name: user.name,
        uuid: user.uuid,
        role: user.role,
        permissions: permissions[user.role]
      }
    }
  } else {
    return {
      status: 401,
      msg: '用户名或密码错误',
      data: {}
    }
  }
})

// 判断是否登录
export default [
  {
    path: '/api/login',
    method: 'post',
    handle ({ body }) {
      const user = userDB.find(e => e.username === body.username && e.password === body.password)
      if (user) {
        return {
          code: 0,
          msg: '登录成功',
          data: {
            ...user,
            token: '8dfhassad0asdjwoeiruty'
          }
        }
      } else {
        return {
          code: 401,
          msg: '用户名或密码错误',
          data: {}
        }
      }
    }
  }
]
```

### 6.2 第二步，在 src/API 中定义调用 mock 中 API 的接口，取到用户信息

```javascript
// src\api\sys.login.js

import request from '@/plugin/axios'

export function AccountLogin (data) {
  return request({
    url: '/login',
    method: 'post',
    data
  })
}

export function getUserInfo (data) {
  return request({
    url: '/userinfo',
    method: 'post',
    data
  })
}
```

### 6.3 第三步，在 router/index.js 中添加钩子函数。

- 获取跳转路由的 meta 信息，判断该路由是否需要验证。
- 若需验证，则从sessionStroage中获取用户的权限列表，如果该用户的权限列表中没有此路由信息，则跳转至401页面提示无权访问此页面，如果权限列表中有此路由信息，则将该用户对此路由的权限信息（如：增删改查）添加至路由的 meta 信息中，并跳转页面。
- 若不需要验证，直接跳转。

```javascript
// \src\router\index.js

import Vue from 'vue'
import VueRouter from 'vue-router'

// 进度条
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'

import store from '@/store/index'

import util from '@/libs/util.js'

// 路由数据
import routes from './routes'

Vue.use(VueRouter)

// 导出路由 在 main.js 里使用
const router = new VueRouter({
  routes
})

/**
 * 路由拦截
 * 权限验证
 */
router.beforeEach((to, from, next) => {
  // 进度条
  NProgress.start()
  // 关闭搜索面板
  store.commit('d2admin/search/set', false)
  // 获取路由的meta信息，判断当前页面是否需要验证（登录验证，权限验证等等）
  // 验证当前路由所有的匹配中是否需要有登录验证的
  if (to.matched.some(r => r.meta.auth)) {
    // 这里暂时将cookie里是否存有token作为验证是否登录的条件
    // 请根据自身业务需要修改
    const token = util.cookies.get('token') // 获取cookie中的token
    if (token && token !== 'undefined') {
      // 用户已登录，从sessionStorage中获取用户权限信息
      let userInfo = JSON.parse(sessionStorage.getItem('userInfo'));
      console.log(token, userInfo);
      if (userInfo) {
        let permissions = userInfo.permissions;
        console.log('router permissions:  ', permissions);
        let allow = false;
        permissions.forEach(p => {
          let item = router.match(p.path)  // 找到权限列表中匹配改条路由对应的权限信息
          if (item) {
            // 匹配到路由后，将权限信息添加在路由的meta中
            item.meta.operations = p.operations
          }

          // 完全匹配或者前缀匹配
          console.log('path:  ', to.path, p.path);
          if (p.path === to.path || to.path.startsWith(p.path)) {
            allow = true;
          }
        })

        // 根据allow判断是否可以跳转到下一个页面
        if (allow) {
          next()
        } else {
          next({ name: '401' })
        }
      } else {
        next()
      }
    } else {
      // 没有登录的时候跳转到登录界面
      // 携带上登陆成功之后需要跳转的页面完整路径
      next({
        name: 'login',
        query: {
          redirect: to.fullPath
        }
      })
      // https://github.com/d2-projects/d2-admin/issues/138
      NProgress.done()
    }
  } else {
    // 不需要身份校验 直接通过
    next()
  }
})

router.afterEach(to => {
  // 进度条
  NProgress.done()
  // 多页控制 打开新的页面
  store.dispatch('d2admin/page/open', to)
  // 更改标题
  util.title(to.meta.title)
})

export default router
```

### 6.4 第四步，在 store 中根据权限处理数据

- （用户登录的API）登录成功后把 userInfo 存至本地的 sessionStorage 中，注销登录后将用户信息从sessionStorage 中删除
- 通过取到 router 对象( `this.$route.meta.[operations]` )的 meta 中的权限信息(如：增删改查)，得出具体某个页面的操作权限信息，针对不同用户权限做出不同的操作。

> 注意：
>
> - $router: 表示全局的 router 对象
> - $route: 表示当前页面的路由对象

```javascript
// src\store\modules\d2admin\modules\account.js

import { Message, MessageBox } from 'element-ui'
import util from '@/libs/util.js'
import router from '@/router'
import { AccountLogin, getUserInfo } from '@api/sys.login'

export default {
  namespaced: true,
  actions: {
    /**
     * @description 登录
     * @param {Object} param context
     * @param {Object} param username {String} 用户账号
     * @param {Object} param password {String} 密码
     * @param {Object} param route {Object} 登录成功后定向的路由对象 任何 vue-router 支持的格式
     */
    login ({ dispatch }, {
      username = '',
      password = ''
    } = {}) {
      return new Promise((resolve, reject) => {
        // 开始请求登录接口
        AccountLogin({
          username,
          password
        })
          .then(async res => {
            // 设置 cookie 一定要存 uuid 和 token 两个 cookie
            // 整个系统依赖这两个数据进行校验和存储
            // uuid 是用户身份唯一标识 用户注册的时候确定 并且不可改变 不可重复
            // token 代表用户当前登录状态 建议在网络请求中携带 token
            // 如有必要 token 需要定时更新，默认保存一天
            util.cookies.set('uuid', res.uuid)
            util.cookies.set('token', res.token)

            // 登录成功后，加载用户信息和权限信息
            getUserInfo({ uuid: res.uuid, token: res.token }).then(res => {
              console.log('get user info:  ', res);
              if (res.status == 401) {
                return;
              } else {
                let userInfo = res.data;
                sessionStorage.setItem('userInfo', JSON.stringify(userInfo));
              }
            })

            // 设置 vuex 用户信息
            await dispatch('d2admin/user/set', {
              name: res.name
            }, { root: true })
            // 用户登录后从持久化数据加载一系列的设置
            await dispatch('load')
            // 结束
            resolve()
          })
          .catch(err => {
            console.log('err: ', err)
            reject(err)
          })
      })
    },
    /**
     * @description 注销用户并返回登录页面
     * @param {Object} param context
     * @param {Object} param confirm {Boolean} 是否需要确认
     */
    logout ({ commit, dispatch }, { confirm = false } = {}) {
      /**
       * @description 注销
       */
      async function logout () {
        // 删除cookie
        util.cookies.remove('token')
        util.cookies.remove('uuid')

        // 退出后将sessionStorage中的用户信息删除
        sessionStorage.removeItem('userInfo')

        // 清空 vuex 用户信息
        await dispatch('d2admin/user/set', {}, { root: true })
        // 跳转路由
        router.push({
          name: 'login'
        })
      }
      // 判断是否需要确认
      if (confirm) {
        commit('d2admin/gray/set', true, { root: true })
        MessageBox.confirm('注销当前账户吗?  打开的标签页和用户设置将会被保存。', '确认操作', {
          confirmButtonText: '确定注销',
          cancelButtonText: '放弃',
          type: 'warning'
        })
          .then(() => {
            commit('d2admin/gray/set', false, { root: true })
            logout()
          })
          .catch(() => {
            commit('d2admin/gray/set', false, { root: true })
            Message({
              message: '放弃注销用户'
            })
          })
      } else {
        logout()
      }
    },
    /**
     * @description 用户登录后从持久化数据加载一系列的设置
     * @param {Object} state vuex state
     */
    load ({ dispatch }) {
      return new Promise(async resolve => {
        // DB -> store 加载用户名
        await dispatch('d2admin/user/load', null, { root: true })
        // DB -> store 加载主题
        await dispatch('d2admin/theme/load', null, { root: true })
        // DB -> store 加载页面过渡效果设置
        await dispatch('d2admin/transition/load', null, { root: true })
        // DB -> store 持久化数据加载上次退出时的多页列表
        await dispatch('d2admin/page/openedLoad', null, { root: true })
        // DB -> store 持久化数据加载侧边栏折叠状态
        await dispatch('d2admin/menu/asideCollapseLoad', null, { root: true })
        // DB -> store 持久化数据加载全局尺寸
        await dispatch('d2admin/size/load', null, { root: true })
        // end
        resolve()
      })
    }
  }
}
```

### 6.5 第五步，渲染页面后，即可看到当前用户具体有哪些操作权限

- 针对表格操作，可以根据 meta 信息中存储的操作权限来判断当前用户所具有的权限，最后对其权限之外的按钮等进行隐藏或者disable。

```html
// src\views\permission\index.vue
<template>
  <d2-container>
    <template slot="header">页面权限验证</template>
    <el-card>
    <d2-crud
      ref="d2Crud"
      :columns="columns"
      :data="data"
      add-title="我的新增"
      edit-title="我的修改"
      :add-template="addTemplate"
      :edit-template="editTemplate"
      :rowHandle="rowHandle"
      :form-options="formOptions"
      @row-add="handleRowAdd"
      @row-edit="handleRowEdit"
      @row-remove="handleRowRemove"
      @dialog-cancel="handleDialogCancel">
    <el-button slot="header" type="primary" style="margin-bottom: 5px;" @click="addRow">新增</el-button>
    </d2-crud>
    </el-card>
    <template slot="footer">ZAdmin Created By <a href="">@ZBW</a> for D2-admin</template>
  </d2-container>
</template>

<script>
export default {
  data () {
    return {
      columns: [
        {
          title: 'ID',
          key: 'id',
          width: 50
        },
        {
          title: '日期',
          key: 'date',
          width: 150
        },
        {
          title: '姓名',
          key: 'name'
        },
        {
          title: '地址',
          key: 'address'
        },
        {
          title: '会员',
          key: 'role',
          width: 100
        }
      ],
      data: [
        {
          id: 1,
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄',
          role: '普通会员',
          showEditButton: this.$route.meta.operations.includes('modify'),
          showRemoveButton: this.$route.meta.operations.includes('delete')
        },
        {
          id: 2,
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1517 弄',
          role: '普通会员',
          showEditButton: this.$route.meta.operations.includes('modify'),
          showRemoveButton: this.$route.meta.operations.includes('delete')
        },
        {
          id: 3,
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄',
          role: '普通会员',
          showEditButton: this.$route.meta.operations.includes('modify'),
          showRemoveButton: this.$route.meta.operations.includes('delete')
        },
        {
          id: 4,
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1516 弄',
          role: '普通会员',
          showEditButton: this.$route.meta.operations.includes('modify'),
          showRemoveButton: this.$route.meta.operations.includes('delete')
        },
        {
          id: 5,
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄',
          role: '普通会员',
          showEditButton: this.$route.meta.operations.includes('modify'),
          showRemoveButton: this.$route.meta.operations.includes('delete')
        }
      ],
      rowHandle: {
        columnHeader: '编辑表格',
        edit: {
          icon: 'el-icon-edit',
          text: '编辑',
          size: 'small',
          show (index, row) {
            if (row.showEditButton) {
              return true
            }
            return false
          },
          disabled (index, row) {
            if (row.forbidEdit) {
              return true
            }
            return false
          }
        },
        remove: {
          icon: 'el-icon-delete',
          size: 'small',
          fixed: 'right',
          confirm: true,
          show (index, row) {
            if (row.showRemoveButton) {
              return true
            }
            return false
          },
          disabled (index, row) {
            if (row.forbidRemove) {
              return true
            }
            return false
          }
        }
      },
      addTemplate: {
        date: {
          title: '日期',
          value: '2018-11-11',
          component: {
            name: 'el-date-picker',
            span: 12
          }
        },
        role: {
          title: '会员',
          value: '普通会员',
          component: {
            name: 'el-select',
            options: [
              {
                value: '普通会员',
                label: '普通会员'
              },
              {
                value: '金卡会员',
                label: '金卡会员'
              }
            ],
            span: 12
          }
        },
        name: {
          title: '姓名',
          value: ''
        },
        address: {
          title: '地址',
          value: ''
        }
      },
      editTemplate: {
        date: {
          title: '日期',
          value: '2018-11-11',
          component: {
            name: 'el-date-picker',
            span: 12
          }
        },
        role: {
          title: '会员',
          value: '普通会员',
          component: {
            name: 'el-select',
            options: [
              {
                value: '普通会员',
                label: '普通会员'
              },
              {
                value: '金卡会员',
                label: '金卡会员'
              }
            ],
            span: 12
          }
        },
        name: {
          title: '姓名',
          value: ''
        },
        address: {
          title: '地址',
          value: ''
        },
        forbidEdit: {
          title: '禁用按钮',
          value: false,
          component: {
            show: false
          }
        },
        showEditButton: {
          title: '显示按钮',
          value: true,
          component: {
            show: false
          }
        }
      },
      formOptions: {
        labelWidth: '80px',
        labelPosition: 'left',
        saveLoading: false
      }
    }
  },
  methods: {
    addRow () {
      this.$refs.d2Crud.showDialog({
        mode: 'add'
      })
    },
    handleRowAdd (row, done) {
      this.formOptions.saveLoading = true
      setTimeout(() => {
        console.log(row)
        this.$message({
          message: '添加成功',
          type: 'success'
        });
        done({
          showEditButton: this.$route.meta.operations.includes('modify'),
          showRemoveButton: this.$route.meta.operations.includes('delete')
        })
        this.formOptions.saveLoading = false
      }, 300);
    },
    handleRowEdit ({index, row}, done) {
      this.formOptions.saveLoading = true
      setTimeout(() => {
        console.log(index)
        console.log(row)
        this.$message({
          message: '编辑成功',
          type: 'success'
        })
        done()
        this.formOptions.saveLoading = false
      }, 300)
    },
    handleDialogCancel (done) {
      this.$message({
        message: '操作已取消',
        type: 'warning'
      });
      done()
    },
    handleRowRemove ({index, row}, done) {
      setTimeout(() => {
        console.log(index)
        console.log(row)
        this.$message({
          message: '删除成功',
          type: 'success'
        })
        done()
      }, 300)
    }
  }
}
</script>

<style scoped>

</style>
```

**admin：**
![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142410444-1889575352.png)

**editer：**
![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142425894-1257811508.png)

**user：**
![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1669633-20190628142500329-1882903140.png)

