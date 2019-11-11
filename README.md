# nem2-hd-wallets

[![npm version](https://badge.fury.io/js/nem2-hd-wallets.svg)](https://badge.fury.io/js/nem2-hd-wallets)
[![Build Status](https://travis-ci.org/nemfoundation/nem2-hd-wallets.svg?branch=master)](https://travis-ci.org/nemfoundation/nem2-hd-wallets)
[![Slack](https://img.shields.io/badge/chat-on%20slack-green.svg)](https://nem2.slack.com/messages/CB0UU89GS//)

:warning: **This package is currently still in development, please do not use in production.** *The author of this package cannot be held responsible for any loss of money or any malintentioned usage forms of this package. Please use this package with caution.*

NEM HD Wallets generator to generate hyper-deterministic wallets for the Catapult (NEM2) platform.

This is a PoC to validate the proposed [NIP? Multi-Account Hierarchy for Deterministic Wallets](https://github.com/nemtech/NIP/issues/12). When stable, the repository will be moved to the [nemtech](https://github.com/nemtech) organization.

## Installation

`npm install nem2-hd-wallets`

## Examples

### Generating a mnemonic pass phrase

```typescript
import {MnemonicPassPhrase} from 'nem2-hd-wallets';

// random 24-words mnemonic
const mnemonic = MnemonicPassPhrase.createRandom();

// random 12-words mnemonic
const mnemonic = MnemonicPassPhrase.createRandom('english', 128);

// random 24-words mnemonic with french wordlist
const mnemonic = MnemonicPassPhrase.createRandom('french');

// random 24-words mnemonic with japanese wordlist
const mnemonic = MnemonicPassPhrase.createRandom('japanese');
```

### Generating a password-protected mnemonic pass phrase seed (for storage)

```typescript
import {MnemonicPassPhrase} from 'nem2-hd-wallets';

// Example 1: generate password-protected seed for random pass phrase
const mnemonic = MnemonicPassPhrase.createRandom();
const secureSeedHex = mnemonic.toSeed('your-password');

// Example 2: empty password for password-protected seed
const mnemonic = MnemonicPassPhrase.createRandom();
const secureSeedHex = mnemonic.toSeed(); // omit password means empty password: ''
```

### Generating a root (master) extended key

```typescript
import {MnemonicPassPhrase} from 'nem2-hd-wallets';

// Example 1: generate BIP32 master seed for random pass phrase
const mnemonic = MnemonicPassPhrase.createRandom();
const bip32Seed = mnemonic.toSeed();

// Example 2: generate BIP32 master seed for known pass phrase
const words = 'alpha pattern real admit vacuum wall ready code '
            + 'correct program depend valid focus basket whisper firm '
            + 'tray fit rally day dance demise engine mango';
const mnemonic = new MnemonicPassPhrase(words);

 // the following seed can be used with `ExtendedKey.createFromSeed()`
const bip32Seed = mnemonic.toSeed();
```

### Generating an extended _private_ key from a mnemonic pass phrase

```typescript
import {MnemonicPassPhrase, ExtendedKey} from 'nem2-hd-wallets';

// using BIP39 mnemonic pass phrase for BIP32 extended keys generation
const mnemonic = MnemonicPassPhrase.createRandom();
const bip32Seed = mnemonic.toSeed();
const bip32Node = ExtendedKey.createFromSeed(bip32Seed.toString('hex'));

// the extended private key (never share, base of private keys tree)
const xprvKey = bip32Node.toBase58();
```

### Generating an extended _public_ key from a mnemonic pass phrase

```typescript
import {MnemonicPassPhrase, ExtendedKey} from 'nem2-hd-wallets';

// using BIP39 mnemonic pass phrase for BIP32 extended keys generation
const mnemonic = MnemonicPassPhrase.createRandom();
const bip32Seed = mnemonic.toSeed();
const bip32Node = ExtendedKey.createFromSeed(bip32Seed.toString('hex'));

// the extended public key (base of public keys tree)
const xpubKey = bip32Node.getPublicNode().toBase58();
```

### Derive child path of an extended key

```typescript
import {MnemonicPassPhrase, ExtendedKey} from 'nem2-hd-wallets';

// using BIP39 mnemonic pass phrase for BIP32 extended keys generation
const mnemonic = MnemonicPassPhrase.createRandom();
const bip32Seed = mnemonic.toSeed();
const bip32Node = ExtendedKey.createFromSeed(bip32Seed.toString('hex'));

// derive BIP44 tree root
const bip44Root = bip32Node.derivePath("m/44'");

// the extended private key (never share, base of private keys tree)
const xprvKey = bip32Node.toBase58();

// the extended public key (base of public keys tree)
const xpubKey = bip32Node.getPublicNode().toBase58();
```

### Derive default wallet from a mnemonic pass phrase

```typescript
import {MnemonicPassPhrase, ExtendedKey} from 'nem2-hd-wallets';

// using BIP39 mnemonic pass phrase for BIP32 extended keys generation
const mnemonic = MnemonicPassPhrase.createRandom();
const bip32Seed = mnemonic.toSeed();
const bip32Node = ExtendedKey.createFromSeed(bip32Seed.toString('hex'));

// derive default wallet path "m/44'/43'/0'/0/0"
const defaultWallet = bip32Node.derivePath("m/44'/43'/0'/0'/0'");

// the extended private key (never share, base of private keys tree)
const xprvKey = defaultWallet.toBase58();

// the extended public key (default wallet base of public keys tree)
const xpubKey = defaultWallet.getPublicNode().toBase58();
```

### Derive second account from a mnemonic pass phrase

```typescript
import {MnemonicPassPhrase, ExtendedKey} from 'nem2-hd-wallets';

// using BIP39 mnemonic pass phrase for BIP32 extended keys generation
const mnemonic = MnemonicPassPhrase.createRandom();
const bip32Seed = mnemonic.toSeed();
const bip32Node = ExtendedKey.createFromSeed(bip32Seed.toString('hex'));

// derive default wallet path "m/44'/43'/1'/0/0"
const secondWallet = bip32Node.derivePath("m/44'/43'/1'/0'/0'"); // second hardened account

// the extended private key (never share, base of private keys tree)
const xprvKey = secondWallet.toBase58();

// the extended public key (default wallet base of public keys tree)
const xpubKey = secondWallet.getPublicNode().toBase58();
```

### Generating a hyper-deterministic wallet (CATAPULT compatible)

```typescript
const xkey = ExtendedKey.createFromSeed('000102030405060708090a0b0c0d0e0f');
const wallet = new Wallet(xkey);

// get master account
const masterAccount = wallet.getAccount();

// get DEFAULT ACCOUNT
const defaultAccount = wallet.getChildAccount();

// derive specific child path
const childAccount = wallet.getChildAccount('m/44\'/43\'/0\'/0\'/0\'');

// get read-only wallet
const readOnlyWallet = new Wallet(xkey.getPublicNode());
const readOnlyAccount = readOnlyWallet.getPublicAccount();

// get read-only DEFAULT ACCOUNT
const readOnlyDefaultAccount = readOnlyWallet.getChildPublicAccount();
```

### Signing with a hyper-deterministic wallet (CATAPULT compatible)

```typescript
    TBD
```

## Changelog

Important versions listed below. Refer to the [Changelog](CHANGELOG.md) for a full history of the project.

- [0.4.0](CHANGELOG.md#v040) - 2019-05-08
- [0.3.1](CHANGELOG.md#v031) - 2019-04-27
- [0.3.0](CHANGELOG.md#v030) - 2019-04-26
- [0.2.0](CHANGELOG.md#v020) - 2019-04-20
- [0.1.0](CHANGELOG.md#v010) - 2019-03-08

## License

Licensed under the [BSD-2 License](LICENSE).
