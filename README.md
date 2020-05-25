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


Modifier le fichier : src/index.html

```
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Mon application Angular</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <app-root></app-root>
</body>
</html>
```

Modifier le fichier : src/app/app.component.html

```
<nav class="navbar navbar-expand-lg navbar-light bg-light">
    <!-- Links -->
    <ul class="navbar-nav">
        <li class="nav-item">
            <a routerLink="employees" class="nav-link" routerLinkActive="active">Employee List</a>
        </li>
        <li class="nav-item">
            <a routerLink="add" class="nav-link" routerLinkActive="active">Add Employee</a>
        </li>
    </ul>
</nav>
<div class="container">
    <br>
    <h2>{{title}}</h2>
    <hr>
    <router-outlet></router-outlet> 
</div>

<footer class="footer">
    <div class="container">
        <span>Test Footer</span>
    </div>
</footer>
```

Modifier le fichier : src/app/app.component.ts

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'Angular + Spring Boot CRUD';
}
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

## Les Modules

Activation des modules HTTP et du module pour les formulaires

On modifie : src/app/app.module.ts

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import {HttpClientModule} from '@angular/common/http';
import { FormsModule } from '@angular/forms';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { CreateEmployeeComponent } from './create-employee/create-employee.component';
import { EmployeeDetailsComponent } from './employee-details/employee-details.component';
import { EmployeeListComponent } from './employee-list/employee-list.component';
import { UpdateEmployeeComponent } from './update-employee/update-employee.component';

@NgModule({
  declarations: [
    AppComponent,
    CreateEmployeeComponent,
    EmployeeDetailsComponent,
    EmployeeListComponent,
    UpdateEmployeeComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    FormsModule,
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## Le service



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

## Liste des employés

On modifie src/app/employee-list/employee-list.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { Employee } from '../model/employee';
import { EmployeeService } from '../service/employee.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-employee-list',
  templateUrl: './employee-list.component.html',
  styleUrls: ['./employee-list.component.css']
})
export class EmployeeListComponent implements OnInit {

  employees: Array<Employee>;

  constructor(private employeeService: EmployeeService, private router: Router) { }

  ngOnInit(): void {
    this.reloadData();
  }

  reloadData() {
    this.employeeService.getAll()
      .subscribe(
        values => {
          this.employees = values;
        },
        errors => {
          console.log("GET call in error", errors);
        },
        () => {
          console.log("The GET observable is now completed.");
        });
  }


  deleteEmployee(id: number) {
    this.employeeService.deleteEmployee(id)
      .subscribe(
        data => {
          console.log("Delete Employee is Ok :", data);
          this.reloadData();
        },
        error => console.log("Error Deleting Employee :",error),
        () => {
          console.log("Delete operation is now completed.");
        });
  }

  employeeDetails(id: number){
    this.router.navigate(['details', id]);
  }

}
```



```
<div class="card">
    <div class="card-header">
        <h2>Employee List</h2>
    </div>
    <div class="card-body">
        <table class="table table-striped">
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Firstname</th>
                    <th>Lastname</th>
                    <th>Email</th>
                    <th>Active</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr *ngFor="let employee of employees">
                    <td>{{employee.id}}</td>
                    <td>{{employee.firstName}}</td>
                    <td>{{employee.lastName}}</td>
                    <td>{{employee.email}}</td>
                    <td>{{employee.active}}</td>
                    <td><button (click)="deleteEmployee(employee.id)"   class="btn btn-danger">Delete</button>
                        <button (click)=""                              class="btn btn-warning" style="margin-left: 10px">Update</button>
                        <button (click)="employeeDetails(employee.id)"  class="btn btn-info"    style="margin-left: 10px">Details</button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</div>
```
