#### Related: [[Anti-Patterns]]

* **Namespace distributed code**  
  You shouldn't worry about prefixing internal code, but anything you plan to OpenSource should be namespaced
  * The `ng-` is reserved for core directives.
  * Purpose-namespacing (`i18n-` or `geo-`) is better than owner-namespacing (`djs-` or `igor-`)
  * Checkout [ui-alias](https://github.com/angular-ui/ui-alias) to remove 3rd party prefixes
* **Only use `.$broadcast()`, `.$emit()` and `.$on()` for atomic events**  
  Events that are relevant globally across the entire app (such as a user authenticating or the app closing). If you want events specific to modules, services or widgets you should consider Services, Directive Controllers, or 3rd Party Libs
  * `$scope.$watch()` should replace the need for events
  * Injecting services and calling methods directly is also useful for direct communication
  * Directives are able to directly communicate with each other through directive-controllers
* **Always let users use expressions whenever possible**  
  * `ng-href` and `ng-src` are plaintext attributes that support `{{}}`
  * Use `$attrs.$observe()` since expressions are _async_ and could change
* **Extend directives by using Directive Controllers**  
  You can place methods and properties into a directive-controller, and access that same controller from other directives. You can even override methods and properties through this relationship
* **Add teardown code to controllers and directives**  
  Controller and directives emit an event right before they are destroyed. This is where you are given the opportunity to tear down your plugins and listeners and pretty much perform garbage collection.
  * Subscribe to the `$scope.$on('$destroy', ...)` event
* **Leverage modules _properly_**  
  Instead of slicing your app across horizontals that can't be broken up, group your code into related bundles. This way if you remove a module, your app still works.
  * Checkout [angular-app/angular-app](https://github.com/angular-app/angular-app/tree/master/client/src/app) for a good example
  * `app.controllers`, `app.services`, etc will break your app if you remove a module
  * `app.users`, `app.users.edit`, `app.users.admin`, `app.projects`, etc allows you to group and nest related components together and create loose coupling
  * Spread route definitions across multiple module `.config()` methods
  * Modules can have their own dependencies (including external)
  * **Folder structure _should_ reflect module structure**
* **Add Bower Support**  
  This has become the standard for AngularJS so it's a good idea to familiarize yourself