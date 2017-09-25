title: JQuery隐藏/显示元素
tags: Js/JQuery
categories: Js/JQuery
---

### 1.方法一 ###

	$("#id").hide(); //隐藏元素
	$("#id").show(); //显示元素

### 2.方法二 ###
	$("id").css("display","none"); //隐藏元素
	$("id").css("display","block"); //显示元素

### 3.方法三 ###
	$("#id")[0].style.display="none"; //隐藏元素
	$("#id")[0].style.display="block"; //显示元素

### 4.方法四 ###
	$("id").toggle(); //切换元素状态
