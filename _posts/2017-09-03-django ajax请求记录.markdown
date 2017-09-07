---
layout:     post
title:      "django + ajax 避免csrf 保护 "
subtitle:   " \"django ajax 请求记录\""
date:       2017-09-03 11:02:20
author:     "Daisy"
header-img: "img/post_django"
catalog: true
tags:
    - django
---

> “django ”

# django ajax 请求记录 #

> 关于csrf是什么东西请看[这里](http://www.jianshu.com/p/8ae7d3734381)，我们需要知道它是一种安全机制。ajax的解决方式原理上面的链接已经说的很清楚。下面开始记录

#### get请求 ####
**前端**

	$('#btn_get').click(function () {
        $.get('/get/', {'key1': 'value1'}, function (data,status) {
            alert(data+" status :"+status)
        })
    });

**django 处理**

    def deal_get(request):
    if request.method == 'GET':
        value = request.GET['key1']
        context = {'key': value, 'method': 'get'}
        return HttpResponse(json.dumps(context))


#### post请求 ####
> post请求和get请求方式大同小异，但是需要注意一个的就是django的csrf跨站点防护。

**前端，和get无差别**

	$('#btn_post').click(function () {
        $.post('/post/', {'key1': 'value1'}, function (data,status) {
            alert(data+" status :"+status)
        })
    });

需要注意的是，需要在$(document).ready()方法调用如下方法获取cookie中的csrf cookie。

    /*====================django ajax ======*/
	jQuery(document).ajaxSend(function (event, xhr, settings) {
	    function getCookie(name) {
	        var cookieValue = null;
	        if (document.cookie && document.cookie != '') {
	            var cookies = document.cookie.split(';');
	            for (var i = 0; i < cookies.length; i++) {
	                var cookie = jQuery.trim(cookies[i]);
	                // Does this cookie string begin with the name we want?
	                if (cookie.substring(0, name.length + 1) == (name + '=')) {
	                    cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
	                    break;
	                }
	            }
	        }
	        return cookieValue;
	    }
	
	    function sameOrigin(url) {
	        // url could be relative or scheme relative or absolute
	        var host = document.location.host; // host + port
	        var protocol = document.location.protocol;
	        var sr_origin = '//' + host;
	        var origin = protocol + sr_origin;
	        // Allow absolute or scheme relative URLs to same origin
	        return (url == origin || url.slice(0, origin.length + 1) == origin + '/') ||
	            (url == sr_origin || url.slice(0, sr_origin.length + 1) == sr_origin + '/') ||
	            // or any other URL that isn't scheme relative or absolute i.e relative.
	            !(/^(\/\/|http:|https:).*/.test(url));
	    }
	
	    function safeMethod(method) {
	        return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
	    }
	
	    if (!safeMethod(settings.type) && sameOrigin(settings.url)) {
	        xhr.setRequestHeader("X-CSRFToken", getCookie('csrftoken'));
	    }
	});
	/*===============================django ajax end===*/

**后台**

    def deal_post(request):
    if request.method == 'POST':
        value = request.POST['key1']
        context = {'key': value, 'method': 'post'}
        return HttpResponse(json.dumps(context))

很简单的功能，记录下来以备后记。
源码地址：[https://github.com/loopq/django_ajax_imple](https://github.com/loopq/django_ajax_imple)
