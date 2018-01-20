# Building TypeScript for AWS Lambda

There's two main methods for preparing TypeScript to be ran on [AWS Lambda](https://aws.amazon.com/lambda/).

This guide assumes you've already created the function, either via CLI or GUI.

## Method 1: Build to single file

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

The resulting `index.js` is ready for copy-paste to AWS Lambda. You can also upload it via the CLI:

    $ brew install awscli

If you don't have a global config for the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html), you can easily set up a temporary one:

```
$ read AWS_ACCESS_KEY_ID # type in your key ID without it going to your shell history
$ read -s AWS_SECRET_ACCESS_KEY # same as above, also hiding any input
$ AWS_DEFAULT_REGION=eu-central-1 # make sure this is the region you want
$ env | grep AWS # to verify your config
```

Then, compress & upload:

    $ zip lambda.zip index.js
    $ aws lambda update-function-code --function-name YourFunctionName --zip-file fileb://lambda.zip

## Method 2: Build individual files

You'll need the following dependencies:

    $ npm install --save typescript

This has the upside of being simpler, as our example `index.ts` no longer needs any Clever Hacks :tm: for attaching `handler` to the global `exports` of the entrypoint module:

```ts
import { something } from './a-local-module';
import { somethingElse } from 'an-npm-module';

exports.handler = (_event: any, _context: any, callback: any) => {
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

Your stack traces will be more straightforward, since modules (both yours and npm ones) stay in their own files. Also, any non-JS assets you may have will be included just as everything else.

The downside is that — depending on how much stuff you have in your `node_modules` — the resulting bundle can easily grow quite large. As mentioned before, [smaller builds are better](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html#function-code). You can exclude unnecessary files by playing with the `zip` command, but it can get unwieldy fast.

Anyway, to build & compress:

    $ ./node_modules/.bin/tsc -p . && zip -r lambda.zip .

The same as above, the resulting `lambda.zip` can be either uploaded via the GUI, or the CLI.

## Reference TS config

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
