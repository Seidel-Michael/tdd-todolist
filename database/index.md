# Database connection

We will use [DBeaver](https://dbeaver.io/) as a GUI for the postgres database.
Please be advised that in order to keep this example project simple we won't secure the database.
Running and using the database with the postgres user and an insecure password is not advised for a production environment.

If you use Visual Studio Code, the extension `SQL tagged template literals` is recommended.

1. Start a postgres database in docker. Please note that this is not a production setup. You'll be running unsafe and if you delete the container all your data will be lost.

```sh
docker run --name todolist-postgres -p 5432:5432 -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

2. Connect with DBeaver

- _Create Connection_
- PosgreSQL
- _Next_
- Host: localhost
- Port: 5432
- Database: empty
- User: postgres
- Password: mysecretpassword
- Show all databases: checked
- _Finish_

3. Create Database

- _Right click on PostgreSQL_
- _Create -> Database_
- Database name: todolist
- _OK_
- _Right click on todolist_
- _Set active_

---

- _Right click on public schema_
- _Create -> Table_
- _Properties_
- Table Name: todos

---

- _Right click in table_
- _Create new column_
- Name: id
- Data type: SERIAL
- Not Null: checked
- _OK_

---

- _Right click in table_
- _Create new column_
- Name: title
- Data type: varchar
- Length: 50
- Not Null: checked
- _OK_

---

- _Right click in table_
- _Create new column_
- Name: state
- Data type: bool
- Not Null: checked
- _OK_

---

- _Constrains_
- _Right click in table_
- _Create New Constraint_
- id: checked
- _OK_
- _Save_
- _Persist_

---

4. We use slonik as the database connector. We have to install the package and the type definitions for typescript.

```sh
npm i -S slonik
npm i -D @types/slonik
```

5. For testing the database connector we use a small helper script.

`test-helper/normalizeQuery.ts`

```ts
const inlineCommentRule = /(.*?)--.*/g;
const multilineCommentRule = /\/\*[\s\S]*?\*\//gm;
const whiteSpaceRule = /\s+/g;

export function normalizeQuery(input: string): string {
  return input
    .replace(inlineCommentRule, (match, p1) => {
      return p1;
    })
    .replace(multilineCommentRule, '')
    .replace(whiteSpaceRule, ' ')
    .trim();
}
```

6. The next step is to create the stub class for the DbConnector.

`src/db/db-connection.ts`

```ts
import { DatabasePoolType } from 'slonik';

class DbConnection {
  constructor(pool: DatabasePoolType) {
    throw new Error('Not implemented!');
  }

  public async addTodo(title: string) {
    throw new Error('Not implemented!');
  }

  public async removeTodo(id: string) {
    throw new Error('Not implemented!');
  }

  public async changeTodoState(id: string, state: boolean) {
    throw new Error('Not implemented!');
  }

  public async getTodos() {
    throw new Error('Not implemented!');
  }
}
```

7. Now we write the test for the `addTodo` to begin with.

`src/db/db-connection.spec.ts`

```ts
import { DbConnection } from './db-connection';
import { expect } from 'chai';
import chai from 'chai';
import sinon from 'sinon';
var chaiAsPromised = require('chai-as-promised');
chai.use(chaiAsPromised);
import { normalizeQuery } from '../../test-helper/normalizeQuery';

import { DatabasePoolType, ConnectionError } from 'slonik';

describe('DbConnection', () => {
  let poolStub: any;
  let connection: DbConnection;
  beforeEach(() => {
    poolStub = <DatabasePoolType>{};
    poolStub.query = sinon.stub();
    poolStub.any = sinon.stub();

    connection = new DbConnection(poolStub);
  });

  describe('addTodo', () => {
    it('should reject with Error if database connection failed', async () => {
      poolStub.query.rejects(new ConnectionError('getaddrinfo ENOTFOUND postgresql postgresql:5432'));

      return expect(connection.addTodo('abcdefg')).to.be.rejectedWith('Connection to database failed.');
    });

    it('should reject with Error if insert into database failed', async () => {
      poolStub.query.rejects(new Error('error: relation "todos" does not exist'));

      return expect(connection.addTodo('abcdefg')).to.be.rejectedWith('Oops... something went wrong.');
    });

    it('should insert todo in database and resolve', async () => {
      await connection.addTodo('abcdefg');

      expect(poolStub.query.callCount).to.equal(1);
      expect(normalizeQuery(poolStub.query.firstCall.args[0].sql)).to.equal(
        'INSERT INTO todos(title, state) VALUES ($1, true)'
      );
      expect(poolStub.query.firstCall.args[0].values).to.deep.equal(['abcdefg']);
    });
  });
});
```

8. If you run `npm test` again you should see all the new test failing. We now can implement the addTodo function.

`src/db/db-connection.ts`

```ts
import { DatabasePoolType, sql, ConnectionError } from 'slonik';
import { TodoItem } from '../models/todo-item';

export class DbConnection {
  private pool: DatabasePoolType;

  constructor(pool: DatabasePoolType) {
    this.pool = pool;
  }

  public async addTodo(title: string) {
    try {
      await this.pool.query(sql`INSERT INTO todos(title, state) VALUES (${title}, true)`);
    } catch (error) {
      if (error instanceof ConnectionError) {
        throw new Error('Connection to database failed.');
      }

      throw new Error('Oops... something went wrong.');
    }
  }

  public async removeTodo(id: string) {
    throw new Error('Not implemented!');
  }

  public async changeTodoState(id: string, state: boolean) {
    throw new Error('Not implemented!');
  }

  public async getTodos() {
    throw new Error('Not implemented!');
  }
}
```

9. Now we can write the rest of the tests and implementation. But always remember. Write the tests first and verify that they fail.

`src/db/db-connection.spec.ts`

```ts
import { DbConnection } from './db-connection';
import { expect } from 'chai';
import chai from 'chai';
import sinon from 'sinon';
var chaiAsPromised = require('chai-as-promised');
chai.use(chaiAsPromised);
import { normalizeQuery } from '../../test-helper/normalizeQuery';

import { DatabasePoolType, ConnectionError } from 'slonik';

describe('DbConnection', () => {
  let poolStub: any;
  let connection: DbConnection;
  beforeEach(() => {
    poolStub = <DatabasePoolType>{};
    poolStub.query = sinon.stub();
    poolStub.any = sinon.stub();

    connection = new DbConnection(poolStub);
  });

  describe('addTodo', () => {
    it('should reject with Error if database connection failed', async () => {
      poolStub.query.rejects(new ConnectionError('getaddrinfo ENOTFOUND postgresql postgresql:5432'));

      return expect(connection.addTodo('abcdefg')).to.be.rejectedWith('Connection to database failed.');
    });

    it('should reject with Error if insert into database failed', async () => {
      poolStub.query.rejects(new Error('error: relation "todos" does not exist'));

      return expect(connection.addTodo('abcdefg')).to.be.rejectedWith('Oops... something went wrong.');
    });

    it('should insert todo in database and resolve', async () => {
      await connection.addTodo('abcdefg');

      expect(poolStub.query.callCount).to.equal(1);
      expect(normalizeQuery(poolStub.query.firstCall.args[0].sql)).to.equal(
        'INSERT INTO todos(title, state) VALUES ($1, true)'
      );
      expect(poolStub.query.firstCall.args[0].values).to.deep.equal(['abcdefg']);
    });
  });

  describe('removeTodo', () => {
    it('should reject with Error if database connection failed', async () => {
      poolStub.query.rejects(new ConnectionError('getaddrinfo ENOTFOUND postgresql postgresql:5432'));

      return expect(connection.removeTodo('id')).to.be.rejectedWith('Connection to database failed.');
    });

    it('should reject with Error if delete from database failed', async () => {
      poolStub.query.rejects(new Error('error: relation "todos" does not exist'));

      return expect(connection.removeTodo('id')).to.be.rejectedWith('Oops... something went wrong.');
    });

    it('should remove todo from database and resolve', async () => {
      await connection.removeTodo('id');

      expect(poolStub.query.callCount).to.equal(1);
      expect(normalizeQuery(poolStub.query.firstCall.args[0].sql)).to.equal('DELETE FROM todos WHERE id=$1');
      expect(poolStub.query.firstCall.args[0].values).to.deep.equal(['id']);
    });
  });

  describe('changeTodoState', () => {
    it('should reject with Error if database connection failed', async () => {
      poolStub.query.rejects(new ConnectionError('getaddrinfo ENOTFOUND postgresql postgresql:5432'));

      return expect(connection.changeTodoState('id', false)).to.be.rejectedWith('Connection to database failed.');
    });

    it('should reject with Error if update database failed', async () => {
      poolStub.query.rejects(new Error('error: relation "todos" does not exist'));

      return expect(connection.changeTodoState('id', false)).to.be.rejectedWith('Oops... something went wrong.');
    });

    it('should update todo state in database and resolve', async () => {
      await connection.changeTodoState('id', false);

      expect(poolStub.query.callCount).to.equal(1);
      expect(normalizeQuery(poolStub.query.firstCall.args[0].sql)).to.equal('UPDATE todos SET state=$1 WHERE id=$2');
      expect(poolStub.query.firstCall.args[0].values).to.deep.equal([false, 'id']);
    });
  });

  describe('getTodos', () => {
    it('should reject with Error if database connection failed', async () => {
      poolStub.any.rejects(new ConnectionError('getaddrinfo ENOTFOUND postgresql postgresql:5432'));

      return expect(connection.getTodos()).to.be.rejectedWith('Connection to database failed.');
    });

    it('should reject with Error if select from database failed', async () => {
      poolStub.any.rejects(new Error('error: relation "todos" does not exist'));

      return expect(connection.getTodos()).to.be.rejectedWith('Oops... something went wrong.');
    });

    it('should resolve with data from database', async () => {
      poolStub.any.resolves([
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
      ]);

      const result = await connection.getTodos();

      expect(poolStub.any.callCount).to.equal(1);
      expect(normalizeQuery(poolStub.any.firstCall.args[0].sql)).to.equal('SELECT * FROM todos ORDER BY ID');

      expect(result).to.deep.equal([
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
      ]);
    });
  });
});
```

`src/db/db-connection.ts`

```ts
import { DatabasePoolType, sql, ConnectionError } from 'slonik';
import { TodoItem } from '../models/todo-item';

export class DbConnection {
  private pool: DatabasePoolType;

  constructor(pool: DatabasePoolType) {
    this.pool = pool;
  }

  public async addTodo(title: string) {
    try {
      await this.pool.query(sql`INSERT INTO todos(title, state) VALUES (${title}, true)`);
    } catch (error) {
      if (error instanceof ConnectionError) {
        throw new Error('Connection to database failed.');
      }

      throw new Error('Oops... something went wrong.');
    }
  }

  public async removeTodo(id: string) {
    try {
      await this.pool.query(sql`DELETE FROM todos WHERE id=${`${id}`}`);
    } catch (error) {
      if (error instanceof ConnectionError) {
        throw new Error('Connection to database failed.');
      }

      throw new Error('Oops... something went wrong.');
    }
  }

  public async changeTodoState(id: string, state: boolean) {
    try {
      await this.pool.query(sql`UPDATE todos SET state=${state} WHERE id=${`${id}`}`);
    } catch (error) {
      if (error instanceof ConnectionError) {
        throw new Error('Connection to database failed.');
      }

      throw new Error('Oops... something went wrong.');
    }
  }

  public async getTodos(): Promise<TodoItem[]> {
    let result;
    try {
      result = await this.pool.any<TodoItem>(sql`SELECT * FROM todos ORDER BY id`);
    } catch (error) {
      if (error instanceof ConnectionError) {
        throw new Error('Connection to database failed.');
      }

      throw new Error('Oops... something went wrong.');
    }

    return result;
  }
}
```

10. All tests should be passing now. You might have noticed, that we don't test for the values to be correct. That is because only we will call the function and can make sure at the calling code that the types are correct. We only want to implement what we really need.

11. We can now delete the `index.spec.ts` file and edit the `index.ts` file to test our written code.

`index.ts`

```ts
import { DbConnection } from './db/db-connection';
import { createPool } from 'slonik';

const connection = new DbConnection(createPool('postgresql://postgres:mysecretpassword@localhost:5432/todolist'));
connection.addTodo('I am a test!');
```

12. If we run `npm start` this example should add a todo item to the database. We can verify this via DBeaver or if the output the `connection.getTodos` result.

`index.ts`

```ts
import { DbConnection } from './db/db-connection';
import { createPool } from 'slonik';

const connection = new DbConnection(createPool('postgresql://postgres:mysecretpassword@localhost:5432/todolist'));
connection.addTodo('I am a test!');

connection.getTodos().then(result => {
  console.log(result);
});
```

13. We don't want to create the database manually every time. So we add an `init.sql` script to the postgres container to automate this.

`init.sql`

```sql
CREATE DATABASE todolist;
\connect -reuse-previous=on "dbname='todolist'"
CREATE TABLE todos (
	id SERIAL NOT NULL PRIMARY KEY,
	title varchar(50) NOT NULL,
	state bool NOT NULL
);
```

```sh
docker run --name todolist-postgres -p 5432:5432 -v$(pwd)/init.sql:/docker-entrypoint-initdb.d/init.sql -e POSTGRES_PASSWORD=mysecretpassword postgres
```

This concludes the database part. We continue with the [api](../api/index.md) part of this project.
