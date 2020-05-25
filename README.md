# Mémo Angular

## Références documentaires

https://www.javaguides.net/p/angular-9-tutorial.html

## Setup Angular

`npm install -g @angular/cli`

`ng --version`

`ng new my-app`

## Démarrage d'une appli Angular

`cd my-app`

`ng serve --open`

## Intégrer Bootstrap et JQuery

`npm install bootstrap jquery --save`

Dans angular.json ajouter :

```javascript

...
 
"styles": [
  "src/styles.css",
  "node_modules/bootstrap/dist/css/bootstrap.min.css"
],
"scripts": [
  "node_modules/jquery/dist/jquery.min.js",
  "node_modules/bootstrap/dist/js/bootstrap.min.js"
]
 
...

```


## Liste des Composants, Services, et Modules

Components
* create-employee
* employee-list
* employee-details
* update-employee

Services
* employee.service.ts - Service for Http Client methods

Modules
* FormsModule
* HttpClientModule
* AppRoutingModule

Employee Class (Typescript class)
* employee.ts: class Employee (id, firstName, lastName, emailId)

## Angular CLI

- ng g s employee
- ng g c create-employee
- ng g c employee-details
- ng g c employee-list
- ng g c update-employee
- ng g class Employee 
