# ReactiveContext

ReactiveContext is a class that manages the reactivity of a context, and manages the subscriptions and helpers.
Instance of this class is returned from the `$reactive` service after using it.

Using this object you can define:

* Meteor Subscriptions - wraps `Meteor.subscribe` and provides the acknowledgment when the subscription updates.
* View Helpers - you can define helpers and use them in your view, and Angular-Meteor will take care of your view reactivity.

## Methods

### constructor

The constructor is private, and called by using $reactive service.

<table class="variables-matrix input-arguments">
  <thead>
  <tr>
    <th>Param</th>
    <th>Type</th>
    <th>Details</th>
    <th>Required</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td><strong>context</strong></td>
    <td>
      Context (Scope or controller's "this" context)
    </td>
    <td>Context to set as reactive - this context will be used to hold the helpers lcoals.</td>
    <td>Yes</td>
  </tr>
  </tbody>
</table>

### attach

This method attaches the context to scope - we need to attach to scope in order to $apply it when changes are made, in order to update the view.

This method is only needed when using `this` context (when using controllerAs or angular2now).

<table class="variables-matrix input-arguments">
  <thead>
  <tr>
    <th>Param</th>
    <th>Type</th>
    <th>Details</th>
    <th>Required</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td><strong>scope</strong></td>
    <td>
      Scope
    </td>
    <td>This scope will be used as the reactive scope, and it will be updated when your data updates.</td>
    <td>Yes</td>
  </tr>
  </tbody>
</table>

### helpers

This method expect to get an object with definitions, it's usage is very similar to Meteor's helpers. Each property in the definition defines a different behavior of your object:
* Using a function - this function will execute each Autorun invalidate - very useful for creating a reactive collection to use in your view.
* Using a object/primitive/string - this will create a reactive variable that you can usem and every change of this object will cause a Autorun invalidate.

The return value of this method is `this` - in order to provide ability to chain the angular-meteor logic (`reactiveContext.helpers(...).subscribe(...)`).

<table class="variables-matrix input-arguments">
  <thead>
  <tr>
    <th>Param</th>
    <th>Type</th>
    <th>Details</th>
    <th>Required</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td><strong>defs</strong></td>
    <td>
      Object
    </td>
    <td>Object contains properties - if the property is a function it will be called each Autorun, if the property is not a function (primitive/string/array/object) - it will treated as reactive variable, and when you set it's value it will cause Autorun to run.</td>
    <td>Yes</td>
  </tr>
  </tbody>
</table>

### subscribe

This method is a wrapper for `Meteor.subscribe`, just with reactivity ability - the subscription will be updated when your reactive variables changes.

<table class="variables-matrix input-arguments">
  <thead>
  <tr>
    <th>Param</th>
    <th>Type</th>
    <th>Details</th>
    <th>Required</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td><strong>subscription_name</strong></td>
    <td>
      String
    </td>
    <td>The name of the subscription, as defined in your Meteor.publish.</td>
    <td>Yes</td>
  </tr>
  <tr>
    <td>Arguments callback</td>
    <td>
      Function
    </td>
    <td>Callback function that returns an array with the subscription arguments.</td>
    <td>No</td>
  </tr>
  </tbody>
</table>

### autorun

This method is a wrapper for `Tracker.autorun`, that will be available for clean angular-meteor usage.

<table class="variables-matrix input-arguments">
  <thead>
  <tr>
    <th>Param</th>
    <th>Type</th>
    <th>Details</th>
    <th>Required</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td><strong>Callback</strong></td>
    <td>
      Function
    </td>
    <td>Callback function that will run in each Autorun cycle.</td>
    <td>Yes</td>
  </tr>
  </tbody>
</table>

### stop

This method takes care of stopping every `subscribe`, `autorun` and `helpers` registration you created.

Note that this method is called automatically when using `$scope` with your reactive context, which means <strong>you need to call this method manually when you do not use `$scope`.</strong>

<table class="variables-matrix input-arguments">
  <thead>
  <tr>
    <th>Param</th>
    <th>Type</th>
    <th>Details</th>
    <th>Required</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td><strong>Callback</strong></td>
    <td>
      Function
    </td>
    <td>Callback function that will run in each Autorun cycle.</td>
    <td>Yes</td>
  </tr>
  </tbody>
</table>


## Usage examples

The core of the usage with this object, is the `subscribe` and `helpers` methods.

For example, let's create a reactive variable:

    angular.module('myApp', []).controller('MyCtrl', ['$scope', '$reactive', function($scope, $reactive)
    {
      let reactiveContext = $reactive(this).attach($scope);

      reactiveContext.helpers({
        myName: 'Dotan'
      });

      // This will print your actual value, and you can use it also in your view!
      console.log(this.myName);
    }]);

And when you will modify the value of `this.myName`, it will cause `Tracker.Autorun` to invalidate and run your reactive logic again.

Also, it is possible to create a collection helpers that returns MongoDB cursor, using `helpers`:

    angular.module('myApp', []).controller('MyCtrl', ['$scope', '$reactive', function($scope, $reactive)
    {
      let reactiveContext = $reactive(this).attach($scope);

      reactiveContext.helpers({
        allUsers: function() {
          return Users.find({});
        }
      });

      // This will print the whole collection, as a normal JavaScript array!
      console.log(this.allUsers);
    }]);



* subscribe - Use this to subscribe to data from your Meteor server side - you need to provide the name of the subscription and a function that will return an array with the subscription arguments (as defined in the publication in the server side).

Using `subscribe` you can register to data, it wraps `Meteor.subscribe`, but it also makes sure to make you view reactive when there are changes.

This is an example for registering to subscription named `relevant_users` with some parameters:

    angular.module('myApp', []).controller('MyCtrl', ['$scope', '$reactive', function($scope, $reactive)
    {
      let reactiveContext = $reactive(this).attach($scope);

      reactiveContext.subscribe('relevant_users', () => {
        return [
          'example' // This is an arguments that will be used in the subscription
        ]
      });
    }]);


Combining both of the feature we just explained, we can create a reactive view with search, that updates the subscription:

    angular.module('myApp', []).controller('MyCtrl', ['$scope', '$reactive', function($scope, $reactive)
    {
      let reactiveContext = $reactive(this).attach($scope);

      reactiveContext.helpers({
        searchText: '',
        users: function() {
          return Users.find({});
        }
      });

      reactiveContext.subscribe('relevant_users', () => {
        return [
          this.searchText
        ];
      });
    }]);

And in our view, we can use `ng-repeat` to display the list of the users, and use `ng-model` to update the search text:

    <div ng-controller="MyCtrl as vm">
      Filter users: <input type="text" ng-model="vm.searchText" /> <br /><br />
      Relevant Users:
      <ul>
        <li ng-repeat="user in vm.users">{ { user.name } }</li>
      </ul>
    </div>

And this is our simple publication in the server side:

    if (Meteor.isServer) {
      Meteor.publish("relevant_users", function (userName) {
        return Users.find({ name: userName});
      });
    }
