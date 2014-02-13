---
author: ankur
date: 2013-07-20 08:32:44+00:00
layout: post
slug: loading-spinners-with-angularjs-and-spin-js
title: Loading Spinners With AngularJS and Spin.js
categories:
    - AngularJS
    - Programming
---

[Spin.js](http://fgnass.github.io/spin.js/) is a tiny JavaScipt library that helps you create beautiful loading spinners on every web browser up from IE6. It is highly customizable, fast, and has zero dependencies. I'm using it in [my AngularJS application](http://github.com/GeneralMaximus/secondhand-web) to display loading spinners inside my `ng-view`s while my REST API responds with the data the view needs to render itself.

I add a `viewLoading` boolean to the `$scope` of each controller that talks to the REST API. The initial value of `viewLoading` is `true`.

{% highlight javascript %}
angular.module('MyApplication')
  .controller('MakesTonsOfAPICallsController', function($scope) {
    $scope.viewLoading = true;
  });
{% endhighlight %}

After all the API calls complete successfully, I set `viewLoading` to false.

{% highlight javascript %}
angular.module('MyApplication')
  .controller('MakesTonsOfAPICallsController', function($scope, MyLargeModel) {
    $scope.viewLoading = true;

    // Grab all MyLargeModel objects.
    MyLargeModel.get({}, function(result) {
      // Do something with the result.
      $scope.viewLoading = false;
    });
  });
{% endhighlight %}

If I have to make multiple calls, I use the [$q service](http://docs.angularjs.org/api/ng.$q) to create a promise for each of them. Each promise is resolved or rejected depending on the status code that the API call returns. I then use `$q.all()` to call a function when all of the promises have been resolved. This function sets `viewLoading` to `false`. I will talk more about $q in another post, but here is a rather simplistic example for now:

{% highlight javascript %}
$q.all([promise1, promise2 ... promiseN]).then(function(data) {
  $scope.viewLoading = false;
});
{% endhighlight %}

I want the loading spinner to be displayed for as long as `$viewLoading` is `true`, and be replaced by the actual view content as soon as `viewLoading` becomes `false`. I use a directive to do this. This is what the markup looks like:

{% highlight html %}
<div ng-controller="MakesTonsOfAPICallsController">
  <div my-loading-spinner="viewLoading">
    <!-- actual view content goes here. -->
  </div>
</div>
{% endhighlight %}

And this is what the directive looks like:

{% highlight javascript %}
angular.module('MyApplication')
  .directive('myLoadingSpinner', function() {
    return {
      restrict: 'A',
      replace: true,
      transclude: true,
      scope: {
        loading: '=myLoadingSpinner'
      },
      templateUrl: 'directives/templates/loading.html',
      link: function(scope, element, attrs) {
        var spinner = new Spinner().spin();
        var loadingContainer = element.find('.my-loading-spinner-container')[0];
        loadingContainer.appendChild(spinner.el);
      }
    };
  });
{% endhighlight %}

For this to work correctly, the Spin.js code has to be loaded before the directive code.

The directive is restricted to attributes only and replaces the original content on the page with the content from my template. I set transclude to true so I can re-insert the original content back into the page later. If you look back at the HTML for the view, you will find that the value of the `myLoadingSpinner` attribute is `viewLoading`. When Angular encounters our markup, it will create a two-way binding between the `loading` variable in the directive's scope and the `viewLoading` variable in the parent controller's scope. If you find this confusing, you may want to [read about directives on the AngularJS website](http://docs.angularjs.org/guide/directive).

Before I explain the `link` function, take a look at the directive's template:

{% highlight html %}
<div>
  <div ng-show="loading" class="my-loading-spinner-container"></div>
  <div ng-hide="loading" ng-transclude></div>
</div>
{% endhighlight %}

The markup is simple enough. The `div` with class `my-loading-spinner-container` is displayed when `loading` is `true`, and hidden if it is `false`. The second `div` is hidden if `loading` is `true`, and displayed if it is `false`. The second `div` also uses `ng-transclude` to re-include into the page the original content that was replaced by our directive.

Finally, the `link` function creates a new loading spinner, finds the `div` with the class `my-loading-spinner-container`, and puts the spinner inside the `div`. Hence, the spinner is displayed as long as `loading` is `true`, and the actual content is shown when it becomes `false`, which is exactly what we want.