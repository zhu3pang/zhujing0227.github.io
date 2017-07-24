---
layout: post
title: sessionStorage实现表单缓存回显
tags: html5
category: html5
#eye_catch: http://jekyllrb.com/img/logo-2x.png
---
html5中的Web Storage包含了两种方式：sessionStorage和localStorage。

### sessionStorage和localStorage的区别

**sessionStorage**用于存本地存储一次会话（session）中的数据，这些数据只能在同一个会话中的页面才能访问，会话结束数据也随之销毁。sessionStorage不是一种持久化得本地存储，仅仅是会话级别的存储。

**localStorage**用于持久化的本地存储，除非主动删除数据，否则数据永不过期。
<!--more-->
<!--more-->

----------

### localStorage和sessionStorage的方法

- setItem存储value

用途： 将value存储到key字段
`sessionStorage.setItem("key","value");     localStorage.setItem("key","value");`

- getItem获取value

用途：获取指定key本地存储的值
`var value = sessionStorage.getItem("key");     localStorage.getItem("key");`

- removeItem删除key

用途：删除指定key本地存储的值
`sessionStorage.removeItem("key");     localStorage.removeItem("key");`

- clear删除所有的key

用途：删除所有的key本地存储的值
`sessionStorage.clear();        localStorage.clear();`

- `.`号操作符和`[]`运算

用途：类似setItem和getItem
`window.sessionStorage.storageKey = object;     window.sessionStorage[storageKey] = object;`    设置值

`var object = window.sessionStorage.storageKey;     var object = window.sessionStorage.[storageKey]`    取值

- storage事件

storage还提供了storage事件，当键值对改变或者clear的时候会触发storage事件

```javaScript
window.addEventListener('storage', function(e){
    //TODO
    console.log(e);
});
```

----------

### 应用 页面数据存储、获取及清除

- 保存表单数据到sessionStorage

```javaScript
function saveDataToStorage(pageName){   //pageName为每个页面独特的名称，用于区别各页面的数据，在storage中用作key
    var history = {};
    for(var i=0; i&lt;$('form input').length; i++){
        var key = $($('form input')[i]).attr('name');
        if($($('form input')[i])[0].type == 'text' || $($('form input')[i])[0].type == 'range'){
            history[key] = $($('form input')[i]).val();
        }else if($($('form input')[i])[0].type == 'radio'){
            history[key] = $($('form input')[i]).attr('checked') ? 'checked' :'';
        }else if($($('form input')[i])[0].type == 'checkbox'){
            history[key] = $($('form input')[i]).attr('checked') ? 'checked' :'';
        }
    }
    //保存button的text
    for(var i = 0; i&lt;$('form button').length; i++){
        if($($('form button')[i])[0].type == 'button'){
            var key = $($('form button')[i]).attr('id');
            history[key] = $($('form button')[i]).text();
        }
    }
    window.sessionStorage.setItem(pageName, history);
}
```

- 加载sessionStorage中数据到表单

```javaScript
function fillSessionStorage(pageName){
    var localMsg;
    if(window.sessionStorage.getItem(pageName)){
        localMsg = window.sessionStorage.getItem(pageName);
    }
    if(localMsg && localMsg!=''){
        for(var i=0; i&lt;$('form input').length; i++){
            var key = $($('form input')[i]).attr('name');
            if($($('form input')[i])[0].type == 'text' || $($('form input')[i])[0].type == 'range'){
                $($('form input')[i]).attr('value', localMsg[key]);
            }else if($($('form input')[i])[0].type == 'radio'){
                $($('form input')[i]).prop('checked', localMsg[key]);
            }else if($($('form input')[i])[0].type == 'checkbox'){
                $($('form input')[i]).prop('checked', localMsg[key]);
            }
        }
        //回显button  text内容
        for(var i=0; i&lt;$('form button').length; i++){
            var key = $($('form input')[i]).attr('id');
            $($('form button)[i]).text(localMsg[key]);
        }
    }
}
```

- 清除sessionStorage内的所有数据，在需要失效的时机手动调用

```javaScript
function clearSessionStorage(){
    window.sessionStorage.clear();
}
```

----------

### 数组方式存储sessionStorage的value

> [HTML5 sessionStorage](http://www.cnblogs.com/Ricky-Huang/p/5736623.html)

```javaScript
/*保存表单数据到sessionStorage*/
function saveDataToStorage(pageName) {
    // debugger;
    var history=[];
    window.sessionStorage.setItem(pageName,'');
    for(var i = 0;i<$('form input').length; i++){
        if($($('form input')[i])[0].type=='text' || $($('form input')[i])[0].type=='range'){
            history.push({'text':$($('form input')[i]).val()})
        }
    }
    /*保存button的text*/
    for(var i = 0; i<$('form button').length; i++){
        if($($('form button')[i])[0].type=='button'){
            history.push({'text':$($('form button')[i]).text()})
        }
    }
    window.sessionStorage.setItem(pageName,JSON.stringify(history));
}

/*清除sessionStorage内的数据*/
function clearSessionStorage() {
    window.sessionStorage.clear();
}

/*加载sessionStorage数据到表单*/
function fillSessionStorage(pageName) {
    var localMsg;
    if(window.sessionStorage.getItem(pageName)){
        localMsg=JSON.parse(window.sessionStorage.getItem(pageName));
    }
    if(localMsg && localMsg.length>=1){
        var realIndex=0;
        for(var i=0;i<$('form input').length;i++){
            if($($('form input')[i])[0].type=='text' || $($('form input')[i])[0].type=='range'){
                $($('form input')[i]).attr('value', localMsg[realIndex].text);
                // console.log(localMsg[realIndex].text);
                realIndex++;
            }
        }
        /*回填button text内容*/
        for(var i = 0; i<$('form button').length;i++){
            if($($('form button')[i])[0].type=='button'){
                $($('form button')[i]).text(localMsg[realIndex].text);
                //.attr('text', localMsg[realIndex].text)
                // console.log(localMsg[realIndex].text);
                realIndex++;
            }
        }
    }
}
```
