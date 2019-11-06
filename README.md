## LNRPC

[![CircleCI](https://img.shields.io/circleci/project/github/RadarTech/lnrpc/master.svg?style=flat)](https://circleci.com/gh/RadarTech/lnrpc)
[![Known Vulnerabilities](https://snyk.io/test/github/RadarTech/lnrpc/badge.svg?targetFile=package.json)](https://snyk.io/test/github/RadarTech/lnrpc?targetFile=package.json)
[![NPM Version](https://img.shields.io/npm/v/@radar/lnrpc.svg?style=flat)](https://www.npmjs.com/package/@radar/lnrpc)
[![License](https://img.shields.io/github/license/radartech/lnrpc.svg?style=flat)](https://img.shields.io/github/license/radartech/lnrpc.svg?style=flat)

Maintained fork of [lnrpc](https://github.com/Matt-Jensen/lnrpc) adding support for generating typescript type definitions.

### Features
- Auto-generates [lnd/lnrpc](https://github.com/lightningnetwork/lnd/tree/master/lnrpc) client and typescript definitons based on target release tag
- Wraps requests in promises
- Easily setup SSL and Macaroons
- Instantiates all gRPC services
- uint64/int64 types cast to string

### Installation
```sh
npm install @radar/lnrpc
# OR
yarn add @radar/lnrpc
```

Install [lnd](https://github.com/lightningnetwork/lnd/blob/master/docs/INSTALL.md) before using this project and ensure you have an lnd instance running with `--no-macaroons`, unless you provide macaroon authentication to your lnrpc instance when created.

### Change LND gRPC release version
To change the gRPC definitions used for all auto-generated types and RPC methods edit the `config.lnd_release_tag` value in `package.json` to the desired [LND release tag](https://github.com/lightningnetwork/lnd/releases) and run the following:

```sh
npm run update-protos
# OR
yarn update-protos

# AND

npm run generate
# OR
yarn generate
```
Newly generated type definitions will be available in `./generated`.
You can now delete the old proto file inside the lnd directory.
Use the generated type definitions to update the types in `lnrpc.d.ts`.


### Usage

Connecting to an lnd instance at `localhost:10001`.

```typescript
import createLnRpc, {
  Invoice,
  InvoiceSubscription,
  WalletBalanceResponse
} from '@radar/lnrpc';

(async () => {
  const lnRpcClient = await createLnRpc();

  // All requests are promisified and typed
  const { confirmedBalance } = await lnRpcClient.walletBalance();

  // ...and you're off!
  console.log(confirmedBalance);

  // subscribe to LND server events
  const subscriber = await lnRpcClient.subscribeInvoices();
  subscriber.on('data', invoice => {
    console.log(invoice); // do something with invoice event
  });
})();
```

### Options

```typescript
import createLnRpc, {
  GetInfoResponse
} from '@radar/lnrpc';

(async () => {
  const lnRpcClient = await createLnRpc({
    /*
     By default lnrpc connects to `localhost:10001`,
     however we can point to any host.
     */
    server: '173.239.209.2:3001',

    /*
     By default  lnrpc looks for your tls certificate at:
     `~/.lnd/tls.cert`, unless it detects you're using macOS and
     defaults to `~/Library/Application\ Support/Lnd/tls.cert`
     however you can configure your own SSL certificate path like:
     */
    tls: './path/to/tls.cert',

    /*
     You can also provide a TLS certificate directly as a string
     (Just make sure you don't commit this to git).
     Overwrites: `tls`
     */
    cert: process.env.MY_SSL_CERT,

    /*
     Optional path to configure macaroon authentication
     from LND generated macaroon file.
     */
    macaroonPath: './path/to/data/admin.macaroon',

    /*
     Optional way to configure macaroon authentication by
     passing a hex encoded string of your macaroon file.
     Encoding: `xxd -ps -u -c 1000 ./path/to/data/admin.macaroon`
     Details: https://github.com/lightningnetwork/lnd/blob/dc3db4b/docs/macaroons.md#using-macaroons-with-grpc-clients
     */
    macaroon: process.env.MY_MACAROON_HEX,
  });

  try {
    const getInfoResponse = await lnRpcClient.getInfo();
    console.log(getInfoResponse);
  } catch (error) {
    console.error(error);
  }
})();
```

### API Reference

[All lnrpc methods documentation can be found here](http://api.lightning.community).

### Usage With Typescript

You must enable the `esModuleInterop` Typescript compiler option if you would like to use a default import.

This can be accomplished by setting `"esModuleInterop": true` in your `tsconfig.json`.

Import Example:
```typescript
import createLnRpc from '@radar/lnrpc';
```

Otherwise, you can import `createLnRpc` using the `as *` syntax.

Import Example:
```typescript
import * as createLnRpc from '@radar/lnrpc';
```

### Usage With BTCPayServer

By default lnrpc assumes SSl certificate pinning.
In order to use lnrpc with a service (like BTCPayServer) which manages your certification,
you'll have to opt to disable certificate pinning by passing `{ tls: false }` within your lnrpc configuration.

### Contributors

To develop on the project please run:

```sh
git clone git@github.com:RadarTech/lnrpc.git && cd $_


npm install
npm start
# OR
yarn
yarn start
```

### License

This project is licensed under the MIT License.
