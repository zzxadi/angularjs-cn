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

2. 那第二个对象是什么呢? {id: @id}吗? 是的, 正如他们所说的, 一行代码可能需要一千行解释, 那么让我们举一个简单的例子:

