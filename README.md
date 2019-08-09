**loopcall /**
[Readme](README.md)
• [Contributing](CONTRIBUTING.md)
• [Changelog](CHANGELOG.md)

# Readme

![Licence](https://img.shields.io/github/license/cosmic-plus/js-loopcall.svg)
[![Dependencies](https://img.shields.io/david/cosmic-plus/js-loopcall)](https://david-dm.org/cosmic-plus/js-loopcall)
![Vulnerabilities](https://img.shields.io/snyk/vulnerabilities/npm/@cosmic-plus/loopcall.svg)
![Size](https://img.shields.io/bundlephobia/minzip/@cosmic-plus/loopcall.svg)
![Downloads](https://img.shields.io/npm/dt/@cosmic-plus/loopcall.svg)

**Loopcall** is a tiny library that enable unlimited complex queries to Horizon
nodes. It takes a _CallBuilder_ and accept a few optional parameter. It returns
an array of records similar to the ones returned by `CallBuilder.call()`.

## Installation

### NPM/Yarn

- NPM: `npm install @cosmic-plus/loopcall`
- Yarn: `yarn add @cosmic-plus/loopcall`

In your script: `const loopcall = require("@cosmic-plus/loopcall")`

### Bower

`bower install cosmic-plus-loopcall`

In your HTML:

```HTML
<script src="./bower_components/stellar-sdk/stellar-sdk.min.js"></script>
<script src="./bower_components/cosmic-plus-loopcall/loopcall.js"></script>
```

### HTML

n your HTML:

```HTML
<script src="https://cdn.cosmic.plus/stellar-sdk"></script>
<script src="https://cdn.cosmic.plus/loopcall@1.x"></script>
```

_Note:_ For production release it is advised to serve your own copy of the
library.

## Usage

### Fetch more than 200 records

To get an arbitrary amount of record:

```js
const callBuilder = server.operations().order("asc")
const the2000FirstOperations = await loopcall(callBuilder, { limit: 2000 })
```

To get all existing records (take care with that one!):

```js
const callBuilder = server.transactions().forAccount("GDE...YBX")
const allTransactions = await loopcall(callBuilder)
```

### Conditional Break

To stop fetching records when a condition is met:

```js
const callBuilder = server.transactions().forAccount("GDE...YBX")
const thisYearTransactions = await loopcall(callBuilder, {
  breaker: record => record.created_at.substr(0, 4) < 2018
})
```

`breaker` is a _Function_ that is called over each fetched record. Once it
returns `true`, the fetching loop breaks and the record that triggered the break
is discarded.

### Conditional Filtering

To filter records by condition:

```js
const callBuilder = server.transactions().forAccount("GDE...YBX")
const transactionsWithoutMemo = await loopcall(callBuilder, {
  filter: record => !record.memo
})
```

`filter`is a _Function_ that is called over each fetched record. When provided,
the records are added to the query results only when it returns `true`.

### Iterating over records on-the-fly

In some situations waiting for the result to be concatenated is not an option.
`breaker` offer the possibility of iterating over records while they are
fetched:

```js
const callBuilder = server.transactions()

async function showTxUntilScreenIsFilled(record) {
  displayTxUsingRecord(record)
  await endOfPageReached()
}

loopcall(callBuilder, { breaker: showTxUntilScreenIsFilled })
```

This example shows a part of the code to implement unlimited scrolling on a
webpage showing the last transactions on a Stellar network. As
`showTxUntilScreenIsFilled` never returns `true`, the loop never breaks.

### Combining parameters

All those parameters may be combined together:

```js
const callBuilder = server.operations().order("asc")

function iterateOver1000RecordsMax() {
  let counter = 0
  return function() {
    counter++
    if (counter > 1000) return true
  }
}

const the20firstAccountCreations = await loopcall(callBuilder, {
  limit: 20,
  breaker: iterateOver1000RecordsMax(),
  filter: record => record.type === "create_account"
})
```

When both are provided, `breaker` is called before `filter`.
