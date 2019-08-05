# Lighthouse Lambda Layer

A Lambda layer with all the required dependencies to run Google Lighthouse.

## Prerequisites

[Nodejs](https://nodejs.org/en/) (at least version 10)

[Yarn](https://yarnpkg.com/lang/en/)

Amazon AWS account and `awscli` installed and configured: <https://aws.amazon.com/getting-started/>

Serverless [CLI](https://serverless.com/framework/docs/getting-started/)

## Setup

```bash
yarn
```

## Deploy

```bash
yarn deploy
```

### Usage

In your `serverless.yml`:

```yaml
functions:
  functionThatUsesLighthouse:
    layers:
      - layerArn # You'll have this after you run the deploy command
```

In you Lambda function:

```ts
const lighthouse = require('lighthouse');
const log = require('lighthouse-logger');
const chromeLauncher = require('chrome-launcher');

// this lets us support invoke local
const chromePath = process.env.IS_LOCAL
  ? undefined
  : '/opt/nodejs/node_modules/chrome-aws-lambda/bin/chromium';

const chromeFlags = [
  '--headless',
  '--disable-dev-shm-usage',
  '--disable-gpu',
  '--no-zygote',
  '--no-sandbox',
  '--single-process',
  '--hide-scrollbars',
];

// utility function to run lighthouse
const runLighthouse = async url => {
  let chrome = null;
  try {
    chrome = await chromeLauncher.launch({ chromeFlags, chromePath });

    const options = {
      port: chrome.port,
      logLevel: 'info',
    };

    log.setLevel(options.logLevel);

    const results = await lighthouse(url, options);
    return results;
  } finally {
    if (chrome) {
      await chrome.kill();
    }
  }
};

// do something with runLighthouse
```
