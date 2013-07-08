#分析一个AngularJS应用程序

在第2章中, 我们已经讨论了一些AngularJS常用的功能, 然后在第3章讨论了该如何结构化开发应用程序. 现在, 我们不再继续深单个的技术点, 第4章将着眼于一个小的, 实际的应用程序进行讲解. 我们将从一个实际有效的应用程序中感受一下我们之前已经讨论过的(示例)所有的部分.

我们将每次介绍一部分, 然后讨论其中有趣和关联的部分, 而不是讨论完整应用程序的前端和核心, 最后在本章的后面我们会慢慢简历这个完整的应用程序.

##应用程序

Guthub是一个简单的食谱管理应用, 我们设计它用于存储我们超级没味的食谱, 同时展示AngularJS应用程序的各个不同的部分. 这个应用程序包含以下内容:

+ 一个两栏的布局
+ 在左侧有一个导航栏
+ 允许你创建新的食谱
+ 允许你浏览现有的食谱列表

主视图在左侧, 其变化依赖于URL, 或者食谱列表, 或者单个食谱的详情, 或者可添加新食谱的编辑功能, 或者编辑现有的食谱. 我们可以在图4-1中看到这个应用的一个截图:

![Guthub](figure/4-1.png)

Figure 4-1. Guthub: A simple recipe management application

这个完整的应用程序可以在我们的Github中的`chapter4/guthub`中得到.

##模型, 控制器和模板之间的关系

在我们深入应用程序之前, 让我们来花一两段文字来讨论以下如何将标题中的者三部分在应用程序中组织在一起工作, 同时来思考一下其中的每一部分.

`model`(模型)是真理. 只需要重复这句话几次. 整个应用程序显示什么, 如何显示在视图中, 保存什么, 等等一切都会受模型的影响. 因此你要额外花一些时间来思考你的模型, 对象的属性将是什么, 以及你打算如何从服务器获取并保存它. 视图将通过数据绑定的方式自动更新, 所以我们的焦点应该集中在模型上.

`controller`保存业务逻辑: 如何获取模型, 执行什么样的操作, 视图需要从模型中获取什么样的信息, 以及如何将模型转换为你所想要的. 验证职责, 使用调用服务器, 引导你的视图使用正确的数据, 大多数情况下所有的这些事情都属于控制器来处理.

最后, `template`代表你的模型将如何显示, 以及用户将如何与你的应用程序交互. 它主要约束以下几点:

+ 显示模型
+ 定义用户可以与你的应用程序交互的方式(点击, 文本输入等等)
+ 应用程序的样式, 并确定何时以及如何显示一些元素(显示或隐藏, hover等等)
+ 过滤和格式化数据(包括输入和输出)

要意识到在Angular中的模型-视图-控制器涉及模式中模板并不是必要的部分. 相关, 视图是模板获取执行被编译后的版本. 它是一个模板和模型的组合.

任何类型的业务逻辑和行为都不应该进入模板中; 这些信息应该被限制在控制器中. 保持模板的简单可以适当的分离关注点, 并且可以确保你只使用单元测试的情况下就能够测试大多数的代码. 而模板必须使用场景测试的方式来测试.

但是, 你可能会问, 在哪里操作DOM呢? DOM操作并不会真正进入到控制器和模板中. 它会存在于Angular的指令中(有时候也可以通过服务来处理, 这样可以避免重复的DOM操作代码). 我们会在我们的Github的示例文件中涵盖一个这样的例子.

废话少说, 让我们来深入探讨一下它们.

##模型

对于应用程序我们要保持模型非常简单. 这一有一个菜谱. 在整个完整的应用程序中, 它们是一个唯一的模型. 它是构建一切的基础.

每个菜谱都有下面的属性:

+ 一个用于保存到服务器的ID
+ 一个名称
+ 一个简短的描述
+ 一个烹饪说明
+ 是否是一个特色的菜谱
+ 一个成份数组, 每个成分的数量, 单位和名称

就是这样. 非常简单. 应用程序的中一切都基于这个简单的模型. 下面是一个让你食用的示例菜谱(如图4-1一样):

	{
		'id': '1',
		'title': 'Cookies',
		'description': 'Delicious. crisp on the outside, chewy' +
			' on the outside, oozing with chocolatey goodness' +
			' cookies. The best kind',
		'ingredients': [
			{
				'amount': '1',
				'amountUnits': 'packet',
				'ingredientName': 'Chips Ahoy'
			}
		],
		'instructions': '1. Go buy a packet of Chips Ahoy\n'+
			'2. Heat it up in an oven\n' +
			'3. Enjoy warm cookies\n' +
			'4. Learn how to bake cookies from somewhere else'
	}

下面我们将会看到如何基于这个简单的模型构建更复杂的UI特性.

##控制器, 指令和服务

现在我们终于可以得到这个让我们牙齿都咬到肉里面去的美食应用程序了. 首先, 我们来看看代码中的指令和服务, 以及讨论以下它们都是做什么的, 然后我们我们会看看这个应用程序需要的多个控制器.

###服务

	//this file is app/scripts/services/services.js

	var services = angular.module('guthub.services', ['ngResource']);

	services.factory('Recipe', ['$resource', function(){
		return $resource('/recipes/:id', {id: '@id'});
	}]);

	services.factory('MultiRecipeLoader', ['Recipe', '$q', function(Recipe, q){
		return function(){
			var delay = $.defer();
			Recipe.query(function(recipes){
				delay.resolve(recipes);
			}, function(){
				delay.reject('Unable to fetch recipes');
			});
			return delay.promise;
		};
	}]);

	services.factory('RecipeLoader', ['Recipe', '$route', '$q', function(Recipe, $route, $q){
		return function(){
			var delay = $q.defer();
			Recipe.get({id: $route.current.params.recipeId}, function(recipe){
				delay.resolve(recipe);
			}, function(){
				delay.reject('Unable to fetch recipe' + $route.current.params.recipeId);
			});
			return delay.promise;
		};
	}]);

首先让我们来看看我们的服务. 在33页的"使用模块组织依赖"小节中已经涉及到了服务相关的知识. 这里, 我们将会更深一点挖掘服务相关的信息.

在这个文件中, 我们实例化了三个AngularJS服务.

有一个菜谱服务, 它返回我们所调用的Angular Resource. 这些是RESETful资源, 它指向一个RESTful服务器. Angular Resource封装了低层的`$http`服务, 因此你可以在你的代码中只处理对象.

注意单独的那行代码 - `return $resource` - (当然, 依赖于`guthub.services`模型), 现在我们可以将`recipe`作为参数传递给任意的控制器中, 它将会注入到控制器中. 此外, 每个菜谱对象都内置的有以下几个方法:

+ Recipe.get()
+ Recipe.save()
+ Recipe.query()
+ Recipe.remove()
+ Recipe.delete()

> 如果你使用了`Recipe.delete`方法, 并且希望你的应用程序工作在IE中, 你应该像这样调用它: `Recipe[delete]()`. 这是因为在IE中`delete`是一个关键字.

对于上面的方法, 所有的查询众多都在一个单独的菜谱中进行; 默认情况下`query()`返回一个菜谱数组.

`return $resource`这行代码用于声明资源 - 也给了我们一些好东西:

1. 注意: URL中的id是指定的RESTFul资源. 它基本上是说, 当你进行任何查询时(`Recipe.get()`), 如果你给它传递一个id字段, 那么这个字段的值将被添加早URL的尾部.

也就是说, 调用`Recipe.get{id: 15})将会请求/recipe/15.

2. 那第二个对象是什么呢? {id: @id}吗? 是的, 正如他们所说的, 一行代码可能需要一千行解释, 那么让我们举一个简单的例子.

比方说我们有一个recipe对象, 其中存储了必要的信息, 并且包含一个id.

然后, 我们只需要像下面这样做就可以保存它:

	//Assuming existingRecipeObj has all the necessary fields,
	//including id(say 13)
	var recipe = new Recipe(existingRecipeObj);
	recipe.$save();

这将会触发一个POST请求到`/recipe/13`.

`@id`用于告诉它, 这里的id字段取自它的对象中同时用于作为id参数. 这是一个附加的便利操作, 可以节省几行代码.

在`apps/scripts/services/services.js`中有两个其他的服务. 它们两个都是加载器(Loaders); 一个用于加载单独的食谱(RecipeLoader), 另一个用于加载所有的食谱(MultiRecipeLoader). 这在我们连接到我们的路由时使用. 在核心上, 它们两个表现得非常相似. 这两个服务如下:

1. 创建一个`$q`延迟(deferred)对象(它们是AngularJS的promises, 用于链接异步函数).
2. 创建一个服务器调用.
3. 在服务器返回值时resolve延迟对象.
4. 通过使用AngularJS的路由机制返回promise.

> **AngularJS中的Promises**
>
> 一个promise就是一个在未来某个时刻处理返回对象或者填充它的接口(基本上都是异步行为). 从核心上讲, 一个promise就是一个带有`then()`函数(方法)的对象.
>
>让我们使用一个例子来展示它的优势, 假设我们需要获取一个用户的当前配置:

	var currentProfile = null;
	var username = 'something';

	fetchServerConfig(function(){
		fetchUserProfiles(serverConfig.USER_PROFILES, username, 
			function(profiles){
				currentProfile = profiles.currentProfile;	
		});	
	});

> 对于这种做法这里有一些问题:
>
> 1. 对于最后产生的代码, 缩进是一个噩梦, 特别是如果你要链接多个调用时.
> 
> 2. 在回调和函数之间错误报告的功能有丢失的倾向, 除非你在每一步中处理它们.
>
> 3. 对于你想使用`currentProfile`做什么, 你必须在内层回调中封装其逻辑, 无论是直接的方式还是使用一个单独分离的函数.
>
> Promises解决了这些问题. 在我们进入它是如何解决这些问题之前, 先让我们来看看一个使用promise对同一问题的实现.

	var currentProfile = fetchServerConfig().then(function(serverConfig){
		return fetchUserProfiles(serverConfig.USER_PROFILES, username);
	}).then(function{
		return profiles.currentProfile;
	}, function(error){
		// Handle errors in either fetchServerConfig or
		// fetchUserProfile here
	});

> 注意其优势:
>
> 1. 你可以链接函数调用, 因此你不会产生缩进带来的噩梦.
>
> 2. 你可以放心前一个函数调用会在下一个函数调用之前完成.
>
> 3. 每一个`then()`调用都要两个参数(这两个参数都是函数). 第一个参数是成功的操作的回调函数, 第二个参数是错误处理的函数.
> 4. 在链接中发生错误的情况下, 错误信息会通过错误处理器传播到应用程序的其他部分. 因此, 任何回调函数的错误都可以在尾部被处理.
>
> 你会问, 那什么是`resolve`和`reject`呢? 是的, `deferred`在AngularJS中是一种创建promises的方式. 调用`resolve`来满足promise(调用成功时的处理函数), 同时调用`reject`来处理promise在调用错误处理器时的事情.

当我们链接到路由时, 我们会再次回到这里.

###指令

我们现在可以转移到即将用在我们应用程序的指令上来. 在这个应用程序中将有两个指令:

`butterbar`

这个指令将在路由发生改变并且页面仍然还在加载信息时处理显示和隐藏任务. 它将连接路由变化机制, 基于页面的状态来自动控制显示或者隐藏是否使用哪个标签.

`focus`

这个`focus`指令用于确保指定的文本域(或者元素)拥有焦点.

让我们来看一下代码:

	// This file is app/scripts/directives/directives.js

	var directive = angular.module('guthub.directives', []);

	directives.directive('butterbar', ['$rootScope', function($rootScope){
		return {
			link: function(scope, element attrs){
				element.addClass('hide');

				$rootScope.$on('$routeChangeStart', function(){
					element.removeClass('hide');
				});

				$routeScope.$on('$routeChangeSuccess', function(){
					element.addClass('hide');
				});
			}
		};
	}]);

	directives.dirctive('focus',function(){
		return {
			link: function(scope, element, attrs){
				element[0].focus();
			}
		};
	});

上面所述的指令返回一个对象带有一个单一的属性, link. 我们将在第六章深入讨论你可以如何创建你自己的指令, 但是现在, 你应该直到下面的所有事情: