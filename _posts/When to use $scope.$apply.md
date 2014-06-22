AngularJS provides wrappers for common native JS async behaviors:
* Events => `ng-click`
* Timeouts => `$timeout`
* jQuery.ajax() => `$http`

This is just a traditional async function with a `$scope.$apply()` called at the end, to tell AngularJS that an asynchronous event just occurred.

**`$scope.$apply()` should occur as close to the async event binding as possible.**

Do **NOT** randomly sprinkle it throughout your code. If you are doing `if (!$scope.$$phase) $scope.$apply()` it's because you are not high enough in the call stack.

Whenever possible, use AngularJS services instead of native. If you're creating an AngularJS service (such as for sockets) it should have a `$scope.$apply()` anywhere it fires a callback.