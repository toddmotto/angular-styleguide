# Angular styleguide

*Opinionated Angular styleguide for teams based off of [@toddmotto](https://github.com/toddmotto/angular-styleguide)'s*

#### What is this?

A standardised approach for developing Angular applications at triplelift. This styleguide touches on concepts, syntax and conventions.

#### Community
[John Papa](https://twitter.com/John_Papa) and [Todd Motto](https://twitter.com/toddmotto) have discussed in-depth styling patterns for Angular and as such have both released separate styleguides. [Check his John Papa's](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md) and [Todd Motto's](https://github.com/toddmotto/angular-styleguide) to compare thoughts.

> See the [original article](http://toddmotto.com/opinionated-angular-js-styleguide-for-teams) that sparked this off

## Table of Contents

  1. [Modules](#modules)
  1. [Controllers](#controllers)
  1. [Services and Factory](#services-and-factory)
  1. [Directives](#directives)
  1. [Filters](#filters)
  1. [Routing resolves](#routing-resolves)
  1. [Publish and subscribe events](#publish-and-subscribe-events)
  1. [Performance](#performance)
  1. [Angular wrapper references](#angular-wrapper-references)
  1. [Comment standards](#comment-standards)
  1. [Minification and annotation](#minification-and-annotation)

## Modules

  - **Definitions**: Declare modules without a variable using the setter and getter syntax

    ```javascript
    // avoid
    var app = angular.module('app', []);
    app.controller();
    app.factory();

    // recommended
    angular
      .module('app', [])
      .controller()
      .factory();
    ```

  - Note: Using `angular.module('app', []);` sets a module, whereas `angular.module('app');` gets the module. Only set once and get for all other instances.


**[Back to top](#table-of-contents)**

## Controllers

  - **controllerAs syntax**: Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the classic "controller with $scope"" syntax.

    ```html
    <!-- avoid -->
    <div ng-controller="MainCtrl">
      {{ someObject }}
    </div>

    <!-- recommended -->
    <div ng-controller="MainCtrl as vm">
      {{ vm.someObject }}
    </div>
    ```

	  ```javascript
	  /* avoid */
	  function CustomerController($scope) {
	      $scope.name = {};
	      $scope.sendMessage = function() { };
	  }
	  ```
	  
	  ```javascript
	  /* recommended - but see next section */
	  function CustomerController() {
	      this.name = {};
	      this.sendMessage = function() { };
	  }
  	  ```
  
    *Why?*: Controllers are constructed, "newed" up, providing a single new instance. The `controllerAs` syntax more closely resembles "JavaScript constructor" than "classic `$scope`" syntax.
    
    *Why?*: Even with `controllerAs`, the controller object is still bound to `$scope`, so you can still bind to the View...this fact ultimately leads some to even see the `controllerAs` syntax as high level syntactic sugar.

    *Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual and easier to read.
    
     *Why?*: Helps avoid using `$parent` in the View to reference a particular property of a parent controller from within a nested controller.
   
     *Why?*: Curbs the temptation to using `$scope` methods inside a controller when they are better placed inside a factory or avoided altogether.
     
     *Why?*: You can, nevertheless, use `$scope` methods, such as `$emit`, `$broadcast`, `$on` or `$watch`, by injecting `$scope` if absolutely necessary.
     
     *why?*: **Most importantly**, `controllerAs` prevents "scope soup". There may be nothing worse in large scale angular development than running into `ng-click="doOnClick(data)"` in the View, looking into the controller "corresponding" to such View and being unable to locate either the `doOnClick` or `data` definitions. At this point, of course, you must begin mapping out the $scope heirarchy until it leads to the nearest declaration of each variable... no fun indeed. Taking advantage of the basic rules of JavaScript prototypal inheritance, placing wonderful `vm` in front of each variable, as in `ng-click="vm.doOnClick(vm.data)"` **ensures** the controller **associated** therewith carries the corresponding definitions or `undefined` will result. That is because when JavaScipt attempts to locate `vm`, it will find the associated controller bound to associated `$scope` - success is guaranteed in the from of the correct controller object. Next, JavaScript will attempt to locate `doOnClick` and `data` **on such controller** and...vuala... either corresponding definitions will be found **bound to such controller** or `undefined` will result.
  	  

  - **controllerAs 'vm'**: Capture the `this` context of the Controller using `vm`, standing for `ViewModel`

  ```javascript
  /* avoid */
  function CustomerController() {
      this.name = {};
      this.sendMessage = function() { };
  }
  ```

  ```javascript
  /* recommended */
  function CustomerController() {
      var vm = this;
      vm.name = {};
      vm.sendMessage = function() { };
  }
  ```

  - *Why?* : The this keyword is contextual and when used within a function inside a controller may change its context. Capturing the context of this avoids encountering this problem.  
  
  - **`$watch`ing in a controller**: When creating watches in a controller using `controller as`, you can watch the `vm.*` member using the following syntax. (Create watches with caution as they add more load to the digest cycle.)  

  ```html
  <input ng-model="vm.title"/>
  ```

  ```javascript
  function SomeController($scope, $log) {
      var vm = this;
      vm.title = 'Some Title';

      $scope.$watch('vm.title', function(current, original) {
          $log.info('vm.title was %s', original);
          $log.info('vm.title is now %s', current);
      });
  }
  ``` 
  
  - **Bindable members up top**: Place bindable members at the top of the controller, *alphabetized*, and not spread through the controller code.
  
  ```javascript
  /* avoid */
  function SessionsController() {
      var vm = this;

      vm.gotoSession = function() {
        /* ... */
      };
      vm.refresh = function() {
        /* ... */
      };
      vm.search = function() {
        /* ... */
      };
      vm.sessions = [];
      vm.title = 'Sessions';
  }
  ```

  ```javascript
  /* recommended */
  function SessionsController(sessionDataService) {
      var vm = this;

      vm.gotoSession = gotoSession;
      vm.refresh = refresh;
      vm.search = search;
      vm.sessions = [];
      vm.title = 'Sessions';
      vm.write = sessionDataService.write; // 1 liner is OK

      ////////////

      function gotoSession() {
        /* */
      }

      function refresh() {
        /* */
      }

      function search() {
        /* */
      }
  }
  ```
  
  
  *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View.
  
  *Why?*: Defining the functions below the bindable members moves the implementation details (& complexity) down.
  
  *Why?*: Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions), even if one function references another.

  - **Presentational logic only (MVVM)**: Presentational logic only inside a controller, avoid Business logic (delegate to Services)

    ```javascript
    // avoid
    function MainCtrl () {
      
      var vm = this;

      $http
        .get('/users')
        .success(function (response) {
          vm.users = response;
        });

      vm.removeUser = function (user, index) {
        $http
          .delete('/user/' + user.id)
          .then(function (response) {
            vm.users.splice(index, 1);
          });
      };

    }

    // recommended
    function MainCtrl (UserService) {

      var vm = this;

      UserService
        .getUsers()
        .then(function (response) {
          vm.users = response;
        });

      vm.removeUser = function (user, index) {
        UserService
          .removeUser(user)
          .then(function (response) {
            vm.users.splice(index, 1);
          });
      };

    }
    ```
    
	*Note*: The `$http` service, its methods and URL paths arent referenced directly.
	
	*Note*: Logic in controllers is used only to bind the appropriate data to the controller object,
	
    *Why?*: Controllers should fetch Model data from Services, avoiding any Business logic. Controllers should act as a ViewModel and control the data flowing between the Model and the View presentational layer. Business logic in Controllers makes testing Services impossible.
    
  - **Keep controllers focused**: Define a controller for a view, and try not to reuse the controller for other views. Instead, move reusable logic to factories and keep the controller simple and focused on its view.
  
    *Why?*: Reusing controllers with several views is brittle and good end-to-end (e2e) test coverage is required to ensure stability across large applications.
    
  - **Assign controllers in route definitions instead of template** 
  
	 ```javascript
	 /* avoid */
	
	 // route-config.js
	 angular
	     .module('app')
	     .config(config);
	
	 function config($routeProvider) {
	     $routeProvider
	         .when('/avengers', {
	           templateUrl: 'avengers.html'
	         });
	 }
	 ```
	
	 ```html
	 <!-- avengers.html -->
	 <div ng-controller="AvengersController as vm">
	 </div>
	 ```
	
	 ```javascript
	 /* recommended */
	
	 // route-config.js
	 angular
	     .module('app')
	     .config(config);
	
	 function config($routeProvider) {
	     $routeProvider
	         .when('/avengers', {
	             templateUrl: 'avengers.html',
	             controller: 'Avengers',
	             controllerAs: 'vm'
	         });
	 }
	 ```

	 ```html
	 <!-- avengers.html -->
	 <div>
	 </div>
	 ```
    *Why?*: Pairing a controller with a template through `$routeProvider` configurations allows for template reuse. In other words, when `ng-controller` is used inline, that's it... a pairing is made between template and controller, which prevents template reuse with another controller.
    
    *Why?*: Having all controller-template pairings upfront creates a sort of index for your application, which makes it easier to see where everything is, what goes with what and how the data flows throughout your application (see **[Routing with promises](#routing-with-promises)** below for more). 
    
    *Note*: Although now even easier to do with the introduction of route definitions, *controller* reuese is *still ill-advised* for the reasons above.
    
    
  - **ES6**: Avoid `var vm = this;` when using ES6

    ```javascript
    // avoid
    function MainCtrl () {
      let vm = this;
      let doSomething = arg => {
        console.log(vm);
      };
      
      // exports
      vm.doSomething = doSomething;
    }

    // recommended
    function MainCtrl () {
      
      let doSomething = arg => {
        console.log(this);
      };
      
      // exports
      this.doSomething = doSomething;
      
    }
    ```

    *Why?* : Use ES6 arrow functions when necessary to access the `this` value lexically

**[Back to top](#table-of-contents)**

## Services and Factory

  - All Angular Services are singletons, using `.service()` or `.factory()` differs the way Objects are created. Since these are so similar to factories, **always use a factory for consistency**.

  **Service**: acts as a `constructor` function and is instantiated with the `new` keyword. Use `this` for public methods and variables

	```javascript
	// service
	angular
	    .module('app')
	    .service('logger', logger);
	
	function logger() {
	  this.logError = function(msg) {
	    /* */
	  };
	}
	```

  **Factory**: is invoked like any ol' function and, accordingly, should return something!
  
	```javascript  
	// factory
	angular
	    .module('app')
	    .factory('logger', logger);
	
	function logger() {
	    return {
	        logError: function(msg) {
	          /* */
	        }
	   };
	}
	```



**[Back to top](#table-of-contents)**

## Directives

  - **Declaration restrictions**: Only use `custom element` and `custom attribute` methods for declaring your Directives (`{ restrict: 'EA' }`) depending on the Directive's role

    ```html
    <!-- avoid -->

    <!-- directive: my-directive -->
    <div class="my-directive"></div>

    <!-- recommended -->

    <my-directive></my-directive>
    <div my-directive></div>
    ```

  - Comment and class name declarations are confusing and should be avoided. Comments do not play nicely with older versions of IE. Using an attribute is the safest method for browser coverage.

  - **Templating**: Use `Array.join('')` for clean templating

    ```javascript
    // avoid
    function someDirective () {
      return {
        template: '<div class="some-directive">' +
          '<h1>My directive</h1>' +
        '</div>'
      };
    }

    // recommended
    function someDirective () {
      return {
        template: [
          '<div class="some-directive">',
            '<h1>My directive</h1>',
          '</div>'
        ].join('')
      };
    }
    ```

    *Why?* : Improves readability as code can be indented properly, it also avoids the `+` operator which is less clean and can lead to errors if used incorrectly to split lines

  - **DOM manipulation**: Takes place only inside Directives, never a controller/service

    ```javascript
    // avoid
    function UploadCtrl () {
      $('.dragzone').on('dragend', function () {
        // handle drop functionality
      });
    }
    angular
      .module('app')
      .controller('UploadCtrl', UploadCtrl);

    // recommended
    function dragUpload () {
      return {
        restrict: 'EA',
        link: function (scope, element, attrs) {
          element.on('dragend', function () {
            // handle drop functionality
          });
        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

  - **Naming conventions**: Never `ng-*` prefix custom directives, they might conflict future native directives

    ```javascript
    // avoid
    // <div ng-upload></div>
    function ngUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('ngUpload', ngUpload);

    // recommended
    // <div drag-upload></div>
    function dragUpload () {
      return {};
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

  - Directives and Filters are the _only_ providers that have the first letter as lowercase; this is due to strict naming conventions in Directives. Angular hyphenates `camelCase`, so `dragUpload` will become `<div drag-upload></div>` when used on an element.

  - **controllerAs**: Use the `controllerAs` syntax inside Directives as well

    ```javascript
    // avoid
    function dragUpload () {
      return {
        controller: function ($scope) {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);

    // recommended
    function dragUpload () {
      return {
        controllerAs: 'vm',
        controller: function () {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```

**[Back to top](#table-of-contents)**

## Filters

  - **Global filters**: Create global filters using `angular.filter()` only. Never use local filters inside Controllers/Services

    ```javascript
    // avoid
    function SomeCtrl () {
      this.startsWithLetterA = function (items) {
        return items.filter(function (item) {
          return /^a/i.test(item.name);
        });
      };
    }
    angular
      .module('app')
      .controller('SomeCtrl', SomeCtrl);

    // recommended
    function startsWithLetterA () {
      return function (items) {
        return items.filter(function (item) {
          return /^a/i.test(item.name);
        });
      };
    }
    angular
      .module('app')
      .filter('startsWithLetterA', startsWithLetterA);
    ```

  - This enhances testing and reusability

**[Back to top](#table-of-contents)**

## Routing resolves

  - **Promises**: Resolve Controller dependencies in the `$routeProvider` (or `$stateProvider` for `ui-router`), not the Controller itself

    ```javascript
    // avoid
    function MainCtrl (SomeService) {
      var _this = this;
      // unresolved
      _this.something;
      // resolved asynchronously
      SomeService.doSomething().then(function (response) {
        _this.something = response;
      });
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);

    // recommended
    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        resolve: {
          // resolve here
        }
      });
    }
    angular
      .module('app')
      .config(config);
    ```

  - **Controller.resolve property**: Never bind logic to the router itself. Reference a `resolve` property for each Controller to couple the logic

    ```javascript
    // avoid
    function MainCtrl (SomeService) {
      this.something = SomeService.something;
    }

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controllerAs: 'vm',
        controller: 'MainCtrl'
        resolve: {
          doSomething: function () {
            return SomeService.doSomething();
          }
        }
      });
    }

    // recommended
    function MainCtrl (SomeService) {
      this.something = SomeService.something;
    }

    MainCtrl.resolve = {
      doSomething: function (SomeService) {
        return SomeService.doSomething();
      }
    };

    function config ($routeProvider) {
      $routeProvider
      .when('/', {
        templateUrl: 'views/main.html',
        controllerAs: 'vm',
        controller: 'MainCtrl'
        resolve: MainCtrl.resolve
      });
    }
    ```

  - This keeps resolve dependencies inside the same file as the Controller and the router free from logic

**[Back to top](#table-of-contents)**

## Publish and subscribe events

  - **$scope**: Use the `$emit` and `$broadcast` methods to trigger events to direct relationship scopes only

    ```javascript
    // up the $scope
    $scope.$emit('customEvent', data);

    // down the $scope
    $scope.$broadcast('customEvent', data);
    ```

  - **$rootScope**: Use only `$emit` as an application-wide event bus and remember to unbind listeners

    ```javascript
    // all $rootScope.$on listeners
    $rootScope.$emit('customEvent', data);
    ```

  - Hint: Because the `$rootScope` is never destroyed, `$rootScope.$on` listeners aren't either, unlike `$scope.$on` listeners and will always persist, so they need destroying when the relevant `$scope` fires the `$destroy` event

    ```javascript
    // call the closure
    var unbind = $rootScope.$on('customEvent'[, callback]);
    $scope.$on('$destroy', unbind);
    ```

  - For multiple `$rootScope` listeners, use an Object literal and loop each one on the `$destroy` event to unbind all automatically

    ```javascript
    var unbind = [
      $rootScope.$on('customEvent1'[, callback]),
      $rootScope.$on('customEvent2'[, callback]),
      $rootScope.$on('customEvent3'[, callback])
    ];
    $scope.$on('$destroy', function () {
      unbind.forEach(function (fn) {
        fn();
      });
    });
    ```

**[Back to top](#table-of-contents)**

## Performance

  - **One-time binding syntax**: In newer versions of Angular (v1.3.0-beta.10+), use the one-time binding syntax `{{ ::value }}` where it makes sense

    ```html
    // avoid
    <h1>{{ vm.title }}</h1>

    // recommended
    <h1>{{ ::vm.title }}</h1>
    ```
    
    *Why?* : Binding once removes the watcher from the scope's `$$watchers` array after the `undefined` variable becomes resolved, thus improving performance in each dirty-check
    
  - **Consider $scope.$digest**: Use `$scope.$digest` over `$scope.$apply` where it makes sense. Only child scopes will update

    ```javascript
    $scope.$digest();
    ```
    
    *Why?* : `$scope.$apply` will call `$rootScope.$digest`, which causes the entire application `$$watchers` to dirty-check again. Using `$scope.$digest` will dirty check current and child scopes from the initiated `$scope`

**[Back to top](#table-of-contents)**

## Angular wrapper references

  - **$document and $window**: Use `$document` and `$window` at all times to aid testing and Angular references

    ```javascript
    // avoid
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs) {
          document.addEventListener('click', function () {

          });
        }
      };
    }

    // recommended
    function dragUpload ($document) {
      return {
        link: function ($scope, $element, $attrs) {
          $document.addEventListener('click', function () {

          });
        }
      };
    }
    ```

  - **$timeout and $interval**: Use `$timeout` and `$interval` over their native counterparts to keep Angular's two-way data binding up to date

    ```javascript
    // avoid
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs) {
          setTimeout(function () {
            //
          }, 1000);
        }
      };
    }

    // recommended
    function dragUpload ($timeout) {
      return {
        link: function ($scope, $element, $attrs) {
          $timeout(function () {
            //
          }, 1000);
        }
      };
    }
    ```

**[Back to top](#table-of-contents)**

## Comment standards

  - **jsDoc**: Use jsDoc syntax to document function names, description, params and returns

    ```javascript
    /**
     * @name SomeService
     * @desc Main application Controller
     */
    function SomeService (SomeService) {

      /**
       * @name doSomething
       * @desc Does something awesome
       * @param {Number} x - First number to do something with
       * @param {Number} y - Second number to do something with
       * @returns {Number}
       */
      this.doSomething = function (x, y) {
        return x * y;
      };

    }
    angular
      .module('app')
      .service('SomeService', SomeService);
    ```

**[Back to top](#table-of-contents)**

## Minification and annotation

  - **ng-annotate**: Use [ng-annotate](//github.com/olov/ng-annotate) for Gulp as `ng-min` is deprecated, and comment functions that need automated dependency injection using `/** @ngInject */`

    ```javascript
    /**
     * @ngInject
     */
    function MainCtrl (SomeService) {
      this.doSomething = SomeService.doSomething;
    }
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);
    ```

  - Which produces the following output with the `$inject` annotation

    ```javascript
    /**
     * @ngInject
     */
    function MainCtrl (SomeService) {
      this.doSomething = SomeService.doSomething;
    }
    MainCtrl.$inject = ['SomeService'];
    angular
      .module('app')
      .controller('MainCtrl', MainCtrl);
    ```

**[Back to top](#table-of-contents)**

## Angular docs
For anything else, including API reference, check the [Angular documentation](//docs.angularjs.org/api).

## Contributing

Open an issue first to discuss potential changes/additions.

## License

#### (The MIT License)

Copyright (c) 2015-2016 Todd Motto

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
