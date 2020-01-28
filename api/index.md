# Todo API


1. We'll use `koa` instead of the often use `express` package to host our API. For testing we'll use `chai-http`.

```sh
npm i -S koa koa-router koa-bodyparser koa-logger
npm i -D @types/koa @types/koa-router @types/koa-bodyparser @types/koa-logger chai-http
```


2. The folder structure might seem a bit exaggerated for a small project like this. But this structure is practical if the project is growing.

```
src
|-
  |-api
    |- controllers
      |- todo-controller.ts
    |- routes
      |- todo-routes.ts
    |- tests
      |- todo.spec.ts
  |- api.ts
```

3. We'll start with creating the stub classes.

`todo-controller.ts`
```ts
import { DbConnection } from "../../db/db-connection";
import { ParameterizedContext } from "koa";

export class TodoController
{
    constructor(connection: DbConnection)
    {
        throw Error('Not implemented!')
    }

    async addTodo(ctx: ParameterizedContext)
    {
        throw Error('Not implemented!')
    }

    async removeTodo(ctx: ParameterizedContext)
    {
        throw Error('Not implemented!')
    }

    async changeTodoState(ctx: ParameterizedContext)
    {
        throw Error('Not implemented!')
    }

    async getTodos(ctx: ParameterizedContext)
    {
        throw Error('Not implemented!')
    }
}
```

`todo-routes.ts`
```ts
import Router from 'koa-router';
import { DbConnection } from '../../db/db-connection';

export class TodoRoutes
{
    public router: Router;

    constructor(connection: DbConnection)
    {
        throw Error('Not implemented!')
    }
}
```

`api.ts`
```ts
import { Server } from 'http';
import Koa from 'koa';
import koaBodyParser from 'koa-bodyparser';
import koaLogger from 'koa-logger';

import {TodoRoutes} from './routes/todo-routes'
import { DbConnection } from '../db/db-connection';

export class Api {
    public app: Koa;

    constructor(connection: DbConnection)
    {
        throw Error('Not implemented!')
    }
}
```

4. As always the next step is to write some tests and verify that they fail first.

`todo.spec.ts`
```ts
import { Api } from './../api';
import { expect } from 'chai';
import chai from 'chai'
import sinon from 'sinon';
import chaiHttp from 'chai-http';
chai.use(chaiHttp);

import { DbConnection } from './../../db/db-connection';
import { Server } from 'http';


describe('/api/v1', () => {
    let baseUrl = '/api/v1';
    let api: Api;
    let server: Server;
    let connectionStub: sinon.SinonStubbedInstance<DbConnection>;
    beforeEach((done) => {
        connectionStub = sinon.createStubInstance(DbConnection);
        api = new Api(connectionStub as unknown as DbConnection);
        server = api.app.listen().on('listening', done);
    });

    afterEach(() => {
        server.close();
    });

    describe('POST /todos', () => {
        it('should return error message with status 500 if addTodo rejects with an error', async () => {
            connectionStub.addTodo.rejects(new Error('Some error.'));

            const result = await chai.request(server).post(`${baseUrl}/todos`).send({ title: 'test' });

            expect(connectionStub.addTodo.callCount).to.equal(1);
            expect(result).to.have.status(500);
            expect(result.text).to.equal('Some error.');
        });

        it('should return status 400 if request body does not contain valid json with title', async () => {
            const result = await chai.request(server).post(`${baseUrl}/todos`).send({ abc: 'test' });

            expect(connectionStub.addTodo.callCount).to.equal(0);
            expect(result).to.have.status(400);
        });

        it('should return status 400 if title in reuqets body is not a string', async () => {
            const result = await chai.request(server).post(`${baseUrl}/todos`).send({ title: 123 });

            expect(connectionStub.addTodo.callCount).to.equal(0);
            expect(result).to.have.status(400);
        });

        it('should add todo and return no body with staus 204', async () => {
            const result = await chai.request(server).post(`${baseUrl}/todos`).send({ title: 'abcdefg' });

            expect(connectionStub.addTodo.callCount).to.equal(1);
            expect(connectionStub.addTodo.firstCall.args[0]).to.equal('abcdefg');
            expect(result).to.have.status(204);
        })
    });

    describe('DELETE /todos/:id', () => {
        it('should return error message with status 500 if removeTodo rejects with an error', async () => {
            connectionStub.removeTodo.rejects(new Error('Some error.'));

            const result = await chai.request(server).delete(`${baseUrl}/todos/abc`);

            expect(connectionStub.removeTodo.callCount).to.equal(1);
            expect(result).to.have.status(500);
            expect(result.text).to.equal('Some error.');
        });

        it('should remove todo and return no body with staus 204', async () => {
            const result = await chai.request(server).delete(`${baseUrl}/todos/abc`);

            expect(connectionStub.removeTodo.callCount).to.equal(1);
            expect(connectionStub.removeTodo.firstCall.args[0]).to.equal('abc');
            expect(result).to.have.status(204);
        })
    });

    describe('PUT /todos/:id', () => {
        it('should return error message with status 500 if changeTodoState rejects with an error', async () => {
            connectionStub.changeTodoState.rejects(new Error('Some error.'));

            const result = await chai.request(server).put(`${baseUrl}/todos/abc`).send({ state: false });

            expect(connectionStub.changeTodoState.callCount).to.equal(1);
            expect(result).to.have.status(500);
            expect(result.text).to.equal('Some error.');
        });

        it('should return status 400 if request body does not contain valid json with state', async () => {
            const result = await chai.request(server).put(`${baseUrl}/todos/abc`).send({ abc: 'test' });

            expect(connectionStub.changeTodoState.callCount).to.equal(0);
            expect(result).to.have.status(400);
        });

        it('should return status 400 if state in reuqets body is not a boolean', async () => {
            const result = await chai.request(server).put(`${baseUrl}/todos/abc`).send({ state: 123 });

            expect(connectionStub.changeTodoState.callCount).to.equal(0);
            expect(result).to.have.status(400);
        });

        it('should change todo state and return no body with staus 204', async () => {
            const result = await chai.request(server).put(`${baseUrl}/todos/abc`).send({ state: false });

            expect(connectionStub.changeTodoState.callCount).to.equal(1);
            expect(connectionStub.changeTodoState.firstCall.args[0]).to.equal('abc');
            expect(connectionStub.changeTodoState.firstCall.args[1]).to.equal(false);
            expect(result).to.have.status(204);
        })
    });

    describe('GET /todos', () => {
        it('should return error message with status 500 if getTodos rejects with an error', async () => {
            connectionStub.getTodos.rejects(new Error('Some error.'));

            const result = await chai.request(server).get(`${baseUrl}/todos`);

            expect(connectionStub.getTodos.callCount).to.equal(1);
            expect(result).to.have.status(500);
            expect(result.text).to.equal('Some error.');
        });

        it('should return todos as json with staus 200', async () => {
            connectionStub.getTodos.resolves([
                {
                    id: '62c460f0-426f-4339-addb-5aab3a4bbc14',
                    title: 'Test 1',
                    state: true
                },
                {
                    id: '94a9e91a-a914-41e6-8f1c-9dcc16c71cac',
                    title: 'Test 2',
                    state: false
                },
                {
                    id: '75002238-941d-4738-8728-a81eae96cb4f',
                    title: 'Test 4',
                    state: true
                },
                {
                    id: '9cad782a-3e3c-4523-94ba-7b892379f6a9',
                    title: 'Test 5',
                    state: true
                }
            ]);

            const result = await chai.request(server).get(`${baseUrl}/todos`);

            expect(connectionStub.getTodos.callCount).to.equal(1);
            expect(result).to.have.status(200);
            expect(result).to.have.json;
            expect(result.body).to.deep.equal([
                {
                    id: '62c460f0-426f-4339-addb-5aab3a4bbc14',
                    title: 'Test 1',
                    state: true
                },
                {
                    id: '94a9e91a-a914-41e6-8f1c-9dcc16c71cac',
                    title: 'Test 2',
                    state: false
                },
                {
                    id: '75002238-941d-4738-8728-a81eae96cb4f',
                    title: 'Test 4',
                    state: true
                },
                {
                    id: '9cad782a-3e3c-4523-94ba-7b892379f6a9',
                    title: 'Test 5',
                    state: true
                }
            ]);
        })
    });
});
```

5. `npm test` will report all tests failing. We can start the implantation now.

`todo-controller.ts`
```ts
import { DbConnection } from "../../db/db-connection";
import { ParameterizedContext } from "koa";

export class TodoController {
    static connection: DbConnection;

    static async addTodo(ctx: ParameterizedContext) {
        if (ctx.request.body.title === undefined) {
            ctx.status = 400;
            return
        }

        if (typeof ctx.request.body.title !== 'string') {
            ctx.status = 400;
            return
        }

        try {
            await TodoController.connection.addTodo(ctx.request.body.title);
            ctx.status = 204;
        } catch (error) {
            ctx.body = error.message;
            ctx.status = 500;
        }
    }

    static async removeTodo(ctx: ParameterizedContext) {
        try {
            await TodoController.connection.removeTodo(ctx.params.id);
            ctx.status = 204;
        } catch (error) {
            ctx.body = error.message;
            ctx.status = 500;
        }
    }

    static async changeTodoState(ctx: ParameterizedContext) {
        if (ctx.request.body.state === undefined) {
            ctx.status = 400;
            return
        }

        if (typeof ctx.request.body.state !== 'boolean') {
            ctx.status = 400;
            return
        }

        try {
            await TodoController.connection.changeTodoState(ctx.params.id, ctx.request.body.state);
            ctx.status = 204;
        } catch (error) {
            ctx.body = error.message;
            ctx.status = 500;
        }
    }

    static async getTodos(ctx: ParameterizedContext) {
        try {
            const result = await TodoController.connection.getTodos();
            ctx.set('Content-Type', 'application/json');
            ctx.body = JSON.stringify(result);
            ctx.status = 200;
        } catch (error) {
            ctx.body = error.message;
            ctx.status = 500;
        }
    }
}
```

`todo-routes.ts`
```ts
import Router from 'koa-router';
import { DbConnection } from '../../db/db-connection';
import { TodoController } from '../controllers/todo-controller';

const BASE_URL = '/api/v1/todos';

export class TodoRoutes
{
    public router: Router;

    constructor(connection: DbConnection)
    {
        this.router = new Router();

        TodoController.connection = connection;

        this.router.get(`${BASE_URL}`, TodoController.getTodos);
        this.router.post(`${BASE_URL}`, TodoController.addTodo);
        this.router.delete(`${BASE_URL}/:id`, TodoController.removeTodo);
        this.router.put(`${BASE_URL}/:id`, TodoController.changeTodoState);
    }
}
```

`api.ts`
```ts
import { Server } from 'http';
import Koa from 'koa';
import koaBodyParser from 'koa-bodyparser';
import koaLogger from 'koa-logger';

import { TodoRoutes } from './routes/todo-routes'
import { DbConnection } from '../db/db-connection';

export class Api {
    public app: Koa;

    constructor(connection: DbConnection) {
        this.app = new Koa();
        this.app.use(koaLogger());
        this.app.use(koaBodyParser());

        const todoRoutes = new TodoRoutes(connection);

        this.app.use(todoRoutes.router.routes());
    }
}
```

6. The last step ist to host the API in the index.ts file.

`index.ts`
```ts
import { DbConnection } from './db/db-connection';
import { createPool } from 'slonik';
import { Api } from './api/api';

const pool = createPool('postgresql://postgres:mysecretpassword@localhost:5432/todolist');
const connection = new DbConnection(pool);

const api = new Api(connection);
const port = process.env.port || 8080;

api.app.listen(port)
    .on('error', err => {
        // tslint:disable-next-line:no-console
        console.error(err);
    })
    .on('listening', () => {
        // tslint:disable-next-line:no-console
        console.log(`Server listening on port ${port}`);
    });
```

7. If you don't want to host the UI on the same host and port as the API, you have to activate CORS.

```sh
npm install -S @koa/cors
npm install -D @types/koa__cors
```

`api.ts`
```ts
import { Server } from 'http';
import Koa from 'koa';
import koaBodyParser from 'koa-bodyparser';
import koaLogger from 'koa-logger';
import Cors from '@koa/cors';

import { TodoRoutes } from './routes/todo-routes'
import { DbConnection } from '../db/db-connection';

export class Api {
    public app: Koa;

    constructor(connection: DbConnection) {
        this.app = new Koa();
        this.app.use(koaLogger());
        this.app.use(koaBodyParser());
        this.app.use(Cors());

        const todoRoutes = new TodoRoutes(connection);

        this.app.use(todoRoutes.router.routes());
    }
}
```



This concludes the api part. We continue with the [UI](../ui/index.md) part of this project.