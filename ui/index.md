# Setup

1. Ensure that you have the Angular CLI installed gloablly. 

```sh
npm install -g @angular/cli
```


2. Create a new Angular project with the wizzard.

```sh
ng new

What name would you like to use for the new workspace and initial project? tdd-todolist-frontend
Would you like to add Angular routing? (y/N) y
Which stylesheet format would you like to use? Less
```

3. You can start the initial test app and try it out on: http://localhost:4200/

```sh
cd tdd-todolist-frontend
ng serve
```

4. We start by creating the TodoItem class. As always we'll first create the stub class, implement the tests and do the implementation last.

```sh
ng generate class models/TodoItem
```

`models/todo-item.ts`
```ts
export class TodoItem {
    id: string;
    title: string;
    state: boolean;

    constructor(values: object = {}) {
        throw new Error('Not Implemented!');
    }
}
```

`models/todo-item.spec.ts`
```ts
import { TodoItem } from './todo-item';

describe('TodoItem', () => {
  it('should create an instance', () => {
    expect(new TodoItem()).toBeTruthy();
  });

  it('should set values in constructor', () => {
    let todo = new TodoItem({
      id:'abc',
      title: 'hello',
      state: true
    });
    expect(todo.id).toEqual('abc');
    expect(todo.title).toEqual('hello');
    expect(todo.state).toEqual(true);
  });
});
```

5. Run the tests with `ng test` and ensure that they are red. Then we can do the implementation.

`models/todo-item.ts`
```ts
export class TodoItem {
    id: string;
    title: string;
    state: boolean;

    constructor(values: object = {}) {
        Object.assign(this, values);
    }
}
```

6. In order to keep the base url of the api flexible we're gonna add the `baseApi` to the angular environment. This is not suitable for production enviroments because the base url is compiled into the app and not changeable later.

`environments/enviroment.test.ts`
```ts
export const environment = {
  production: false,
  baseApi: 'http://localhost:8080'
};
```

`environments/enviroment.ts`
```ts
export const environment = {
  production: true,
  baseApi: 'http://localhost:8080'
};
```

7. We start with the todo service that is responsible for talking to our api.

```sh
ng generate service services/Todo
```

`services/todo.service.ts`
```ts
import { Injectable } from '@angular/core';
import { TodoItem } from './../models/todo-item';
import { Observable } from 'rxjs';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { environment } from 'src/environments/environment';

const API_URL = `${environment.baseApi}/api/v1/todos`;

@Injectable({
  providedIn: 'root'
})
export class TodoService {

  constructor(private http: HttpClient) { }

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
import { TestBed, getTestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController, TestRequest } from '@angular/common/http/testing';

import { TodoService } from './todo.service';
import { TodoItem } from '../models/todo-item';
import { environment } from 'src/environments/environment';

describe('TodoServiceService', () => {
  let http: HttpTestingController;
  const API_URL = `${environment.baseApi}/api/v1/todos`;

  beforeEach(() => {
    TestBed.configureTestingModule({ imports: [HttpClientTestingModule] });

    http = getTestBed().get(HttpTestingController);
  });

  it('should be created', () => {
    const service: TodoService = TestBed.get(TodoService);
    expect(service).toBeTruthy();
  });

  describe('addTodo', () => {
    const url = `${API_URL}`;

    it('should call api endpoint correctly', (done) => {
      const service: TodoService = TestBed.get(TodoService);
      const item = new TodoItem({ title: 'abc' });
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

    it('should set correct body', (done) => {
      const service: TodoService = TestBed.get(TodoService);
      const item = new TodoItem({ title: 'abc' });
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
    const url = `${API_URL}/abc`;

    it('should call api endpoint correctly', (done) => {
      const service: TodoService = TestBed.get(TodoService);
      const item = new TodoItem({ id: 'abc' });
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

    it('should call api endpoint correctly', (done) => {
      const service: TodoService = TestBed.get(TodoService);
      const item = new TodoItem({ title: 'abc', id: 'id', state: true });
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

    it('should set correct body', (done) => {
      const service: TodoService = TestBed.get(TodoService);
      const item = new TodoItem({ title: 'abc', id: 'id', state: true });
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

    it('should call api endpoint correctly', (done) => {
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

    it('should resolve with response', (done) => {
      const service: TodoService = TestBed.get(TodoService);
      service.getTodos().subscribe((res) => {
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


8. The tests schould now be red and we can start with the implemenation.

`services/todo.service.ts`
```ts
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { TodoItem } from '../models/todo-item';
import { environment } from 'src/environments/environment';

const API_URL = `${environment.baseApi}/api/v1/todos`;

@Injectable({
  providedIn: 'root'
})
export class TodoService {

  constructor(private http: HttpClient) { }

  public addTodo(item: TodoItem): Observable<void> {
    const headers: HttpHeaders = new HttpHeaders().set(
      'Content-Type',
      'application/json'
    );
    return this.http.post<void>(`${API_URL}`, { title: item.title }, { headers });
  }

  public removeTodo(item: TodoItem): Observable<void> {
    return this.http.delete<void>(`${API_URL}/${item.id}`);
  }

  public changeTodoState(item: TodoItem): Observable<void> {
    const headers: HttpHeaders = new HttpHeaders().set(
      'Content-Type',
      'application/json'
    );
    return this.http.put<void>(`${API_URL}/${item.id}`, { state: item.state }, { headers });
  }

  public getTodos(): Observable<TodoItem[]> {
    return this.http.get<TodoItem[]>(`${API_URL}`);
  }
}

```

9. Now it is time to create the components for the ui. We're starting with the `TodoListHeader` component..

```sh
npm i -S @angular/forms
ng generate component components/todo-list-header
```

`components/todo-list-header/todo-list-header.component.ts`
```ts
import { Component, OnInit, Output, EventEmitter } from '@angular/core';
import { TodoItem } from 'src/app/models/todo-item';

@Component({
  selector: 'app-todo-list-header',
  templateUrl: './todo-list-header.component.html',
  styleUrls: ['./todo-list-header.component.less']
})
export class TodoListHeaderComponent implements OnInit {

  newTodoItem: TodoItem = new TodoItem();

  @Output()
  add: EventEmitter<TodoItem>;

  constructor() {
    this.add = new EventEmitter();
  }

  ngOnInit() {
  }

  addTodo()
  {
    throw new Error('Not Implemented!');
  }

}

```

`components/todo-list-header/todo-list-header.component.spec.ts`
```ts
import { async, ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';


import { TodoListHeaderComponent } from './todo-list-header.component';

describe('TodoListHeaderComponent', () => {
  let component: TodoListHeaderComponent;
  let fixture: ComponentFixture<TodoListHeaderComponent>;

  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [TodoListHeaderComponent],
      imports: [FormsModule]
    })
      .compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(TodoListHeaderComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should emit add event if addTodo is called', () => {
    const emitSpy = spyOn(component.add, 'emit');
    component.newTodoItem.title = 'abc';

    component.addTodo();

    expect(emitSpy).toHaveBeenCalledTimes(1);
    expect(emitSpy.calls.mostRecent().args[0].title).toEqual('abc');
  });

  it('should create new newTodo instance after addTodo was called', () => {
    component.newTodoItem.title = 'abc';

    component.addTodo();

    expect(component.newTodoItem.title).toEqual(undefined);
  });
});

```


10. After verifying that the test are red we can start implementing the component and add the html and less files. We're not gonna test the HTML file. It is possible to check the databinding in unit tests. We do this later with e2e tests.

`components/todo-list-header/todo-list-header.component.ts`
```ts
import { Component, OnInit, Output, EventEmitter } from '@angular/core';
import { TodoItem } from 'src/app/models/todo-item';

@Component({
  selector: 'app-todo-list-header',
  templateUrl: './todo-list-header.component.html',
  styleUrls: ['./todo-list-header.component.less']
})
export class TodoListHeaderComponent implements OnInit {

  newTodoItem: TodoItem = new TodoItem();

  @Output()
  add: EventEmitter<TodoItem>;

  constructor() {
    this.add = new EventEmitter();
  }

  ngOnInit() {
  }

  addTodo() {
    this.add.emit(this.newTodoItem);
    this.newTodoItem = new TodoItem();
  }

}

```

`components/todo-list-header/todo-list-header.component.html`
```html
<header class="header">
    <h1>Todos</h1>
    <input class="new-todo-item" placeholder="What needs to be done?" autofocus="" [(ngModel)]="newTodoItem.title"
        (keyup.enter)="addTodo()">
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
	box-shadow: inset 0 -2px 1px rgba(0,0,0,0.03);
}
```


11. The next component is the `TodoListItem` component.

```sh
ng generate component components/todo-list-item
```

`components/todo-list-item/todo-list-item.component.ts`
```ts
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';
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

  ngOnInit() {
  }

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
import { async, ComponentFixture, TestBed } from '@angular/core/testing';

import { TodoListItemComponent } from './todo-list-item.component';
import { TodoItem } from 'src/app/models/todo-item';
import { FormsModule } from '@angular/forms';

describe('TodoListItemComponent', () => {
  let component: TodoListItemComponent;
  let fixture: ComponentFixture<TodoListItemComponent>;

  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [TodoListItemComponent],
      imports: [FormsModule]
    })
      .compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(TodoListItemComponent);
    component = fixture.componentInstance;
    component.todoItem = new TodoItem({});
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should emit remove event of removeTodo is called', () => {
    const emitSpy = spyOn(component.remove, 'emit');
    const item = new TodoItem({});
    component.todoItem = item;

    component.removeTodo();

    expect(emitSpy).toHaveBeenCalledTimes(1);
    expect(emitSpy).toHaveBeenCalledWith(item);
  });

  it('should emit toggleComplete event of toggleTodoComplete is called', () => {
    const emitSpy = spyOn(component.toggleComplete, 'emit');
    const item = new TodoItem({});
    component.todoItem = item;

    component.toggleTodoComplete();

    expect(emitSpy).toHaveBeenCalledTimes(1);
    expect(emitSpy).toHaveBeenCalledWith(item);
  });
});

```

12. The `TodoListItem` component implementation.

`components/todo-list-item/todo-list-item.component.ts`
```ts
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';
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

  ngOnInit() {
  }

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
    <input class="toggle" type="checkbox" (change)="toggleTodoComplete()" [(ngModel)]="todoItem.state">
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

13. The last component to create is the `TodoList` component.

```sh
ng generate component components/todo-list
```

`components/todo-list/todo-list.component.ts`
```ts
import { Component, OnInit, EventEmitter, Input, Output } from '@angular/core';
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

  ngOnInit() {
  }

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
import { async, ComponentFixture, TestBed } from '@angular/core/testing';

import { TodoListComponent } from './todo-list.component';
import { TodoListItemComponent } from './../todo-list-item/todo-list-item.component';
import { TodoItem } from 'src/app/models/todo-item';
import { FormsModule } from '@angular/forms';

describe('TodoListComponent', () => {
  let component: TodoListComponent;
  let fixture: ComponentFixture<TodoListComponent>;

  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [TodoListComponent, TodoListItemComponent],
      imports: [FormsModule]
    })
      .compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(TodoListComponent);
    component = fixture.componentInstance;
    component.todoItems = [];
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should emit remove event if onRemoveTodo is called', () => {
    const emitSpy = spyOn(component.remove, 'emit');
    const item = new TodoItem({});

    component.onRemoveTodo(item);

    expect(emitSpy).toHaveBeenCalledTimes(1);
    expect(emitSpy).toHaveBeenCalledWith(item);
  });

  it('should emit toggleComplete event if onToggleTodoComplete is called', () => {
    const emitSpy = spyOn(component.toggleComplete, 'emit');
    const item = new TodoItem({});

    component.onToggleTodoComplete(item);

    expect(emitSpy).toHaveBeenCalledTimes(1);
    expect(emitSpy).toHaveBeenCalledWith(item);
  });
});

```

14. The `TodoList` component implementation

`components/todo-list/todo-list.component.ts`
```ts
import { Component, OnInit, EventEmitter, Input, Output } from '@angular/core';
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

  ngOnInit() {
  }

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
            <app-todo-list-item [todoItem]="todoItem" (toggleComplete)="onToggleTodoComplete($event)"
                (remove)="onRemoveTodo($event)"></app-todo-list-item>
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


15. In the last steps we're gonna bring everthing together in the app component.

`app.module.ts`
```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { TodoListHeaderComponent } from './components/todo-list-header/todo-list-header.component';
import { TodoListItemComponent } from './components/todo-list-item/todo-list-item.component';
import { TodoListComponent } from './components/todo-list/todo-list.component';
import { FormsModule } from '@angular/forms';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [
    AppComponent,
    TodoListHeaderComponent,
    TodoListItemComponent,
    TodoListComponent
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

`app.component.ts`
```ts
import { Component, OnInit } from '@angular/core';
import { TodoService } from './services/todo.service';
import { TodoItem } from './models/todo-item';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.less']
})
export class AppComponent implements OnInit {
  todoItems: TodoItem[];

  constructor(private todoService: TodoService) {

  }

  ngOnInit() {
  }

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
import { TestBed, ComponentFixture } from '@angular/core/testing';
import { RouterTestingModule } from '@angular/router/testing';
import { AppComponent } from './app.component';
import { TodoItem } from './models/todo-item';
import { TodoService } from './services/todo.service';
import { of, } from 'rxjs';
import { TodoListHeaderComponent } from './components/todo-list-header/todo-list-header.component';
import { TodoListComponent } from './components/todo-list/todo-list.component';
import { TodoListItemComponent } from './components/todo-list-item/todo-list-item.component';
import { FormsModule } from '@angular/forms';
import { HttpClientTestingModule } from '@angular/common/http/testing';

const EMPTY = of('' as unknown as void);

describe('AppComponent', () => {
  let fixture: ComponentFixture<AppComponent>;
  let app: AppComponent;
  let todoService: TodoService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [
        RouterTestingModule,
        FormsModule,
        HttpClientTestingModule
      ],
      declarations: [
        AppComponent,
        TodoListHeaderComponent,
        TodoListComponent,
        TodoListItemComponent
      ]
    });

    fixture = TestBed.createComponent(AppComponent);
    app = fixture.debugElement.componentInstance;

    todoService = TestBed.get(TodoService);

    fixture.detectChanges();
  });


  it('should create the app', () => {
    expect(app).toBeTruthy();
  });

  describe('onAddTodo', () => {
    it('should call TodoService.addTodo if called', () => {
      const addSpy = spyOn(todoService, 'addTodo').and.returnValue(EMPTY);
      const item = new TodoItem({});

      app.onAddTodo(item);

      expect(addSpy).toHaveBeenCalledTimes(1);
      expect(addSpy).toHaveBeenCalledWith(item);
    });

    it('should refresh todoItems if called', () => {
      const addSpy = spyOn(todoService, 'addTodo').and.returnValue(EMPTY);
      const item = new TodoItem({});
      const getSpy = spyOn(todoService, 'getTodos').and.returnValue(of([item]));

      app.onAddTodo(item);

      expect(addSpy).toHaveBeenCalledBefore(getSpy);
      expect(app.todoItems).toEqual([item]);
    });
  });

  describe('onToggleTodoComplete', () => {
    it('should call TodoService.addTodo if called', () => {
      const changeSpy = spyOn(todoService, 'changeTodoState').and.returnValue(EMPTY);
      const item = new TodoItem({});

      app.onToggleTodoComplete(item);

      expect(changeSpy).toHaveBeenCalledTimes(1);
      expect(changeSpy).toHaveBeenCalledWith(item);
    });

    it('should refresh todoItems if called', () => {
      const changeSpy = spyOn(todoService, 'changeTodoState').and.returnValue(EMPTY);
      const item = new TodoItem({});
      const getSpy = spyOn(todoService, 'getTodos').and.returnValue(of([item]));

      app.onToggleTodoComplete(item);

      expect(changeSpy).toHaveBeenCalledBefore(getSpy);
      expect(app.todoItems).toEqual([item]);
    });
  });

  describe('onRemoveTodo', () => {
    it('should call TodoService.addTodo if called', () => {
      const removeSpy = spyOn(todoService, 'removeTodo').and.returnValue(EMPTY);
      const item = new TodoItem({});

      app.onRemoveTodo(item);

      expect(removeSpy).toHaveBeenCalledTimes(1);
      expect(removeSpy).toHaveBeenCalledWith(item);
    });

    it('should refresh todoItems if called', () => {
      const removeSpy = spyOn(todoService, 'removeTodo').and.returnValue(EMPTY);
      const item = new TodoItem({});
      const getSpy = spyOn(todoService, 'getTodos').and.returnValue(of([item]));

      app.onRemoveTodo(item);

      expect(removeSpy).toHaveBeenCalledBefore(getSpy);
      expect(app.todoItems).toEqual([item]);
    });
  });
});

```


16. The last implementation.

`app.component.ts`
```ts
import { Component, OnInit } from '@angular/core';
import { TodoService } from './services/todo.service';
import { TodoItem } from './models/todo-item';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.less']
})
export class AppComponent implements OnInit {
  todoItems: TodoItem[];

  constructor(private todoService: TodoService) {

  }

  private getTodos() {
    this.todoService.getTodos().subscribe(result => this.todoItems = result);
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