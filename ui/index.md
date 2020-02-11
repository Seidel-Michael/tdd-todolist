# Setup

1. Ensure that you have the Angular CLI installed gloablly.

```sh
npm install -g @angular/cli
```

2. Create a new Angular project with the wizzard.

```sh
ng new
```

```
What name would you like to use for the new workspace and initial project? tdd-todolist-frontend
Would you like to add Angular routing? (y/N) y
Which stylesheet format would you like to use? Less
```

3. You can start the initial test app and try it out on: http://localhost:4200/

```sh
cd tdd-todolist-frontend
ng serve
```

4. We start by creating the `TodoItem` interface. As this is only an interface, we won't need any tests. Interfaces in TypeScript won't be present in the compile JavaScript code.

```sh
ng generate interface models/todo-item
```

`models/todo-item.ts`

```ts
export interface TodoItem {
  id?: string;
  title: string;
  state: boolean;
}
```

5. In order to keep the base URL of the backend flexible we're gonna add the `baseApi` to the Angular environment. This is not suitable for production environments because the base URL is compiled into the app and cannot be changed unless the app is rebuilt.

`environments/enviroment.ts`

```ts
export const environment = {
  production: false,
  baseApi: 'http://localhost:8080'
};
```

`environments/enviroment.prod.ts`

```ts
export const environment = {
  production: true,
  baseApi: 'http://localhost:8080'
};
```

6. We start with the `TodoService` that is responsible for talking to our REST API.

```sh
ng generate service services/todo
```

`services/todo.service.ts`

```ts
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { environment } from 'src/environments/environment';
import { TodoItem } from '../models/todo-item';

const API_URL = `${environment.baseApi}/api/v1/todos`;

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  constructor(private http: HttpClient) {}

  public addTodo(item: TodoItem): Observable<void> {
    throw new Error('Not Implemented!');
  }

  public removeTodo(item: TodoItem): Observable<void> {
    throw new Error('Not Implemented!');
  }

  public changeTodoState(item: TodoItem): Observable<void> {
    throw new Error('Not Implemented!');
  }

  public getTodos(): Observable<TodoItem[]> {
    throw new Error('Not Implemented!');
  }
}
```

`services/todo.service.spec.ts`

```ts
import { HttpClientTestingModule, HttpTestingController, TestRequest } from '@angular/common/http/testing';
import { TestBed } from '@angular/core/testing';
import { environment } from 'src/environments/environment';
import { TodoItem } from '../models/todo-item';
import { TodoService } from './todo.service';

describe('TodoServiceService', () => {
  let http: HttpTestingController;
  const API_URL = `${environment.baseApi}/api/v1/todos`;

  beforeEach(() => {
    TestBed.configureTestingModule({ imports: [HttpClientTestingModule] });

    http = TestBed.get(HttpTestingController);
  });

  describe('general', () => {
    it('should be created', () => {
      const service: TodoService = TestBed.get(TodoService);
      expect(service).toBeTruthy();
    });
  });

  describe('addTodo', () => {
    const url = `${API_URL}`;

    it('should call api endpoint correctly', done => {
      const service: TodoService = TestBed.get(TodoService);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };
      service.addTodo(item).subscribe(() => {
        expect().nothing();
        done();
      });

      const req: TestRequest = http.expectOne(url);
      expect(req.request.method).toBe('POST');
      expect(req.request.headers.get('Content-Type')).toBe('application/json');
      req.flush('', { status: 204, statusText: '204' });
      http.verify();
    });

    it('should set correct body', done => {
      const service: TodoService = TestBed.get(TodoService);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };
      service.addTodo(item).subscribe(() => {
        expect().nothing();
        done();
      });

      const req: TestRequest = http.expectOne(url);
      expect(req.request.body).toEqual({ title: 'abc' });
      req.flush('', { status: 204, statusText: '204' });
      http.verify();
    });
  });

  describe('removeTodo', () => {
    const url = `${API_URL}/id`;

    it('should call api endpoint correctly', done => {
      const service: TodoService = TestBed.get(TodoService);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };
      service.removeTodo(item).subscribe(() => {
        expect().nothing();
        done();
      });

      const req: TestRequest = http.expectOne(url);
      expect(req.request.method).toBe('DELETE');
      req.flush('', { status: 204, statusText: '204' });
      http.verify();
    });
  });

  describe('changeTodoState', () => {
    const url = `${API_URL}/id`;

    it('should call api endpoint correctly', done => {
      const service: TodoService = TestBed.get(TodoService);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };
      service.changeTodoState(item).subscribe(() => {
        expect().nothing();
        done();
      });

      const req: TestRequest = http.expectOne(url);
      expect(req.request.method).toBe('PUT');
      expect(req.request.headers.get('Content-Type')).toBe('application/json');
      req.flush('', { status: 204, statusText: '204' });
      http.verify();
    });

    it('should set correct body', done => {
      const service: TodoService = TestBed.get(TodoService);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };
      service.changeTodoState(item).subscribe(() => {
        expect().nothing();
        done();
      });

      const req: TestRequest = http.expectOne(url);
      expect(req.request.body).toEqual({ state: true });
      req.flush('', { status: 204, statusText: '204' });
      http.verify();
    });
  });

  describe('getTodo', () => {
    const url = `${API_URL}`;
    const response = [
      {
        id: '1',
        title: 'Test 1',
        state: true
      },
      {
        id: '2',
        title: 'Test 2',
        state: false
      },
      {
        id: '3',
        title: 'Test 4',
        state: true
      },
      {
        id: '4',
        title: 'Test 5',
        state: true
      }
    ];

    it('should call api endpoint correctly', done => {
      const service: TodoService = TestBed.get(TodoService);
      service.getTodos().subscribe(() => {
        expect().nothing();
        done();
      });

      const req: TestRequest = http.expectOne(url);
      expect(req.request.method).toBe('GET');
      req.flush(response);
      http.verify();
    });

    it('should resolve with response', done => {
      const service: TodoService = TestBed.get(TodoService);
      service.getTodos().subscribe(res => {
        expect(res).toEqual(response);
        done();
      });

      const req: TestRequest = http.expectOne(url);
      req.flush(response);
      http.verify();
    });
  });
});
```

7. The tests schould now be red and we can start with the implemenation.

`services/todo.service.ts`

```ts
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { environment } from 'src/environments/environment';
import { TodoItem } from '../models/todo-item';

const API_URL = `${environment.baseApi}/api/v1/todos`;

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  constructor(private http: HttpClient) {}

  addTodo(item: TodoItem): Observable<void> {
    const headers = new HttpHeaders().set('Content-Type', 'application/json');
    return this.http.post<void>(`${API_URL}`, { title: item.title }, { headers });
  }

  removeTodo(item: TodoItem): Observable<void> {
    return this.http.delete<void>(`${API_URL}/${item.id}`);
  }

  changeTodoState(item: TodoItem): Observable<void> {
    const headers = new HttpHeaders().set('Content-Type', 'application/json');
    return this.http.put<void>(`${API_URL}/${item.id}`, { state: item.state }, { headers });
  }

  getTodos(): Observable<TodoItem[]> {
    return this.http.get<TodoItem[]>(`${API_URL}`);
  }
}
```

8. Now it is time to create the _Components_. We're starting with the `TodoListHeader` component.

```sh
ng generate component components/todo-list-header
```

`components/todo-list-header/todo-list-header.component.ts`

```ts
import { Component, EventEmitter, OnInit, Output } from '@angular/core';
import { TodoItem } from 'src/app/models/todo-item';

@Component({
  selector: 'app-todo-list-header',
  templateUrl: './todo-list-header.component.html',
  styleUrls: ['./todo-list-header.component.less']
})
export class TodoListHeaderComponent implements OnInit {
  title = '';

  @Output()
  add: EventEmitter<TodoItem>;

  constructor() {
    this.add = new EventEmitter();
  }

  ngOnInit() {}

  addTodo() {
    throw new Error('Not Implemented!');
  }
}
```

`components/todo-list-header/todo-list-header.component.spec.ts`

```ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';
import { TodoListHeaderComponent } from './todo-list-header.component';

describe('TodoListHeaderComponent', () => {
  let component: TodoListHeaderComponent;
  let fixture: ComponentFixture<TodoListHeaderComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [TodoListHeaderComponent],
      imports: [FormsModule]
    });

    fixture = TestBed.createComponent(TodoListHeaderComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  describe('general', () => {
    it('should create', () => {
      expect(component).toBeTruthy();
    });
  });

  describe('addTodo', () => {
    it('should emit add event if addTodo is called', () => {
      const emitSpy = spyOn(component.add, 'emit');
      component.title = 'abc';

      component.addTodo();

      expect(emitSpy).toHaveBeenCalledTimes(1);
      expect(emitSpy.calls.mostRecent().args[0].title).toEqual('abc');
    });

    it('should create new newTodo instance after addTodo was called', () => {
      component.title = 'abc';

      component.addTodo();

      expect(component.title).toEqual('');
    });
  });
});
```

9. After verifying that the test are red we can start implementing the component and add the html and less files. We're not gonna test the HTML file. It is possible to check the databinding in unit tests. We do this later with e2e tests.

`components/todo-list-header/todo-list-header.component.ts`

```ts
import { Component, EventEmitter, OnInit, Output } from '@angular/core';
import { TodoItem } from 'src/app/models/todo-item';

@Component({
  selector: 'app-todo-list-header',
  templateUrl: './todo-list-header.component.html',
  styleUrls: ['./todo-list-header.component.less']
})
export class TodoListHeaderComponent implements OnInit {
  title = '';

  @Output()
  add: EventEmitter<TodoItem>;

  constructor() {
    this.add = new EventEmitter();
  }

  ngOnInit() {}

  addTodo() {
    this.add.emit({ title: this.title, state: false });
    this.title = '';
  }
}
```

`components/todo-list-header/todo-list-header.component.html`

```html
<header class="header">
  <h1>Todos</h1>
  <input
    class="new-todo-item"
    placeholder="What needs to be done?"
    autofocus=""
    [(ngModel)]="title"
    (keyup.enter)="addTodo()"
  />
</header>
```

`components/todo-list-header/todo-list-header.component.less`

```less
h1 {
  top: -155px;
  width: 100%;
  font-size: 100px;
  font-weight: 100;
  text-align: center;
  color: rgba(0, 0, 0, 1);
  -webkit-text-rendering: optimizeLegibility;
  -moz-text-rendering: optimizeLegibility;
  text-rendering: optimizeLegibility;
}

.new-todo-item,
.edit {
  position: relative;
  margin: 0;
  width: 100%;
  font-size: 24px;
  font-family: inherit;
  font-weight: inherit;
  line-height: 1.4em;
  border: 0;
  color: inherit;
  padding: 6px;
  border: 1px solid #999;
  box-shadow: inset 0 -1px 5px 0 rgba(0, 0, 0, 0.2);
  box-sizing: border-box;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.new-todo-item {
  padding: 16px 16px 16px 16px;
  border: none;
  background: rgba(0, 0, 0, 0.003);
  box-shadow: inset 0 -2px 1px rgba(0, 0, 0, 0.03);
}
```

10. The next component is the `TodoListItem` component.

```sh
ng generate component components/todo-list-item
```

`components/todo-list-item/todo-list-item.component.ts`

```ts
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
import { TodoItem } from 'src/app/models/todo-item';

@Component({
  selector: 'app-todo-list-item',
  templateUrl: './todo-list-item.component.html',
  styleUrls: ['./todo-list-item.component.less']
})
export class TodoListItemComponent implements OnInit {
  @Input() todoItem: TodoItem;

  @Output()
  remove: EventEmitter<TodoItem>;

  @Output()
  toggleComplete: EventEmitter<TodoItem>;

  constructor() {
    this.remove = new EventEmitter();
    this.toggleComplete = new EventEmitter();
  }

  ngOnInit() {}

  toggleTodoComplete() {
    throw new Error('Not Implemented!');
  }

  removeTodo() {
    throw new Error('Not Implemented!');
  }
}
```

`components/todo-list-item/todo-list-item.component.spec.ts`

```ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';
import { TodoItem } from 'src/app/models/todo-item';
import { TodoListItemComponent } from './todo-list-item.component';

describe('TodoListItemComponent', () => {
  let component: TodoListItemComponent;
  let fixture: ComponentFixture<TodoListItemComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [TodoListItemComponent],
      imports: [FormsModule]
    });

    fixture = TestBed.createComponent(TodoListItemComponent);
    component = fixture.componentInstance;

    component.todoItem = { id: 'id', state: true, title: 'abc' };

    fixture.detectChanges();
  });

  describe('general', () => {
    it('should create', () => {
      expect(component).toBeTruthy();
    });
  });

  describe('removeTodo', () => {
    it('should emit remove event of removeTodo is called', () => {
      const emitSpy = spyOn(component.remove, 'emit');
      const item: TodoItem = { id: 'abc', state: true, title: 'abc' };
      component.todoItem = item;

      component.removeTodo();

      expect(emitSpy).toHaveBeenCalledTimes(1);
      expect(emitSpy).toHaveBeenCalledWith(item);
    });
  });

  describe('toffleTodoComplete', () => {
    it('should emit toggleComplete event of toggleTodoComplete is called', () => {
      const emitSpy = spyOn(component.toggleComplete, 'emit');
      const item: TodoItem = { id: 'abc', state: true, title: 'abc' };
      component.todoItem = item;

      component.toggleTodoComplete();

      expect(emitSpy).toHaveBeenCalledTimes(1);
      expect(emitSpy).toHaveBeenCalledWith(item);
    });
  });
});
```

11. The `TodoListItem` component implementation.

`components/todo-list-item/todo-list-item.component.ts`

```ts
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
import { TodoItem } from 'src/app/models/todo-item';

@Component({
  selector: 'app-todo-list-item',
  templateUrl: './todo-list-item.component.html',
  styleUrls: ['./todo-list-item.component.less']
})
export class TodoListItemComponent implements OnInit {
  @Input() todoItem: TodoItem;

  @Output()
  remove: EventEmitter<TodoItem>;

  @Output()
  toggleComplete: EventEmitter<TodoItem>;

  constructor() {
    this.remove = new EventEmitter();
    this.toggleComplete = new EventEmitter();
  }

  ngOnInit() {}

  toggleTodoComplete() {
    this.toggleComplete.emit(this.todoItem);
  }

  removeTodo() {
    this.remove.emit(this.todoItem);
  }
}
```

`components/todo-list-item/todo-list-item.component.html`

```html
<div class="view" [class.completed]="todoItem.state">
  <input class="toggle" type="checkbox" (change)="toggleTodoComplete()" [(ngModel)]="todoItem.state" />
  <label>{{todoItem.title}}</label>
  <button class="destroy" (click)="removeTodo()">x</button>
</div>
```

`components/todo-list-item/todo-list-item.component.less`

```less
label {
  word-break: break-all;
  line-height: 1.2;
  transition: color 0.4s;
}

.completed label {
  color: #d9d9d9;
  text-decoration: line-through;
}

.toggle {
  width: 30px;
  height: 25px;
  text-align: center;
  border: none; /* Mobile Safari */
}

.toggle:checked:before {
  color: #737373;
}

button {
  margin: 0;
  padding: 0;
  border: 0;
  background: none;
  font-size: 100%;
  vertical-align: baseline;
  font-family: inherit;
  font-weight: inherit;
  color: inherit;
  -webkit-appearance: none;
  appearance: none;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.destroy {
  top: 0;
  right: 10px;
  bottom: 0;
  width: 40px;
  height: 40px;
  margin: auto 0;
  font-size: 30px;
  color: #868686;
  margin-bottom: 11px;
  transition: color 0.2s ease-out;
}

.destroy:hover {
  color: #ce6f72;
}
```

12. The last component to create is the `TodoList` component.

```sh
ng generate component components/todo-list
```

`components/todo-list/todo-list.component.ts`

```ts
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
import { TodoItem } from 'src/app/models/todo-item';

@Component({
  selector: 'app-todo-list',
  templateUrl: './todo-list.component.html',
  styleUrls: ['./todo-list.component.less']
})
export class TodoListComponent implements OnInit {
  @Input()
  todoItems: TodoItem[];

  @Output()
  remove: EventEmitter<TodoItem>;

  @Output()
  toggleComplete: EventEmitter<TodoItem>;

  constructor() {
    this.remove = new EventEmitter();
    this.toggleComplete = new EventEmitter();
  }

  ngOnInit() {}

  onToggleTodoComplete(todoItem: TodoItem) {
    throw new Error('Not Implement');
  }

  onRemoveTodo(todoItem: TodoItem) {
    throw new Error('Not Implement');
  }
}
```

`components/todo-list/todo-list.component.spec.ts`

```ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';
import { TodoItem } from 'src/app/models/todo-item';
import { TodoListItemComponent } from './../todo-list-item/todo-list-item.component';
import { TodoListComponent } from './todo-list.component';

describe('TodoListComponent', () => {
  let component: TodoListComponent;
  let fixture: ComponentFixture<TodoListComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [TodoListComponent, TodoListItemComponent],
      imports: [FormsModule]
    });

    fixture = TestBed.createComponent(TodoListComponent);
    component = fixture.componentInstance;

    component.todoItems = [];

    fixture.detectChanges();
  });

  describe('general', () => {
    it('should create', () => {
      expect(component).toBeTruthy();
    });
  });

  describe('onRemoveTodo', () => {
    it('should emit remove event if onRemoveTodo is called', () => {
      const emitSpy = spyOn(component.remove, 'emit');
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };

      component.onRemoveTodo(item);

      expect(emitSpy).toHaveBeenCalledTimes(1);
      expect(emitSpy).toHaveBeenCalledWith(item);
    });
  });

  describe('onToggleTodoComplete', () => {
    it('should emit toggleComplete event if onToggleTodoComplete is called', () => {
      const emitSpy = spyOn(component.toggleComplete, 'emit');
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };

      component.onToggleTodoComplete(item);

      expect(emitSpy).toHaveBeenCalledTimes(1);
      expect(emitSpy).toHaveBeenCalledWith(item);
    });
  });
});
```

13. The `TodoList` component implementation

`components/todo-list/todo-list.component.ts`

```ts
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
import { TodoItem } from 'src/app/models/todo-item';

@Component({
  selector: 'app-todo-list',
  templateUrl: './todo-list.component.html',
  styleUrls: ['./todo-list.component.less']
})
export class TodoListComponent implements OnInit {
  @Input()
  todoItems: TodoItem[];

  @Output()
  remove: EventEmitter<TodoItem>;

  @Output()
  toggleComplete: EventEmitter<TodoItem>;

  constructor() {
    this.remove = new EventEmitter();
    this.toggleComplete = new EventEmitter();
  }

  ngOnInit() {}

  onToggleTodoComplete(todoItem: TodoItem) {
    this.toggleComplete.emit(todoItem);
  }

  onRemoveTodo(todoItem: TodoItem) {
    this.remove.emit(todoItem);
  }
}
```

`components/todo-list/todo-list.component.html`

```ts
<section class="main" *ngIf="todoItems && todoItems.length > 0">
  <ul class="todo-list">
    <li *ngFor="let todoItem of todoItems">
      <app-todo-list-item
        [todoItem]="todoItem"
        (toggleComplete)="onToggleTodoComplete($event)"
        (remove)="onRemoveTodo($event)"
      ></app-todo-list-item>
    </li>
  </ul>
</section>
```

`components/todo-list/todo-list.component.less`

```less
.todo-list {
  margin: 0;
  padding: 0;
  list-style: none;
}

.todo-list li {
  position: relative;
  font-size: 24px;
  border-bottom: 1px solid #ededed;
}

.todo-list li:last-child {
  border-bottom: none;
}

.todo-list li.editing {
  border-bottom: none;
  padding: 0;
}

.todo-list li.editing .edit {
  display: block;
  width: 506px;
  padding: 12px 16px;
  margin: 0 0 0 43px;
}

.todo-list li.editing .view {
  display: none;
}

.todo-list li .toggle {
  text-align: center;
  width: 40px;
  /* auto, since non-WebKit browsers doesn't support input styling */
  height: auto;
  position: absolute;
  top: 0;
  bottom: 0;
  margin: auto 0;
  border: none; /* Mobile Safari */
  -webkit-appearance: none;
  appearance: none;
}

.todo-list li .toggle:after {
  content: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="-10 -18 100 135"><circle cx="50" cy="50" r="50" fill="none" stroke="#ededed" stroke-width="3"/></svg>');
}

.todo-list li .toggle:checked:after {
  content: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="-10 -18 100 135"><circle cx="50" cy="50" r="50" fill="none" stroke="#bddad5" stroke-width="3"/><path fill="#5dc2af" d="M72 25L42 71 27 56l-4 4 20 20 34-52z"/></svg>');
}
```

14. In the last steps we're gonna bring everthing together in the app component.

`app.module.ts`

```ts
import { HttpClientModule } from '@angular/common/http';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { BrowserModule } from '@angular/platform-browser';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { TodoListHeaderComponent } from './components/todo-list-header/todo-list-header.component';
import { TodoListItemComponent } from './components/todo-list-item/todo-list-item.component';
import { TodoListComponent } from './components/todo-list/todo-list.component';

@NgModule({
  declarations: [AppComponent, TodoListHeaderComponent, TodoListItemComponent, TodoListComponent],
  imports: [BrowserModule, AppRoutingModule, FormsModule, HttpClientModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

`app.component.ts`

```ts
import { Component, OnInit } from '@angular/core';
import { TodoItem } from './models/todo-item';
import { TodoService } from './services/todo.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.less']
})
export class AppComponent implements OnInit {
  todoItems: TodoItem[];

  constructor(private todoService: TodoService) {}

  ngOnInit() {}

  onAddTodo(todoItem: TodoItem) {
    throw new Error('Not Implemented!');
  }

  onToggleTodoComplete(todoItem: TodoItem) {
    throw new Error('Not Implemented!');
  }

  onRemoveTodo(todoItem: TodoItem) {
    throw new Error('Not Implemented!');
  }
}
```

`app.component.spec.ts`

```ts
import { HttpClientTestingModule } from '@angular/common/http/testing';
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';
import { RouterTestingModule } from '@angular/router/testing';
import { of } from 'rxjs';
import { AppComponent } from './app.component';
import { TodoListHeaderComponent } from './components/todo-list-header/todo-list-header.component';
import { TodoListItemComponent } from './components/todo-list-item/todo-list-item.component';
import { TodoListComponent } from './components/todo-list/todo-list.component';
import { TodoItem } from './models/todo-item';
import { TodoService } from './services/todo.service';

const EMPTY = of(('' as unknown) as void);

describe('AppComponent', () => {
  let fixture: ComponentFixture<AppComponent>;
  let app: AppComponent;
  let todoService: TodoService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [RouterTestingModule, FormsModule, HttpClientTestingModule],
      declarations: [AppComponent, TodoListHeaderComponent, TodoListComponent, TodoListItemComponent]
    });

    fixture = TestBed.createComponent(AppComponent);
    app = fixture.debugElement.componentInstance;

    todoService = TestBed.get(TodoService);

    fixture.detectChanges();
  });

  describe('general', () => {
    it('should create the app', () => {
      expect(app).toBeTruthy();
    });
  });

  describe('onAddTodo', () => {
    it('should call TodoService.addTodo if called', () => {
      const addSpy = spyOn(todoService, 'addTodo').and.returnValue(EMPTY);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };

      app.onAddTodo(item);

      expect(addSpy).toHaveBeenCalledTimes(1);
      expect(addSpy).toHaveBeenCalledWith(item);
    });

    it('should refresh todoItems if called', () => {
      const addSpy = spyOn(todoService, 'addTodo').and.returnValue(EMPTY);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };
      const getSpy = spyOn(todoService, 'getTodos').and.returnValue(of([item]));

      app.onAddTodo(item);

      expect(addSpy).toHaveBeenCalledBefore(getSpy);
      expect(app.todoItems).toEqual([item]);
    });
  });

  describe('onToggleTodoComplete', () => {
    it('should call TodoService.addTodo if called', () => {
      const changeSpy = spyOn(todoService, 'changeTodoState').and.returnValue(EMPTY);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };

      app.onToggleTodoComplete(item);

      expect(changeSpy).toHaveBeenCalledTimes(1);
      expect(changeSpy).toHaveBeenCalledWith(item);
    });

    it('should refresh todoItems if called', () => {
      const changeSpy = spyOn(todoService, 'changeTodoState').and.returnValue(EMPTY);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };
      const getSpy = spyOn(todoService, 'getTodos').and.returnValue(of([item]));

      app.onToggleTodoComplete(item);

      expect(changeSpy).toHaveBeenCalledBefore(getSpy);
      expect(app.todoItems).toEqual([item]);
    });
  });

  describe('onRemoveTodo', () => {
    it('should call TodoService.addTodo if called', () => {
      const removeSpy = spyOn(todoService, 'removeTodo').and.returnValue(EMPTY);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };

      app.onRemoveTodo(item);

      expect(removeSpy).toHaveBeenCalledTimes(1);
      expect(removeSpy).toHaveBeenCalledWith(item);
    });

    it('should refresh todoItems if called', () => {
      const removeSpy = spyOn(todoService, 'removeTodo').and.returnValue(EMPTY);
      const item: TodoItem = { id: 'id', state: true, title: 'abc' };
      const getSpy = spyOn(todoService, 'getTodos').and.returnValue(of([item]));

      app.onRemoveTodo(item);

      expect(removeSpy).toHaveBeenCalledBefore(getSpy);
      expect(app.todoItems).toEqual([item]);
    });
  });
});
```

15. The last implementation.

`app.component.ts`

```ts
import { Component, OnInit } from '@angular/core';
import { TodoItem } from './models/todo-item';
import { TodoService } from './services/todo.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.less']
})
export class AppComponent implements OnInit {
  todoItems: TodoItem[];

  constructor(private todoService: TodoService) {}

  private getTodos() {
    this.todoService.getTodos().subscribe(result => (this.todoItems = result));
  }

  ngOnInit() {
    this.getTodos();
  }

  onAddTodo(todoItem: TodoItem) {
    this.todoService.addTodo(todoItem).subscribe(() => this.getTodos());
  }

  onToggleTodoComplete(todoItem: TodoItem) {
    this.todoService.changeTodoState(todoItem).subscribe(() => this.getTodos());
  }

  onRemoveTodo(todoItem: TodoItem) {
    this.todoService.removeTodo(todoItem).subscribe(() => this.getTodos());
  }
}
```

For now we are finished. Congratulations!

**TODO:** CI Setup
