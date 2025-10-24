---
title: Angular Services
sidebar_position: 3
---

# Angular Services — `HttpClient` and Data Fetching

In Angular, **services** are used to organize reusable logic that can be shared across components.  
One of the most common use cases is **fetching data** from an API using the **HttpClient**.

---

## Service

A **service** is a class with a specific purpose — for example, handling API calls, managing state, or providing utilities.

Angular services are typically:

- Decorated with `@Injectable()`
- Registered in the **dependency injection system**
- Injected into components or other services

---

## Example: Fetch Users from an API

Let’s build a simple `UserService` that fetches user data from a REST API.

```ts title="user.service.ts"
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { User } from './user.model';

@Injectable({ providedIn: 'root' })
export class UserService {
  private apiUrl = 'https://jsonplaceholder.typicode.com/users';

  constructor(private http: HttpClient) { }

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  getUsersButWireCut(): Observable<User[]> {
    return this.http.get<User[]>('https://127.0.0.1/unknowend');
  }
}
```

What’s happening here?

HttpClient is injected into the service.

Each method returns an Observable of a typed response (User[]).

Observable makes it easy to handle asynchronous responses.
---
### Consuming the Service in a Component

Now we’ll create a component that uses this service to load users and handle errors.
---
```component 
import { Component } from '@angular/core';
import { JsonPipe } from '@angular/common';
import { Observable } from 'rxjs';
import { UserService } from './user.service';
import { User } from './user.model';

@Component({
  selector: 'app-user-list',
  imports: [JsonPipe],
  templateUrl: './user-list.component.html',
  styleUrl: './user-list.component.scss'
})
export class UserListComponent {
  users: User[] = [];
  error: any = null;

  constructor(private userService: UserService) { }

  loadFrom(source: Observable<User[]>) {
    this.users = [];
    this.error = null;

    source.subscribe({
      next: (data) => this.users = data,
      error: (err) => this.error = err
    });
  }

  load() {
    this.loadFrom(this.userService.getUsers());
  }

  loadFail() {
    this.loadFrom(this.userService.getUsersButWireCut());
  }
}
```

``` Template
<ul>
  @for(user of users; track user.id) {
    <li>
      <strong>{{ user.name }}</strong> — {{ user.email }}
    </li>
  }
</ul>

@if(error) {
  {{ error | json }}
}

<button (click)="load()">Load</button>
<button (click)="loadFail()">Load Fail</button>
```
---
### Explanation:

load() calls the real API and displays the user list.

loadFail() simulates a failed request (404 or connection error).

The template shows both successful data and any error message.

in code you will see it's have .subscribe for service HttpClient returns Observables (from RxJS), not Promises.

this will explain later in RxJs section
