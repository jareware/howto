# Building TypeScript for AWS Lambda

This assumes the following `tsconfig.json`:

```js
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "allowJs": false,
    "downlevelIteration": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "types": [
      "node"
    ],
    "lib": [
      "es2015",
      "dom" // for Blob et al (see https://github.com/Microsoft/TypeScript/issues/14897)
    ]
  },
  "include": [
    "**/*.ts"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

At the time of writing, 6.10 is the most recent node runtime supported by Lambda. Eventually other `target` values can make more sense.

## Method 1: Compile to single file

You'll need the following dependencies:

    $ npm install --save browserify tsify

This is convenient if your function is going to be rather compact, as the result can be simply copy-pasted to the AWS GUI and you're off to the races. As an added benefit, since we use a bundler (`browserify`), anything in your `node_modules` that isn't explicitly `import`ed by you will be left out of the bundle. Keeping the function size small [results in faster cold-start times for your Lambda](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html#function-code).

The downside is that stacktraces will become less useful, since you'll lose filename and line number information in the process. Function and variable names will remain unmangled, however.

Example `index.ts` content:

```ts
declare var lambda: any; // this will be added by our build process

import { something } from './a-local-module';
import { somethingElse } from 'an-npm-module';

lambda.handler = (_event: any, _context: any, callback: any) => {
  callback(null, {
    statusCode: 200,
    body: 'index.ts says hi!',
    isBase64Encoded: false,
    headers: {
      'Content-Type': 'text/plain',
      'Cache-Control': 'no-cache',
    },
  });
};
```

To build this to plain-old JS:

    $ echo 'var lambda = exports;' > index.js && ./node_modules/.bin/browserify -p tsify --node index.ts >> index.js

Note the initial `var lambda = exports`; it allows the bundler to create an isolated function scope for all modules (including the entrypoint), while still allowing the Lambda runtime to call our entrypoint.

The resulting `index.js` is ready for copy-paste (or upload via CLI) to AWS Lambda.
