# AngularDart

Le portage d’Angular dans l’écosystème Dart vient tout juste d’atteindre la version 1.0. 
Cette nouvelle version garantit une stabilisation des APIs et permet d’envisager une mise en production du framework. 
Certains diront que Dart n’a pas d’avenir car étant un énième meta-langage compilant vers du JavaScript, mais rappelons le Dart, aujourd’hui, c’est avant tout : 
* un standard: l’ECMA-408. Dart a été standardisé en Juin dernier par l’ECMA. Pour rappel l’ECMA ou European Computer Manufacturers Association est l’organisme ayant standardisé, il y a plus de 15 ans, JavaScript
* un des 20 langages les plus populaires selon le Tiobe index qui rappelons le est un index classant les langages de programmations les plus en vogues en se basant sur le nombre de pages internet répondant à une requête d’un moteur de recherche

Cependant AngularDart devrait attirer bien plus de développeurs que ceux intéréssés par Dart; 
en effet, c’est pour le développeur JavaScript habitué à manipuler AngularJS une façon de s’intéresser au futur, de découvrir les nouvelles conventions et de s’habituer à la nouvelle architecture d’une application web construite avec un Angular tout neuf qui se rapproche pour certains de manière très proche d’AngularJS 2.0.

## Historique
AngularDart a été développé sous l’impulsion d’une équipe de Google ayant pour objectif initial d’enrichir le CRM interne de Google. 
Ce CRM interne était codé en Java / GWT et plusieurs essais réutilisant le code legacy avaient déjà été réalisé  mais tous avaient échoué. 
Lors de JFokus, Seth Ladd qualifiera ce projet de “widow maker project”  - un projet qui fait des veuves. 
L’équipe Green Tea, forte de ses 12 développeurs, décida donc de ne pas tenir compte de la legacy et de repartir “from scratch” en recodant l’application en Dart, 
mais profitant de l’effervescence autour d’AngularJS, les développeurs décidèrent d’apporter la puissance d’AngularJS dans 
l’écosystème Dart, c’est ainsi que débuta le projet sous l’égide de 26 développeurs disséminés au sein de trois équipes : 
* Green tea 
* Angular core
* Dart core
A titre d’anecdote, le projet de réécriture du CRM interne fût un succès et l’application fût livré en six mois.

## AngularDart

Si je devais résumer AngularDart en une phrase cela serait : 
“Un framework regroupant toute la puissance et la philosophie d’Angular, les derniers standards web et codé avec un *
langage moderne et structuré fonctionnant sur une VM offrant d’intéressantes possibilités”.

En réalité AngularDart n'est pas un simple portage mais une `réécriture complète` d'AngularJS dont la méthodologie choisi est la suivante : 

1. Réécrire tous les tests d'AngularJS en Dart
2. Réécrire le cœur d'AngularDart from scratch
3. S'assurer que tous les tests passent pour garantir le côté iso-fonctionnel des deux versions

Cela permet notamment deux choses : 
* Ne pas reproduire les erreurs que l'on a pu faire dans la première écriture du framework. En effet, soyons honnête, toute personne réécrivant un algorithme déjà écrit le réécrira plus vite et avec une meilleure qualité.
* Profiter des possibilités offertes par Dart

### Les annotations

Un des exemples les plus flagrants concernant l'utilisation d'une possibilité offerte par Dart est, sans conteste, l'utilisation d'annotations absentes en JavaScript.
Les annotations permettent à AngularDart de standardiser au maximum les APIs afin de simplifier l'utilisation et la prise en main du framework sous Dart.
Un exemple d'utilisation serait de déclarer un service devant être injectable dans l'application, pour cela il suffit d'annoter la classe avec `@Injectable` et le tour est joué : 
```Dart
@Injectable()
class Contacts {
  List<Contact> contacts = [ new Contact(...), new Contact()];
}

``` 


### Une nouvelle injection de dépendance
Tout développeur ayant utilisé AngularJS sait qu'une partie de la puissance d'Angular provient de son injection de dépendances. 

Cependant, l'injection de dépendances de la version JavaScript présente un inconvénient majeur, elle est basé sur des chaînes de caractères et peut donc poser de nombreux problèmes lors des étapes de minifications aujourd'hui répandues pour la distribution d'une application web.
Le nouveau né de la famille Angular ne reproduira pas l'erreur de son parent et se base désormais sur les types pour l'injection de dépendances.

En reprenant le service précédemment déclaré, on peut facilement l'injecter dans le constructeur pour le rendre accessible et ainsi récupérer la liste des contacts: 
```Dart
@Component(...)
class VCardList  {
  Contacts contactsSvc;
  List<Contact> contacts;
 
  VCardList(this.contactsSvc) {
    contacts = contactsSvc.contacts;
  }
}

```

### Au revoir Directives, Controllers et bienvenu Decorators, Components

Au sein d'AngularJS, il y a une franche séparation entre : 

* La manipulation du DOM assurée par les `directives`
* La logique de l'application assurée par les `controllers`

Le premier groupe a connaissance du DOM et est garant de son comportement alors que le second groupe ne doit surtout pas en être conscient, il manipule des données qui seront rendues via le mécanismes de 2-way binding.

Cependant dans l'écosystème AngularDart, les contrôleurs disparaissent et les directives sont scindées en deux sous-catégories: 

* Les `decorators` dont le rôle est d'enrichir les éléments existant en leur ajoutant du comportement. Ces éléments existants peuvent soit faire partie de la norme HTML, soit être des `components` externes.
* Les `components` qui ne sont rien de moins qu'un sous-ensemble des web-components et dont le rôle est de permettre d'étendre le HTML afin de lui ajouter nos propres balises.

La disparition des contrôleurs, quant à elle, correspond au rapprochement avec les web-components. Désormais les expressions sont évalués dans le contexte du `Component` englobant.

#### Decorators

La syntaxe des `decorators` est une nouvelle fois basée sur les annotations et la déclaration d'un tooltip customisé pourrait se faire de la façon suivante : 

```Dart
@Decorator(selector: '[tooltip]')
class Tooltip {
  
  Element _elm; // DOM element injected
  
  @NgOneWay('tooltip')
  Contact tooltip; // One way binding of the contact
  
  Tooltip(this._elm) {
    this._elm.onMouseEnter.listen((MouseEvent e) {
      // Using string interpolation to ease the writing
      DivElement div = new Element.html("<div id='tooltip'>${tooltip.address} - ${tooltip.phone}</div>");
      
      // Using the Dart method cascades to avoid repeating div.style.
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
    
    window.onHashChange.listen((e) {
      var tooltip = document.querySelector('#tooltip');
      if (tooltip != null) {
        tooltip.remove();
      }
    });
  }
}
```
On peut donc constater : 
* qu'un décorateur est, avant tout, une classe annotée avec la meta-information `@Decorator`
* qu'un décorateur est aussi caractérisé par un attribut `selector` qui est un sélecteur CSS indiquant la condition d'activation de celui-ci, dans notre cas nous indiquons que le tooltip sera actif pour tous les éléments possédant l'attribut `tooltip`. 
* qu'il est possible de récupérer des variables du scope englobant via des annotations `@NgOneWay`, `@NgTwoWay` ou encore par l'attribut [map](https://docs.angulardart.org/#angular/angular-core-annotation.Directive@id_map) de la meta-information `@Decorator`.

#### Component

Les `Components` sont la brique servant à étendre le HTML, il n'est désormais plus nécessaire de se rappeler quelles sont les valeurs possibles pour l'attribut `restrict` d'une directive AngularJS, les composants sont limités aux éléments ('E'). 

Comme je l'ai indiqué précédemment, un des objectifs d'AngularDart est d'uniformiser les APIs pour cela nous allons retrouver pour les `Components` l'utilisation des annotations comme précédemment pour les `Decorators`.

La déclaration d'un composant se fait de la manière suivante : 
```Dart
@Component(
    selector: 'vcard',
    templateUrl: 'components/vcard/vcard_component.html',
    cssUrl: 'components/vcard/vcard_component.css',
    map: const {
      'contact': '<=>contact'
    }
)
class VCard implements ScopeAware {
  Scope scope;
  Contact contact;
}
```
On peut constater la présence d'un certains nombre de paramètres : 
* `selector` le sélecteur CSS indiquant que le tag créé s'appellera vcard
* `templateUrl` l'URL du template correspondant au composant
* `cssUrl` l'URL du fichier CSS appliqué à l'élément
* `map` définit comment les variables seront "bindées" entre le composant vcard et le composant englobant.

Attention, depuis la version 1.0 les URL `templateUrl` et `cssUrl` prennent en paramètre des chemins relatifs par rapport à la librairie déclarant les composants.

Une fois le composant définit, il reste à déclarer le template pour cela rien de spécifique juste un simple template HTML : 
```HTML
<div class="contact-card">
  <div class="contact-card-inner">
    <h4>{{contact.firstName}} {{contact.lastName | uppercase}}</h4>

    <div class="contact-address">{{contact.address}}</div>
    <div class="contact-phone">{{contact.phone}}</div>
  </div>
</div>
```

Voilà, notre premier composant est défini et nous n'avons plus qu'à l'utiliser : 

```HTML
<vcard contact="contact" class="span4" ng-repeat="contact in contacts"></vcard>
```

Une fois utilisé ce composant va se baser sur le shadow DOM et, par conséquent, dans les DevTools de notre navigateur nous ne verrons que l'élément `vcard` comme s'il faisait partie du standard HTML et nous ne verrons nullement l'imbrication de `div` définit dans notre template. Cela s'oppose à AngularJS qui fonctionnerait par insertion du template au sein de notre `vcard`.

Dans le cas où le support d'Internet Explorer 9 serait requis, il sera nécessaire de désactiver l'utilisation du shadow DOM avec la propriété `useShadowDom = false` au sein de la déclaration du `@Component`.

### Plus de Filters mais des Formatters

La documentation officiel d'AngularJS indique qu'un filtre sert à formater le résultat d'une expression à des fins d'affichages (["A filter formats the value of an expression for display to the user"](https://docs.angularjs.org/guide/filter)). Cette définition pour le moins explicite permet de comprendre la raison de ce renommage de `Filter` à `Formatter`. La collision de nommage entre les `filters` et le filtre `filter` peut aussi être vu comme une autre raison permettant d'expliquer cette décision.

La déclaration d'un formatter se fait par l'utilisation d'un "callable object" déclaré de la manière suivante : 
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
On peut ainsi noter que contrairement aux décorateurs et composants la déclaration d'un formatter n'utilise pas de `selector` mais un `name` car le formatter n'a pas vocation à apparaître dans l'arbre DOM pour activer un quelconque comportement. Le `Formatter` est une fonction qui sera appelée pour formater des données lors de l'affichage.

## Zones : la fin du $apply

Une zone est un concept propre à Dart défini comme l'étendue d'un appel de méthode incluant les exécutions synchrones et asynchrones - "A zone represents the asynchronous dynamic extent of a call". L'intérêt d'un tel concept est que tout appel de méthode se retrouve encapsulé dans une zone et cela va permettre d'annihiler le besoin d'utiliser `$scope.$apply` même lorsque l'on intègre une librairie externe.

```JavaScript
controller("FooCtrl", function($scope){
  $scope.square = 2;

  setInterval(function(){
    $scope.$apply(function(){
      $scope.square = $scope.square * $scope.square;
    });
  }, 2500);
})
```

Sur ce bout de code JavaScript, on peut constater qu'il est nécessaire d'encapsuler le code situé à l'intérieur du setInterval dans un $scope.$apply. En effet, la fonction native setInterval va être déclenchée hors du scope d'Angular et le système de binding ne sera donc pas alerté qu'une modification du modèle a eu lieu. Pour cette raison, nous utilisons le $scope.$apply pour être sûr qu'Angular sera bien informé de la mise à jour de la valeur et mettra à jour l'UI en conséquence.

La version Dart maintenant entraîne quelques changements, tout d'abord nous passons d'un contrôleur à un composant mais la logique reste la même : 
```Dart
@Component(
    selector: "square",
    template: '{{square}}'
)
class Square {
  num square = 0;

  Count(){
    new Timer.periodic(new Duration(seconds: 1), (_) { 
      square = square * square
    });
  }
}
```
Grâce au fonctionnement des zones, nous n'avons pas besoin dans la version Dart de nous poser la question : "est-ce que la modification a eu lieu au sein du scope d'Angular ou non ?", les zones garantissent qu'à la fin de l'exécution d'une méthode le $digest sera déclenché et la vue sera bien rafraichie.

## Conclusion 

Pour résumer, AngularDart prend le parti de s'éloigner de son parent AngularJS afin d'essayer d'uniformiser les APIs et de se rapprocher des nouveaux standards web.

Mon avis est aussi que les développeurs habitués au back-end seront très vite opérationnel via l'utilisation de classes, d'une injection de dépendances basée sur les types et l'utilisation d'annotations.

Enfin, AngularDart nous simplifie la vie par l'utilisation des zones afin d'éviter d'avoir à se soucier des $apply. Cela a, cependant, un coût, la suppression des controllers nous amène à ré-imaginer la structuration de nos applications web se basant sur un sous-ensemble des web-components, travail que certains auront déjà réalisé via l'utilisation de polymer.
