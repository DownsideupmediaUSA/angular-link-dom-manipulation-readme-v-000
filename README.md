# Using the link function for DOM manipulation

## Overview

We've had a look at the link function, now lets do some DOM manipulation with it.

## Objectives

- Use a native event API
- Use a Controller inside Directive
- Update Controller layer using native events

## Native Events

Now that we've got access to the actual DOM nodes, we can do our own, manual DOM events on the elements. Whilst generally we'd use `ng-click` and let Angular do the dirty work for us, we can do it ourselves manually if we are using 3rd-party plugins that aren't compatible with Angular.

Let's take a simple directive:

```js
function SomeDirective() {
	return {
		template: [
			'<div>',
				'<span>Click on me!</span>',
			'</div>'
		].join(''),
		link: function (scope, elem, attrs) {

		}
	}
}

angular
	.module('app')
	.directive('someDirective', SomeDirective);
```

Now it's important to note that `elem` isn't *actually* the raw DOM node. It is a jqLite (or jQuery if you have loaded jQuery before Angular) element. However, we can access the raw DOM node via `elem[0]`.

Let's add a click event to our `<span />`.

```js
function SomeDirective() {
	return {
		template: [
			'<div>',
				'<span>Click on me!</span>',
			'</div>'
		].join(''),
		link: function (scope, elem, attrs) {
			var actualElement = elem[0];

			var spanElement = actualElement.querySelector('span');

			spanElement.addEventListener('click', function () {
				alert('You clicked me!');
			});
		}
	}
}

angular
	.module('app')
	.directive('someDirective', SomeDirective);
```

Here, we will get an alert showing when the user clicks on the span. Awesome! Now, say that we want to actually update the `scope` values when the user clicks on the span - how do we do this? First of all, let's put a scope value in our view and controller:

```js
function SomeDirective() {
	return {
		template: [
			'<div>',
				'<span>Click on me!</span>',
				'{{ status }}',
			'</div>'
		].join(''),
		controller: function ($scope) {
			$scope.status = 'Not clicked!';
		},
		link: function (scope, elem, attrs) {
			var actualElement = elem[0];

			var spanElement = actualElement.querySelector('span');

			spanElement.addEventListener('click', function () {
				scope.status = 'Clicked!';
			});
		}
	}
}

angular
	.module('app')
	.directive('someDirective', SomeDirective);
```

Now you can see that we're updating `scope` in our link function. However, this won't actually work. We're outside the scope of Angular here, so Angular isn't able to tell that we've updated our scope.

However, we can manually push Angular to update, by using `scope.$apply()`. Don't worry about this magic, we'll go into detail with this soon.

```js
function SomeDirective() {
	return {
		template: [
			'<div>',
				'<span>Click on me!</span>',
				'{{ status }}',
			'</div>'
		].join(''),
		controller: function ($scope) {
			$scope.status = 'Not clicked!';
		},
		link: function (scope, elem, attrs) {
			var actualElement = elem[0];

			var spanElement = actualElement.querySelector('span');

			spanElement.addEventListener('click', function () {
				scope.status = 'Clicked!';

				scope.$apply();
			});
		}
	}
}

angular
	.module('app')
	.directive('someDirective', SomeDirective);
```

Sorted! This will update our `scope` with the new value.

## Update the controller

One problem - we don't *really* use `$scope` anymore - we use controller values. Well, much like when we required the parent controller in our previous README, we can actually request our own directives controller for use in the link function.

To do this, we add `require` with the value `ngController`.

```js
function SomeDirective() {
	return {
		template: [
			'<div>',
				'<span>Click on me!</span>',
				'{{ some.status }}',
			'</div>'
		].join(''),
		require: 'ngController',
		controller: function () {
			this.status = 'Not clicked!';
		},
		controllerAs: 'some',
		link: function (scope, elem, attrs, ctrl) {
			var actualElement = elem[0];

			var spanElement = actualElement.querySelector('span');

			spanElement.addEventListener('click', function () {
				ctrl.status = 'Clicked!';

				scope.$apply();
			});
		}
	}
}

angular
	.module('app')
	.directive('someDirective', SomeDirective);
```

Have a look at the minor changes we've made - we've now attached `status` to our controller instead of scope, updated the view to reflect that and added the require statement we talked about. We've now got that fourth argument in our link function too, and we're updating `ctrl.status` in our event.

We're still using `scope.$apply()` though - why? We still use it because `$apply()` is a function only available on scope, and we still need to tell Angular that we've updated our view.