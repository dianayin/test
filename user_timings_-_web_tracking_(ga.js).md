##User Timings - Web Tracking (ga.js)
___
[介绍][1]   
[设置用户时间][2] Setting Up User Timings   
[记录花费时间][3]Tracking Time Spent   
[多用户使用][4] Working With Multiple User    
[记录][5] XMLHttpRequests Tracking XMLHttpRequests   
[避免发送错误数据][6] Avoid Sending Bad Data   
[重叠样品比率][7] Overriding the Sample Rate   
[记录其他时间事件][8] Tracking Other Time Events    
[Working Demo](http://analytics-api-samples.googlecode.com/svn/trunk/src/tracking/javascript/v5/user-timing/user-timing.html)
[Sample Code](https://code.google.com/p/analytics-api-samples/source/browse/trunk/src/tracking/javascript/v5/user-timing/user-timing.js)



###介绍 {#p1}
---
研究说明减少页面加载时间会提高网站整体的用户体验。GA有多个强大的报告可以自动记录页面加载时间。但是如果你不想记录一个资源的加载时间呢？

比如说，如果一个JS库加载速度慢会不会降低一些用户的体验?

用户时间可以回答这个问题，因为他提供一个本土方式来记录在GA的一段时间(native way)。

[可以使用版本](#p10)请看用户时间的[样品代码](#p11)

<a href="#top">↑ 回到顶部</a>


###设置用户时间 {#p2} Setting Up User Timings  
---
收集用户时间数据需要用 [_trackTiming](https://developers.google.com/analytics/devguides/collection/gajs/methods/gaJSApiUserTiming),用来给GA发时间数据的。

	_gaq.push([‘_trackTiming’, category, variable, time, opt_label, opt_sample]);

参数介绍

 
|       参数     |   函数 | 必有 | 介绍
| -------------- | ------ | --- |---
|    category    | string | yes | 一个字符串用来归类所有用户的时间变量成逻辑组，报告时更容易。例如，你可以使用jQuery的价值，如果你在跟踪加载一个特定的JavaScript库所花费的时间。
|    variable    | string | yes | 一个字符串用来指示被跟踪的资源的名称的行动。例如，如果你想跟踪加载jQuery JavaScript库所花费的时间，您可能使用``JavaScript Load``值。需要注意的是，相同的变量有可能跨分类的被使用来跟踪计时，如``Javascript Load``和``Page Ready Time``等。
|      time      | number | yes | 用毫秒报给GA的已过时间。如果jQuery库加载了20毫秒，那么所送价值是20。
|    opt_label   | string |  no | 一个字符串，可用于在可视化用户报告中的时间增加更多灵活性。标签也可以用于关注同类别的不同子实验和变量组合。例如，如果我们从Google Content Delivery Network载入jQuery，我们将使用``Google CDN``的价值。
| opt_sampleRate | number |  no | 一个数字用来覆盖发送给GA的计时点击的游客百分比。默认设置跟一般网站数据采集是相同的数字，是基于访问者的百分比。所以，如果你想跟踪_trackTiming命中100％的参观者，您可以使用``100``。请注意，每个点击数都会被算进500次点击一个session。


<a href="#top">↑ 回到顶部</a>

###记录花费时间 {#p3} Tracking Time Spent   
---
当使用``_trackTiming``，您可以定义``time``参数花了多少毫秒。作为开发者，您可以随意写追捕时间这段代码。最容易实现的方法就是在一段时间的开始和结束各创建一个时间戳。采取的这两个时间戳记之间的差异来得到花费时间。

简单例子：

	var startTime = new Date().getTime();

	setTimeout(myCallback, 200);

	function myCallback(event) {

	var endTime = new Date().getTime();
	var timeSpent = endTime - startTime;

	_gaq.push(['_trackTiming', 'Test', 'callback_timeout', timeSpent, 'animation']);
	}

开始时间戳会被获取当一个新``date``object被创建并以毫秒为时间单位被获取。接下来，``setTimeout``函数用于200毫秒调用``myCallback``功能。一旦``myCallback``被执行，结束时间戳是通过创建一个新的``Date``object被获取。结束和开始时间的差异，计算出来就是花费的时间。最终花费时间被发送到谷歌Analytics（分析）。

这个例子很简单，但足以说明如何追踪时间的概念。让我们来看看一个更现实的例子。

<a href="#top">↑ 回到顶部</a>

###记录JS资源加载花费时间  Tracking Time Spent Loading a JavaScript Resource
---
今天，许多网站会使用第三方JS库，或从JSON objects请求的数据。在您家里，您的网站可能会快速的加载这些资源但是在其他国家的用户，相同的资源，加载速度有可能非常慢。这些加载速度慢的资源可能会降低国际用户的经验。

用户网站速度计时功能可以帮助您收集和报告这些资源需要多久才能加载。

下面是一个简单的例子演示如何记录一个异步加载JavaScript资源的功能所花费的时间。

	var startTime;

	function loadJs(url, callback) {
	var js = document.createElement('script');
	js.async = true;
	js.src = url;
	var s = document.getElementsByTagName('script')[0];

	js.onload = callback;
	startTime = new Date().getTime();

	s.parentNode.insertBefore(js, s);
	}
	
	function myCallback(event) {
	var endTime = new Date().getTime();
	var timeSpent = endTime - startTime;
	_gaq.push(['_trackTiming', 'jQuery', 'Load Library', timeSpent, 'Google CDN']);

	// Library has loaded. Now you can use it.
	};

请注意前面的例子和这个例子非常相似。

在这个例子中，``loadJs``是一个实用功能，通过动态创建一个程序元素加载的JavaScript资源，和将其附加到浏览器的DOM。该函数接受两个参数：一个URL字符串，和script加载后执行一次的一个回调函数。

在``loadJs``里面，开始时间戳存储在``startTime``里。一旦资源被加载，回调函数会被执行。在回调函数中，最终时间戳会被调取和用来计算加载JavaScript资源所花费的时间。花费时间会通``_trackTiming``过发送GA。

因此，通过调用：

	loadJs(‘//ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js’, callback);

会从Google Content Delivery 网络异步加载jQuery库， 一旦完成会执行(callback function)回调函数, 把加载时间发给GA。

<a href="#top">↑ 回到顶部</a>

###多用户使用 {#p4} Working With Multiple User   
---
假设你想使用上面的代码加载多个JavaScript资源。 ``startTime``变量是全站点的，每次记录一段时间内的开始，``startTime``变量会被覆盖，产生错误的花费时间。

因此，最好保持每个跟踪开始和结束时间要保持唯一实例。

还要注意的是``_trackTiming``的类别和变量参数是不能改的。果您使用``loadJs``来加载多个资源，在GA报告里是不能区分每个资源。

要解决这两个问题，可以把时间和``_trackTiming``参数存储在一个JavaScript object里面。

**创建一个JavaScript object来存储用户计时**

下面是一个简单的JavaScript object，可以用来存储​​用每个被追踪的资源的户时间数据：

	function TrackTiming(category, variable, opt_label) {
		this.category = category;
		this.variable = variable;
		this.label = opt_label ? opt_label : undefined;
		this.startTime;
		this.endTime;
		return this;
	}

	TrackTiming.prototype.startTime = function() {
		this.startTime = new Date().getTime();
		return this;
	}

	TrackTiming.prototype.endTime = function() {
		this.endTime = new Date().getTime();
		return this;
	}

	TrackTiming.prototype.send = function() {
		var timeSpent = this.endTime - this.startTime;
		window._gaq.push(['_trackTiming', this.category, this.variable, timeSpent, this.label]);
		return this;
	}

现在我们可以使用这个object使``loadJs``有多个请求。

**发送存储的用户时间**

现在，我们有一种方法来存储我们要跟踪的每个资源的时间数据，这里是如何更新loadJs使用的：


	var tt = new TrackTiming('jQuery', 'Load Library', 'Google CDN');

	loadJs(‘//ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js’, myCallback, tt);

	function loadJs(url, callback, tt) {
		var js = document.createElement('script');
		js.async = true;
		js.src = url;
		js.onload = callback;
		var s = document.getElementsByTagName('script')[0];

	tt.startTime();
	js.tt = tt;

	s.parentNode.insertBefore(js, s);
	}

	function myCallback(event) {
		var e = event || window.event;
		var target = e.target ? e.target : e.srcElement;

	target.tt.endTime().send();

	// Library has loaded. Now you can use it.
	}

上面的代码首先由创建一个新``TrackTiming``object，将分类函数和选择(category, variable, option)传递给构造(constructor)。 ``TrackTiming``object作为参数会被传到``loadJs``功能。

在```loadJs``里面，``startTime``method被召唤来获取和存储开始时间戳。

在前面的例子中，回调函数(callback function)可以轻松地使用``startTime``variable，因为它是全站点的。现在``startTime``是``TrackTiming``object的一部分，所以我们需要一种方法把这个object从loadJS传递到回调函数(callback function)。

为了解决这个问题的一个方法是，把``TrackTiming``object添加成script element的属性。由于回调函数执行是在``onload``方法的脚本上，回调对象作为参数传递一个事件。此事件对象可以被用于恢复原始脚本触发事件的对象，脚本对象可以在用来访问我们``TrackTiming``对象。

一旦我们能够接触我们原来的``TrackTiming``对象，该脚本可以结束的时候，和发送的数据。

在我们的[样本网站的现场演示这个例子](http://analytics-api-samples.googlecode.com/svn/trunk/src/tracking/javascript/v5/user-timing/user-timing.html)。

这种模式的添加的TrackTiming对象作为属性到跟踪的对象，往往用来跟踪其他的异步加载机制也好用，像使用``XMLHttpRequest``对象。

<a href="#top">↑ 回到顶部</a>


###记录 XMLHttpRequests {#p5} Tracking XMLHttpRequests   
另一种常用的步加载网页资源的方法是使用``XMLHttpRequest``object。加载这些资源所花费时可以同时使用``_trackTiming``的方法和``TimeTracker``object来记录。

下面是一个从服务器加载quotes的例子。

	var url = ‘//myhost.com/quotes.txt’;
	var tt = new TrackTime('xhr demo', 'load quotes');

	makeXhrRequest(url, myCallback, tt);

	function makeXhrRequest(url, callback, tt) {
		if (window.XMLHttpRequest) {
			var xhr = new window.XMLHttpRequest;
    		xhr.open('GET', url, true);
    		xhr.onreadystatechange = callback;

    		tt.startTime();
    		xhr.tt = tt;

    		xhr.send();
		}
	}

	function myCallback(event) {
		var e = event || window.event;
		var target = e.target ? e.target : e.srcElement;

		if (target.readyState == 4) {
			if (target.status == 200) {

      			target.tt.endTime().send();

      			// Do something with the resource.
    		}
		}
	}
	
这个例子看起来真很loadJs的例子很相似。[demo请看这里](http://analytics-api-samples.googlecode.com/svn/trunk/src/tracking/javascript/v5/user-timing/user-timing.html)。

<a href="#top">↑ 回到顶部</a>
	
###避免发送错误数据 {#p6} Avoid Sending Bad Data   
在上面的例子中，以得到所花费时间，代码把开始时间和结束时间向减。通常，如果开始时间比结束时间小，这样是没问题的。但是如果用户在开始时间开始计算后调整了机器时间，如果在浏览器中的变化的时间。如果用户改变他们的机器时间设置的开始时间后，坏的数据会被发送到GA那边。一个大问题是，一个被发送的错误值会扭曲你的平均和总指标。

因此，最好再发送数据到GA之前确定花费的时间是大于0和小于一段时间。我们可以通过修改``TimeTracker``的发送method，来执行检查：

	TrackTiming.prototype.send = function() {
		var timeSpent = this.endTime - this.startTime;

		var hourInMillis = 1000 * 60 * 60;

		if (0 < timeSpent && timeSpent < hourInMillis) {
			window._gaq.push(['_trackTiming', this.category, this.variable, 			timeSpent, this.label]);
		}

		return this;
	}


###重叠样品比率 {#p7} Overriding the Sample Rate   
---
GA全部收集的网站的速度指标会被``_trackTiming``method同平率的发送给GA。默认情况下，此设置为1％的访问者。

对于大量的流量的网站，默认的应该是没问题的。但对于较少的流量的网站，你可以通过设置自定采样率参数，来提高采样率。例如：

	_gaq.push(['_trackTiming', 'jQuery', 'Load Library', timeSpent, 'Google CDN', 50]);

会收取50%访问者的 _trackTiming数据。

此外，您可以设置[_setSiteSpeedSampleRate](https://developers.google.com/analytics/devguides/collection/gajs/methods/gaJSApiBasicConfiguration#_gat.GA_Tracker_._setSiteSpeedSampleRate/) method来设置所有网站的速度的采样率，包括的``_trackTiming``method。例如：

	_gaq.push([‘_setSiteSpeedSampleRate’, 50]);

也会收取50%访问者的 _trackTiming数据。

通常情况下，当您测试和验证一个GA实施，你一般会有很少的流量到你的网站。因此，测试时一般会把采样率增加到100％。

<a href="#top">↑ 回到顶部</a>

###记录其他时间事件 {#p8} Tracking Other Time Events    
---
上面的例子中都使用的``_trackTiming``method来跟踪需要多长时间加载资源, 这个method也可以用来追踪持续时间。例如，你可以跟踪：

* 访问者花费在观看视频的时间。
* 游戏中完成一个级别所花的时间。
* 访问者阅读一段的网站的时间。

在这些情况下，你可以重复使用同样的``TimeTracker``js object来简化收集和传送所花费的时间数据给GA。


  <a href="#top">↑ 回到顶部</a>
</p>

[1]: #p1
[2]: #p2
[3]: #p3
[4]: #p4
[5]: #p5
[6]: #p6
[7]: #p7
[8]: #p8
[9]: #p9
