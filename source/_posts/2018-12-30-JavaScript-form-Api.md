---
title: JavaScript表单校验Api
date: 2018-06-16 20:00:23
tags: js
categories: 前端
comments: true
---


JavaScript表单元素提供了一些内置的验证方法和属性，来对用户输入进行校验和错误显示，比如

- checkValidity()，input 元素中的数据是合法的返回 true，否则返回 false
- 设置 input 元素的 validationMessage属性，用于自定义错误提示信息的方法。
使用 setCustomValidity 设置了自定义提示后，validity.customError 就会变成true，则 checkValidity 总是会返回false

例子：

    <input id="id1" type="number" min="100" max="300" required>
    <button onclick="myFunction()">验证</button>
    <script>
    function myFunction() {
        var inpObj = document.getElementById("id1");
        if (inpObj.checkValidity() == false) {
            document.getElementById("demo").innerHTML = inpObj.validationMessage;
        }
    }
    </script>


### 验证 DOM 属性

##### validity
布尔属性值，返回 input 输入值是否合法
input元素的 validity属性包含一系列关于数据属性

- customError	设置为 true, 如果设置了自定义的 validity 信息。
- patternMismatch	设置为 true, 如果元素的值不匹配它的模式属性。
- rangeOverflow	设置为 true, 如果元素的值大于设置的最大值。
- rangeUnderflow	设置为 true, 如果元素的值小于它的最小值。
- stepMismatch	设置为 true, 如果元素的值不是按照规定的 step 属性设置。
- tooLong	设置为 true, 如果元素的值超过了 maxLength 属性设置的长度。
- typeMismatch	设置为 true, 如果元素的值不是预期相匹配的类型。
- valueMissing	设置为 true，如果元素 (required 属性) 没有值。
- valid	设置为 true，如果元素的值是合法的。

例子：

    <input id="id1" type="number" max="100">
    <button onclick="myFunction()">验证</button>
    <script>
    function myFunction() {
        var txt = "";
        if (document.getElementById("id1").validity.rangeOverflow) {
           txt = "输入的值太大了";
        }
        document.getElementById("demo").innerHTML = txt;
    }
    </script>

##### validationMessage
浏览器错误提示信息
使用setCustomValidity（error : string）来设置validationMessage，并可手动取消他们

    setCustomValidity('')；
    setCustomValidity(null)
    setCustomValidity(undefined)

##### willValidate
表示input 是否需要验证
比如,如果表单元素设置了required特性或pattern特性，则willValidate属性的值为true



