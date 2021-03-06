Unit tests are generally used to test a small piece of code and ensure that it
is doing what was intended. Unlike acceptance tests, they are narrow in scope
and do not require the Ember application to be running.

As it is the basic object type in Ember, being able to test a simple
`Ember.Object` sets the foundation for testing more specific parts of your
Ember application such as controllers, components, etc. Testing an `Ember.Object`
is as simple as creating an instance of the object, setting its state, and
running assertions against the object. By way of example lets look at a few
common cases.

### Testing Computed Properties

Let's start by looking at an object that has a `computedFoo` computed property
based on a `foo` property.

```javascript {data-filename=app/models/some-thing.js}
export default Ember.Object.extend({
  foo: 'bar',

  computedFoo: function() {
    return 'computed ' + this.get('foo');
  }.property('foo')
});
```

Within the test we'll create an instance, update the `foo` property (which
should trigger the computed property), and assert that the logic in our
computed property is working correctly.

```javascript {data-filename=tests/unit/models/some-thing-test.js}
import SomeThing from '<your-app-name>/models/some-thing';

moduleFor('model:some-thing', 'Unit: some-thing');

test('computedFoo correctly concats foo', function(assert) {
  var someThing = SomeThing.create({});

  someThing.set('foo', 'baz');

  assert.equal(someThing.get('computedFoo'), 'computed baz');
});
```

See that we have used `moduleFor` one of the several [unit-test helpers](../unit-test-helpers/) provided
by Ember-Qunit.

### Testing Object Methods

Next let's look at testing logic found within an object's method. In this case
the `testMethod` method alters some internal state of the object (by updating
the `foo` property).

```javascript {data-filename=app/models/some-thing.js}
export default Ember.Object.extend({
  foo: 'bar',
  testMethod: function() {
    this.set('foo', 'baz');
  }
});
```

To test it, we create an instance of our class `SomeThing` as defined above, 
call the `testMethod` method and assert that the internal state is correct as a 
result of the method call.

```javascript {data-filename=tests/unit/models/some-thing-test.js}
import SomeThing from '<your-app-name>/models/some-thing';

moduleFor('model:some-thing', 'Unit: some-thing');

test('calling testMethod updated foo', function(assert) {
  var someThing = SomeThing.create({});

  someThing.testMethod();

  assert.equal(someThing.get('foo'), 'baz');
});
```

In the event the object's method returns a value you can simply assert that the
return value is calculated correctly. Suppose our object has a `calc` method
that returns a value based on some internal state.

```javascript {data-filename=app/models/some-thing.js}
export default Ember.Object.extend({
  count: 0,
  calc: function() {
    this.incrementProperty('count');
    return 'count: ' + this.get('count');
  }
});
```

The test would call the `calc` method and assert it gets back the correct value.

```javascript {data-filename=tests/unit/models/some-thing-test.js}
import SomeThing from '<your-app-name>/models/some-thing';

moduleFor('model:some-thing', 'Unit: some-thing');

test('calc returns incremented count', function(assert) {
  var someThing = SomeThing.create({});
  assert.equal(someThing.calc(), 'count: 1');
  assert.equal(someThing.calc(), 'count: 2');
});
```

### Testing Observers

Suppose we have an object that has a property and a method observing that property.

```javascript {data-filename=app/models/some-thing.js}
export default Ember.Object.extend({
  foo: 'bar',
  other: 'no',
  doSomething: function(){
    this.set('other', 'yes');
  }.observes('foo')
});
```

In order to test the `doSomething` method we create an instance of `SomeThing`,
update the observed property (`foo`), and assert that the expected effects are present.

```javascript {data-filename=tests/unit/models/some-thing-test.js}
import SomeThing from '<your-app-name>/models/some-thing';

moduleFor('model:some-thing', 'Unit: some-thing');

test('doSomething observer sets other prop', function() {
  var someThing = App.SomeThing.create();
  someThing.set('foo', 'baz');
  equal(someThing.get('other'), 'yes');
});
```

