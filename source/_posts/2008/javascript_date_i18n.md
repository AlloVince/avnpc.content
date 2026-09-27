---
title: "Javasript Date对象的扩展、国际化、总结及其他"
date: "2008-06-04 17:05:16"
slug: "javascript_date_i18n"
published: true
author: "AlloVince"
legacy_id: 95
comment_status: "closed"
comments: false
tags:
  - "DateFormat"
  - "I18n"
  - "javascript"
  - "YD的程序员葛阁"
---
一个优秀的程序必须要兼顾I18n和L10N，但Javascript在Date处理这一方面做得很不友好，问题反映在以下几个方面：

1. 虽然Js也设置了UTC时间函数，但脚本运行时的时区是自动从系统获得的，且无法更改。而在实际应用中，往往需要接受一个非当前系统时区的时间数据进行转换，这时候Js的自作聪明就带来了许多麻烦。
2. Js对日期的自动解析和格式化输出根据系统环境、浏览器环境不同，表现也不同，这主要反映在Date.parse和toLocaleString方法上，有兴趣的同学可以自行测试。

为了尽可能简便的改善Js的Date处理能力，这里只对Js的Date对象做了两个扩展

```js
	/**
	 * 获得当前时间的UTC时间戳
	 * @return {int} unix timestamp
	 */
	Date.prototype.getTimeUTC = function() {
		return this.getTime() + this.getTimezoneOffset() * 60 * 1000;
	}
	/**
	 * 将当前操作的时间变更时区(主要用于转换一个其他时区的时间)
	 *
	 * @param {int} tzo 原时区 -12~13
	 * @param {int} tzo 目标时区 -12~13 默认为当前时区
	 */
	Date.prototype.changeTimezone = function(tzo,tzn) {
		tzo = tzo * 60;
		tzn = tzn ? tzn * 60 : -this.getTimezoneOffset();
		this.setTime(this.getTime() - (tzo - tzn) * 60 * 1000);
	}
```

至此，就可以基于这个扩展过的Js Date对象进行新的开发。

思路很简单，就是把任意格式任意时区的时间先通过changeTimezone方法转到和当前系统同一时区下，然后再对getTimeUTC生成的UTC Unix时间戳进行操作。

比如我要计算 +8 时区下 4 Jun 2008, 16:30时刻 与 +9 时区下的当前时刻相差的时间

```js
	//自动解析成unix时间戳
	var p = Date.parse('4 Jun 2008, 16:30');

	var time_parse = new Date(p);

	//转换到要对比的时区
	time_parse.changeTimezone(8,9);

	var time_now = new Date();

	//都转换为UTC进行对比
	var der = time_now.getTimeUTC() - time_parse.getTimeUTC();

	alert('相差' + parseInt(der / 1000 / 60) + '分钟');
```

当然有更简单的编码，但在复杂运用中理清思路更加不容易出错。

如果想要实现本Blog左侧栏XX天XX月前这样更人性化的提示，这里可以根据自己的需要做进一步扩展。实现的函数如下

```js
	/**
	 * 表示指定时间与现在的差值
	 *
	 * @param {int} t 所要比较的时间 unix timestamp (UTC)
	 * @param {int} n 作为标准的时间，默认为现在时刻 unix timestamp (UTC)
	 * @return {string} 对相差时间的表述
	 */
	Date.prototype.derTime = function(t,n) {
		var n = n ? n : this.getTimeUTC();
		function ms2min(ms) {
			return parseInt(ms / 1000 / 60);
		}
		var der = ms2min(n - t);
		var ba = der > 0 ? '前' : '后';
		der = Math.abs(der);
		var res = '';
		if(der == 0) {
			res = '刚刚';
		}
		else if(0 < der && der < 60) {
			res = der + '分钟' + ba;
		}
		else if(60 <= der && der < 24 * 60) {
			var min = der % 60 == 0 ? '' : String(der % 60) + '分钟';
			res = String(parseInt(der / 60)) + '小时' + min + ba;
		}
		else if(der >= 24 * 60 && der < 24 * 60 * 31) {
			res = String(parseInt(der / 60 / 24)) + '天' + ba;
		}
		else if(der >= 24 * 60 * 31 && der < 24* 60 * 365) {
			res = String(parseInt(der / 60 / 24 / 31)) + '个月' + ba;
		}
		else if(der > 24 * 60 * 365) {
			res = String(parseInt(der / 60 / 24 / 365)) + '年' + ba;
		}
		return res;
	}
	/**
	 * 解析一个时间字段与当前时间的差
	 * @param {string} i
	 * @param {int} 时区 -12~13
	 */
	function time_der(i,tz) {
		var p = Date.parse(i);
		if(!p) {
			return i;
		}
		var time_parse = new Date(p);
		if(tz != undefined) {
			time_parse.changeTimezone(tz);
		}
		var time_now = new Date();
		return time_now.derTime(time_parse.getTimeUTC());
	}
```
