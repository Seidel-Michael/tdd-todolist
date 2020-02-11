# Initial backend project setup

This project setup utilizes GitHub actions for the continuous integration.
If you want to use this, you have to set up a GitHub project first and clone it.
Otherwise, you can use any version control system or none if you're into that.

1. Change to the folder you want to set up the backend project in.
2. Run the `npm init` command to set up a node project. You can use the default value or customize them to your desire. We're gonna change the important settings later on.

```sh
npm init
```

3. With the following command we'll install and setup all the basic npm packages we need to get started with the test driven development.

```sh
npm install -D typescript gts mocha @types/mocha chai @types/chai ts-node nyc source-map-support @types/node sinon @types/sinon chai-as-promised @types/chai-as-promised @istanbuljs/nyc-config-typescript tslint-no-unused-expression-chai
```

```
npx gts init
```

Answer no if you are aksed to overwrite the version of `typescript` and `@types/node`.

4. We want to slightly modify our typescript compiler options. The default Google config already has most of what we need configured. We just need to add two options to the `lib` section.

`tsconfig.json`

```diff
{
  "extends": "./node_modules/gts/tsconfig-google.json",
  "compilerOptions": {
    "rootDir": ".",
    "outDir": "build",
+   "esModuleInterop": true,
+   "lib": ["es2016", "DOM"]
  },
  "include": ["src/**/*.ts", "test/**/*.ts"]
}
```

`tslint.json`

```diff
{
  "extends": [
    "gts/tslint.json",
+   "tslint-no-unused-expression-chai"
  ],
  "linterOptions": {
    "exclude": ["**/*.json"]
  }
}

```

5. We also want to change our `package.json` file. For our convenience we add the start script which will compile and start our app.

`package.json`

```diff
"description": "",
  "main": "build/src/index.js",
  "scripts": {
+   "test": "nyc mocha",
+   "start": "npm run compile && node .",
    "check": "gts check",
    "clean": "gts clean",
    "compile": "tsc -p .",
    "fix": "gts fix",
    "prepare": "npm run compile",
    "pretest": "npm run compile",
    "posttest": "npm run check"
  },
```

6. We also need to add a config for the code coverage reporter and our test runner.

`.nycrc.json`

```json
{
  "extends": "@istanbuljs/nyc-config-typescript",
  "include": ["src/**/*.ts"],
  "reporter": ["text-summary", "html", "cobertura", "lcov"],
  "all": true
}
```

`test/mocha.opts`

```
--require ts-node/register
--require source-map-support/register
--recursive
src/**/*.spec.ts
```

7. After this we're creating the basic folder structure and a dummy test to be able setup GitHub actions for the CI.

`Folder structure`

```
project-root/
├── node_modules/
└── src/
    ├── index.ts
    └── index.spec.ts
```

8. As already mentioned the core to test driven development is to implement the test first. Still, the go to strategy is to start with the stubs of methods or classes to test first. This helps you think about what to test and enables your IDE to provide code completion.

`index.ts`

```ts
export function hello(name: string): string {
  throw new Error('Not implemeted!');
}

export default hello;
```

This is just the stub for a simple hello world function.

8. Now we have to write the test.

`index.spec.ts`

```ts
import { expect } from 'chai';
import hello from './index';

describe('hello', () => {
  it('should return Hello <name>!', () => {
    const input = 'test';
    const result = hello(input);
    expect(result).to.equal('Hello test!');
  });
});
```

9. If we now run `npm test` in the console we should be able to see the failing test.

```sh
npm run test

> project@1.0.0 test /home/seidelm/Test/bla
> nyc mocha



  hello
    1) should return Hello <name>!


  0 passing (14ms)
  1 failing

  1) hello
       should return Hello <name>!:
     Error: Not implemeted!
      at Object.hello (src/index.ts:1:1561)
      at Context.<anonymous> (src/index.spec.ts:1:3932)



npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! project@1.0.0 test: `nyc mocha -r ts-node/register src/**/*.spec.ts src/**/**/*.spec.ts`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the project@1.0.0 test script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/seidelm/.npm/_logs/2020-01-23T15_26_28_534Z-debug.log
```

10. Now that we verified that the test fails, we can implement the function.

`index.ts`

```ts
export function hello(name: string): string {
  return `Hello ${name}!`;
}

export default hello;
```

11. If we run `npm test` now, we should see the successful test result.

```sh
npm run test

> project@1.0.0 test /home/seidelm/Test/bla
> nyc mocha



  hello
    ✓ should return Hello <name>!


  1 passing (9ms)
```

12. With our first running test we can create a GitHub actions pipeline to automate the testing. We also run the `npm run check` command to verify that our code is linted correctly. To display the code coverage we use the service of [codecov](https://codecov.io/). You have to add your GitHub repository there to generate a token. This token has to be setup as a secret called `CODECOV_TOKEN` in your GitHub repository.

`github/workflows/ci.yml`

```yaml
name: Node.js CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [8.x, 10.x, 12.x]

    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
      - uses: codecov/codecov-action@v1
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          file: ./coverage/lcov.info
      - run: npm run tslint
```

This concludes our first steps. We now have a project setup that is able to run tests. We continue with the [database](../database/index.md) part of this project.
