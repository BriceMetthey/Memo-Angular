# Mémo Angular

* [Exemple de CRUD](#exemple-de-crud)


## Exemple de CRUD

### Le back-end Java

Rappel : `mvn spring-boot:run` 

Fichier : Employee.java

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;
    
    @Column(nullable = false)
    private String firstName;

    @Column(nullable = false)
    private String lastName;

    @Column(nullable = false)
    private String email;

    @Column(nullable = false)
    private boolean active;
 
    public Employee() { }
 
    public Employee(String firstName, String lastName, String email, boolean active) {
         this.firstName = firstName;
         this.lastName = lastName;
         this.email = email;
         this.active = active;
    }

```

Fichier : EmployeeRepository.java

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import fr.bme.demo.model.Employee;

@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long>{
    
}
```

Fichier : EmployeeController.java

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.validation.Valid;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import fr.bme.demo.model.Employee;
import fr.bme.demo.repository.EmployeeRepository;

@RestController 
@CrossOrigin(origins = "http://localhost:4200")
@RequestMapping("/api/v1")
public class EmployeeController {
    
    @Autowired
    private EmployeeRepository employeeRepository;

    @GetMapping("/employees")
    public List<Employee> getAllEmployees() {
        return employeeRepository.findAll();
    }

    @GetMapping("/employees/{id}")
    public ResponseEntity<Employee> getEmployeeById(@PathVariable(value = "id") Long employeeId)
        throws ResourceNotFoundException {
        
        Employee employee = employeeRepository.findById(employeeId)
          .orElseThrow(() -> new ResourceNotFoundException("Employee not found for this id :: " + employeeId));
        
          return ResponseEntity.ok().body(employee);
    }
    
    @PostMapping("/employees")
    public Employee createEmployee(@Valid @RequestBody Employee employee) {
        return employeeRepository.save(employee);
    }

    @PutMapping("/employees/{id}")
    public ResponseEntity<Employee> updateEmployee(@PathVariable(value = "id") Long employeeId,
         @Valid @RequestBody Employee employeeDetails) throws ResourceNotFoundException {
        
        Employee employee = employeeRepository.findById(employeeId)
        .orElseThrow(() -> new ResourceNotFoundException("Employee not found for this id :: " + employeeId));

        employee.setActive(employeeDetails.getActive());
        employee.setEmail(employeeDetails.getEmail());
        employee.setLastName(employeeDetails.getLastName());
        employee.setFirstName(employeeDetails.getFirstName());
        
        final Employee updatedEmployee = employeeRepository.save(employee);
        
        return ResponseEntity.ok(updatedEmployee);
    }

    @DeleteMapping("/employees/{id}")
    public Map<String, Boolean> deleteEmployee(@PathVariable(value = "id") Long employeeId)
         throws ResourceNotFoundException {
        
        Employee employee = employeeRepository.findById(employeeId)
       .orElseThrow(() -> new ResourceNotFoundException("Employee not found for this id :: " + employeeId));

        employeeRepository.delete(employee);
        Map<String, Boolean> response = new HashMap<>();
        response.put("deleted", Boolean.TRUE);
        
        return response;
    }
}
```

Fichier : ErrorDetails.java

```java
import java.util.Date;

public class ErrorDetails {
    private Date timestamp;
    private String message;
    private String details;

    public ErrorDetails(Date timestamp, String message, String details) {
        super();
        this.timestamp = timestamp;
        this.message = message;
        this.details = details;
    }

    public Date getTimestamp() {
        return timestamp;
    }

    public String getMessage() {
         return message;
    }

    public String getDetails() {
         return details;
    }
}

```

Fichier : GlobalExceptionHandler.java

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;

@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<?> resourceNotFoundException(ResourceNotFoundException ex, WebRequest request) {
         ErrorDetails errorDetails = new ErrorDetails(new Date(), ex.getMessage(), request.getDescription(false));
         return new ResponseEntity<>(errorDetails, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> globleExcpetionHandler(Exception ex, WebRequest request) {
        ErrorDetails errorDetails = new ErrorDetails(new Date(), ex.getMessage(), request.getDescription(false));
        return new ResponseEntity<>(errorDetails, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}

```

Fichier : ResourceNotFoundException.java

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends Exception{

    private static final long serialVersionUID = 1L;

    public ResourceNotFoundException(String message){
        super(message);
    }
}
```

### Setup Angular

`npm install -g @angular/cli`

`ng --version`

`ng new my-app`

### Démarrage d'une appli Angular

`cd my-app`

`ng serve --open`

### Intégrer Bootstrap et JQuery

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

### Avant de commencer


Le fichier : src/index.html

```html
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

Le fichier : src/app/app.component.html

```html
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


<nav class="navbar fixed-bottom navbar-expand-sm navbar-dark bg-dark">
    <a class="navbar-brand" href="#">Angular app</a>
</nav>
```

Le fichier : src/app/app.component.ts

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'App Angular';
}
```

### Liste des Composants, Services, et Modules

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

### Angular CLI

`cd src/app`

- `ng generate service service/employee`
- `ng generate component create-employee`
- `ng generate component employee-details`
- `ng generate component employee-list`
- `ng generate component update-employee`
- `ng generate class model/Employee`

### Le modèle

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

### Les Modules

Activation des modules HTTP et du module pour les formulaires.
Ici les formulaires sont de type : Template-driven forms.

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

### Le service



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

### Le routage

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

### Liste des éléments

Fichier : src/app/employee-list/employee-list.component.ts

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

  updateEmployee(id: number) {
    this.router.navigate(['update', id]);
  }

}
```

Fichier :  src/app/employee-list/employee-list.component.html

```html
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
                    <td>
                        <div *ngIf="employee.active; then thenBlock else elseBlock"></div>
                        <ng-template #thenBlock><button type="button" class="btn btn-outline-success btn-sm">ON</button></ng-template>
                        <ng-template #elseBlock><button type="button" class="btn btn-outline-danger btn-sm">OFF</button></ng-template>
                    </td>
                    <td><button (click)="deleteEmployee(employee.id)"   class="btn btn-danger">Delete</button>
                        <button (click)="updateEmployee(employee.id)"   class="btn btn-warning" style="margin-left: 10px">Update</button>
                        <button (click)="employeeDetails(employee.id)"  class="btn btn-info"    style="margin-left: 10px">Details</button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</div>
```

### Style général

Le fichier : src/styles.css

```css
 
.ng-valid[required], .ng-valid.required  {
    border-left: 5px solid #42A948;
}
  
.ng-invalid:not(form)  {
    border-left: 5px solid #a94442;   
}
```

### Formulaire de création

Fichier : src/app/create-employee/create-employee.component.ts

```typescript

import { Component, OnInit } from '@angular/core';
import { FormBuilder } from '@angular/forms';

import { EmployeeService } from '../service/employee.service';
import { Employee } from '../model/employee';
import { Router } from '@angular/router';

@Component({
  selector: 'app-create-employee',
  templateUrl: './create-employee.component.html',
  styleUrls: ['./create-employee.component.css']
})
export class CreateEmployeeComponent implements OnInit {

  model: Employee;
  submitted = false;

  constructor(
    private employeeService: EmployeeService,
    private router: Router
  ) {   }

  ngOnInit() {
    this.model = new Employee();
    this.model.active = true; // Radio button pre-selectinonné
  }

 
  onSubmit() { 
    this.submitted = true;
    
    this.employeeService.createEmployee(this.model).subscribe(
      data => console.log(data),
      error => console.log(error)
    );
  }

  gotoList() {
    this.router.navigate(['/employees']);
  }

  // C'est un getter. On l'appel comme une propriété.
  get diagnostic() { 
    return JSON.stringify(this.model);
  }

}


```

Fichier : src/app/create-employee/create-employee.component.html

```html

<div class="card">
  <div class="card-header">
    <h2>Create new Employee</h2>
  </div>
  <div class="card-body">




    <div [hidden]="submitted">

      <p>
        Diagnostique en temps réel (Option1) :
        {{diagnostic}}
      </p>

      <form (ngSubmit)="onSubmit()" #employeeForm="ngForm">

        <div class="form-group">

          <label for="labelFirstName">First name</label>

          <input type="text" class="form-control" id="firstName" 
            required
            [(ngModel)]="model.firstName"  name="firstName"
            #firstName="ngModel" #spy>
                                         
            <div [hidden]="firstName.valid || firstName.pristine" class="alert alert-danger">FirstName is required </div>

            *Le modele en cours : {{model.firstName}}
            <br>*Etat du input #spy : {{spy.className}}
        </div>

        <div class="form-group">
          <label for="labelLastName">Last name</label>
          <input type="text" class="form-control" id="lastName"
            required   
            [(ngModel)]="model.lastName" name="lastName"
            #lastName="ngModel">

          <div [hidden]="lastName.valid || lastName.pristine" class="alert alert-danger">LastName is required </div>

          *Le modele en cours : {{model.lastName}}
        </div>

        <div class="form-group">
          <label for="labelEmail">Email</label>
          <input type="email" class="form-control" id="email"
            required
            [(ngModel)]="model.email" name="email"
            placeholder="name@example.com"
            #email="ngModel"> 

            <div [hidden]="email.valid || email.pristine" class="alert alert-danger">Email is required </div>

          *Le modele en cours : {{model.email}}
        </div>

        <div class="form-group">
 
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="active" [(ngModel)]="model.active" [value]="true">
            <label class="form-check-label" for="radio1">Active</label>
          </div>
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="active" [(ngModel)]="model.active" [value]="false">
            <label class="form-check-label" for="radio2">Not Active</label>
          </div>
          <p> *Le modele en cours : {{model.active}}</p>

        </div>

        <button type="submit" class="btn btn-primary" [disabled]="!employeeForm.form.valid">Submit</button>

      </form>
    </div>

    <p>
      Diagnostique en temps réel (Option2) :
      {{ employeeForm.value | json }}
    </p>



    <div [hidden]="!submitted">
      <h2>You submitted the following:</h2>
      <div class="row">
        <div class="col-xs-3">First Name : </div>
        <div class="col-xs-9">{{ model.firstName }}</div>
      </div>
      <div class="row">
        <div class="col-xs-3">Last Name : </div>
        <div class="col-xs-9">{{ model.lastName }}</div>
      </div>
      <div class="row">
        <div class="col-xs-3">Email : </div>
        <div class="col-xs-9">{{ model.email }}</div>
      </div>
      <div class="row">
        <div class="col-xs-3">Active : </div>
        <div class="col-xs-9">{{ model.active }}</div>
      </div>
      <br>
      <button class="btn btn-primary" (click)="submitted=false">Review</button>
      <button class="btn btn-info" (click)="gotoList()" style="margin-left: 10px">List</button>
    </div>

  </div>
</div>

```

### Formulaire de modification

Fichier : 

src/app/update-employee/update-employee.component.ts

```typescript

import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { EmployeeService } from '../service/employee.service';
import { Employee } from '../model/employee';

@Component({
  selector: 'app-update-employee',
  templateUrl: './update-employee.component.html',
  styleUrls: ['./update-employee.component.css']
})
export class UpdateEmployeeComponent implements OnInit {

  id: number;
  model: Employee;

  constructor(
    private route: ActivatedRoute,
    private router: Router,
    private employeeService: EmployeeService) { }

  ngOnInit() {
    this.model = new Employee();

    this.id = this.route.snapshot.params['id'];
    
    this.employeeService.getEmployee(this.id)
      .subscribe(
        data => {
          console.log(data);
          this.model = data;
        }, 
        error => console.log(error)
      );
  }

  updateEmployee() {
        
    this.employeeService.updateEmployee(this.id, this.model)
      .subscribe(
        data => console.log(data),
        error => console.log(error)
      );
    this.model = new Employee();
    this.gotoList();
  }

  onSubmit() {
    this.updateEmployee();    
  }

  gotoList() {
    this.router.navigate(['/employees']);
  }
}


```

Fichier : src/app/update-employee/update-employee.component.html

```html

<div class="card">
    <div class="card-header">
        <h2>Update employee #{{model.id}}</h2>
    </div>
    <div class="card-body">



        <form (ngSubmit)="onSubmit()" #employeeForm="ngForm">


            <div class="form-group">

                <label for="labelFirstName">First name</label>

                <input type="text" class="form-control" id="firstName" required [(ngModel)]="model.firstName"
                    name="firstName" #firstName="ngModel" #spy>

                <div [hidden]="firstName.valid || firstName.pristine" class="alert alert-danger">FirstName is required
                </div>

                *Le modele en cours : {{model.firstName}}
                <br>*Etat du input #spy : {{spy.className}}
            </div>



            <div class="form-group">
                <label for="labelLastName">Last name</label>
                <input type="text" class="form-control" id="lastName" required [(ngModel)]="model.lastName"
                    name="lastName" #lastName="ngModel">

                <div [hidden]="lastName.valid || lastName.pristine" class="alert alert-danger">LastName is required
                </div>

                *Le modele en cours : {{model.lastName}}
            </div>



            <div class="form-group">
                <label for="labelEmail">Email</label>
                <input type="email" class="form-control" id="email" required [(ngModel)]="model.email" name="email"
                    placeholder="name@example.com" #email="ngModel">

                <div [hidden]="email.valid || email.pristine" class="alert alert-danger">Email is required </div>

                *Le modele en cours : {{model.email}}
            </div>


            <div class="form-group">

                <div class="form-check form-check-inline">
                    <input class="form-check-input" type="radio" name="active" [(ngModel)]="model.active"
                        [value]="true">
                    <label class="form-check-label" for="radio1">Active</label>
                </div>
                <div class="form-check form-check-inline">
                    <input class="form-check-input" type="radio" name="active" [(ngModel)]="model.active"
                        [value]="false">
                    <label class="form-check-label" for="radio2">Not Active</label>
                </div>
                <p> *Le modele en cours : {{model.active}}</p>

            </div>

            <button type="submit" class="btn btn-warning" [disabled]="!employeeForm.form.valid">Modify</button>
        </form>


    </div>
</div>

```

### Affichage des détails d'un élément

Fichier : src/app/employee-details/employee-details.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { Employee } from '../model/employee';
import { ActivatedRoute, Router } from '@angular/router';
import { EmployeeService } from '../service/employee.service';

@Component({
  selector: 'app-employee-details',
  templateUrl: './employee-details.component.html',
  styleUrls: ['./employee-details.component.css']
})
export class EmployeeDetailsComponent implements OnInit {

  id: number;
  model: Employee;

  constructor(
    private route: ActivatedRoute,
    private router: Router,
    private employeeService: EmployeeService
    ) { }

  ngOnInit() {
    this.model = new Employee();

    this.id = this.route.snapshot.params['id'];
    
    this.employeeService.getEmployee(this.id)
      .subscribe(
        data => {
          console.log(data)
          this.model = data;
        }, 
      error => console.log(error)
      );
  }

  list(){
    this.router.navigate(['employees']);
  }
}

```

Fichier : src/app/employee-details/employee-details.component.html

```html

<div class="card">
    <div class="card-header">
        <h2>Employee Details</h2>
    </div>
    <div class="card-body">

        
        <div *ngIf="model">
         

            <form>

                <div class="form-group">
        
                  <label for="labelFirstName">First name</label>
        
                  <input type="text" class="form-control" id="firstName" 
                    [(ngModel)]="model.firstName"  name="firstName" disabled>

                </div>
        
                <div class="form-group">
                  <label for="labelLastName">Last name</label>
                  <input type="text" class="form-control" id="lastName"
                    [(ngModel)]="model.lastName" name="lastName" disabled>
        
                </div>
        
                <div class="form-group">
                  <label for="labelEmail">Email</label>
                  <input type="email" class="form-control" id="email"
                    [(ngModel)]="model.email" name="email" disabled> 
        
                </div>
        
                <div class="form-group">
                    <div *ngIf="model.active; then thenBlock else elseBlock"></div>
                    <ng-template #thenBlock><button type="button" class="btn btn-outline-success btn-sm">ON</button></ng-template>
                    <ng-template #elseBlock><button type="button" class="btn btn-outline-danger btn-sm">OFF</button></ng-template>
                </div>
        
                <button class="btn btn-secondary" (click)="list()" >Retrun</button>
        
              </form>
        </div>

       

    </div>
</div>

```
