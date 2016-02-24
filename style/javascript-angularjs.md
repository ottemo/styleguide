# JavaScript & AngularJS Styleguide

1. [File Naming & Hierarchy](#file-naming--hierarchy)
1. [File Contents](#file-contents)
1. [Modules](#modules)
1. [Controllers](#controllers)
    1. names should be `UpperCamelCase`
    1. names should have the suffix `Controller`
    1. use the `controllerAs` syntax
    1. capture the variable for this in `vm`
    1. use function declarations to hide implementation details `function doThing() {}`
    1. put bindable members up top
    1. use an `activate` method to house any initialization logic
1. [Services](#services)

## File Naming & Hierarchy
1. `kebab-case.html` or `kebab-case.type.js` all files

1. template, css, and controller should have similar names, and live in the same folder
  * pdp/view.html
  * pdp/view.scss
  * pdp/view.controller.js -> PdpViewController

## File Contents
Each file should contain one "thing"; module definition, controller, service, etc

## Modules
A module declaration should use the setter syntax
```js
// good: setter syntax to declare module
angular.module('app', []);

// good: getter syntax to recall module
angular.module('app')
.controller('SomeController', SomeController);
```

## Controllers
1. Controller names should be `UpperCamelCase`

1. Controller names should be suffixed with `Controller` ie. `CustomerController` or `CheckoutController`

1. Prefer the `controllerAs` syntax, this promotes the use of _dotted_ objects
    * if the controller is the view model controller try to name it `vm` for consistency
    * if the controller is embedded then consider naming it something else

    Declaring the controller in the config:
    ```js
    // recommended
    angular.module('app')
    .config(function($routeProvider) {
        $routeProvider
            .when('/avengers', {
                templateUrl: 'avengers.html',
                controller: 'Avengers',
                controllerAs: 'vm'
            })
    });
    ```

    Declaring the controller in the html:
    ```html
    <!-- bad -->
    <div ng-controller="CustomerController">{{ name }}</div>

    <!-- good -->
    <div ng-controller="CustomerController as customer">{{ customer.name }}</div>
    ```

1. Capture the variable for `this` with `vm` in the controller code as well, for consistency
    ```js
    // good
    function CustomerController() {
        var vm = this;
        vm.name = {};
    }
    ```

1. Use Function Declarations to hide implementation details
    Use function declarations to hide implementation details. Keep your bindable members up top. When you need to bind a function in a controller, point it to a function declaration that appears later in the file. This is tied directly to the section Bindable Members Up Top. For more details see [this post](http://www.johnpapa.net/angular-function-declarations-function-expressions-and-readable-code/).

    Why?: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View. (Same as above.)

    Why?: Placing the implementation details of a function later in the file moves that complexity out of view so you can see the important stuff up top.

    Why?: Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions).

    Why?: You never have to worry with function declarations that moving var a before var b will break your code because a depends on b.

    Why?: Order is critical with function expressions

    ```js
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

    Notice that the important stuff is scattered in the preceding example. In the example below, notice that the important stuff is up top. For example, the members bound to the controller such as vm.avengers and vm.title. The implementation details are down below. This is just easier to read.
    ```js
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

1. Put the bindable members up top
    *  If the function is a 1 liner consider keeping it right up top, as long as readability is not affected.

    ```js
    /* bad */
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

    /* good */
    function SessionsController() {
        var vm = this;

        vm.gotoSession = gotoSession;
        vm.refresh = refresh;
        vm.search = search;
        vm.sessions = [];
        vm.title = 'Sessions';

        ////////////

        function gotoSession() {
          /* ... */
        }
        function refresh() {
          /* ... */
        }
        function search() {
          /* ... */
        }
    }
    ```

1. Use an `activate()` method to house any initialization logic
    ```js
    function AvengersController(avengersService, logger) {
        var vm = this;
        vm.avengers = [];
        vm.getAvengers = getAvengers;
        vm.title = 'Avengers';

        activate();

        ////////////////////

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

## Services
1. Keep the accessible methods at the top, and var declarations at the top.
    ```js
    /* avoid */
    function dataService() {
      var someValue = '';
      function save() {
        /* */
      };
      function validate() {
        /* */
      };

      return {
          save: save,
          someValue: someValue,
          validate: validate
      };
    }
    ```

    ```js
    /* recommended */
    function dataService() {
        var someValue = '';
        var service = {
            save: save,
            someValue: someValue,
            validate: validate
        };
        return service;

        ////////////

        function save() {
            /* */
        };

        function validate() {
            /* */
        };
    }
    ```

1. Use function declarations to hide implementation details
    Why: Same reasons as listed in the controllers section
    ```js
    // bad
    var getAvengers = function() {
    }

    // good
    function getAvengers() {
    }
    ```
