## Dart, it's Angular too!

[Pierre Reliquet](http://github.com/PierreReliquet) - @preliquet

Consultant [@ZenikaIT](http://zenika.com/)



## Agenda

1. Dart, a quick tour
1. AngularDart
1. What's in the future?



## Dart
<figure>
  <img src="img/dart.png" />
</figure>



## What ?

* `Web` programming language
* `Client` and `server` side
* Open source and `standardized` (ECMA-408)
* One of the `20 most popular programming language` according to Tiobe index
* Active community
* Developped and supported by `Google`



## Why ? 

<figure>
   <img src="img/no_javascript.gif"/>
</figure>



## Dart goals

* Create a `whole ecosystem`:
 * Package manager : pub
 * Dedicated IDE or plugins
 * Dartium & dart2js
* Easy to learn
* High productivity & [performances](https://www.youtube.com/watch?v=FqsU3TbUw_s)



Dart targets the `modern` web
<figure>
   <img src="img/browsers.png"/>
</figure>



## Dart is familiar

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



## Dart is not JavaScript

No `undefined` just `null`

Only `true` is truthy

Real lexical scoping



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



## Dart's Future are great

```Dart
asyncOp1()
 .then((var firstData) => asyncOp2(firstData))
 .then((var secondData) => asyncOp3(secondData))
 .then((var thirdData) => treatData(thirdData))
 .catchError((e) => print(e));
```


