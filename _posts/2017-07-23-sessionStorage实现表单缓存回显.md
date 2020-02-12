---
layout: post
title: sessionStorage 实现表单缓存会先
categories: html5
tags: [html5]
comments: true
description: html5 使用 sessionStorage 实现表单缓存回显
---


# sessionStorage 和 localStorage 的区别

html5 中的 Web Storage 包含了两种方式：sessionStorage 和 localStorage.
==sessionStorage==用于存本地存储一次会话（session）中的数据, 这些数据只能在同一个会话中的页面才能访问, 会话结束数据也随之销毁.sessionStorage 不是一种持久化得本地存储, 仅仅是会话级别的存储.
==localStorage==用于持久化的本地存储, 除非主动删除数据, 否则数据永不过期.


# localStorage 和 sessionStorage 的方法

-   setItem 存储 value

用途： 将 value 存储到 key 字段
\`sessionStorage.setItem("key","value");     localStorage.setItem("key","value");\`

-   getItem 获取 value

用途：获取指定 key 本地存储的值
\`var value = sessionStorage.getItem("key");     localStorage.getItem("key");\`

-   removeItem 删除 key

用途：删除指定 key 本地存储的值
\`sessionStorage.removeItem("key");     localStorage.removeItem("key");\`

-   clear 删除所有的 key

用途：删除所有的 key 本地存储的值
\`sessionStorage.clear();        localStorage.clear();\`

-   \`.\`号操作符和\`[]\`运算

用途：类似 setItem 和 getItem
\`window.sessionStorage.storageKey = object;     window.sessionStorage[storageKey] = object;\`    设置值

\`var object = window.sessionStorage.storageKey;     var object = window.sessionStorage.[storageKey]\`    取值

-   storage 事件

storage 还提供了 storage 事件, 当键值对改变或者 clear 的时候会触发 storage 事件

    window.addEventListener('storage', function(e){
        //TODO
        console.log(e);
    });


# 应用 页面数据存储、获取及清除

-   保存表单数据到 sessionStorage
    
        function saveDataToStorage(pageName){   //pageName 为每个页面独特的名称, 用于区别各页面的数据, 在 storage 中用作 key
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
            //保存 button 的 text
            for(var i = 0; i&lt;$('form button').length; i++){
                if($($('form button')[i])[0].type == 'button'){
                    var key = $($('form button')[i]).attr('id');
                    history[key] = $($('form button')[i]).text();
                }
            }
            window.sessionStorage.setItem(pageName, history);
        }

-   加载 sessionStorage 中数据到表单
    
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
                //回显 button  text 内容
                for(var i=0; i&lt;$('form button').length; i++){
                    var key = $($('form input')[i]).attr('id');
                    $($('form button)[i]).text(localMsg[key]);
                }
            }
        }

-   清除 sessionStorage 内的所有数据, 在需要失效的时机手动调用
    
        function clearSessionStorage(){
            window.sessionStorage.clear();
        }


# 数组方式存储 sessionStorage 的 value

> [HTML5 sessionStorage](http://www.cnblogs.com/Ricky-Huang/p/5736623.html)

    /*保存表单数据到 sessionStorage*/
    function saveDataToStorage(pageName) {
        // debugger;
        var history=[];
        window.sessionStorage.setItem(pageName,'');
        for(var i = 0;i<$('form input').length; i++){
            if($($('form input')[i])[0].type=='text' || $($('form input')[i])[0].type=='range'){
                history.push({'text':$($('form input')[i]).val()})
            }
        }
        /*保存 button 的 text*/
        for(var i = 0; i<$('form button').length; i++){
            if($($('form button')[i])[0].type=='button'){
                history.push({'text':$($('form button')[i]).text()})
            }
        }
        window.sessionStorage.setItem(pageName,JSON.stringify(history));
    }
    
    /*清除 sessionStorage 内的数据*/
    function clearSessionStorage() {
        window.sessionStorage.clear();
    }
    
    /*加载 sessionStorage 数据到表单*/
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
            /*回填 button text 内容*/
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

