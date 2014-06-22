#### Related: [[Best Practices]]

1. Don't wrap `element` inside of `$()`. All AngularJS elements are already jq-objects
2. Don't do `if (!$scope.$$phase) $scope.$apply()`, it means your `$scope.$apply()` isn't high enough in the call stack.
3. Don't use jQuery to generate templates or DOM
4. Don't create a new plugin without trying to discover, fork and pull request existing plugins first