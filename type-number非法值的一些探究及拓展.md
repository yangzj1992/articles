title: 关于表单input type="number"非法值时的一些探究及拓展
date: 2015-09-01 13:58:50
categories: HTML
tags: [浏览器,表单,input,HTML5,兼容性]
---

## 问题描述

### 需求是这样的

今天在处理表单验证时发现了一个很诡异的现象，遂记录下来。

事情是这样的：
产品MM提了个需求，在微信端要求做一个表单，提交一些信息，然后其中有一个input需要直接调用数字键盘

### 最初思路

既然调用数字键盘，那么`input type`肯定就是number或者tel。因为tel不能输入小数点，所以`input type`就为number了，然后在填表单时要做一些验证来及时反馈吧，所以代码大概就是这样的了

	<input type="number" name="retail_price" id="retail_price" placeholder="">
	
	$("#retail_price").on('keyup',function(e){
            var code = e.keyCode;
            if(code != 37 && code != 39 && code != 46 && code != 8){
                console.log($(this)[0].validity);
                var val = $.trim(this.value + "" );
                var matches = defaults.reg.exec(val);
                if(matches && matches[0] != undefined){
                    if($(this).val() != ""){
                        if(!validateNum($(this).val()) || checklength($(this).val()) > 4){
                            if(!validateNum($(this).val())){
                                $(this).val($(this).val().slice(0,-1));
                                jsAlert('整数位最多不超过4位,小数位最多不超过2位');
                            }else{
                                if($(this).val().indexOf(dot) > -1){
                                    if(checklength($(this).val()) > 7){
                                        $(this).val($(this).val().slice(0,-1));
                                        jsAlert('整数位最多不超过4位,小数位最多不超过2位');
                                    }
                                }else{
                                    $(this).val($(this).val().slice(0,-1));
                                    jsAlert('整数位最多不超过4位,小数位最多不超过2位');
                                }
                            }
                        }
                    }
                }else{
                    $(this).val($(this).val().slice(0,-1));
                    jsAlert('整数位最多不超过4位,小数位最多不超过2位');
                }
            };
        });
        
这样出现了一个问题在电脑chrome,iphone上，对于`input type="number"`的input都会将value变为"";而在安卓的微信上却显示为正确的input number格式（如输入值为"1...."时，由于值并不为数字，非法，大多数按照[W3C相关规范]( http://www.w3.org/TR/html5/forms.html#number-state-(type=number)\)会将它处理为"",而在android微信上为"1."）


## 相关知识

这里主要就是input的number;其为html5加入的type类型。

[input html5添加内容介绍](http://www.w3school.com.cn/html5/html_5_form_input_types.asp)

## 解决思路

   由于在W3C规范中，如果输入了一些非数字的字符，就会返回空字符串。

   但这样其实比较坑，导致验证的时候如果输入非数字的时候，直接使用`.value`( 或者`$('.selector').val()`)都拿不到值，而拿不到值的情况下就会认为没有填写这个输入框。
	
   这里的解决方案就是在input的属性中有一个validity属性:
   
   在chrome中，input元素的validity.badInput这个属性里，可以判断值是否合法，如果填入了非法值，这个属性就是true，正常值的话就是false。

   但是火狐下.validity里没有badInput属性，如下图，它可以直接通过.value正常返回非数字的字符串。（微信X5同样也这样..)
   ![安卓微信](http://qcyoung.qiniudn.com/qcyoung/type-number非法值的一些探究及拓展/安卓微信，valid对象有的属性.png)

   而IE8、9则也可以直接.value获取到非数字的字符串值，不会返回空字符串。
   
