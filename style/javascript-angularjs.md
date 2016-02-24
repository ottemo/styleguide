# JavaScript & AngularJS Styleguide

1. [General](#general)
1. [File Naming & Hierarchy](#file-naming--hierarchy)
1. [File Contents](#file-contents)
1. [Modules](#modules)
1. [Controllers](#controllers)
1. [Services](#services)
1. [Directives](#directives)

## General
1. Review the [.jshintrc](/style/.jshintrc) file for general style guidelines; additionally you can run jshint on any of the js repos.

## File Naming & Hierarchy
1. `kebab-case.html` all file names; lowercase, dash-separated

1. Module declaration files should be named `init.js`

1. Add a `.type` suffix for all other js files; `pdp/ot-related-products.directive.js`, `pdp/view.controller.js`

1. Templates, css, and js should have similar names, and live in the same folder
  * `pdp/view.html`, `pdp/view.scss`, `pdp/view.controller.js (PdpViewController)`
  * `pdp/ot-related-products.html`, `pdp/ot-related-products.directive.js`

## File Contents
Each file should contain one "thing"; module definition, controller, service, etc

## Modules
1. Module declaration files should be called `init.js`

1. A module declaration should use the setter syntax, other files should use the getter syntax
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

1. Controller names should be a representation of the file system location
    `PdpViewController` = `/pdp/view.controller.js`

1. Use Function Declarations to hide implementation details
    
    Keep your bindable members up top. When you need to bind a function in a controller, point it to a function declaration that appears later in the file. This is tied directly to the section Bindable Members Up Top. For more details see [this post](http://www.johnpapa.net/angular-function-declarations-function-expressions-and-readable-code/).

    Why?: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View. (Same as above.)

    Why?: Placing the implementation details of a function later in the file moves that complexity out of view so you can see the important stuff up top.

    Why?: Function declaration are hoisted so there are no concerns over using a function before it is defined (as there would be with function expressions).

    Why?: You never have to worry with function declarations that moving var a before var b will break your code because a depends on b.

    Why?: Order is critical with function expressions

    ```js
    /**
     * bad
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
     * good
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

1. View controllers should not be re-used across multiple views
    + _Why:_ In theory reusing code like this is a good way to keep the codebase DRY, in practice this leads to confusion and errors.
    + View controllers should not be embedded in other views, we did this with the CartController and LoginController let those mistakes serve as a lesson to us.

1. Prefer the `controllerAs` syntax, this promotes the use of _dotted_ objects
    + if the controller is the view model controller try to name it `vm` for consistency
    + if the controller is embedded then consider naming it something else

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


## Services
1. Service names should be `lowerCamelCased`

1. Service names should only be suffixed with `Service` when it is not apparent what they are
    logger, customerService, 

1. Services should **not** be prefixed with a dollar sign `$` this is considered reserved for angular internals

1. Prefer `factory` over `service`
    + all services are singletons, since they are so similar just stick with factories for consistency.

1. Keep the accessible methods at the top, and var declarations at the top.
    ```js
    /* bad */
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
    /* good */
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

    + _Why:_ Same reasons as listed in the controllers section
    
    ```js
    // bad
    var getAvengers = function() {
    }

    // good
    function getAvengers() {
    }
    ```

1. Data calls should always return a promise


## Directives
1. One directive per file
1. Prefix ottemo directives with `ot`
1. Restrict to attributes and elements `EA`
1. Notes
    1. Use `$attributes.$observe()` to watch evaluated attributes
    2. `$element` is already an angular / jquery wrapped object