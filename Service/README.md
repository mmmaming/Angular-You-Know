#### 所有的服务都是单例
服务/阶段	 | provider  | factory  | service  | value | constant
---|---|---|---|---|---
config阶段	 | Yes | No | No | No | Yes
run阶段	     | Yes | Yes | Yes | Yes | Yes
>1 value(name, value);

value可以是数字、字符串、数组、对象或者函数。

value服务中不能再注入其他服务，并且不能在config阶段注入，但是可以被decorator进行装饰。

```
angular.module('app', []).value('value1', {
    author: 'mmmaming'
});

// angular.module('app', []).value('value1', 'mmmaming');

// 使用decorator进行装饰
angular.module('app').config(function($provide) {
    $provide.decorator('value1', function($delegate) {
        $delegate.author = 'handsome mmmaming';
        return $delegate;
    });
});
// 使用
angular.module('app').constroller('ctrl', ['value1', function(value1) {
    console.log(value1.author);// 'handsome mmmaming'                                     
}]);



```

注： 不能在config阶段注入，否则会报错.angular程序会报错。Unknown provider: value1
```
angular.module('app').config(function(value1) {
    console.log(value1); // Unknown provider: value1
});
```

>2 constant(name, value);

value可以是数字、字符串、数组、对象或者函数。

constant服务中不能再注入其他服务,但是可以在config阶段注入。但是不能被decorator装饰。

```
angular.module('app', []).constant('constant1', {
    author: 'mmmaming'
});

angular.module('app').config(function(constant1) {
    console.log(constant1); // {author: mmmaming}
});
```

注： 不能被decorator装饰，否则会报错。Unknown provider: constant1Provider

```
// Unknown provider: constant1Provider
angular.module('app').config(function($provide) {
    $provide.decorator('constant1', function($delegate) {
        $delegate.author = 'handsome mmmaming';
        return $delegate;
    });
});
```

constant和value都可以被修改，但是constant不能被decorator装饰，value不能在config阶段注入。

>3 service(name, constructor);

constructor是一个构造函数，最终调用服务的是它的实例对象。

```
// 注册
angular.module('app').service('service1', function() {
    this.x = 1;
    this.add = function() {
        return 3;
    }
});

// 使用
angular.module('app').constroller('ctrl', ['service1', function(service1) {
    console.log(service1.add());// 3 
    console.log(service1.x);// 1  
                                    
}]);
```

>4 factory(name, $getFn);

$getFn是一个函数，函数内部返回一个对象

```
// 注册
angular.module('app').factory('factory1', function() {
    return {
        x: 1,
        add: function() {
            return 3;
        }
        
    }
    
});

// 使用
angular.module('app').constroller('ctrl', ['factory1', function(factory1) {
    console.log(factory1.add());// 3 
    console.log(factory1.x);// 1  
                                    
}]);

```

注： service和factory的区别在于,它第二个参数传入的是一个构造函数,最后被注入的服务是这个构造函数实例化以后的结果.所以基本上使用service创建的服务的,也都可以使用factory来创建。

>5 provider(name, provider);

provider是一个构造函数。

```
// 注册
angular.module('app').provider('my', function() {
    this.name ='';
    return {
        $get : function() {
            var that = this;
            var age = 23;
            function getAge() {
                return age;
            }
            return {
                getName: function() {
                    return that.name;
                },
                getAge: getAge
            }
        }

    }
});

// 在config里包装,注意这里要在my后面加Provider
angular.module('app').config(function(myProvider) {
    myProvider.name = 'mmmaming';
});

// 使用
angular.module('app').constroller('ctrl', ['my', function(my) {
    console.log(my.getName());// mmmaming
    console.log(my.getAge());// 23                               
}]);

```

>6 decorator(name, decorator);  可以拦截一个服务的创建，从而修改或覆盖服务的行为。

decorator有两种写法

- $provide.decorator

```
// 使用上面的myProvider
angular.module('app').config(function($provide) {
    $provide.decorator('my', ['$delegate', function($delegate) {
         // 给my服务增加getTom方法
         $delegate.getTom = function() {
            return 'tom';
         };
         // 返回修改后的服务
         return $delegate;
    }])
});

// 使用
angular.module('app').constroller('ctrl', ['my', function(my) {
    console.log(my.getName());// mmmaming
    console.log(my.getAge());// 23                               
    console.log(my.getName());// Tom  装饰后的my，拥有了getName方法。                              
}]);
```

- module.decorator

类似于$provide.decorator,允许在模块中配置装饰器，可以装饰service，directive和filter。但需要注意书写顺序，服务必须在装饰器之前注册，否则会提示服务没有注册。三种写法具体可参见[https://code.angularjs.org/1.5.8/docs/guide/decorators](https://code.angularjs.org/1.5.8/docs/guide/decorators)