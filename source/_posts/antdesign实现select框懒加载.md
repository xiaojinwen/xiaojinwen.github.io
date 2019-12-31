---
title: antdesign实现select框懒加载
date: 2019-12-23 10:52:49
categories: ['前端'] 
tags: 前端
comments: true
---

select框 还可以搜索
### 实现原理
即是手动去修改option的长度

- 将所有的option保存起来
- 首次显示默认条数
- 当select框选择内容滚动到底部 再增加条数
- 输入搜索内容时 清空之前的option 替换为搜索匹配到的新option(不要去重新赋值一个数组，而是修改原来的数组)

- 已知问题 initialValue值存在时会导致首次下拉选择框滚动到底部 加载更多option时的滚动条位置错误
- 不完美的解决 push数据前先清楚 initialValue值 操作完再恢复 
- 副作用 滚动到底部后select框的值会清空了 失焦后才恢复

参考文章 https://www.cnblogs.com/clairelss/p/11063808.html
```javascript
// 这是封装的组件  我拆出来了一部分
// template 部分
<a-select allowClear
        v-if="item.type == 'select'||item.type == 'multiple'"
        :getPopupContainer="triggerNode => triggerNode.parentNode"
        :mode="item.type == 'multiple'?'multiple':undefined"
        :showSearch="item.showSearch"
        v-decorator="[
                    `${item.decorator}`,
                    {
                    rules: [{
                        required: item.required,
                        message: `${item.Tips} field should be no-null`,
                    },...(item.rules && item.rules instanceof Array ? item.rules : [])],
                    initialValue:item.initialValue || undefined
                    }
                ]"
        :placeholder="item.isShowPlaceHolder?item.Tips:''"
        @change="handleInputChange($event,item.decorator)"
        :disabled="item.disabled"

        @popupScroll="selectScroll($event,item)"
        @search="fetchSelectSearchData($event,item)"
        @dropdownVisibleChange="SelectDropdownVisibleChange($event,item)">
<a-select-option :value="child.value"
                    v-for="(child,key) in item.options"
                    :key="key"
                    :disabled="child.disabled">{{child.name}}</a-select-option>
</a-select>
```

上面要看的主要是 
``````
@popupScroll="selectScroll($event,item)"
@search="fetchSelectSearchData($event,item)"
@dropdownVisibleChange="SelectDropdownVisibleChange($event,item)"
``````
这三个回调 其他的不用看

```javascript
 // script部分 mixin.ts
import { Vue, Component } from 'vue-property-decorator'
import {
  deepClone,
  isObject,
  distinct
} from '@/assets/ts';

declare module 'vue/types/vue' {
  interface Vue {
    timerId: number; // 用于防抖 解决输入框快速输入卡顿
    dataLen: number; // 数据每次加载的条数
    allOptionsDatakey: symbol; // 保存所有数据的 key
    isReady: symbol; // 用于判断数据是否被自己修改过
    searchValue: any; // 上一次输入框输入的值
    originOptions: any[]; // 保存修改过的option
    getSelectSearchDataList(item: any, value?: any): void; // 处理搜索和滚动时的数据
    selectScroll(e: any, item: any): void; // select滚动时调用 @popupScroll回调
    fetchSelectSearchData(value: any, item: any): void; // 输入内容搜索时调用 @search回调
    SelectDropdownVisibleChange(e: any, item: any): void; // select内容选择框显示时调用 @dropdownVisibleChange回调
  }
}

@Component
export default class selecctMixin extends Vue {
  /***
   * select 懒加载 开始
   * 实现原理即是 手动去修改option的长度
   * 
   * 将所有的option保存起来 (SelectDropdownVisibleChange)
   * 首次显示默认条数
   * 当select框选择内容滚动到底部 再增加条数(不要去重新赋值一个数组，而是修改原来的数组)
   * 输入搜索内容时 清空之前的option 替换为搜索匹配到的新option
   * 
   * 已知问题 initialValue值存在时会导致首次下拉选择框滚动到底部 加载更多option时的滚动条位置错误
   * 不完美的解决 push数据前先清楚 initialValue值 操作完再恢复 
   * 副作用 滚动到底部后select框的值会清空了 失焦后才恢复
   */
  timerId: number = 0; // 用于防抖 解决输入框快速输入卡顿
  dataLen: number = 100; // 数据每次加载的条数
  allOptionsDatakey: symbol = Symbol.for("allOptionsDatakey"); // 保存所有数据的 key
  isReady: symbol = Symbol.for("isReady"); // 用于判断数据是否被自己修改过
  searchValue: any; // 上一次输入框输入的值
  originOptions: any[] = []; // 保存修改过的option
  // select滚动时调用
  selectScroll(e: any, item: any) {
    const {
      target
    } = e;
    if (target.scrollTop + target.offsetHeight === target.scrollHeight) {
      item.scrollPage = item.scrollPage ? item.scrollPage + 1 : 1;
      this.getSelectSearchDataList(item, this.searchValue); // 调用处理数据方法
    }
  }

  // 输入内容搜索时调用
  fetchSelectSearchData(value: any, item: any) {
    if (this.timerId) {
      clearTimeout(this.timerId);
    }
    this.timerId = setTimeout(() => {
      item.scrollPage = 0;
      this.getSelectSearchDataList(item, value); // 调用处理数据方法
      this.searchValue = value;
    }, 300);
  }

  // 处理搜索和滚动时的数据
  getSelectSearchDataList(item: any, searchValue?: any) {
    const start: number = item.scrollPage ? item.scrollPage * this.dataLen : 0;
    const end: number = item.scrollPage ? (item.scrollPage + 1) * this.dataLen : this.dataLen;
    // console.log("start", start);
    // console.log("end", end);
    const allOptions: any[] = deepClone(item[this.allOptionsDatakey])
    const filterOption: any[] = searchValue ? [] : allOptions;
    // 执行 清空initialValue值之后 push完数据后再恢复原本值 原因是该值存在时会导致首次下拉选择框滚动到底部 加载更多option时的滚动条位置错误 如果没有这种问题可以不做此操作
    const initialValue: any = item.initialValue
    item.initialValue = undefined
    if (searchValue) {
      for (let innerItem of allOptions) {
        new RegExp(searchValue, "ig").test(
          isObject(innerItem) ? innerItem.value : innerItem
        ) && filterOption.push(innerItem);
      }
    }
    // 修改过输入框时 options清空
    this.searchValue !== searchValue && item.options.splice(0);
    filterOption.slice(start, end).length && item.options.push(...filterOption.slice(start, end));
    this.$nextTick(() => {
      item.initialValue = initialValue
    })
  }

  // select内容选择框显示时调用
  SelectDropdownVisibleChange(visible: any, item: any) {
    if (!item[this.isReady]) {
      // 把所有的数据保存起来 并且标记状态
      item[this.isReady] = true;
      item[this.allOptionsDatakey] = deepClone(distinct(item.options));
      this.originOptions.push(item)
    }
    // options赋值 this.dataLen 条数据
    item.options = deepClone(item[this.allOptionsDatakey]).slice(0, this.dataLen)
    if (!visible) {
      item.scrollPage = undefined;
      this.searchValue = undefined;
    }
  }

  public beforeDestroy() {
    // 恢复默认值
    this.originOptions.forEach((item: any) => {
      if (item[this.isReady]) {
        item.options = deepClone(item[this.allOptionsDatakey])
        delete item[this.allOptionsDatakey]
        delete item[this.isReady]
      }
    });
    this.originOptions = []
  }
  // select 懒加载结束
}

```


