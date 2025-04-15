---
title: "Unit testing and mocking fs.ReadFileSync"
date: 2020-05-28T07:32:45
tags: ["jest", "unit testing", "twilio", "rabbitmq", "twilio taskrouter", "mssql", "sync", "twilio sync", "taskrouter"]
---

I'd just ran `npm run test` in a newly created package I'd added to a monorepo ([lerna](https://lerna.js.org/)) I'd created for a project I was working on that integrates with Twilio Sync, RabbitMQ, Twilio TaskRouter and MSSQL, and I go this:

```ps
*******************************consumers\packages\eda [CRMBROK-233 +0 ~2 -0 !]> npm run test

> @cf247/eda@1.0.2 test *******************************consumers\packages\eda
> jest

 FAIL  __tests__/eda.test.js
  ● Test suite failed to run

    ENOENT: no such file or directory, open '.env'

      2 | const fs = require('fs')
      3 | const dotenv = require('dotenv')
    > 4 | const envConfig = dotenv.parse(fs.readFileSync(`.env`))
        |                                   ^
      5 | for (const k in envConfig) {
      6 |     process.env[k] = envConfig[k]
      7 | }

      at Object.<anonymous> (lib/setenv.js:4:35)
      at Object.<anonymous> (lib/eda.js:1:1)

Test Suites: 1 failed, 1 total
Tests:       0 total
Snapshots:   0 total
Time:        1.772 s
Ran all test suites.
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! @cf247/eda@1.0.2 test: `jest`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the @cf247/eda@1.0.2 test script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.
npm WARN Local package.json exists, but node_modules missing, did you mean to install?

npm ERR! A complete log of this run can be found in:
npm ERR!     *******************************\npm-cache\_logs\2020-05-28T08_04_32_271Z-debug.log
*******************************consumers\packages\eda [CRMBROK-233 +0 ~3 -0 !]>
```

Not great but hey, first run and all!

The error message tells me everything I need to know:

```ps
ENOENT: no such file or directory, open '.env'

      2 | const fs = require('fs')
      3 | const dotenv = require('dotenv')
    > 4 | const envConfig = dotenv.parse(fs.readFileSync(`.env`))
```  

Which is that it can't find an `.env` file. And it wouldn't. Later refactoring would remove this file dependency but for now, all I want to do is to get my test working.

This _was_ the unit test code:

```js
'use strict'

const eda = require('..')

describe('@cf247/eda', () => {
    it('no tests', () => {
    })
})
```

This is the code from the module it was _importing_ via the `require('..')` statement:

```js
require('./setenv')
const amqp = require('amqplib/callback_api');

module.exports = (io, emitter) => {
    ...
```

The top line is importing code from this file:

_I've highlighted the problematic line of code_

{{< highlight js "linenos=table,hl_lines=3" >}}

const fs = require('fs')
const dotenv = require('dotenv')
const envConfig = dotenv.parse(fs.readFileSync(`.env`))
for (const k in envConfig) {
    process.env[k] = envConfig[k]
}

{{< / highlight >}}

The quickest (IMO) way to deal with this and move forward is to Mock the `fs` class.  I did this by included a jest module mock into my unit test file:

_I've highlighted the mock related code_

{{< highlight js "linenos=table,hl_lines=2 4-8" >}}

'use strict'
const fs = require('fs')
const eda = require('..')
jest.mock('fs', () => ({
    readFileSync: jest.fn((file_name) => {
        return []
    })
}))

describe('@cf247/eda', () => {
    it('no tests', () => {
    })
});
{{< / highlight >}}

What this does is, when the `readFileSync` class function is called, it always returns an empty array `[]`.  As the unit code does not have a dependency on environment variables, this mocked response will work fine.