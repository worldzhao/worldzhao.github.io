---
title: React计算属性-get关键字的妙用
date: 2018-02-06
tags: [react]
categories: react
---

> 前置阅读：[取值函数（getter）和存值函数（setter）](http://es6.ruanyifeng.com/#docs/class#%E5%8F%96%E5%80%BC%E5%87%BD%E6%95%B0%EF%BC%88getter%EF%BC%89%E5%92%8C%E5%AD%98%E5%80%BC%E5%87%BD%E6%95%B0%EF%BC%88setter%EF%BC%89)

## 计算属性

初次见到计算属性一词是在 Vue 官方文档-[计算属性和侦听器](https://cn.vuejs.org/v2/guide/computed.html)一节中。

> 模板内的表达式非常便利，但是设计它们的初衷是用于简单运算的。在模板中放入太多的逻辑会让模板过重且难以维护。

回想我们编写的 React 代码，是否也在 jsx（render 函数）中放入了太多的逻辑导致 jsx（render 函数）过于沉重？

通过`get`关键字，我们可以一样效果。

本例来源：[react-cookbook](https://github.com/shimohq/react-cookbook#%E8%AE%A1%E7%AE%97%E5%B1%9E%E6%80%A7)-石墨文档

```jsx
// 使用 getters 封装 render 所需要的状态或条件的组合
// 对于返回 boolean 的 getter 使用 is- 前缀命名

// bad
render () {
    return (
        <div>
        {
            this.state.age > 18
            && (this.props.school === 'A'
                || this.props.school === 'B')
            ? <VipComponent />
            : <NormalComponent />
        }
        </div>
    )
}

// good
get isVIP() {
    return
        this.state.age > 18
        && (this.props.school === 'A'
            || this.props.school === 'B')
    }
render() {
    return (
        <div>
        {this.isVIP ? <VipComponent /> : <NormalComponent />}
        </div>
    )
}
```

可以简单理解为：通过`get`关键字，抽离计算逻辑，根据 state 与 props 计算出衍生值。

props 以及 state 的变化会导致 render 函数调用，进而重新计算衍生值。与 Vue 中的定义基本一致。

```
props/state => render => get
```

可能有同学会问为什么不直接定义一个方法调用呢？

好吧，Vue 官方文档中进行了解释："我们可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。然而，不同的是计算属性是基于它们的依赖进行缓存的。只在相关依赖发生改变时它们才会重新求值。"

## 使用计算属性实现最优雅的表单联动

表单之间联动一直是后台管理类项目中的高频场景，简单描述为：一个（源）表单域的变化会导致另外一个（目标）表单域的变化。

假设存在业务类别以及处罚内容，并且不同类别对应不同的处罚内容，点击不同的业务类别，处罚内容是动态变化的，以业务类别为网上超市为例，见下图：

![交互.png](https://upload-images.jianshu.io/upload_images/4869616-2b162a367810ed3c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

完整对应关系如下（数字为对应 code）：

```
业务类别：网上超市-0
处罚内容：协议-0 商品-1 店铺-2 搜索-6

业务类别：在线询价-1
处罚内容：询价单报价-4

业务类别：协议供货-2
处罚内容：协议-0 商品-1

业务类别：反项竞价-4
处罚内容：发起竞价-5
```

现在我们来完成这个小需求 :)

### 准备工作-抽象对应数据结构

*为了避免在代码中出现无意义的神仙数字*以及*大量的 if-else*，我们给每一个 code 命名，同时将其文案以及对应内容进行抽象，如下（看个大概结构就好，我也觉得太长了）。

```js
/* 业务类型code */
export const businessTypeCode = {
  LUNATONE_CODE: 0, // 网超
  ENQUIRY_CODE: 1, // 在线询价
  PROTOCOL_CODE: 2, // 协议供货
  REVERSE_CODE: 4 // 反向竞价
}

/* 处罚内容code */
export const punishContentCode = {
  PROTOCOL_CODE: 0, // 协议
  ITEM_CODE: 1, // 商品
  SHOP_CODE: 2, // 店铺
  ENQUIRY_CODE: 4, // 询价单报价
  BIDDING_CODE: 5, // 发起竞价
  SEARCH_CODE: 6 // 搜索
}

/* 业务类型 code 及其对应的描述 desc 以及处罚内容 code */
export const businessTypeMap = {
  [businessTypeCode.LUNATONE_CODE]: {
    desc: '网上超市',
    matchedPunishContent: [
      punishContentCode.PROTOCOL_CODE,
      punishContentCode.ITEM_CODE,
      punishContentCode.SHOP_CODE,
      punishContentCode.SEARCH_CODE
    ]
  },
  [businessTypeCode.ENQUIRY_CODE]: {
    desc: '在线询价',
    matchedPunishContent: [punishContentCode.ENQUIRY_CODE]
  },
  [businessTypeCode.PROTOCOL_CODE]: {
    desc: '协议供货',
    matchedPunishContent: [
      punishContentCode.PROTOCOL_CODE,
      punishContentCode.ITEM_CODE
    ]
  },
  [businessTypeCode.REVERSE_CODE]: {
    desc: '反向竞价',
    matchedPunishContent: [punishContentCode.BIDDING_CODE]
  }
}
/* 处罚内容 code 及其对应的描述 desc */
export const punishContentMap = {
  [punishContentCode.PROTOCOL_CODE]: { desc: '协议' },
  [punishContentCode.ITEM_CODE]: { desc: '商品' },
  [punishContentCode.SHOP_CODE]: { desc: '店铺' },
  [punishContentCode.ENQUIRY_CODE]: { desc: '询价单报价' },
  [punishContentCode.BIDDING_CODE]: { desc: '发起竞价' },
  [punishContentCode.SEARCH_CODE]: { desc: '搜索' }
}
```

### 巧用计算属性

通常，我们是通过监听源表单域`onChange`事件，改变一个 state 属性，目标表单域则根据该 state 进行动态渲染。

譬如在此例中，我们可以在 state 中维护一个`matchedPunishContent`的字段，每次改变业务类别时，根据其 code 找到匹配的处罚内容，然后 setState 保存，取出 state 渲染处罚内容。

大体流程如下：

```
粗粒度：ui change => state change => ui change

细粒度：Radio=> onChange => 取得code找到对应处罚内容 => setState => 重新渲染处罚内容
```

如果表单联动关系较多，就会维护很多个 state 以及 onChange 函数，增加维护成本。

我们可以换一种思路，既然 Radio 的 value 值改变了，那么 getFieldValue 取得对应表单域的值也改变了。

通过如下方式，我们可以动态计算出处罚内容，而不需要去维护一个中间 state 以及一个 onChange 方法，大大减少了代码量。

```jsx
 /* 根据 businessType 获取 punishContent */
  get matchedPunishContent() {
    const businessType = this.props.form.getFieldValue('businessType')
    return businessTypeMap[businessType].matchedPunishContent
  }
```

完整代码见附录。

## 回填表单，从未如此简单

使用计算属性，你甚至可以享受到回填表单的美妙之处。

每一次回填表单数据，最难熬的莫过于不但要把数据填上去，并且需要还原表单状态，譬如某个表单域是选中的话，会导致一个表单域 disable，另一个表单域 hide 等等，之前是需要去维护 一个一个的 state，而有了计算属性，这一切操作，你只需要进行一次`setFieldsValue`（以及编写一些计算属性逻辑）。

以上文为例，回填数据时，对业务类型（bussiness）进行`setFieldsValue`操作（会导致`getFieldValue('businessType')`的改变），可以自动计算出处罚内容，而无需手动去还原。

### 实战

如图，如果勾选了`永久`(permanent)，该时间选择器禁用。

同时，回填时如果 permanent 字段为 PERMANENT_CODE，则要将时间选择器还原为禁用状态。

![disableDatePicker.png](https://upload-images.jianshu.io/upload_images/4869616-353f9b2038c5a16b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决如下：

```jsx
  get isDatePickerDisabled() {
    const { getFieldValue } = this.props.form;
    const permanent = getFieldValue('permanent') || [];
    return permanent.includes(PERMANENT_CODE);
  }
```

你只需要这般使用：

```jsx
<DatePicker disabled={this.isDatePickerDisabled} />
```

不论是用户手动点击亦或是开发者回填表单`setFieldsValue({permanent:[PERMANENT_CODE]})`

DatePicker 会自己乖乖禁用，而无需手动进行任何操作。

感谢阅读，欢迎探讨。

参考文章：

1. [Vue.js-计算属性和侦听器](https://cn.vuejs.org/v2/guide/computed.html)
2. [react-cookbook](https://github.com/shimohq/react-cookbook#%E8%AE%A1%E7%AE%97%E5%B1%9E%E6%80%A7)

---

附录：

```jsx
import React, { Component } from 'react'
import { Form, Radio } from 'doraemon'
import {
  businessTypeCode,
  punishContentCode,
  businessTypeMap,
  punishContentMap
} from './config'

const FormItem = Form.Item
const RadioGroup = Radio.Group

@Form.create()
class Test extends Component {
  /* 根据 businessType 动态计算 punishContent */
  get matchedPunishContent() {
    const businessType = this.props.form.getFieldValue('businessType')
    return businessTypeMap[businessType].matchedPunishContent
  }

  /* 渲染业务类别 */
  renderBusinessType = () => {
    const { getFieldDecorator } = this.props.form
    const businessTypeConfig = {
      rules: [{ required: true, message: '请选择业务类别' }],
      initialValue: businessTypeCode.LUNATONE_CODE
    }
    return (
      <FormItem label="业务类别">
        {getFieldDecorator('businessType', businessTypeConfig)(
          <RadioGroup>
            {Object.keys(businessTypeMap).map(value => {
              const { desc } = businessTypeMap[value]
              return (
                <Radio key={value} value={Number(value)}>
                  {desc}
                </Radio>
              )
            })}
          </RadioGroup>
        )}
      </FormItem>
    )
  }

  /* 渲染处罚内容 */
  renderPunishContent = () => {
    const { getFieldDecorator } = this.props.form
    const punishContentConfig = {
      rules: [{ required: true, message: '请选择处罚内容' }],
      initialValue: punishContentCode.PROTOCOL_CODE
    }
    return (
      <FormItem label="处罚内容">
        {getFieldDecorator('punishContent', punishContentConfig)(
          <RadioGroup>
            {this.matchedPunishContent.map(value => {
              const { desc } = punishContentMap[value]
              return (
                <Radio key={value} value={value}>
                  {desc}
                </Radio>
              )
            })}
          </RadioGroup>
        )}
      </FormItem>
    )
  }

  render() {
    return (
      <Form>
        {this.renderBusinessType()}
        {this.renderPunishContent()}
      </Form>
    )
  }
}
```
