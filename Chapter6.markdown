#指令

对于指令, 你可以扩展HTML来以添加声明性语法来做任何你喜欢做的事情. 通过这样做, 你可以替换一些特定于你的应用程序的通用的\<div\>s和\<span\>s元素和属性的实际意义. 它们都带有Angular提供的基础功能, 但是你可以创建特定于应用程序的你自己想做的事情.

首先我们要复习以下指令API以及它在Angular启动和运行生命周期里是如何运作的. 从那里, 我们将使用这些只是来创建一个指令类型. 在本将完成时我们将学习到如何编写指令的单元测试和使它们运行得更快.

但是首先, 我们来看看一些使用指令的语法说明.

##指令和HTML验证

在本书中, 我们已经使用了Angular内置指令的`ng-directive-name`语法. 例如`ng-repeat`, `ng-view`和`ng-controller`. 这里, `ng`部分是Angular的命名空间, 并且dash之后的部分便是指令的名称.

虽然我们喜欢这个方便输入的语法, 但是在大部分的HTML验证机制中它不是有效的. 为了支持这些, Angular指令允许你以几种方式调用任意的指令. 以下在表6-1中列出的语法, 都是等价的并能够让你偏爱的[首选的]验证器正常工作

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

###为你的指令命名

你可以用模块的指令函数为你的指令创建一个名称, 如下所示:

	myModule.directive('directiveName', function factory(injectables){...});

虽然你可以使用任何你喜欢的名字命名你的指令, 该符号会选择一个前缀命名空间标识你的指令, 同时避免与可能包含在你的项目中的外部指令冲突.

你当然不希望它们使用一个`ng-`前缀, 因为这可能与Angular自带的指令相冲突. 如果你从事于SuperDuper MegaCorp, 你可以选择一个super-, superduper-, 或者甚至是superduper-megacorp-, 虽然你可能选择第一个选项, 只是为了方便输入.

正如前面所描述的, Angular使用一个标准化的指令命名机制, 并且试图有效的在模板中使用驼峰式的指令命名方式来确保在5个不同的友好的验证器中正常工作. 例如, 如果你已经选择了`super-`作为你的前缀, 并且你在编写一个日期选择(datepicker)组件, 你可能将它命名为`superDatePicker`. 在模板中, 你可以像这样来使用它: `super-date-picker`, `super:date-picker`, `data-super-date-picker`或者其他多样的形式.

##指令定义对象

正如前面提到的, 在指令定义中大多数的选项都是可选的. 实际上, 这里并没有硬性的要求必须选择哪些选项, 并且你可以构造出许多有利于指令的子集参数. 让我们来逐步讨论这些选项是做什么的.

####restrict

`restrict`属性允许你指定你的指令声明风格--也就是说, 它是否能够用于作为元素名称, 属性, 类[className], 或者注释. 你可以根据表6-3来指定一个或多个声明风格, 只需要使用一个字符串来表示其中的每一中风格:

Table 6-3 指令声明用法选项

<table>
	<thead>
		<tr>
			<th>Character</th>
			<th>Declaration style</th>
			<th>Example</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>E</td>
			<td>element</td>
			<td>&lt;my-menu title=<i>Products</i>&gt;&lt;/my-menu&gt;</td>
		</tr>
		<tr>
			<td>A</td>
			<td>attribute</td>
			<td>&lt;div my-menu=<i>Products</i>&gt;&lt;/div&gt;</td>
		</tr>
		<tr>
			<td>C</td>
			<td>class</td>
			<td>&lt;div class=my-menu:<i>Products</i>&gt;&lt;/div&gt;</td>
		</tr>
		<tr>
			<td>M</td>
			<td>comment</td>
			<td>&lt;!--directive:my-menu Products--&gt;</td>
		</tr>
	</tbody>
</table>

如果你希望你的指令用作一个元素或者一个属性, 那么你应该传递`EA`作为`restrict`字符串.

如果你忽略了`restrict`属性, 则默认为`A`, 并且你的指令只能用作一个属性(属性指令).

如果你计划支持IE8, 那么基于attribute-和class-的指令就是你最好的选择, 因为它需要额外的努力来使新元素正常工作. 可以查看Angular文档来详细了解这一点.

####Priorities




