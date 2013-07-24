#指令

对于指令, 你可以扩展HTML来以添加声明性语法来做任何你喜欢做的事情. 通过这样做, 你可以替换一些特定于你的应用程序的通用的\<div\>s和\<span\>s元素和属性的实际意义. 它们都带有Angular提供的基础功能, 但是你可以创建特定于应用程序的你自己想做的事情.

首先我们要复习以下指令API以及它在Angular启动和运行生命周期里是如何运作的. 从那里, 我们将使用这些只是来创建一个指令类型. 在本将完成时我们将学习到如何编写指令的单元测试和使它们运行得更快.

但是首先, 我们来看看一些使用指令的语法说明.

##指令和HTML验证

在本书中, 我们已经使用了Angular内置指令的`ng-directive-name`语法. 例如`ng-repeat`, `ng-view`和`ng-controller`. 这里, `ng`部分是Angular的命名空间, 并且dash之后的部分便是指令的名称.

虽然我们喜欢这个方便输入的语法, 但是在大部分的HTML验证机制中它不是有效的. 为了支持这些, Angular指令允许你以几种方式调用任意的指令. 以下在表61中列出的语法, 都是等价的并能够让你偏爱的[首选的]验证器正常工作

Table 6-1 HTML Validation Schemes

<table>
	<thead>
		<tr>
			<th>Validator</th>
			<th>Format</th>
			<th>Example</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>none</td>
			<td>namespace-name</td>
			<td>ng-repeat=<i>item in items</i></td>
		</tr>
		<tr>
			<td>XML</td>
			<td>namespace:name</td>
			<td>ng:repeat=<i>item in items</i></td>
		</tr>
		<tr>
			<td>HTML5</td>
			<td>data-namespace-name</td>
			<td>data-ng-repeat=<i>item in items</i></td>
		</tr>
		<tr>
			<td>xHTML</td>
			<td>x-namespace-name</td>
			<td>x-ng-repeat=<i>item in items</i></td>
		</tr>
	</tbody>
</table>

由于你可以使用任意的这些形式, [AngularJS文档](http://docs.angularjs.org/)中列出了一个驼峰式的指令, 而不是任何这些选项. 例如, 在`ngRepeat`标题下你可以找到`ng-repeat`. 稍后你会看到, 在你定义你自己的指令时你将会使用这种命名格式.

如果你不适用HTML验证器(大多数人都不使用), 你可以很好的使用在目前你所见过的例子中的命名空间-指令[namespace-directive]语法

##API预览

下面是一个创建任意指令伪代码模板

	var myModule = angular.module(...);

	myModule.directive('namespaceDirectiveName', function factory(injectables) {
		var directiveDefinitionObject = {
			restrict: string,
			priority: number,
			template: string,
			templateUrl: string,
			replace: bool,
			transclude: bool,
			scope: bool or object,
			controller: function controllerConstructor($scope, $element, $attrs, $transclude){...},
			require: string,
			link: function postLink(scope, iElement, iAttrs) {...},
			compile: function compile(tElement, tAttrs, transclude){
				return: {
					pre: function preLink(scope, iElement, iAttrs, controller){...},
					post: function postLink(scope, iElement, iAttrs, controller){...}
				}
			}
		};
		return directiveDefinitionObject;
	});

有些选项是互相排斥的, 它们大多数都是可选的, 并且它们都有有价值的详细说明:

当你使用每个选项时, 表6-2提供了一个概述.

Table 6-2 指令定义选项

<table>
	<thead>
		<tr>
			<th>Property</th>
			<th>Purpose</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>restrict</td>
			<td>声明指令可以作为一个元素, 属性, 类, 注释或者任意的组合如何用于模板中</td>
		</tr>
		<tr>
			<td>priority</td>
			<td>设置模板中相对于其他元素上指令的执行顺序</td>
		</tr>
		<tr>
			<td>template</td>
			<td>指令一个作为字符串的内联模板. 如果你指定一个模板URL就不要使用这个模板属性.</td>
		</tr>
		<tr>
			<td>templateUrl</td>
			<td>指定通过URL加载的模板. 如果你指定了字符串的内联模板就不需要使用这个.</td>
		</tr>
		<tr>
			<td>replace</td>
			<td>如果为true, 则替换当前元素. 如果为false或者未指定, 则将这个指令追加到当前元素上.</td>
		</tr>
		<tr>
			<td>transclude</td>
			<td>让你将一个指令的原始自节点移动到心模板位置内.</td>
		</tr>
		<tr>
			<td>scope</td>
			<td>为这个指令创建一个新的作用域而不是继承父作用域.</td>
		</tr>
		<tr>
			<td>controller</td>
			<td>为跨指令通信创建一个发布的API.</td>
		</tr>
		<tr>
			<td>require</td>
			<td>需要其他指令服务于这个指令来正确的发挥作用.</td>
		</tr>
		<tr>
			<td>link</td>
			<td>以编程的方式修改生成的DOM元素实例, 添加事件监听器, 设置数据绑定.</td>
		</tr>
		<tr>
			<td>compile</td>
			<td>以编程的方式修改一个指令的DOM模板的副本特性, 如同使用`ng-repeat`时. 你的编译函数也可以返回链接函数来修改生成元素的实例.</td>
		</tr>
	</tbody>
</table>

下面让我们深入细节来看看.

