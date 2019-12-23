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

        @popupScroll="searchScroll($event,item)"
        @search="fetchSearchData($event,item)"
        @dropdownVisibleChange="searchDropdownVisibleChange($event,item)">
<a-select-option :value="child.value"
                    v-for="(child,key) in item.options"
                    :key="key"
                    :disabled="child.disabled">{{child.name}}</a-select-option>
</a-select>
```

上面要看的主要是 
``````
@popupScroll="searchScroll($event,item)"
@search="fetchSearchData($event,item)"
@dropdownVisibleChange="searchDropdownVisibleChange($event,item)"
``````
这三个回调 其他的不用看

```javascript
 // script部分
/***
   * select 懒加载 开始
   */
   // 滚动回调
  private searchScroll(e: any, item: any) {
    const { target } = e;
    // if (target.scrollTop + target.offsetHeight + 80 >= target.scrollHeight) {
    if (target.scrollTop + target.offsetHeight === target.scrollHeight) {
      item.scrollPage = item.scrollPage ? item.scrollPage + 1 : 1;
      this.getSearchDataList(item, this.searchValue); // 调用api方法
      // console.log("加载");
      // console.log("this.searchValue", this.searchValue);
    }
  }

  private timerId: number = 0; // 用于防抖 解决输入框快速输入卡顿
  private fetchSearchData(value: string, item: any) {
    if (this.timerId) {
      clearTimeout(this.timerId);
    }
    this.timerId = setTimeout(() => {
      item.scrollPage = 0;
      // this.setState({ companyData: [], fetching: true });
      this.getSearchDataList(item, value); // 关键字模糊查询api
      this.searchValue = value;
      // console.log("搜索", value);
    }, 300);
  }

  private dataLen: number = 100; // 数据每次加载的条数
  // 查询数据
  private getSearchDataList(item: any, value?: string) {
    // console.log("item", item);
    // console.log("item.scrollPage", item.scrollPage);
    const start: number = item.scrollPage ? item.scrollPage * this.dataLen : 0;
    const end: number = item.scrollPage
      ? (item.scrollPage + 1) * this.dataLen
      : this.dataLen;
    // console.log("start", start);
    // console.log("end", end);
    // console.log("item.options", item.options);
    const filterOption: any = value ? [] : item[this.allOptionsDatakey];
    if (value) {
      // console.log("匹配");
      for (let innerItem of item[this.allOptionsDatakey]) {
        new RegExp(value, "ig").test(
          isObject(innerItem) ? innerItem.value : innerItem
        ) && filterOption.push(innerItem);
      }
    }
    // 修改过输入框时 清空
    this.searchValue !== value && item.options.splice(0);
    // console.log("push", filterOption.slice(start, end));
    filterOption.slice(start, end).length &&
      item.options.push(...filterOption.slice(start, end));
  }

  private allOptionsDatakey: symbol = Symbol.for("allOptionsDatakey"); // 保存所有数据的 key
  private isReady: symbol = Symbol.for("isReady"); // 用于判断数据是否被自己修改过
  private searchValue: any; // 上一次输入框输入的值
  private searchDropdownVisibleChange(e: any, item: any) {
    // console.log('e', e);
    console.log("item", item);
    if (!item[this.isReady]) {
      // console.log("把所有的数据保存起来");
      item[this.isReady] = true;
      // item[this.allOptionsDatakey] = Array.from(deepClone(item.options))
      item[this.allOptionsDatakey] = deepClone(item.options);
      item.options = (item.options || []).slice(0, this.dataLen);
    } else {
      // 赋值所有数据 只修改原数组
      item.options.splice(0);
      item.options.push(...item[this.allOptionsDatakey].slice(0, this.dataLen));
    }
    item.scrollPage = undefined;
    this.searchValue = undefined;
  }
  // select 懒加载结束
```


