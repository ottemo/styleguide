# JavaScript & AngularJS Styleguide

1. [General](#general)
1. [File Naming & Hierarchy](#file-naming--hierarchy)
1. [File Contents](#file-contents)
1. [Modules](#modules)
1. [Controllers](#controllers)
1. [Services](#services)
1. [Directives](#directives)

## General
1. Review the general enforcement files; [JSHint](/style/.jshintrc), [JSBeautify](/style/.jsbeautifyrc), [Editor Config](/style/.editorconfig)

    jshint can be run with the `gulp vet` task on any of the repos, here is a short-list though:

    + 4-spaces for indenting
    + unix line endings; 'lf'
    + use single quotes `'single quotes'`
    + use tripple equals `===` instead of `==`
    + always use curly braces
    + max depth of nested blocks is 5
    + max params is 10

1. Avoid using the `delete` keyword
    
    _Why:_ In modern JavaScript engines, changing the number of properties on an object is much slower than reassigning the values.

    ```
    // bad
    Foo.prototoype.dispose = function() {
        delete this.property;
    }

    // good
    Foo.prototoype.dispose = function() {
        this.property = null;
    }
    ```

1. Use methods that are included in our libraries if it makes the code more legible
    + `angular.forEach` is much nicer than a for loop with hasOwnProperty checks
    + `_.map()`, `_.find()`, `_.sortBy()`, `_.filter()`, `_.kebabCase()`, `_.debounce()`

## File Naming & Hierarchy
1. `kebab-case.html` all file names; lowercase, dash-separated

1. Module declaration files should be named `init.js`

1. Add a `.type` suffix for all other js files
    + `pdp/ot-related-products.directive.js`
    + `pdp/view.controller.js`

1. Templates, css, and js should have similar names, and live in the same folder
    + `pdp/view.html`, `pdp/view.scss`, `pdp/view.controller.js (PdpViewController)`
    + `pdp/ot-related-products.html`, `pdp/ot-related-products.directive.js`

## Themes
We use themes for sharing common frontend code between stores

1. Directory structure:

```
    js
    src/
        default/
        <theme>/
```
**Notes**

      `default` is a distinct git repository that implements default theme, contains all necessary frontend code to build a default store. 
    
     `<theme>` contains specific code for a store.

1. To setup a new store:  
     * Create `src/default` and clone default theme repository into it. Add this directory to .gitignore in the store.
     * Create custom store theme in `src/<themeName>`. 
     * Config themes in `gulp.config.js`  
```
    js  
    base = 'default', // src/default Default theme
    theme = 'ultimo'; // src/ultimo  Store theme
```

1. Files from `<theme>` overwrite files from `default`. If you need to modify code in `default`, create a file with the same name and path in `<theme>`.
2. To overwrite styles, copy `app.scss` from `default` into `theme`. You should place all sass imports in `app.scss`, avoid importing in other sass files. Then you are able to overwrite every sass file from `default` and import new styles in your theme
3. If you need to change controllers, prefer controller inheritance instead of overwriting

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

1. Controller filename should be suffixed with 'controller.js'  
    `accordion.controller.js`

1. Controller inheritance in themes.  
    Most of the time we need to slightly modify or extend default controllers. Use controller inheritance in such cases.  
    Steps to create a child controller:  
    + Child controller filename should be the same as parent and prefixed with `_`   
        ```
        parent:     src/default/checkout/accordiion.controller.js
        child:      src/ultimo/checkout/_accordiion.controller.js
        ``` 
        
    + To implement inheritance use the next code in child controller   
    `src/ultimo/checkout/_accordiion.controller.js`:   
    
        ```js
        .controller('checkoutAccordionController', [
        '$controller', '$scope'
        function($controller, $scope) {
            $controller('_checkoutAccordionController', { $scope: $scope });
        }]);
        ```  
        Notice that we use parent name for child controller:  
        ```js
        .controller('checkoutAccordionController' ...
        ```
        but prefix it with `_` in $controller call
        ```js
        $controller('_checkoutAccordionController'...
        ```
        
    + Overwrite parent methods, add new code:
        ```js
        angular.module('checkoutModule')
        .controller('checkoutAccordionController', [
            '$controller',
            '$scope',
            function (
                $controller,
                $scope
            ) {
                $controller('checkoutAccordionController', { $scope: $scope});
        
                // Overwrite 'next' method
                $scope.next = function(step) {
                    switch (step) {
                        case 'billingAddress':
                            $scope._actionBillingAddress(step);
                            break;
                        case 'shippingAddress':
                            $scope._actionShippingAddress(step);
                            break;
                        case 'shippingMethod':
                            $scope._actionShippingMethod(step);
                            break;
                        case 'paymentMethod':
                            $scope._actionPaymentMethod(step);
                            break;
                        case 'customerInfo':
                            $scope._actionCustomerAdditionalInfo(step);
                            break;
                        // Add new step 
                        case: 'newStep':
                            $scope._actionNewStep(step);
                            break;
                        default:
                            $scope._actionDefault(step);
                    }
                };
        
                // New step method
                $scope._actionNewStep = function(step) {
                    // Do something here...
                };
        ]);
        ```
    
    + Controller methods that are allowed to be overriden in child controllers should be present in the scope.  
        ```js
        // Can be overriden in child controller
        $scope._info = function() {
                return checkoutService.update().then(function (checkout) {
                    ...
                   
                });
            };
        
        // Cannot be changed in child
        function info() {
            return checkoutService.update().then(function (checkout) {
                    ...
                    
                });
        }
        ```
    + Avoid invoking any functions directly in controller. Move initialization to the template using `ng-init`
        ```js
        /** bad
        *  if we create child visitorAccountController, 
        *  activate() will be invoked before any modifications 
        */
        .controller("visitorAccountController", [
        "$scope",
        ...
        function($scope, ...) {
            $scope.visitor = visitorLoginService.getVisitor();
            ...
            activate();
    
            ////////////////////////////////
    
            function activate() {
                ...
            }
        ```
        
        ```js
        /* good
        *  don't call any functions on controller instantiation
        */
        angular.module("visitorModule")
        .controller("visitorAccountController", [
            "$scope",
            ...
            function($scope, ...) {
                $scope.changePswCredentials = {};
                ...
                ////////////////////////////////
        
                $scope.activate = function () {
                    ...
                };
        ```
        
        ```html
        <!-- in the template -->
        <i class="init-controller" ng-init="activate()"></i>
        ```

    

1. Use an `activate()` method to house any initialization logic
    ```js
    function AvengersController(avengersService, logger) {
        var vm = this;
        vm.avengers = [];
        vm.getAvengers = getAvengers;
        vm.title = 'Avengers';

        .///////////////////

        $scope.activate = function(){
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
    // good
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

1. Prefix ottemo directives with `ot-`

1. Never prefix with `ng-`

1. Restrict to attributes and elements `EA`

1. Template with `[].join('')` instead of string concatenation

    _Why:_ Improves readability as code can be indented properly, it also avoids the + operator which is less clean and can lead to errors if used incorrectly to split lines

    ```js
    // bad
    function someDirective() {
        return {
            template: '<div>'+
            '<i>' + hello + '</i>' +
            '</div>';
        };
    }

    // good
    function someDirective() {
        return {
            template: [
                '<div>',
                    '<i>',hello,'</i>',
                '</div>',
            ].join('')
        };
    }
    ```

1. Notes
    1. Use `$attributes.$observe()` to watch evaluated attributes
    2. `$element` is already an angular / jquery wrapped object
