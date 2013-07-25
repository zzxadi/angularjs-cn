#与服务器通信

到现在为止，我们已经看到下面这些内容的主要部分，这些内容是你的Angular应用如何规划设计、不同的AngularJS部件如何装配在一起并正常工作以及AngularJS中的模板代码运行机制的一小部分内容。把这些结合在一起，你就可以搭建一些苗条性感的Web应用，但他们的运作主要还是限制在客户端.在前面第二章,我们看了一点用`$http`服务做服务器端通信的内容,但是在这一章,我们将会深入探讨一下如何在现实世界的应用中使用它(`$http`)

在这一章，我们将讨论一下AngularJS如何帮你同服务器端通信，这其中包括在最底抽象等级的层面或者用它提供的优雅的封装器。而且我们将会深入探讨AngularJS如何用内建缓存机制来帮你加速你的应用.如果你想用`SocketIO`开发一个实时的Angular应用,那么第八章有一个例子，演示了如何把·SocketIO·封装成一个指令然后如何使用这个指令，在这一章，我们就不涉及这方面内容了.

##通过$http进行通行

从Ajax应用(使用XMLHttpRequests)发动一个请求到服务器的传统方式包括：得到一个XMLHttpRequest对象的引用、发起请求、读取响应、检验错误代码然后最后处理服务器响应。它就是下面这样：
    
    var xmlhttp = new XMLHttpRequest();
    xmlhttp.onreadystatechange = function() {
        if (xmlhttp.readystate == 4 && xmlhttp.status == 200) {
            var response = xmlhttp.responseText;
        } else if (xmlhttp.status == 400) { // or really anything in the 4 series
            // Handle error gracefully
        }
    };
    // Setup connection
    xmlhttp.open(“GET”, “http://myserver/api”, true);
    // Make the request
    xmlhttp.send();
    
对于这样一个简单、常用且经常重复的任务，上面这个代码量比较大.如果你想重复性地做这件事,你最终可能会做一个封装或者使用现成的库.

AngularJS XHR(XMLHttpRequest) API遵循Promise接口.因为XHRs是异步方法调用，服务器响应将会在未来一个不定的时间返回(当然希望是越快越好).Promise接口保证了这样的响应将会如何处理,它允许Promise接口的消费者以一种可预计的方式使用这些响应.

假设我们想从我们的服务器取回用户的信息.如果接口在/api/user地址可用,并且接受id作为url参数，那么我们的XHR请求就可以像下面这样使用Angular的核心$http服务:
    
    $http.get('api/user', {params: {id: '5'}
    }).success(function(data, status, headers, config) {
    // Do something successful.
    }).error(function(data, status, headers, config) {
    // Handle the error
    });
    
如果你来自jQuery世界,你可能会注意到：AngularJS和jQuery处理异步需求的方式很相似.

我们上面例子中使用的$htttp.get方法仅仅是AngularJS核心服务$http提供的众多简便方法之一.类似的，如果你想使用AngularJS向相同URL带一些POST请求数据发起一个POST请求，你可以像下面这样做：

    var postData = {text: 'long blob of text'};
    // The next line gets appended to the URL as params
    // so it would become a post request to /api/user?id=5 
    var config = {params: {id: '5'}};
    $http.post('api/user', postData, config
    ).success(function(data, status, headers, config) {
    // Do something successful
    }).error(function(data, status, headers, config) {
    // Handle the error
    });

AngularJS为大多数常用请求类型都提供了类似的简便方法，他们包括：

+ GET
+ HEAD
+ POST
+ DELETE
+ PUT
+ JSONP

###进一步配置你的请求

有时，工具箱提供的标准请求配置还不够,它可能是因为你想做下面这些事情:

+ 你可能想为请求添加权限验证的头信息
+ 改变请求数据的缓存方式
+ 在请求被发送或者响应返回时，对数据以一些方式做一定的转换处理

在上面这样的情况之下,你可以进一步配置请求，通过可选的传递进请求的配置对象.在之前的例子中,我们使用配置对象来标明可选的URL参数，即便我们哪儿演示的GET和POST方法是简便方法。内部的原生方法可能看上面像相面这样：

    $http(config)

下面演示的是一个调用这个方法的伪代码模板:

    $http({
        method: string,
        url: string,
        params: object,
        data: string or object,
        headers: object,
        transformRequest: function transform(data, headersGetter) or an array of functions,
        transformResponse: function transform(data, headersGetter) or an array of functions,
        cache: boolean or Cache object,
        timeout: number,
        withCredentials: boolean
    });

GET、POST和其它的简便方法已经设置了请求的method类型,所以不需要再设置这个，config配置对象是传给·$http.get·、·$http.post·方法的最后一个参数,所以当你使用任何简便方法的时候，你任何能用这个config配置对象.

你也可以通过传入含有下面这些键的属性集config对象来改变已有的request对象

+ method : 一个表示http请求类型的字符串，比如GET,或者POST
+ url : 一个URL字符串代表要请求资源的绝对或相对URL
+ params : 一个对象(准确的说是键值映射)包含字符串到字符串内容，它代表了将会转换为URL参数的键值对，比如下面这样：
    [{key1: 'value1', key2: 'value2'}]
它将会被转换为:
    ?key1=value&key2=value2
这串字符将会加在URL后面，如果在value的位置你用一个对象取代字符串或数字，那这个对象将会转换为JSON字符串.
+ data ：一个字符串或一个对象，它将会被作为请求消息数据被发送.
+ timeout : 这是请求被认定为过期之前所要等待的毫秒数.

还有部分另外的选项可以被配置,在下面的章节中，我们将会深度探索这些选项.

###设定HTTP头信息(Headers)

AngularJS有一个默认的头信息,这个头信息将会对所有的发送请求使用,它包含以下信息:
    1.Accept: application/json, text/plain, /
    2.X-Requested-With:XMLHttpRequest

如果你想设置任何特定的头信息,这儿有两种方法来做这件事：

第一种方法,如果你相对所有的发送请求都使用这些特定头信息,那你需要把特定有信息设置为Angular默认头信息的一部分.可以在`$httpProvider.defaults.headers`配置对象里面设置这个,这个步骤通常会在你的app设置config部分来做.所以如果你想移除"Requested-With"头信息且对所有的GET请求启用"DO NOT TRACK"设置,你可以简单地通过以下代码来做:

    angular.module('MyApp',[]).
        config(function($httpProvider) {
            // Remove the default AngularJS X-Request-With header
            delete $httpProvider.default.headers.common['X-Requested-With'];
            // Set DO NOT TRACK for all Get requests
            $httpProvider.default.headers.get['DNT'] = '1';
    });
    
如果你只想对某个特定的请求设置头信息,而不是设置默认头信息.那么你可以通过给$http服务传递包含指定头信息的config对象来做.相同的定制头信息可以作为第二个参数传递给GET请求,第一个参数是URL字符串：
    
    $http.get('api/user', {
    // Set the Authorization header. In an actual app, you would get the auth
    // token from a service
    headers: {'Authorization': 'Basic Qzsda231231'},
    params: {id: 5}
    }).success(function() { // Handle success });
    
如何在应用中处理权限验证头信息的成熟示例将会在第八章的Cheetsheets示例部分给出.

###缓存响应数据

AngularJS为HTTP GET请求提供了一个开箱即用的简单缓存系统.缺省情况下,它对所有的请求都是禁用的,但是如果你想对你的请求启用缓存系统，你可以使用以下代码:

    $http.get('http://server/myapi', {
        cache: true
    }).success(function() { // Handle success });

这段代码启用了缓存系统，然后AngularJS将会缓存来自Server的响应数据.但对相同的URL的请求第二次发出时,AngularJS将会从缓存里面取出前一次的响应数据作为响应返回.这个缓存系统也很智能,即使你同时对相同URL发出多个请求,只有一个请求会发向Server,这个请求的响应数据将会反馈给所有(同时发起的)请求。


















