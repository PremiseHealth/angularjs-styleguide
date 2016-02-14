# AngularJS Best Practices & Style Guide

Note: The updated version of this guide is still very much a Work in Progress.

This styleguide was forked from Todd Motto's version in August of 2014. A lot of things have changed since then, including the recent release of Angular 1.5. This styleguide is being updated to include best practices as of February 2016, including best practices for preparing an Angular 1.x app for an eventual upgrade to Angular 2.

This will be an opinionated guide for how we (Premise Health) develop our Angular apps. Some of the ideas will be ours, and some/many will come from various members of the Angular community. We'll give a shout out below to thank those whose inspired our guide.

Special thanks to:
* [John Papa's Styleguide] (https://github.com/johnpapa/angular-styleguide)
* [Todd Moto's Styleguide] (https://github.com/toddmotto/angular-styleguide)

## Table of Contents

  1. [General](#general)
  1. [Files](#files)
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
  1. [Dependencies](#dependencies)
  2. [Start Up Logic](#start-up-logic)

## General
  - When possible, use angular.element(), etc. instead of jQuery lookups and DOM manipulation.
  - Don't wrap element inside of $(). All AngularJS elements are already jqobjects.
  - Do not use $ prefix for the names of variables, properties and methods. This prefix is reserved for AngularJS usage.
  - When you need to set the src of an image dynamically use ng-src instead of src with {{}}.
  - When you need to set the href of an anchor tag dynamically use ng-href instead of href with {{}}
  - Avoid using $rootScope. It's ok to use $rootScope for emitting an event, but storing data on the $rootScope should be avoided. Use a service for your data instead.

## Files

Note: Some of these opinions about file structure, naming, etc are due to the fact that I use and recommend [Browserify](http://browserify.org/) as part the build process.

  - **One module per file**: Each file should have only one module definition. Exceptions are your app definition file (usually app.js), and any modules that need a config module.

  - **Each file should get its own namespace**: The namespace should follow its directory structure. The module namespace is /<project name>/path/to/file. Note that you leave the src root out of the filepath (the src root is typically /app). Ex: If your project is called LocAdmin, your file is a controller for the LocationsListing directive and is named LocationsListingCtrl it will likely have the following path on your filesystem: /LocAdmin/app/components/locationsListing. 
   ```javascript
    .module('locAdmin.components.locationsListing.locationsListingCtrl', [
    ]); 
   ```
   

  - **Each filename should match the controller/service/etc name**: A file with a .controller(‘mainCtrl’) definition should be named mainCtrl.js
 
  - **Function name and file name should match**: Given the the function definition below, you would name your file locationsListingCtrl.js. Note that filenames should start with a lowercase letter.
 
    ```javascript
    function LocationsListingCtrl($scope) {
    }
    ```
  - **Each file should be ES6 module compatible**: This means using export and import. 
  
   ```javascript
    // recommended
    function LocationsListingCtrl() {  
    }  
   	  
    export default angular
      .module('locAdmin.components.locationsListing.locationsListingCtrl', [                           
        require('./locationsListingService').name  
      ])  
      .controller('LocationsListingCtrl', LocationsListingCtrl);
   ```
  - **Directory structure** TODO: Update to replace less with sass
    
    ```
    /MyProject
    --/src
    ----index.html (the index.html for the SPA)
    ----app.js (the app definition for the Angular app)
    ----/assets (images)
    ------logo.png
    ----/less (less files for things other than pages/directives)
    ------main.less (the main less file for the app. Should import LESS files from directives & pages
    ------variables.less
    ----/app (angular app files)
    ------/components (directives go here)
    --------/myDirective (directory for myDirective directive
    ----------myDirective.js (directive file)
    ----------myDirective.tpl.html (directive template/partial)
    ----------myDirectiveCtrl.js (the directive's controller)
    ----------myDirectiveService.js (if directive requires a service, used ONLY by this directive)
    ----------myDirective.less (LESS file for this directive, if needed)
    ------/pages (top level pages/views go here. Subdirectories follow the same logic as the directives directory)
    --------/main (the main page/view)
    ----------mainCtrl.js (the controller for the main view)
    ----------main.tpl.html (the template/partial for the main view)
    ----------main.less (LESS file for the main view)
    ------/services (shared services, used by multiple controllers, go here)
    --------mySharedService.js
    ------/utils (other helper files used throughout the app
    --------stringUtils.js
    --------viewUtils.js
    ```

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

  - **Methods**: Pass functions into module methods rather than assign as a callback

    ```javascript
    // avoid
    angular
      .module('app', [])
      .controller('MainCtrl', function MainCtrl () {

      })
      .service('SomeService', function SomeService () {

      });

    // recommended
    function MainCtrl () {

    }
    angular
      .module('app', [])
      .controller('MainCtrl', MainCtrl);
    ```

  - This aids with readability and reduces the volume of code "wrapped" inside the Angular framework


**[Back to top](#table-of-contents)**

## Controllers

  - **controllerAs syntax**: Controllers are classes, so use the `controllerAs` syntax at all times

    ```html
    <!-- avoid -->
    <div ng-controller="MainCtrl">
      {{ someObject }}
    </div>

    <!-- recommended -->
    <div ng-controller="MainCtrl as main">
      {{ main.someObject }}
    </div>
    ```

  - In the DOM we get a variable per controller, which aids nested controller methods, avoiding any `$parent` calls

  - The `controllerAs` syntax uses `this` inside controllers, which gets bound to `$scope`

    ```javascript
    // avoid
    function MainCtrl ($scope) {
      $scope.someObject = {};
      $scope.doSomething = function () {

      };
    }

    // recommended use this or self
    function MainCtrl () {
      this.someObject = {};
      this.doSomething = function () {

      };
    }
    
    function MainCtrlTwo() {
      var self = this;
      self.someObject = {};
      self.doSomething = function() {
    };
    ```

  - Only use `$scope` in `controllerAs` when necessary; for example, publishing and subscribing events using `$emit`, `$broadcast`, `$on` or `$watch`. Try to limit the use of these, however, and treat `$scope` as a special use case

  
  - **controllerAs 'self'**: Capture the `this` context of the Controller using `self' (this is further explanation of MainCtrlTwo() in the example above)

    ```javascript
    // avoid
    function MainCtrl () {
      this.doSomething = function () {

      };
    }

    // recommended
    function MainCtrl (SomeService) {
      var self = this;
      self.doSomething = SomeService.doSomething;
    }
    ```

    *Why?* : Function context changes the `this` value, use it to avoid `.bind()` calls and scoping issues

  - **Presentational logic only (MVVM)**: Presentational logic only inside a controller, avoid Business logic (delegate to Services)

    ```javascript
    // avoid
    function MainCtrl () {
      
      var self = this;

      $http
        .get('/users')
        .success(function (response) {
          self.users = response;
        });

      vm.removeUser = function (user, index) {
        $http
          .delete('/user/' + user.id)
          .then(function (response) {
            self.users.splice(index, 1);
          });
      };

    }

    // recommended
    function MainCtrl (UserService) {

      var self = this;

      UserService
        .getUsers()
        .then(function (response) {
          self.users = response;
        });

      self.removeUser = function (user, index) {
        UserService
          .removeUser(user)
          .then(function (response) {
            self.users.splice(index, 1);
          });
      };

    }
    ```

    *Why?* : Controllers should fetch Model data from Services, avoiding any Business logic. Controllers should act as a ViewModel and control the data flowing between the Model and the View presentational layer. Business logic in Controllers makes testing Services impossible.
    
  **Bindable Members Up Top**
  - Place bindable members at the top of the controller, alphabetized, and not spread through the controller code.

    *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View.

    *Why?*: Setting anonymous functions in-line can be easy, but when those functions are more than 1 line of code they can reduce the readability. Defining the functions below the bindable members (the functions will be hoisted) moves the implementation details down, keeps the bindable members up top, and makes it easier to read.

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
  function SessionsController() {
      var vm = this;

      vm.gotoSession = gotoSession;
      vm.refresh = refresh;
      vm.search = search;
      vm.sessions = [];
      vm.title = 'Sessions';

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
  **Function Declarations to Hide Implementation Details**
  - Use function declarations to hide implementation details. Keep your bindable members up top. When you need to bind a function in a controller, point it to a function declaration that appears later in the file. This is tied directly to the section Bindable Members Up Top. For more details see [this post](http://www.johnpapa.net/angular-function-declarations-function-expressions-and-readable-code/).

    *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View. (Same as above.)

    *Why?*: Placing the implementation details of a function later in the file moves that complexity out of view so you can see the important stuff up top.

    *Why?*: Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions).

    *Why?*: You never have to worry with function declarations that moving `var a` before `var b` will break your code because `a` depends on `b`.

    *Why?*: Order is critical with function expressions

  ```javascript
  /**
   * avoid
   * Using function expressions.
   */
  function AvengersController(avengersService, logger) {
      var vm = this;
      vm.avengers = [];
      vm.title = 'Avengers';

      var activate = function() {
          return getAvengers().then(function() {
              logger.info('Activated Avengers View');
          });
      }

      var getAvengers = function() {
          return avengersService.getAvengers().then(function(data) {
              vm.avengers = data;
              return vm.avengers;
          });
      }

      vm.getAvengers = getAvengers;

      activate();
  }
  ```

  Notice that the important stuff is scattered in the preceding example. In the example below, notice that the important stuff is up top. For example, the members bound to the controller such as `vm.avengers` and `vm.title`. The implementation details are down below. This is just easier to read.

  ```javascript
  /*
   * recommend
   * Using function declarations
   * and bindable members up top.
   */
  function AvengersController(avengersService, logger) {
      var vm = this;
      vm.avengers = [];
      vm.getAvengers = getAvengers;
      vm.title = 'Avengers';

      activate();

      function activate() {
          return getAvengers().then(function() {
              logger.info('Activated Avengers View');
          });
      }

      function getAvengers() {
          return avengersService.getAvengers().then(function(data) {
              vm.avengers = data;
              return vm.avengers;
          });
      }
  }
  ```
  
  **Assigning Controllers**
  - When a controller must be paired with a view and either component may be re-used by other controllers or views, define controllers along with their routes.

    Note: If a View is loaded via another means besides a route, then use the `ng-controller="Avengers as vm"` syntax.

    *Why?*: Pairing the controller in the route allows different routes to invoke different pairs of controllers and views. When controllers are assigned in the view using [`ng-controller`](https://docs.angularjs.org/api/ng/directive/ngController), that view is always associated with the same controller.

 ```javascript
  /* avoid - when using with a route and dynamic pairing is desired */

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
              controllerAs: 'ctrl'
          });
  }
  ```

  ```html
  <!-- avengers.html -->
  <div>
  </div>
  ```
  **Controller Activation Promises**
  - Resolve start-up logic for a controller in an `activate` function.

    *Why?*: Placing start-up logic in a consistent place in the controller makes it easier to locate, more consistent to test, and helps avoid spreading out the activation logic across the controller.

    *Why?*: The controller `activate` makes it convenient to re-use the logic for a refresh for the controller/View, keeps the logic together, gets the user to the View faster, makes animations easy on the `ng-view` or `ui-view`, and feels snappier to the user.

    Note: If you need to conditionally cancel the route before you start using the controller, use a [route resolve](#style-y081) instead.

  ```javascript
  /* avoid */
  function AvengersController(dataservice) {
      var vm = this;
      vm.avengers = [];
      vm.title = 'Avengers';

      dataservice.getAvengers().then(function(data) {
          vm.avengers = data;
          return vm.avengers;
      });
  }
  ```

  ```javascript
  /* recommended */
  function AvengersController(dataservice) {
      var vm = this;
      vm.avengers = [];
      vm.title = 'Avengers';

      activate();

      ////////////

      function activate() {
          return dataservice.getAvengers().then(function(data) {
              vm.avengers = data;
              return vm.avengers;
          });
      }
  }
  ```
**[Back to top](#table-of-contents)**

## Services and Factory

  - All Angular Services are singletons, using `.service()` or `.factory()` differs the way Objects are created.

  **Services**: act as a `constructor` function and are instantiated with the `new` keyword. Use `this` for public methods and variables (or var self=this, and use self as noted in the Controller As example above)

    ```javascript
    function SomeService () {
      this.someMethod = function () {

      };
    }
    angular
      .module('app')
      .service('SomeService', SomeService);
    ```

  **Factory**: Business logic or provider modules, return an Object or closure

  - Always return a host Object instead of the revealing Module pattern due to the way Object references are bound and updated

    ```javascript
    function AnotherService () {
      var AnotherService = {};
      AnotherService.someValue = '';
      AnotherService.someMethod = function () {

      };
      return AnotherService;
    }
    angular
      .module('app')
      .factory('AnotherService', AnotherService);
    ```

    *Why?* : Primitive values cannot update alone using the revealing module pattern

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

  - **^Templating**: Use external templates instead of inline string templates for larger html blocks. These templates should be stored in Angular’s template cache. Inline String templates should only be 	used in rare circumstances (when the template is very short)

  - **DOM manipulation**: Takes place only inside Directives (link function), never a controller/service

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
        link: function ($scope, $element, $attrs) {
          $element.on('dragend', function () {
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
        controllerAs: 'dragUpload',
        controller: function () {

        }
      };
    }
    angular
      .module('app')
      .directive('dragUpload', dragUpload);
    ```
  - **Each directive should live in its own directory**: This directory will include the directive js file, the template, and any services that are specific to the directive. The parent director for all directives is typically /components.
  - Example:
  
  ```
  /components/listing/listing.js (the directive)
  /components/listing/listing.tpl.html (the template/partial for the directive)
  /components/listing/listingService.js (if the directive needs a service, used ONLY be this directive)
  /components/listing/listingCtrl.js (the controller for this directive)
  ```

  - **Use isolate scope in directives whenever possible**: This isn't an absolute, hence the _whenever possible_ phrase. Allowing directives to rely on inherited/shared scope can make the code brittle. Be explicit about what data your directive needs by passing it into the scope. Note that workarounds can be found if you have an element with multiple directives (since Angular only allows 1 directive per element to have an isolate scope).
  
  - **Provide a Unique Directive Prefix**
  Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo` which would be declared in HTML as `acme-sales-customer-info`.

    *Why?*: The unique short prefix identifies the directive's context and origin. For example a prefix of `cc-` may indicate that the directive is part of a CodeCamper app while `acme-` may indicate a directive for the Acme company.

  - **Use bindToController when using 'controller as'**
  Use `bindToController = true` when using `controller as` syntax with a directive when you want to bind the outer scope to the directive's controller's scope.

    *Why?*: It makes it easy to bind outer scope to the directive's controller scope.

    Note: `bindToController` was introduced in Angular 1.3.0.

  ```html
  <div my-example max="77"></div>
  ```

  ```javascript
  angular
      .module('app')
      .directive('myExample', myExample);

  function myExample() {
      var directive = {
          restrict: 'EA',
          templateUrl: 'app/feature/example.directive.html',
          scope: {
              max: '='
          },
          controller: ExampleController,
          controllerAs: 'ctrl',
          bindToController: true
      };

      return directive;
  }

  function ExampleController() {
      var self = this;
      self.min = 3;
      console.log('CTRL: ctrl.min = %s', ctrl.min);
      console.log('CTRL: ctrl.max = %s', ctrl.max);
  }
  ```

  ```html
  <!-- example.directive.html -->
  <div>hello world</div>
  <div>max={{ctrl.max}}<input ng-model="ctrl.max"/></div>
  <div>min={{ctrl.min}}<input ng-model="ctrl.min"/></div>
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

  - **^Promises**: When possible, resolve Controller dependencies in the `$stateProvider`(prefer `ui-router` instead of `ng-route`), not the Controller itself

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
    function config ($stateProvider) {
      $stateProvider
      .state('main', {
        url: '/main',
        controller: 'MainCtrl as mainCtrl',
        templateUrl: 'pages/main/main.tpl.html'
        resolve: {
          // resolve here
          locations: function () {
            return SomeService.getLocations();
        }
      });
    }
    angular
      .module('app')
      .config(config);
      
    function MainCtrl (SomeService, locations) {
      var self = this;
      self.locations = locations;
    }
  
    ```

  **Route Errors**

  - Handle and log all routing errors using [`$routeChangeError`](https://docs.angularjs.org/api/ngRoute/service/$route#$routeChangeError).

    *Why?*: Provides a consistent way to handle all routing errors.

    *Why?*: Potentially provides a better user experience if a routing error occurs and you route them to a friendly screen with more details or recovery options.

    ```javascript
    /* recommended */
    var handlingRouteChangeError = false;

    function handleRoutingErrors() {
        /**
         * Route cancellation:
         * On routing error, go to the dashboard.
         * Provide an exit clause if it tries to do it twice.
         */
        $rootScope.$on('$routeChangeError',
            function(event, current, previous, rejection) {
                if (handlingRouteChangeError) { return; }
                handlingRouteChangeError = true;
                var destination = (current && (current.title ||
                    current.name || current.loadedTemplateUrl)) ||
                    'unknown target';
                var msg = 'Error routing to ' + destination + '. ' +
                    (rejection.msg || '');

                /**
                 * Optionally log using a custom service or $log.
                 * (Don't forget to inject custom service)
                 */
                logger.warning(msg, [current]);

                /**
                 * On routing error, go to another route/state.
                 */
                $location.path('/');

            }
        );
    }
    ```

 
**[Back to top](#table-of-contents)**

## Publish and subscribe events

  - **$rootScope**: Use only `$emit` as an application-wide event bus and remember to unbind listeners

    ```javascript
    // all $rootScope.$on listeners
    $rootScope.$emit('customEvent', data);
    ```

  - Hint: `$rootScope.$on` listeners are different from `$scope.$on` listeners and will always persist, so they need destroying when the relevant `$scope` fires the `$destroy` event

    ```javascript
    // call the closure
    var unbind = $rootScope.$on('customEvent'[, callback]);
    $scope.$on('$destroy', unbind);
    ```

  - For multiple `$rootScope` listeners, use an Object literal and loop each one on the `$destroy` event to unbind all automatically

    ```javascript
    var rootListeners = {
      'customEvent1': $rootScope.$on('customEvent1'[, callback]),
      'customEvent2': $rootScope.$on('customEvent2'[, callback]),
      'customEvent3': $rootScope.$on('customEvent3'[, callback])
    };
    for (var unbind in rootListeners) {
      $scope.$on('$destroy', rootListeners[unbind]);
    }
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
    
    *Why?* : Binding once removes the `$$watchers` count after the `undefined` variable becomes resolved, thus reducing performance in each dirty-check
    
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
    function dragUpload () {
      return {
        link: function ($scope, $element, $attrs, $document) {
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
       * @param {Number} x First number to do something with
       * @param {Number} y Second number to do something with
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

## Dependencies

  **Manually Identify Dependencies**

  - Use `$inject` to manually identify your dependencies for Angular components.

    *Why?*: This technique mirrors the technique used by [`ng-annotate`](https://github.com/olov/ng-annotate), which I recommend for automating the creation of minification safe dependencies. If `ng-annotate` detects injection has already been made, it will not duplicate it.

    *Why?*: This safeguards your dependencies from being vulnerable to minification issues when parameters may be mangled. For example, `common` and `dataservice` may become `a` or `b` and not be found by Angular.

    *Why?*: Avoid creating in-line dependencies as long lists can be difficult to read in the array. Also it can be confusing that the array is a series of strings while the last item is the component's function.

    ```javascript
    /* avoid */
    angular
        .module('app')
        .controller('DashboardController',
            ['$location', '$routeParams', 'common', 'dataservice',
                function Dashboard($location, $routeParams, common, dataservice) {}
            ]);
    ```

    ```javascript
    /* avoid */
    angular
      .module('app')
      .controller('DashboardController',
          ['$location', '$routeParams', 'common', 'dataservice', Dashboard]);

    function Dashboard($location, $routeParams, common, dataservice) {
    }
    ```

    ```javascript
    /* recommended */
    angular
        .module('app')
        .controller('DashboardController', DashboardController);

    DashboardController.$inject = ['$location', '$routeParams', 'common', 'dataservice'];

    function DashboardController($location, $routeParams, common, dataservice) {
    }
    ```

    Note: When your function is below a return statement the `$inject` may be unreachable (this may happen in a directive). You can solve this by moving the Controller outside of the directive.

    ```javascript
    /* avoid */
    // inside a directive definition
    function outer() {
        var ddo = {
            controller: DashboardPanelController,
            controllerAs: 'vm'
        };
        return ddo;

        DashboardPanelController.$inject = ['logger']; // Unreachable
        function DashboardPanelController(logger) {
        }
    }
    ```

    ```javascript
    /* recommended */
    // outside a directive definition
    function outer() {
        var ddo = {
            controller: DashboardPanelController,
            controllerAs: 'vm'
        };
        return ddo;
    }

    DashboardPanelController.$inject = ['logger'];
    function DashboardPanelController(logger) {
    }
    ```

  **Module Dependencies**

  - The application root module depends on the app specific feature modules and any shared or reusable modules.

    ![Modularity and Dependencies](https://raw.githubusercontent.com/johnpapa/angular-styleguide/master/assets/modularity-1.png)

    *Why?*: The main app module contains a quickly identifiable manifest of the application's features.

    *Why?*: Each feature area contains a manifest of what it depends on, so it can be pulled in as a dependency in other applications and still work.

    *Why?*: Intra-App features such as shared data services become easy to locate and share from within `app.core` (choose your favorite name for this module).

    Note: This is a strategy for consistency. There are many good options here. Choose one that is consistent, follows Angular's dependency rules, and is easy to maintain and scale.

    > My structures vary slightly between projects but they all follow these guidelines for structure and modularity. The implementation may vary depending on the features and the team. In other words, don't get hung up on an exact like-for-like structure but do justify your structure using consistency, maintainability, and efficiency in mind.

    > In a small app, you can also consider putting all the shared dependencies in the app module where the feature modules have no direct dependencies. This makes it easier to maintain the smaller application, but makes it harder to reuse modules outside of this application.

**[Back to top](#table-of-contents)**

## Start Up Logic

  **Configuration**

  - Inject code into [module configuration](https://docs.angularjs.org/guide/module#module-loading-dependencies) that must be configured before running the angular app. Ideal candidates include providers and constants.

    *Why?*: This makes it easier to have less places for configuration.

  ```javascript
  angular
      .module('app')
      .config(configure);

  configure.$inject =
      ['routerHelperProvider', 'exceptionHandlerProvider', 'toastr'];

  function configure (routerHelperProvider, exceptionHandlerProvider, toastr) {
      exceptionHandlerProvider.configure(config.appErrorPrefix);
      configureStateHelper();

      toastr.options.timeOut = 4000;
      toastr.options.positionClass = 'toast-bottom-right';

      ////////////////

      function configureStateHelper() {
          routerHelperProvider.configure({
              docTitle: 'NG-Modular: '
          });
      }
  }
  ```

  **Run Blocks**

  - Any code that needs to run when an application starts should be declared in a factory, exposed via a function, and injected into the [run block](https://docs.angularjs.org/guide/module#module-loading-dependencies).

    *Why?*: Code directly in a run block can be difficult to test. Placing in a factory makes it easier to abstract and mock.

  ```javascript
  angular
      .module('app')
      .run(runBlock);

  runBlock.$inject = ['authenticator', 'translator'];

  function runBlock(authenticator, translator) {
      authenticator.initialize();
      translator.initialize();
  }
  ```
  **[Back to top](#table-of-contents)**

## Angular docs
For anything else, including API reference, check the [Angular documentation](//docs.angularjs.org/api).

## Contributing

Open an issue first to discuss potential changes/additions.

## License

#### (The MIT License)

Copyright (c) 2016 Premise Health

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
