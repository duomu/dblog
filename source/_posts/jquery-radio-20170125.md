title: JQuery操作单选按钮radio
tags: Js/JQuery
categories: Js/JQuery
---
### 1.获取当前选中项的值 ###
	$('input:radio:checked').val();
	$("input[type='radio']:checked").val();
	$("input[name='radioName']:checked").val();


### 2.设置选中第一个radio ###
	$('input:radio:first').attr('checked', 'checked');
or

	$('input:radio:first').attr('checked', 'true');

### 3.设置选中最后一个radio ###
	$('input:radio:last').attr('checked', 'checked');
or

	$('input:radio:last').attr('checked', 'true');

### 4.设置选中指定值的radio ###
	$("input:radio[value='男']").attr('checked','true');
or

	$("input[value='男']").attr('checked','true');

### 5.删除指定值的radio ###
	$("input:radio[value='男']").remove();

### 6.删除指定索引的radio ###
	$("input:radio").eq(i).remove();//i=0,1,2....

### 7.遍历radio ###
	$('input:radio').each(function(index,domEle){

	});
