angular-migrate
===============

An utility library with a bunch of ECMAScript 7 decorators that allow
you to write angular 1.x applications with ES6 classes to align more
with what next generation frameworks will look like to easy the
migration path.  
For an example of use check my [bookstore example application](https://github.com/olanod/bookstore-example-ng1). The library is in `src/lib/angular-migrate`. I'll just tweak it a bit more before making it a stand-alone library in this repository.

##Why
Because you are concerned about developing this new cool app in
angular1.x and then have to re-write everything from scratch when
angular2 or that other cool framework comes out.  
So if you've read about preparing your app for the future, the first
recommendation you probably found is to start writing your code with
the new ES6 syntax, that means using classes. This isn't hard to do with
controllers and services, with controllers you have the `controllerAs`
syntax and with services you have `angular.service` which receives a
constructor function(a class) as parameter. But what about the other
angular elements you use a lot? they cannot be written as classes and
thats where this library comes in, allowing you to write other
components of the framework as classes with the help of **decorators**.

Decorators
----------
Currently there are two main decorators that transform an ES6 class into the
corresponding angular component.

###ngModule
Instead of declaring a module in the old boring way which is too angular1-ish you can now declare it as a class which is much cooler(and clearer?).

```
import {ngModule} from 'angular-migrate/decorators';
import MyOtherModule from './subsection/my-other-module';

// just decorate the class and you got your module
// pass an array of dependencies with the classic string format
// or a constructor to extract its name from the class name
@ngModule(['ngRoute', MyOtherModule])
class MyApp {
}
```
One thing you use a lot when declaring a module is using the `module.run` and `module.config` so I made it easy to access them from inside the class.
```
@ngModule(['ngRoute'])
class MyApp {
  // the construcor body is the body of module.run
  // this way it feels more aligned with next-gen
  constructor($rootScope) {
    $rootScope.greeting = 'Hello World!';
  }

  // same as module.config
  static config($routeProvider) {
    // configure your services here
  }
}
```
What about services, directives, controllers? easy just declare them inside a
static array in the class
```
@ngModule
class MyApp {
  // ES7 static properties
  static controllers = [SomeController, OtherController];
  static service = [SuperService];
  static directives = [MyAwesomeButton];
}
```
####Extra
There are some helper decorators that make it even more easy
```
@ngModule
@withController(SomeController, OtherController)
@withDirective(SuperService)
@withService(MyAwesomeButton)
class MyApp {}
```

###ngDirective
Directives have always been problematic for developers to understand and write,
no wonder the DDO(directive definition object) is dead. With this decorator you
can write them in a much clearer way. By default it uses some conventions based
on the most common use cases and best practices for migration.

```
@ngDirective({
  template: "<button class='awesome'>Pure Awesomeness!</button>"
})
class MyAwesomeButton {
  constructor(MyService) {
    this.myService = myService;
  }
}
```
this is the same as writing
```
var ddo = {
  restrict: 'E', // element
  scope: {}, // isolated scope
  controller: function(MyService) {
      this.myService = MyService;
    }
  controllerAs: 'vm', // convention used a lot lately
  bindToController: true, // expose scope properties in the controller
  template: '<button class="awesome">Pure Awesomeness!</button>'
}
```
`link` and `compile` should be used less now that you are using the cool `controllerAs` and `bindToController` but you have them in the class if you want.
```
@ngDirective
class MyAwesomeButton {
  static link() {}
  static compile() {}
}
```
#### Extra
What about exposed properties of the scope? the argument to ngDirective is a
DDO that will extend the defaults so you can pass a scope property, but much
cooler is to use `withProperty` helper decorator.

```
@ngDirective
@withProperty('fuu') // same as {scope: {fuu: '='}}
class MyAwesomeButton {}
```
Want declare more properties and be more explicit about the tipe of binding?
```
@ngDirective
// pass the binding as an array of 2 or 3
// you can pass several properties separated by comma
@withProperty(['fuu','@'], ['bar','&','baz']) // same as {scope:{fuu:'@',bar:'&baz'}}
// or just put another decorator
@withProperty('otherProperty')
class MyAwesomeButton {}
```

### Extra Extra

###inject
If you are worried about minification there is another helper decorator for that.
```
@ngModule
@inject('$rootScope')
class MyApp {
  constructor(scope) {}

  @inject('$routerProvider')
  static config(router) {}
}
```
`inject` should work on any method or class constructor that needs angular DI.

###bootstrap
This just manually bootstraps your app when passing a module.

```
// app.js
import {ngModule} from 'angular-migrate/decorators';
import {bootstrap} from 'angular-migrate/util';
@ngMogule
class MyApp {}

export function start() {
  bootstrap(MyApp);
}
```
And let's say you use `SystemJs` and `jspm` your index could look like:
```
<html>
<head>...</head>
<body>
  <script src="packages/system.js"></script>
  <script src="app/config.js"></script>
  <script>System.import('app/app').then(function(app){app.start()})</script>
</body>
</html>
```
