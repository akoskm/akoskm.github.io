---
title: How to tweak your existing Mocha setup and start using Jest
layout: post
type: post
date: 2020-01-22 00:00:01
featured: yes
---

Jest is a simple testing library that works with zero configuration. Because of its great API - if you're already using Mocha - with a few Find & Replace and minimal effort you can move to Jest, without having to rewrite your tests.

In this post I'll show you how I moved to Jest from an existing Mocha configuration. How I tweak the existing environment setup, the Babel configuration, and refactored some functions.

If you're new to Jest, don't forget to check out their [Getting Started](https://jestjs.io/docs/en/getting-started) page as well.

I personally got hooked on Jest when I realized it comes with a configuration option [testEnvironment](https://jestjs.io/docs/en/configuration#testenvironment-string), that gives you a test environment with a JSDOM that works out of the box, without actually configuring JSDOM. I've lost count, across all projects, how many times I wanted to update the testing environment and ended up not doing it because I couldn't get JSDOM to work with my existing tests.

# Configuration

## Mocha
I did it through `mocha.opts`, the contents are important because I'm going to reference some of these files later:

```
--reporter spec
--ui bdd
--colors
--file test_helpers/babel_loader.js
--file test_helpers/jsdom_setup.js
--file test_helpers/test_setup.js
```

and this is how I used to run Mocha:
```
mocha --opts mocha.opts
```


## Jest
You can configure Jest with `jest.config.js`, a js or JSON file with the `--config` flag, and in `package.json`, see [Configuring Jest](https://jestjs.io/docs/en/configuration.html). I suggest starting with `package.json`. Once the configuration gets more complicated, move it to a separate file.
In `package.json` you should use the key `"jest"` on top level, like this:

```
{
  "name": "my-project",
  "jest": {
    "verbose": true
  }
}
```
now you should be able to run Jest as:

```
jest
```

# Setup

This is the only place where depending on what other libraries you use in your tests, you might want to tweak the setup a bit.

## Mocha
In my case `test_helpers/test_setup.js` had the environment setup.
If you're using Enzyme or Sinon for example, this is the place where you keep that configuration as well.
Let's say your file has something similar:
```
global.sinonSandbox = sinon.createSandbox();
afterEach(function() {
  return global.sinonSandbox.restore();
});
```
Then you should be able to reuse this.

## Jest
Jest does its environment setup with [`setupFilesAfterEnv`](https://jestjs.io/docs/en/configuration#setupfilesafterenv-array).
This option tells Jest which file(s) to run before each test. Put this into `package.json` after `verbose`:

```
{
  "name": "my-project",
  "jest": {
    "verbose": true,
    "setupFilesAfterEnv": ["<rootDir>/test_helpers/test_setup.js"]
  }
}
```

Side-note: this doesn't exactly match the behavior of the Mocha configuration. Now we create a sandbox before each test, while earlier we created it only when Mocha first loaded the file. However, there should be no difference in the way how Sinon interacts in your tests.

# Hooks

Mocha operates with a wide variety of hooks. I mostly used `describe()`, `context()`, `it()`, `before()`, `after()`, `beforeEach()`, and `afterEach()`.

Jest uses a bit different naming, in general, you're safe to replace:

`before` ➡️ `beforeAll`

`after` ➡️ `afterAll`

`context` doesn't have an equivalent in Jest, so I decided to use `describe`.
You can find & replace all occurrences of `context`, `before`, and `after` or put the following code into `test_helpers/test_setup.js`:

```
// mocha keyword mappings
window.context = describe;
window.before = beforeAll;
window.after = afterAll;
```

# Function parameters

With Mocha, you were able to give hooks a description:
```
beforeEach(description, fn)
afterEach(description, fn)
```

in Jest, hooks only accept a function and an optional timeout:

```
beforeEach(fn, timeout)
afterEach(fn, timeout)
```

If your `beforeEach` blocks looked something like this:

```
beforeEach('login', () => {
```

to strip out the descriptions from every `beforeEach` run the following regular expression in your editor:

```
beforeEach\([^(]*
```
and replace it with
```
beforeEach(
```


# Babel

## Mocha

We set up Babel in `test_helpers/babel_loader.js`:

```
// Run before any test files are loaded to use Babel for subsequent parsing
require('@babel/register')({
  ignore: [/node_modules\//],
  extensions: ['.js', '.jsx', '.ts', '.tsx'],
});
```

## Jest

Install `babel-jest`:
```
yarn add --dev babel-jest
```

then add `transfer` to `package.json` that now should look like this:
```
{
  "name": "my-project",
  "jest": {
    "verbose": true,
    "setupFilesAfterEnv": ["<rootDir>/test_helpers/test_setup.js"],
    transform: {
      '^.+\\.[t|j]sx?$': 'babel-jest',
    }
  }
}
```

See more on [using Babel with Jest](https://jestjs.io/docs/en/getting-started#using-babel).

# Conclusion

At this point, you're pretty much ready to run your existing tests with Jest.

What I liked about Jest is the simple configuration and the "work great by default" philosophy.

Even when installed in an empty package and run, it gives you a meaningful output:

```
% jest
No tests found, exiting with code 1
Run with `--passWithNoTests` to exit with code 0
In /Users/akoskm/Projects/test
  2 files checked.
  testMatch: **/__tests__/**/*.[jt]s?(x), **/?(*.)+(spec|test).[tj]s?(x) - 0 matches
  testPathIgnorePatterns: /node_modules/ - 2 matches
  testRegex:  - 0 matches
Pattern:  - 0 matches
```

It doesn't ask for configuration files or necessary parameters it simply works.

This is different from how we used to interact with libraries and, in my opinion, a philosophy we should follow when building new tools.

What framework you use to test your application?
You are in the middle of a similar migration and stuck? Let me know in the comments or find me on <a href="https://twitter.com/akoskm">Twitter</a>.