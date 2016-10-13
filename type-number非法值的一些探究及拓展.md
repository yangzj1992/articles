---
title: 关于表单input type="number"非法值时的一些探究及拓展
date: 2015-09-01 13:58:50
categories: HTML
tags: [浏览器,表单,input,HTML5,兼容性]
---

## 问题描述

### 需求是这样的

今天在处理表单验证时发现了一个很诡异的现象，遂记录下来。

事情是这样的：
产品MM提了个需求，在微信端要求做一个表单，提交一些信息，然后其中有一个 input 需要直接调用数字键盘

### 最初思路

既然调用数字键盘，那么 `input type` 肯定就是 number 或者 tel。因为 tel 不能输入小数点，所以`input type`就为 number 了，然后在填表单时要做一些验证来及时反馈吧，所以代码大概就是这样的了

	<input type="number" name="retail_price" id="retail_price" placeholder="">
	
	alertDebug:function(formbug){
        let alertcontent ={
          6:'整数位最多不超过4位,小数位最多不超过2位',
        }
        let texts = {}
        texts.text = alertcontent[formbug]
        this.$dispatch('alertMsg',texts);
      },
      submitForm: function(event){
        let eventname = '';
        let formbug = false;
        if(this.$data.retail_price != ''){
          if(!this.validateNum(this.$data.retail_price) || this.checklength(this.$data.retail_price) > 4){
            if(!this.validateNum(this.$data.retail_price)){
              if(!eventname){
                formbug = 6;
              }
              if(eventname == 'retail_price'){
                formbug = 6;
                let temp = this.retail_price;
                this.retail_price = temp.toString().slice(0,-1);
                event.target.value = temp.toString().slice(0,-1);
              }
            }else{
              let retailprice = this.$data.retail_price + '';
              if(retailprice.indexOf('.') > -1){
                if(this.checklength(this.$data.retail_price) > 7){
                  if(!eventname){
                    formbug = 6;
                  }
                  if(eventname == 'retail_price'){
                    formbug = 6;
                    let temp = this.retail_price;
                    this.retail_price = temp.toString().slice(0,-1);
                    event.target.value = temp.toString().slice(0,-1);
                  }
                }
              }else{
                if(!eventname){
                  formbug = 6;
                }
                if(eventname == 'retail_price'){
                  formbug = 6;
                  let temp = this.retail_price;
                  this.retail_price = temp.toString().slice(0,-1);
                  event.target.value = temp.toString().slice(0,-1);
                }
              }
            }
          }
        }
        if(formbug){
          this.alertDebug(formbug);
          return false;
        }
      },
        
这样出现了一个问题在电脑 Chrome,iphone上，对于`input type="number"`的 input 都会将 value 变为"";而在安卓的微信上却显示为正确的 input number格式（如输入值为"1...."时，由于值并不为数字，非法，大多数按照[W3C相关规范]( http://www.w3.org/TR/html5/forms.html#number-state-(type=number)\)会将它处理为"",而在 Android 微信上为"1."）


## 相关知识

这里主要就是 input 的 number;其为 html5 加入的 type 类型。

[input html5 添加内容介绍](http://www.w3school.com.cn/html5/html_5_form_input_types.asp)

## 解决思路

   由于在 W3C 规范中，如果输入了一些非数字的字符，就会返回空字符串。

   但这样其实比较坑，导致验证的时候如果输入非数字的时候，直接使用 `.value`( 或者`$('.selector').val()`)都拿不到值，而拿不到值的情况下就会认为没有填写这个输入框。
	
   这里的解决方案就是在 input 的属性中有一个 validity 属性:
   
   在 Chrome 中，input 元素的 validity.badInput 这个属性里，可以判断值是否合法，如果填入了非法值，这个属性就是 true，正常值的话就是 false。

   但是火狐下.validity 里没有 badInput 属性，如下图，它可以直接通过.value 正常返回非数字的字符串。（微信 X5 同样也这样..）
   ![安卓微信](http://qcyoung.qiniudn.com/qcyoung/type-number非法值的一些探究及拓展/安卓微信，valid对象有的属性.png)

   而 IE8、9 则也可以直接 .value 获取到非数字的字符串值，不会返回空字符串。
   
