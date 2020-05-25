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

## Avant de commencer

Vider le fichier : src/app/app.component.html

```
<router-outlet></router-outlet>
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

Modification de src/app/model/employee.ts

```typescript

export class Employee {
    id?: number;
    firstName: string;
    lastName: string;
    email: string;
    active: boolean;
}

```

## Le service

Activation du module pour le client Http

On modifie : src/app/app.module.ts

```typescript
import {HttpClientModule} from '@angular/common/http';

@NgModule({
    declarations: [
        AppComponent
    ],
    imports: [
        BrowserModule,
        HttpClientModule
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule {
}
```

On modifie : src/app/service/employee.service.ts

```typescript
import { Injectable } from '@angular/core';

import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Employee } from '../model/employee';


@Injectable({
  providedIn: 'root'
})
export class EmployeeService {

  private baseUrl = 'http://localhost:8080/api/v1/employees';

  constructor(private http: HttpClient) { }


  getAll(): Observable<Employee[]> {
    return this.http.get<Employee[]>(`${this.baseUrl}`);
  }

  getEmployee(id: number): Observable<Employee> {
    return this.http.get<Employee>(`${this.baseUrl}/${id}`);
  }

  createEmployee(employee: Employee): Observable<Employee> {
    return this.http.post<Employee>(`${this.baseUrl}`, employee);
  }

  updateEmployee(id: number, value: any): Observable<Employee> {
    return this.http.put<Employee>(`${this.baseUrl}/${id}`, value);
  }

  deleteEmployee(id: number): Observable<any> {
    // Renvoi une map en mode texte  Ex: ("deleted", Boolean.TRUE);
    return this.http.delete(`${this.baseUrl}/${id}`, { responseType: 'text' });
  }


}
```

## Le routage

On modifie : src/app/app-routing.module.ts

```typescript

import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { EmployeeListComponent } from './employee-list/employee-list.component';
import { CreateEmployeeComponent } from './create-employee/create-employee.component';
import { UpdateEmployeeComponent } from './update-employee/update-employee.component';
import { EmployeeDetailsComponent } from './employee-details/employee-details.component';


const routes: Routes = [
  { path: '', redirectTo: 'employees', pathMatch: 'full' },
  { path: 'employees', component: EmployeeListComponent },
  { path: 'add', component: CreateEmployeeComponent },
  { path: 'update/:id', component: UpdateEmployeeComponent },
  { path: 'details/:id', component: EmployeeDetailsComponent },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }

```

## Partie Lister les employés

On modifie src/app/employee-list/employee-list.component.ts
