## AngularDart 1.0 and beyond!

[Pierre Reliquet](http://github.com/PierreReliquet) - @preliquet

Consultant [@ZenikaIT](http://zenika.com/)



## Agenda

1. Dart, a quick tour
1. AngularDart
1. What's in the future?



<figure>
  <img src="img/dart.png" />
</figure>


## What ?

* `Modern web` programming language
* `Client` and `server` side
* Open source and `standardized` (ECMA-408)
* One of the `20 most popular programming language`
* Developped and supported by `Google`
* Ships with a whole `ecosystem`

<figure>
   <img src="img/browsers.png"/>
</figure>


## It is familiar

```Dart
@Injectable
class Conference {
  List<Attendee> attendees = []; // var attendees = [];

  Conference(this.attendees);
  Conference operator +(Conference other) {
    return new Conference(this.attendees + other.attendees);
  }
  void askQuestions({int number :1}) { ... }
  String toString() => "${attendees.length} attendees participated to the conference";
}
```


## "Callback Hell" aka "Pyramid of Doom"

```Dart
myapp.asyncOperation1(function(firstData) {
  // Success
  myapp.asyncOperation2(firstData, function(secondData) {
   // Success
   myapp.asyncOperation3(secondData, function(thirdData) {
     myapp.treatData(thirdData);
   }, function(err) {
     // Error handling
   });
  }, function(err) {
     // Error handling
  });
  }, function(err) {
  // Error Handling
});
```


## Native Future are great

```Dart
asyncOp1()
 .then((var firstData) => asyncOp2(firstData))
 .then((var secondData) => asyncOp3(secondData))
 .then((var thirdData) => treatData(thirdData))
 .catchError((e) => print(e));
```


## Many other great things

* Isolates
* Deferred loading
* Tree shaking



<figure>
	<img src="img/angulardart.png"/>
</figure>


## The birth

* `Enhancing` of Google internal CRM
* Legacy : `Java` / `GWT` application
* 12 engineers
* A "`widow maker project`"
* Decisions made:
  * Rewrite the app in Dart
  * Port `Angular` to `Dart`


## How ?
* Porting the `test harness` and make it pass
* Not alone, they had help :
 * Angular core team
 * Dart core team


## More than a port !

* `Fully` rewritten
* Some standard features
 * Directives
 * Filters
* Disappearance of controllers & scopes
* Plus some `Dartisms`


## Dartisms ?

* Type based [DI](http://www.youtube.com/watch?v=_OGGsf1ZXMs)
* `Annotations`
 * Add meta information to existing class
 * Standardization of declaration
   * Component, Decorator, Injectable Formatter ...
* `Web standards` :
 * Web components => Shadow DOM
* [Zones](http://www.youtube.com/watch?v=3IqtmUscE_U)


## Zone ? What's that beast ?

A zone is a "dynamic extent `including asynchronous callbacks` declared in the zone" which means an execution context.
<figure>
  <img src="img/zones.png"/>  
</figure>

Why is that `so great` ?


## Death of $scope.$apply
```Dart
_zone.onTurnDone = () {
  _pendingAsync.increaseCount();
  apply();
  _pendingAsync.decreaseCount();
  _runAsyncFns();
};
```


## Bootstrapping an application

Include dart file and declare your Angular application :

```HTML
<html ng-app>
  <!-- should be script :-) -->
  <sript src="packages/shadow_dom/shadow_dom.min.js"></sript>
  <sript type="application/dart" src="addressbook.dart"></sript>
  <sript src="packages/browser/dart.js"></sript>
</html>
```


## Bootstrapping an application

```Dart
class AddressBook extends Module {
  AddressBook() {
    bind(Contacts);
    bind(VCard);
    bind(VCardList);
    bind(RouteInitializerFn, toValue: addressBookRouter);
    bind(NgRoutingUsePushState, toValue: new NgRoutingUsePushState.value(false));
  }
}

main() {
  applicationFactory()
      ..addModule(new AddressBook())
      ..run();
}
```


## Services

Creating a service :
```Dart
@Injectable()
class Contacts {
  List<Contact> contacts = [ new Contact(...), new Contact(...)];
}
```
Use it through DI :
```Dart
@Component(/* ... */)
class VCardList {
  Contacts contactsSvc;
  List<Contact> contacts;

  VCardList(this.contactsSvc) {
    contacts = contactsSvc.contacts;
  }
}
```


## Directives

* Directives have been splitted into :
 * Decorator
   * Add behaviour to `existing` HTML element
 * Component
   * Create custom HTML element
   * Subset of web components
   * Create a new `context`


## Components
Declare the VCard :
```Dart
@Component(
    selector: 'vcard',
    templateUrl: 'components/vcard/vcard_component.html',
    cssUrl: 'components/vcard/vcard_component.css'
)
class VCard {
  @NgTwoWay('contact')
  Contact contact;
}
```
Use your component :
```Html
<vcard contact="contact" class="span4" ng-repeat="contact in contactList.contacts"></vcard>
```


## Components
The template looks like :
```Html
<div class="contact-card">
  <div class="contact-card-inner">
    <h4>{{contact.firstName}} {{contact.lastName | uppercase}}</h4>

    <div class="contact-address">{{contact.address}}</div>
    <div class="contact-phone">{{contact.phone}}</div>
  </div>
</div>
```
No more `CSS leak`


## Decorators
Customize existing HTML :
```Dart
@Decorator(selector: '[tooltip]')
class Tooltip {
  Element _elm;

  @NgOneWay('tooltip')
  Contact tooltip;

  Tooltip(this._elm) {
    this._elm.onMouseEnter.listen((MouseEvent e) {
      DivElement div = new Element
        .html("<div id='tooltip'>${tooltip.address} - ${tooltip.phone}</div>");
      div.style
        ..position = 'absolute'
        ..left = '${e.page.x + 10}px'
        ..top = '${e.page.y + 10}px'
        ..padding = '5px'
        ..borderRadius = '5px'
        ..backgroundColor = 'white'
        ..border = 'solid 1px black';
      document.body.append(div);
    });
    this._elm.onMouseLeave.listen((MouseEvent e) {
      var tooltip = document.querySelector('#tooltip');
      if (tooltip != null) {
        tooltip.remove();
      }
    });
  }
}
```


## Filters aka Formatters

Filters were meant to `format data` for display

Declare them is easy :
```Dart
@Formatter(name: "doSearch")
class SearchFilter {
  List<Contact> call(List<Contact> contacts, String search) {
    if (search == null) {
      return contacts;
    }
    return contacts.where(
        (Contact c) => (
            c.firstName.toLowerCase().contains(search.toLowerCase()) ||
            c.lastName.toLowerCase().contains(search.toLowerCase()))
        ).toList();
  }
}
```

&#9888; No `selector` but a `name`

&#9888; The method must be named `call`


## Routing is powerful

Similar to `ui-router` :
```Dart
void addressBookRouter(Router router, RouteViewFactory views) {
  views.configure({
    'list': ngRoute(
      path: '/list',
      view: 'partials/list.html',
      defaultRoute : true),
    'contact': ngRoute(
      path: '/contact/:id',
      mount: {
        'edit': ngRoute(
            path: '/edit',
            viewHtml: '<contact-edit></contact-edit>'
        ),
        'view': ngRoute(
            path: '/view',
            viewHtml: '<contact-view></contact-view>'
        )
      })
  });
}
```


## Angular.dart

* `One` year of development to ship 1.0
* Used by `Google` in production
* `New` but `simpler` way to build web applications
* Helps to get familiar with the `future`



<figure>
  <img src="img/future.jpg" />
</figure>


## Angular 2

* Written in `AtScript`
* Angular.dart was a `Proof Of Concept`
 * `Type` based DI
 * `Annotations`
 * `Zones`
 * `Dirty checking`


## What the heck is AtScript ?
<figure>
  <img src="img/atscript.png" />
</figure>


`AngularDart` is not about `Dart`.

Such as `AngularJS` was not about `JS`.

It is all about making the web a better development platform.



## Thank you!


<figure>
  <img src="img/anyquestions.jpg" />
</figure>
