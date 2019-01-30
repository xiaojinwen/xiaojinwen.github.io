---
title: vue-element-控制字段排序与显示小组件
date: 2019-01-04 15:32:38
categories: ['前端'] 
tags: 前端

---
## 效果
![无法展示图片](http://web.xiaojw.xyz/blog_file/table.png)

## 代码
table-column.vue
```vue
<template>
    <div>
        <el-table-column v-if="selection" type="selection" width="50" fixed="left"></el-table-column>
        <el-table-column
                v-for="(item,index) in list" v-if="item.show"
                :prop="item.prop" :label="item.label" :min-width="item.width" :fixed="index===0">
            <template scope="scope">
                <div v-if="item.prop==='rz'" :style="{minWidth:item.width}">
                </div>
                <div v-else :style="{minWidth:item.width}">{{scope.row[item.prop]}}</div>
            </template>
        </el-table-column>
    </div>
</template>
<script>
    export default {
        props: {
            list: Array,
            selection: Boolean
        }
    }
</script>


```
common.js

```javascript
let data = []
switch (this.$route.path) {
    case '/order/inquiry':
      data = [
        {
          list: [
            {prop: 'name', label: '姓名', width: '100', show: true},
            {prop: 'phone', label: '手机号', width: '150', show: true}
          ],
          title: '全部',
          vision: '1'
        },
        {
          list: [
            {prop: 'name', label: '姓名', width: '100', show: true},
            {prop: 'status', label: '状态', width: '150', show: true}
          ],
          title: '未处理',
          vision: '1'
        }
      ]
      break;
    }
this.$emit('getShowContentListChange', data);
```
字段显示编辑组件
order-setting.vue

使用插件 vuedraggable
```vue
<template>
  <div class="order-setting">
    <div class="inner-order-setting"
         v-if="$route.path.includes('order')&&!$route.path.includes('detail')">
      <span style="color: #999;font-size: 16px;">显示字段</span>
      <i class="icon fa fa-cog" style="color: #999;font-size: 20px" @click="dialog=true"></i>
    </div>
    <el-dialog size="tiny" title="显示字段" :visible.sync="dialog">
      <div class="dialog-order-setting">
        <el-row type="flex" justify="space-around">
          <el-col :span="14">
            <div class="main-title">基础信息</div>
            <hr class="long-one m-b-10">
            <el-checkbox
              :indeterminate="isIndeterminate"
              v-model="checkAll"
              @change="handleCheckAllChange">全选
            </el-checkbox>
            <div v-for="(outerItem,index) in showContentListArr">
              <hr class="one m-t-20 m-b-20" v-if="index>=1">
              <!--:label="`${item.index}-${item.prop}`"-->
              <div>{{outerItem.title}}</div>
              <el-checkbox
                @change="handleCheckedChange"
                v-for="(item,index) in outerItem.list"
                v-model="item.show"
                :key="index">{{item.label}}
              </el-checkbox>
            </div>

          </el-col>
          <el-col :span="10">
            <span class="main-title" style="color: red;">编辑顺序</span>
            <hr class="long-one m-b-10">
            <el-radio-group v-model="active" @change="activeChange">
              <el-radio :label="index" v-for="(outerItem,index) in contentList">{{outerItem.title}}
              </el-radio>
            </el-radio-group>
            <draggable
              v-model="showContentList"
              class="co-draggable"
              :options="{group:'people'}"
              @end="onSortStart($event,showContentList)">
              <transition-group>
                <div v-for="(item,index) in showContentList" :key="index.toString()" v-if="item.show">
                  {{item.label}}
                  <i @click="delTap(index)" class="el-icon-close"></i>
                </div>
              </transition-group>
            </draggable>

          </el-col>
        </el-row>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="save">保存</el-button>
        <el-button type="primary" @click="reset">重置</el-button>
        <el-button type="primary" @click="dialog=false">取消</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import draggable from "vuedraggable";
  import {mapGetters, mapMutations} from "vuex";

  export default {
    components: {
      draggable
    },
    props: {
      contentList: {
        type: [Array, Object],
        default() {
          return [];
        }
      }
    },
    data() {
      return {
        dialog: false,
        showContentList: [],
        showContentListArr: [],
        checkAll: false,
        isIndeterminate: true,
        active: 0,
      };
    },
    methods: {
      onSortStart(to, arr) {
        this.showContentListArr[this.active].list = arr
      },
      save() {
        let obj = Object.assign({}, this.getShowContentList, {
          [this.$route.path]: this.showContentListArr
        });
        this.dialog = false;
        this.setShowContentList(obj);
        // 刷新页面 this.$router.go(0) 最好是转跳到一个刷新组件  router.replace({path: '/refresh', query: {path}})  在用组件里执行 router.replace({path: this.$route.query.path})
      },
      reset() {
        this.clearShowContentList();
        this.$router.go(0)
      },
      changeShowContent() {
        if (this.$route.path.includes("order")) {
            if (this.getShowContentList && this.getShowContentList[this.$route.path]) {
                // 当字段更新时 vision 需要修改和之前不一样的值 缓存的值才会重新赋值
                if (this.getShowContentList[this.$route.path].length !== this.contentList.length) {
                  // 重新保存
                  console.log('重新保存')
                  this.showContentListArr = this.contentList
                  this.save()
                  return
                } else {
                  for (let i = 0; i < this.contentList.length; i++) {
                    if (this.contentList[i].list.length !== this.getShowContentList[this.$route.path][i].list.length|| this.contentList[i].vision!==this.getShowContentList[this.$route.path][i].vision) {
                      // 重新保存
                      console.log('重新保存')
                      this.showContentListArr = this.contentList
                      this.save()
                      return
                    }
                  }
                }
              }        
          this.showContentListArr = this.getShowContentList && this.getShowContentList[this.$route.path] ? this.getShowContentList[this.$route.path] : this.contentList
          this.showContentList = this.showContentListArr[this.active].list
          // console.log(this.showContentListArr)
          this.$emit("onSubmit", this.showContentListArr);
        }
      },
      activeChange(active) {
        this.showContentList = this.showContentListArr[active].list
        // console.log(this.showContentList)
      },
      handleCheckAllChange(val) {
        if (val) {
          for (let i = 0; i < this.showContentListArr.length; i++) {
            this.showContentListArr[i].list.forEach((item) => {
              item.show = false
            })
          }
        } else {
          this.showContentListArr = this.contentList
        }
        this.showContentList = this.showContentListArr[this.active].list
        this.isIndeterminate = true;
      },
      handleCheckedChange(value) {
        let isCheckAll = false
        let num = 0, total = 0
        for (let i = 0; i < this.showContentListArr.length; i++) {
          for (let n = 0; n < this.showContentListArr[i].list.length; n++) {
            let item = this.showContentListArr[i].list[n]
            if (item.show) {
              isCheckAll = true
              num++
            }
            total++
          }
        }
        this.checkAll = isCheckAll;
        this.isIndeterminate = num > 0 && num < total;
      },
      delTap(index) {
        this.showContentList[index].show = false;
      },
      ...mapMutations({
        setShowContentList: "setShowContentList",
        clearShowContentList: "clearShowContentList",
      })
    },
    computed: {
      ...mapGetters(["getShowContentList"])
    },
    created() {
        this.changeShowContent();
    },
    watch: {
      contentList() {
        this.changeShowContent();
      }
    }
  };
</script>
<style>
  .dialog-order-setting .el-checkbox {
    margin-left: 0;
    margin-right: 30px;
    width: 100px;
  }

  .dialog-order-setting .el-radio {
    width: 100px;
    margin: 0 30px 10px 0;
  }

  .co-draggable div {
    padding: 6px 20px;
    cursor: all-scroll;
    border: 1px solid #e6e6e6;
    margin-bottom: 8px;
    border-radius: 4px;
    color: #363b42;
  }

  .co-draggable div i {
    float: right;
    color: #266298;
    font-size: 18px;
    vertical-align: middle;
    cursor: pointer;
  }
</style>

```

vuex getting.js
```javascript

const getters = {
    getShowContentList: state => state.showContentList
}

export default getters


```
vuex mutations.js

使用插件 Lockr
```javascript
import Lockr from 'lockr'

const mutations = {
     setShowContentList(state, showContentList) {
            if (showContentList) {
                state.showContentList = showContentList;
                Lockr.set('showContentList', showContentList)
            }
        },
        clearShowContentList(state) {
            state.showContentList = null;
            Lockr.rm('showContentList')
        },
}
export default mutations


```

vuex state.js
```javascript
import Lockr from 'lockr'

const state = {
    showContentList: Lockr.get('showContentList') || []
}
export default state
```

使用的地方
```vue
<template>
    <div>
        <el-table :data="tableData" style="width: 100%" stripe>
            <table-column :selection="true" :list="showContentList[0].list"></table-column>
            <el-table-column label="操作" width="80" fixed="right">
                <template scope="scope">
                </template>
            </el-table-column>
        </el-table>
    </div>
</template>
<script>
import tableColumn from 'common/table-column'

export default {
   props: ['showContentList'],  
   data(){
       return {
           tableData:[]
       }
   }
}
</script>

```


