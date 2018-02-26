# Running npm scripts in parallel

When using `npm run` as your build tool, waiting for several commands to complete serially sometimes ends up wasting valuable time. Waiting for that [husky precommit hook](https://github.com/typicode/husky) to run over and over again gets tedious.

There's a [bunch of ways to achieve this in the shell](https://www.cyberciti.biz/faq/how-to-run-command-or-code-in-parallel-in-bash-shell-under-linux-or-unix/), but none of them are without their cons, and none is particularly portable. Luckily, a node version (or a very close copy) of the GNU `parallel` command exists on npm: [`parallel`](https://www.npmjs.com/package/parallel).

## Before

Assuming you have something like this in your `package.json`:

```json
{
  "scripts": {
    "lint-versions": "check-node-version --package",
    "lint-ts": "tsc --noEmit --project .",
    "lint-prettier": "prettier --list-different '**/*.ts'",
    "test": "mocha --require ts-node/register **/*.spec.ts",
    "precommit": "npm run lint-versions && npm run lint-ts && npm run lint-prettier && npm run test"
  },
  "devDependencies": {
    "...": "..."
  }
}
```

Running `npm run precommit` runs each step **serially**, and exits non-zero if any step fails.

## After

The above can be made run in parallel by `npm install --save parallel` and:

```json
{
  "scripts": {
    "lint-versions": "check-node-version --package",
    "lint-ts": "tsc --noEmit --project .",
    "lint-prettier": "prettier --list-different '**/*.ts'",
    "test": "mocha --require ts-node/register **/*.spec.ts",
    "precommit": "echo lint-versions lint-ts lint-prettier test | parallel --delimiter ' ' --trim npm run --silent {}"
  },
  "devDependencies": {
    "...": "...",
    "parallel": "^1.2.0"
  }
}
```

Running `npm run precommit` runs each step **in parallel**, and exits non-zero if any step fails.

The amount of parallelization default to the number of CPU cores available. There's [other handy options](https://github.com/flesler/parallel#options), too.

## See also

If you're looking for a higher-end solution, with output prefixing and everything, look no further than [kimmobrunfeldt/concurrently](https://github.com/kimmobrunfeldt/concurrently).
