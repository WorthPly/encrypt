# encrypt

[![Pub Package](https://img.shields.io/pub/v/encrypt.svg)](https://pub.dartlang.org/packages/encrypt)
[![Build Status](https://travis-ci.org/leocavalcante/encrypt.svg?branch=master)](https://travis-ci.org/leocavalcante/encrypt)
[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donate_SM.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=E4F45BFVMFVQW)

A set of high-level APIs over PointyCastle for two-way cryptography.

> Looking for password hashing? Please, visit [password](https://github.com/leocavalcante/password-dart).

## API Overview

### `Encrypter(Algorithm algo)`
Acts like a Adapter interface for any algorithm. Exposes:
- `Encrypted encrypt(String text)` that encrypts the given plain-text.
- `String decrypt(Encrypted encrypted)` that decrypts the given `Encrypted` value.
- `String decrypt16(String encoded)` sugar for `decrypt(Encrypted.fromBase16(encoded))`.
- `String decrypt64(String encoded)` sugar for `decrypt(Encrypted.fromBase64(encoded))`.

### `Encrypted(Uint8List bytes)`
Wraps the encrypted bytes. Exposes:
- `Encrypted.fromBase16(String encoded)` that creates an Encrypted object from a hexdecimal string.
- `Encrypted.fromBase64(String encoded)` that creates an Encrypted object from a Base64 string.
- `String base16` that returns a hexdecimal representation of the bytes.
- `String base64` that returns a Base64 representation of the bytes.
- `Uint8List bytes` that returns raw bytes.

### `IV(UintList bytes)`
Represents an Initialization Vector https://en.wikipedia.org/wiki/Initialization_vector. Exposes:
- `IV.fromBase16(String encoded)` that creates an IV from a hexdecimal string.
- `IV.fromBase64(String encoded)` that creates an IV from a Base64 string.
- `IV.fromLength(int length)` that is a sugar for `IV(Uint8List(length))`.

## Algorithms
Current status is:
- AES with SIC mode and PKCS7 padding
- RSA with PKCS1 encoding
- Salsa20

## Usage

### Symmetric

#### AES
```dart
import 'package:encrypt/encrypt.dart';

void main() {
  final plainText = 'Lorem ipsum dolor sit amet, consectetur adipiscing elit';
  final key = Key.fromUtf8('my 32 length key................');
  final iv = IV.fromLength(16);

  final encrypter = Encrypter(AES(key, iv));

  final encrypted = encrypter.encrypt(plainText);
  final decrypted = encrypter.decrypt(encrypted);

  print(decrypted); // Lorem ipsum dolor sit amet, consectetur adipiscing elit
  print(encrypted.base64); // R4PxiU3h8YoIRqVowBXm36ZcCeNeZ4s1OvVBTfFlZRdmohQqOpPQqD1YecJeZMAop/hZ4OxqgC1WtwvX/hP9mw==
}
```

#### Salsa20
```dart
import 'package:encrypt/encrypt.dart';

void main() {
  final plainText = 'Lorem ipsum dolor sit amet, consectetur adipiscing elit';
  final key = Key.fromLength(32);
  final iv = IV.fromLength(8);
  final encrypter = Encrypter(Salsa20(key, iv));

  final encrypted = encrypter.encrypt(plainText);
  final decrypted = encrypter.decrypt(encrypted);

  print(decrypted); // Lorem ipsum dolor sit amet, consectetur adipiscing elit
  print(encrypted.base64); // CR+IAWBEx3sA/dLkkFM/orYr9KftrGa7lIFSAAmVPbKIOLDOzGwEi9ohstDBqDLIaXMEeulwXQ==
}
```

### Asymmetric

#### RSA
```dart
import 'dart:io';
import 'package:encrypt/encrypt.dart';
import 'package:pointycastle/asymmetric/api.dart';

void main() {
  final publicKeyFile = File('/path/to/public_key.pem');
  final privateKeyFile = File('/path/to/private_key.pem');

  final parser = RSAKeyParser();
  final RSAPublicKey publicKey = parser.parse(publicKeyFile.readAsStringSync());
  final RSAPrivateKey privateKey = parser.parse(privateKeyFile.readAsStringSync());

  final plainText = 'Lorem ipsum dolor sit amet, consectetur adipiscing elit';
  final encrypter = Encrypter(RSA(publicKey: publicKey, privateKey: privateKey));

  final encrypted = encrypter.encrypt(plainText);
  final decrypted = encrypter.decrypt(encrypted);

  print(decrypted); // Lorem ipsum dolor sit amet, consectetur adipiscing elit
  print(encrypted.base64); // XWMuHTeO86gC6SsUh14h+jc4iQW7Vy0TDaBKN926QWhg5c3KKoSuF+6uedLWBEis0LYgTON2rhtTOjmb6bU2P27lgf+5JKdLGKqri2F4sCS3+/p/EPb41f60vnr3whX2o5VRJhJagxtrq0V3eu3X4UeRiO2y7yOt6MXyJxMFcXs=
}
```
##### Note
If you are just encrypting or just decrypting, you can ignore the respectives `privateKey` and `publicKey`.
Trying the encrypt without a public key or decrypt without a private key will throw a `StateError`.
