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
* employee.service.ts (HTTP)

Modules
* FormsModule
* HttpClientModule
* AppRoutingModule

Employee Class
* employee.ts: class Employee

## Angular CLI

`cd src/app`

- `ng generate service service/employee`
- `ng generate component create-employee`
- `ng generate component employee-details`
- `ng generate component employee-list`
- `ng generate component update-employee`
- `ng generate class model/Employee`

## Le modèle

```typescript

export class Employee {
    id: number;
    firstName: string;
    lastName: string;
    email: string;
    active: boolean;
}

```
