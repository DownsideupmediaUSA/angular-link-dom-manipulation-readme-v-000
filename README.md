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

Now it's important to note that `elem` isn't *actually* the raw DOM node. It is a jqLite (a light version of jQuery) (or jQuery if you have loaded jQuery before Angular) element. However, we can access the raw DOM node via `elem[0]`.

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

We're adding a DOM event to the actual DOM element rather than using jQuery/jqLite. This is because the API for them slightly differ, and as we can't guarantee what would be loaded on the page, we'll use native JavaScript events instead.

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

Sorted! This will update our `scope` with the new value. This is also effectively how `ng-click` works, which is why we use them instead of doing all of our DOM events in the link function!

## Update the controller

One problem - we don't *really* use `$scope` anymore - we use controller values. Well, much like when we required the parent controller in our previous README, we can actually request our own directives controller for use in the link function.

To do this, we add `require` with the value `someDirective`. Before, we used `^nameOfDirective` to get the parent's directive. Notice how we aren't using a `^` anymore - we are no longer looking upwards to the parents for a controller - instead, we're asking for the controller of the directive (the `someDirective` part).

```js
function SomeDirective() {
	return {
		template: [
			'<div>',
				'<span>Click on me!</span>',
				'{{ some.status }}',
			'</div>'
		].join(''),
		require: 'someDirective',
		controller: function () {
			$scope.status = 'Not clicked!';
		},
		link: function (scope, elem, attrs, ctrl) {
			var actualElement = elem[0];

			var spanElement = actualElement.querySelector('span');

			spanElement.addEventListener('click', function () {
				// ctrl.status = undefined;
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

Have a look at the minor changes we've made - we've now got that fourth argument in our link function too, and we're updating `ctrl.status` in our event.

However, our status value in our controller is still on our `scope`. We need to change this over to the controller, using `controllerAs`. We can then attach the value to `this` instead of the scope.

```js
function SomeDirective() {
	return {
		template: [
			'<div>',
				'<span>Click on me!</span>',
				'{{ some.status }}',
			'</div>'
		].join(''),
		require: 'someDirective',
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


We're still using `scope.$apply()` though - why? We still use it because `$apply()` is a function only available on scope, and we still need to tell Angular that we've updated our view.
