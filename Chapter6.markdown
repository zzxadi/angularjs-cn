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