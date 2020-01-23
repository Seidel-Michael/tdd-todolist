# Initial backend project setup

This project setup utilizes GitHub actions for the continuous integration.
If you want to use this, you have to set up a GitHub project first and clone it.
Otherwise, you can use any version control system or none. 

1. Change to the folder you want to set up the backend project in.
2. Run the `npm init` command to set up a node project. You can use the default value or customize them to your desire. We're gonna change the important settings later on.

```sh
npm init
```

3. With the following command we'll install and setup all the basic npm packages we need to get started with the test driven development. 

```sh 
npm install -D typescript tslint mocha @types/mocha chai @types/chai ts-node nyc source-map-support @types/node sinon @types/sinon chai-as-promised @types/chai-as-promised 
./node_modules/.bin/tsc --init
./node_modules/.bin/tslint --init
```

4. We want to slightly modify our typescript compiler options. We need to activate source maps for the code coverage tool. We also want to change the output directory for a cleaner project folder. Lastly we exclude our tests from the compiler.

`tsconfig.json`
```json
{
  "exclude" : ["src/**/*.spec.ts"],
  "compilerOptions": {

...

    // "declarationMap": true,                /* Generates a sourcemap for each corresponding '.d.ts' file. */
    "sourceMap": true,                        /* Generates corresponding '.map' file. */
    // "outFile": "./",                       /* Concatenate and emit output to single file. */
    "outDir": "./dist",                       /* Redirect output structure to the directory. */
    // "rootDir": "./",  

...
```

5. We also want to change our `package.json` file. We're adding some script and the configuration for the code coverage reporter.

`package.json`
```json
...

  "description": "",
  "main": "dist/src/index.js",
  "scripts": {
    "test": "nyc mocha -r ts-node/register src/**/*.spec.ts src/**/**/*.spec.ts",
    "start": "tsc && node dist/src/index.js",
    "lint": "tslint -p tsconfig.json"
  },
  "nyc": {
    "extension": [
      ".ts"
    ],
    "exclude": [
      "**/*.d.ts"
    ],
    "reporter": [
      "lcov"
    ],
    "all": true
  },

...
```

6. After this we're creating the basic folder structure and a dummy test to be able setup GitHub actions for the CI.

`Folder structure`
```
...

node_modules
src
|- index.ts
|- index.spec.ts

...
```

7. As already mentioned the core to test driven development is to implement the test first. Still, my go to strategy is to start with the stubs of my method or class to test first. This helps me to think about what to test and my IDE can provide me with code completion.

`index.ts`
```ts
export function hello(name: string): string
{
    throw new Error('Not implemeted!');
}

export default hello;
```

This is just the stub for a simple hello world function.


8. Now we have to write the test.

`index.spec.ts`
```ts
import {expect} from 'chai';
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
> nyc mocha -r ts-node/register src/**/*.spec.ts src/**/**/*.spec.ts



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

10. Now that we verified, that the test failed first, we can implement the function.

`index.ts`
```
export function hello(name: string): string
{
    return `Hello ${name}!`;
}

export default hello;
```

11. If we run `npm test` now we should see the successful test result.

```sh
npm run test

> project@1.0.0 test /home/seidelm/Test/bla
> nyc mocha -r ts-node/register src/**/*.spec.ts src/**/**/*.spec.ts



  hello
    ✓ should return Hello <name>!


  1 passing (9ms)
```

12. With our first running test we can create a GitHub actions pipeline to automate the testing. We also run the `npm run lint` command to verify that our code is linted correctly. To display the code coverage we use the service of [codecov](https://codecov.io/). You have to add your GitHub repository there to generate a token. This token has to be setup as a secret called `CODECOV_TOKEN` in your GitHub repository.

`github/workflows/ci.yml` 
```
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