---
title: Replacing Mocha with Jest
layout: post
type: post
date: 2020-01-22 00:00:01
---

When everything works and clients are happy, sometimes we forget to upgrade stuff. That's how a jsdom version from 2016 ended up in our testing stack.

If you used Mocha earlier for testing web apps, you already know that you have to set up jsdom manually.

I decided to replace Mocha with Jest while I was searching jsdom's issue tracker and run into a post.

It was about changing something in the preconfigured jsdom that comes with Jest.

I was like, wait, you don't configure jsdom anymore?

It turned out that Jest has a configuration option [testEnvironment](https://jestjs.io/docs/en/configuration#testenvironment-string), that gives your test environment, a fully configured and working version of jsdom.

From this point, it was all downhill. I had the whole day allocated for the upgrade but I did it in a few hours.

I'll show you how I tweaked our Mocha setup to work with Jest. Where was the environment setup moved which functions needed rename or refactoring and how the Babel configuration was updated.

# Configuration

## Mocha
There are several ways to [configure Mocha]([https://mochajs.org/#configuring-mocha-nodejs]). The `mocha.opts` file we used is already deprecated, but later I'm going to reference its contents:

```
--reporter spec
--ui bdd
--colors
--file test_helpers/babel_loader.js
--file test_helpers/jsdom_setup.js
--file test_helpers/test_setup.js
```

and this is how we used to run Mocha:
```
mocha --opts mocha.opts
```


## Jest
Jest can be configured with `jest.config.js`, any js or JSON file with the `--config` flag, and in `package.json`, see [Configuring Jest](https://jestjs.io/docs/en/configuration.html). I suggest starting with package.json. Once the configuration gets more complicated, move it to a separate file.
In `package.json` you should use the key "jest" on top level, like this:

```
{
  "name": "my-project",
  "jest": {
    "verbose": true
  }
}
```
finally run Jest as:

```
jest
```

# Initializing environments

## Mocha
In our case `test_helpers/test_setup.js` had the environment setup.

To keep things simple, let's say we needed only a [Sinon sandbox](https://sinonjs.org/releases/latest/sandbox/).

If you're using React, this is the place where you keep your Enzyme configuration.

```
global.sinonSandbox = sinon.createSandbox();
afterEach(function() {
  return global.sinonSandbox.restore();
});
```

## Jest
We have a similar approach here with [`setupFilesAfterEnv`](https://jestjs.io/docs/en/configuration#setupfilesafterenv-array).
This option tells Jest which file(s) to run before each test. Put this into `package.json`:

```
{
  "name": "my-project",
  "jest": {
    "verbose": true,
    "setupFilesAfterEnv": ["<rootDir>/test_helpers/test_setup.js"]
  }
}
```

Sidenote: this doesn't exactly match the behavior of the Mocha configuration. Now we create a sandbox before each test, while earlier we created it only when Mocha first loaded the file. However, there should be no differences in the way you interact with Sinon in your tests.

# Hooks

Mocha operates with a wide variety of hooks. We mostly used `describe()`, `context()`, `it()`, `before()`, `after()`, `beforeEach()`, and `afterEach()`.

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

To strip out the descriptions from every `beforeEach` run the following regular expression in your editor:
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

and add the following to `package.json`:
```
// A map from regular expressions to paths to transformers
transform: {
  '^.+\\.[t|j]sx?$': 'babel-jest',
}
```

See more on [using Babel with Jest](https://jestjs.io/docs/en/getting-started#using-babel).

# Conclusion

At this point, you're pretty much ready to run the tests with Jest.

What I liked about Jest is the simple configuration and the "work great by default" philosophy.

When you install Jest in a new package and run it, it'll give you a meaningful output:

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

It doesn't ask for configuration files or necessary parameters it just works.

This is different from how we used to interact with libraries and, in my opinion, a philosophy we should follow when building new tools.