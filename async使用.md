#async使用

## 安装

	npm install async
	
项目地址是： [https://github.com/caolan/async](https://github.com/caolan/async)	

## API分类

async的API大体分为三类：

* 流程控制

* 集合处理

* 工具类

### 流程控制类

#### series

##### 语法

	series(tasks, [callback])

##### 描述

依次执行tasks中的任务，每个任务只有在上一个任务执行完毕之后才可以执行。如果任意一个任务向他的回调函数中传递了一个错误信息，那么后续的任务将不再执行，这个错误信息将会作为第一个参数传递给[callback]，前面任务的执行结果将以数组的形式作为第二个参数传递给[callback]。

##### 参数

* tasks: 一个包含异步任务的数组，每个异步任务都有一个形如callback(error, value)的回调函数，第一个参数是该任务的错误信息，无错误时传递null,第二个参数是可选的，代表该任务产生的结果。

* callback(error, results): 所有tasks都执行完毕后的回调，它是可选参数，第一个参数接收tasks中传递的错误信息，第二个参数以数组的形式接收tasks传递的操作结果。

特别指明的是，假如tasks是以字典的形式组织各异步操作，那么最后的results也是个字典。

	
##### 例子

	var async = require('async');
	
	async.series([
		function (callback) {
			setTimeout(function () {
				console.log('first task finished');
				callback(null, 1000);
			}, 1000);
		},
		function (callback) {
			setTimeout(function () {
				console.log('second task finished');
				callback(null, 2000);
			}, 2000);
		}], 
		function (error, results) {
			console.log('finial error: ', error);
			console.log('finial results: ', results);
	});
		
	//执行结果如下
	first task finished
	second task finished
	finial error:  undefined
	finial results:  [ 1000, 2000 ]
	
更多详细的例子可以看 [https://github.com/zc001/async_example/blob/master/series.js](https://github.com/zc001/async_example/blob/master/series.js)	

#### parallel

##### 语法

	parallel(tasks, [callback]);
	
##### 描述

并行执行tasks中的任务，每一个任务不需要等待上一个任务完成即可执行。如果tasks中任何一个任务向它的回调中传递了一个错误信息，那么[callback]会立即被调用。

##### 参数

* tasks: 一个包含异步任务的数组，每个异步任务都有一个形如callback(error, value)的回调函数，第一个参数是该任务的错误信息，无错误时传递null,第二个参数是可选的，代表该任务产生的结果。

* callback(error, results): 所有任务执行完毕后的回调，它是可选参数，error接收tasks中传递的第一个错误信息，results以数组的形式接收tasks传递的结果，假如tasks中任何一个任务传递了错误信息，那么此时没有完成的任务的操作结果将不会被results接收，但需要按照任务在tasks中定义的顺序占个位置。 

特别指明的是，假如tasks是以字典的形式组织各异步操作，那么最后的results也是个字典。

##### 例子

	var async = require('async');
	
	async.parallel([
		function (callback) {
			console.log('start the first task');
			setTimeout(function () {
				console.log('the first task finished');
				callback(null, 3000);
			}, 3000);
		},
		function (callback) {
			console.log('start the second task');
			setTimeout(function () {
				console.log('the second task finished');
				callback(null, 2000);
			}, 2000);
		}], 
		function (error, results) {
			console.log('finial error: ', error);
			console.log('finial results: ', results);
	});
		
	//执行结果如下
	start the first task
	start the second task
	the second task finished
	the first task finished
	finial error:  undefined
	finial results:  [ 3000, 2000 ]
	
更多详细的例子可以看 [https://github.com/zc001/async_example/blob/master/parallel.js](https://github.com/zc001/async_example/blob/master/parallel.js)	

#### parallelLimit

##### 语法

	parallelLimit(tasks, limit, [callback])

#### waterfall
	
##### 语法	
	
	waterfall(tasks, [callback])
	
##### 描述

依次执行tasks中的任务，每个任务只有在上个任务完成之后才可执行，上一个任务可以把操作结果传递给下个任务。假如任何一个任务向它的回调中传递了一个错误信息，那么后续的任务将不会执行，[callback]回调的第一个参数接收这个错误。

##### 参数

* tasks: 一个包含异步任务的数组，每个任务都有一个形如callback(error, result_1, result_2, …, result_n)的回调，第一个参数是该任务执行时产生的错误，之后是需要传递给下一任务的参数。

* callback(error, [results]): 所有任务执行完毕后的回调，它是一个可选参数。 error接收tasks中任务传递的错误信息，[results]是一个可选参数，接收tasks中最后一个任务传递的操作结果。

##### 例子

	var async = require('async');
	
	async.waterfall([
		function (callback) {
			console.log('start the first task');
			setTimeout(function () {
				console.log('the first task finished');
				callback(null, 1000, 1001);
			}, 1000);
		},
		function (argu_1, argu_2, callback) {
			console.log('start the second task');
			console.log('argu_1: %d, argu_2: %d', argu_1, argu_2);
			setTimeout(function () {
				console.log('the second task finished');
				callback(null, 2000);
			}, 2000);
		}], 
		function (error, result) {
			console.log('enter the finial callback');
			console.log('finial error: ', error);
			console.log('finial result: ', result);
	});
	
	//运行结果如下
	start the first task
	the first task finished
	start the second task
	argu_1: 1000, argu_2: 1001
	the second task finished
	enter the finial callback
	finial error:  null
	finial results:  2000
	
更多详细的例子可以看 [https://github.com/zc001/async_example/blob/master/waterfall.js](https://github.com/zc001/async_example/blob/master/waterfall.js)	

### 集合控制类

#### forEach

##### 语法
	
	each(arr, iterator, callback)
	
##### 描述

对集合arr中的每一个元素，都执行iterator操作，且是并行，所有iterator执行完毕后，最后的callback就会调用。每一个iterator操作完成后，都会调用一个回调，该回调接收一个参数，表示iterator操作过程中产生的错误信息。任何一个iterator向它的回调中传递了一个错误信息，最后的callback就会立即执行，而不需要等待所以的
iterator都执行完毕。该方法只关心iterator执行的过程，忽略产生的结果。

##### 参数

* arr: 数据集合

* iterator(item, callback): 对arr中每个元素执行的操作，item是arr中的一个元素。callback(error)是iterator执行完毕后的回调，error为该回调接收的错误信息，可为null。

* callback(err): 当所有iterator执行完毕，或者有iterator给自己的回调传递错误信息时被调用，err就是iterator传递的错误信息。

##### 例子
	
	var async = require('async')
	,	array = [{name: 'zc', time_delay: 1000}, {name: 'async', time_delay: 2000}];
	
	async.forEach(array, function (item, callback) {
		console.log('start iterator. item: ', item);
		setTimeout(function () {
			console.log('show name: ', item.name);
			callback(null);
		}, item.time_delay);
	}, function (error) {
		console.log('error: ', error);
	});
	
	//打印信息如下
	start iterator. item:  { name: 'zc', time_delay: 2000 }
	start iterator. item:  { name: 'async', time_delay: 1000 }
	show name:  async
	show name:  zc
	error:  undefined
	


