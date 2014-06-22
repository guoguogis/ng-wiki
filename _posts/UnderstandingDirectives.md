### [Generously reshared from AngularUI](https://github.com/angular-ui/angular-ui/blob/master/modules/directives/jq/README.md)

This document is a (tangential) attempt to explain how AngularJS directives and the related compiling engine works so that you're not flailing around like a noodle the first time you try to tackle it yourself.

## Injecting, Compiling, and Linking functions

When you create a directive, there are essentially up to 3 function layers for you to define[ [1]](#footnotes):

```js
myApp.directive('uiJq', function InjectingFunction(){

  // === InjectingFunction === //
  // Logic is executed 0 or 1 times per app (depending on if directive is used).
  // Useful for bootstrap and global configuration

  return {
    compile: function CompilingFunction($templateElement, $templateAttributes) {

      // === CompilingFunction === //
      // Logic is executed once (1) for every instance of ui-jq in your original UNRENDERED template.
      // Scope is UNAVAILABLE as the templates are only being cached.
      // You CAN examine the DOM and cache information about what variables
      //   or expressions will be used, but you cannot yet figure out their values.
      // Angular is caching the templates, now is a good time to inject new angular templates 
      //   as children or future siblings to automatically run..

      return function LinkingFunction($scope, $linkElement, $linkAttributes) {

        // === LinkingFunction === //
        // Logic is executed once (1) for every RENDERED instance.
        // Once for each row in an ng-repeat when the row is created.
        // If ui-if or ng-switch may also affect if this is executed.
        // Scope IS available because controller logic has finished executing.
        // All variables and expression values can finally be determined.
        // Angular is rendering cached templates. It's too late to add templates for angular
        //  to automatically run. If you MUST inject new templates, you must $compile them manually.

      };
    }
  };
})
```

You can _only_ access data in `$scope` inside the **LinkingFunction**. Since the template logic may remove or duplicate elements, you can _only_ rely on the final DOM configuration in the **LinkingFunction**. You still _cannot_ rely upon **children** or **following-siblings** since they have not been linked yet.

## Pre vs Post Linking Functions
Anywhere you can use a `LinkingFunction()`, you can alternatively use an object with a pre and post linking function. [Oddly enough](https://github.com/angular/angular.js/issues/2592), a `LinkingFunction()` is a `PostLinkingFunction()` by default:
```js
link: function LinkingFunction($scope, $element, $attributes) { ... }
...
link: {
  pre: function PreLinkingFunction($scope, $element, $attributes) { ... },
  post: function PostLinkingFunction($scope, $element, $attributes) { ... },
}
```

The difference is that `PreLinkingFunction()` will fire on the parent first, then child, and so on. A `PostLinkingFunction()` goes in reverse, firing on the child first, then parent, and so on. Here's a demo: http://plnkr.co/edit/qrDMJBlnwdNlfBqEEXL2?p=preview

**When do I want this reverse `PostLinking` behavior?** Sometimes jQuery plugins need to know the number and size of children DOM element's (such as slideshows or layout managers like Isotope). There are a few ways to support these:

* **(Worst)** Delay the plugin's execution using [$timeout](http://docs.angularjs.org/api/ng.$timeout)
* Nested directives. If each child has a directive, it can  `require: '^parentDirective'` which will give you access to the `parentDirective` controller.
  * If you use the `PreLinkingFunction()` on `parentDirective`, you can instantiate the container empty, and use then update it every time the 

**This does _NOT_ accomodate for async changes such as loading `$scope` data via AJAX**

If you need to wait till your `$scope` data finishes loading try using [ui-if](http://angular-ui.github.com/#directives-if) to defer linking of a block of DOM.

## $element === angular.element() === jQuery() === $()

To make working with the DOM easier, AngularJS contains a miniaturized version of jQuery called jqlite. This emulates some of the core features of jQuery using an _almost_ identical API as jQuery. Any time you see an AngularJS DOM element, it will be the equivalent to a `jQuery()` wrapped DOM element.

**You do _NOT_ have to wrap AngularJS elements in `jQuery()`**

If you are noticing that the full array of jQuery methods (or plugins) aren't available on an AngularJS element, it's because you either forgot to load the jQuery lib, or you forgot to load it **BEFORE** loading AngularJS. If AngularJS doesn't see jQuery already loaded by the time AngularJS loads, it will use its own jqlite library instead.

## $attributes.$observe()

If you have a _sibling_ attribute that will contain `{{}}` then the attribute will need to be evaluated and could even change multiple times. **Don't do this manually!**

Instead use `$attributes.$observe('myOtherAttribute', function(newValue))` exactly as you would have used `$scope.$watch()`. The only difference in the first argument is the attribute name (not an expression) and the callback function only has `newValue` (already evaluated for you). It will re-fire the callback every single time the evaluation changes too.

**NOTE:** This means that you can only access this attribute _asynchronously_

**NOTE:** If you want to _reliably_ access the attribute pre-evaluation then you should do it in the CompileFunction

## Extending Directives

Lets say you want to use a 3rd-party directive, but you want to extend it without modifying it. There are several ways you can go about doing this.

### Global Configurations
Some well-designed directives (such as those found in AngularUI) can be configured globally so that you do not have to pass in your options into every instance.
### Require Directives
Create a new directive that assumes the first directive has already been applied. You can require it on a parent DOM element, OR on the same DOM element. If you need to access functionality found in the primary directive, make it exposed via the directive controller (this may require submitting a Pull Request or feature request to the plugin developer).  
```js
// <div a b></div>
ui.directive('a', function(){
  return {
    controller: function(){
      this.data = {}
      this.changeData = function( ... ) { ... }
    },
    link: ($scope, $element, $attributes, controller) {
      controller.data = { ... }
    }
  }
})
myApp.directive('b', function(){
  return {
    require: 'a',
    link: ($scope, $element, $attributes, aController) {
      aController.changeData()
      aController.data = { ... }
    }
  }
})
```
### Stacking Directives
You can create a new directive with the exact same name as the original directive. Both directives will be executed. However, you can use the priority to control which directive fires first (again, may require a Pull Request or feature request)
```js
// <div a></div>
ui.directive('a', ... )
myApp.directive('a', ... )
```
### Templating
You can leverage `<ng-include>` or simply create a directive that generates the HTML with the primary directive attached.
```js
// <div b></div>
ui.directive('a', ... )
myApp.directive('b', function(){
  return {
    template: '<div a="someOptions"></div>'
  }
})
```
## Footnotes

1. A [transcluding function](http://docs.angularjs.org/guide/directive) is actually a 4th layer, but this is not used by uiJq