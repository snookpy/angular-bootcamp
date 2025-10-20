---
sidebar_position: 1
---

# Dependency Injection in Angular
"DI" is a design pattern and mechanism for creating and delivering some parts of an app to other parts of an app that require them.

Mostly used to provide services to components or other services.
```typescript
// means this inject at root level by default
@Injectable({
  providedIn: 'root' // tree-shakable providers known for compiler/bundler
})
class ApiService {}


// Inject at module level, all components and services under this module can use it
@NgModule({
  providers: [ApiService] // this inject at module level and its children
  Declarations: [...],
  Imports: [...],
})
class AppModule {}

// At component level
@Component({
  selector: 'share-com',
  template: '...',
  providers: [ApiService] // this inject at component level and its children
})
class ShareCom {}


// Usage in component
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  // new version
  // import { inject } from "@angular/core";
  // const apiService = inject(ApiService);

  constructor(private apiService: ApiService) {} // inject service here
}


```