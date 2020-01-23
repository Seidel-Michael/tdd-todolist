```sh
npm i -S koa koa-router koa-bodyparser koa-logger
npm i -D @types/koa @types/koa-router @types/koa-bodyparser @types/koa-logger chai-http
```

**Todo:** Create folder structure

`todo-controller.ts`
```
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
```
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
```
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

`index.ts`
```
import { DbConnection } from './db/db-connection';
import { createPool } from 'slonik';
import { Api } from './api/api';

const pool = createPool('postgresql://postgres:mysecretpassword@localhost:5432/todolist');
const connection = new DbConnection(pool);

const api = new Api(connection);

const PORT = 8080;

api.app.listen(PORT)
    .on('error', err => {
        console.error(err);
    })
    .on('listening', () => {
        console.log(`Server listening on port ${PORT}`);
    });
```

**TODO:** Add tests and implementation steps