
[![pipeline status](https://gitlab.com/ionspin-github-ci/kotlin-multiplatform-crypto-ci/badges/master/pipeline.svg)](https://gitlab.com/ionspin-github-ci/kotlin-multiplatform-crypto-ci/-/commits/master)

![Danger: Experimental](https://camo.githubusercontent.com/275bc882f21b154b5537b9c123a171a30de9e6aa/68747470733a2f2f7261772e6769746875622e636f6d2f63727970746f7370686572652f63727970746f7370686572652f6d61737465722f696d616765732f6578706572696d656e74616c2e706e67)

# Kotlin Multiplatform Crypto Library

This repository contains two cryptography related projects:

1. Libsodium bindings for Kotiln Multiplatform
2. Pure/Delegated kotlin multiplatform crypto library written from scratch in pure kotlin/delegated to libsodium. [Link to project readme](https://github.com/ionspin/kotlin-multiplatform-crypto/blob/master/multiplatform-crypto-api/README.md)

This readme represents the libsodium bindings project

# Libsodium bindings for Kotiln Multiplatform

Libsodium bindings project uses libsodium c sources, libsodium.js as well as LazySodium Java and Android to provide a kotlin multiplatform wrapper library for libsodium.

## Installation

The libsodium binding library is not published yet, once the sample showing the basic usage is ready, the library will be published. You can track the implementation 
[progress here](https://github.com/ionspin/kotlin-multiplatform-crypto/blob/master/supported_bindings_list.md)


## Usage

Before using the wrapper you need to initialize the underlying libsodium library. You can use either a callback or coroutines approach

```
    LibsodiumInitializer.initializeWithCallback {
        // Libsodium initialized
    }
```

```
    suspend fun initalizeProject() {
        ...
        LibsodiumInitializer.intialize()
        ...
    }
```

After intiailization you can call libsodium functions directly

The API is very close to libsodium but still adapted to kotlin standards, as an example here is the usage of authenticated
encryption api:

**libsodium:**

```
    #define MESSAGE ((const unsigned char *) "test")
    #define MESSAGE_LEN 4
    #define CIPHERTEXT_LEN (crypto_secretbox_MACBYTES + MESSAGE_LEN)
    
    unsigned char key[crypto_secretbox_KEYBYTES];
    unsigned char nonce[crypto_secretbox_NONCEBYTES];
    unsigned char ciphertext[CIPHERTEXT_LEN];
    
    crypto_secretbox_keygen(key);
    randombytes_buf(nonce, sizeof nonce);
    crypto_secretbox_easy(ciphertext, MESSAGE, MESSAGE_LEN, nonce, key);
    
    unsigned char decrypted[MESSAGE_LEN];
    if (crypto_secretbox_open_easy(decrypted, ciphertext, CIPHERTEXT_LEN, nonce, key) != 0) {
        /* message forged! */
    }
```

**kotlin:**
```
    val message = ("Ladies and Gentlemen of the class of '99: If I could offer you " +
                   "only one tip for the future, sunscreen would be it.").encodeToUByteArray()

    val key = LibsodiumRandom.buf(32)

    val nonce = LibsodiumRandom.buf(24)

    val encrypted = SecretBox.easy(message, nonce, key)
    val decrypted = SecretBox.openEasy(encrypted, nonce, key)
``` 
If message cannot be verified, `openEasy` function will throw a `SecretBoxCorruptedOrTamperedDataExceptionOrInvalidKey`

In some cases libsodium C api returns two values, usually encrypted data and a autogenerated nonce. In situations like
those, kotlin API returns a data class wrapping both objects. An example of this behavior is initializing the secret stream, where initialization funciton returns both the header and state:

**libsodium:**
```
    crypto_secretstream_xchacha20poly1305_state state;
    unsigned char key[crypto_secretstream_xchacha20poly1305_KEYBYTES];
    unsigned char header[crypto_secretstream_xchacha20poly1305_HEADERBYTES];
    
    /* Set up a new stream: initialize the state and create the header */
    crypto_secretstream_xchacha20poly1305_init_push(&state, header, key);
```

**kotlin:**
This is what the response data class definition looks like:
```
    data class SecretStreamStateAndHeader(val state: SecretStreamState, val header : UByteArray)
```
And here is the usage sample
```
    val key = LibsodiumRandom.buf(crypto_secretstream_xchacha20poly1305_KEYBYTES)
    val stateAndHeader = SecretStream.xChaCha20Poly1305InitPush(key)
    val state = stateAndHeader.state
    val header = stateAndHeader.header 
```

The functions are mapped from libsodium to kotiln objects, so `crypto_secretstream_xchacha20poly1305_init_push` becomes
`SecretStream.xChaCha20Poly1305InitPush`

At the moment you should refer to original libsodium documentation for instructions on how to use the library

## Supported native platforms

Currently supported native platforms:

|Platform|Pure variant| Delegated variant|
|--------|------------|------------------|
|Linux X86 64|          :heavy_check_mark: | :heavy_check_mark: |
|Linux Arm 64|          :heavy_check_mark: | :heavy_check_mark: |
|Linux Arm 32|          :heavy_check_mark: | :x: |
|macOS X86 64|          :heavy_check_mark: | :heavy_check_mark: |
|iOS x86 64 |           :heavy_check_mark: | :heavy_check_mark: |
|iOS Arm 64 |           :heavy_check_mark: | :heavy_check_mark: |
|iOS Arm 32 |           :heavy_check_mark: | :heavy_check_mark: |
|watchOS X86 32 |       :heavy_check_mark: | :heavy_check_mark: |
|watchOS Arm 64(_32) |  :heavy_check_mark: | :heavy_check_mark: |
|watchos Arm 32 |       :heavy_check_mark: | :heavy_check_mark: |
|tvOS X86 64 |          :heavy_check_mark: | :heavy_check_mark: |
|tvOS Arm 64 |          :heavy_check_mark: | :heavy_check_mark: |
|minGW X86 64|          :heavy_check_mark: | :heavy_check_mark: |
|minGW X86 32|          :x:                | :x: | 


### TODO:
- Copy/adapt code documentation, currently only some functions have documentation that is a copy-paste from libsodium website
- Replace LazySodium with direct JNA calls, and add build scripts for required libraries if missing
- Android testing 
- Fix browser testing, both locally and in CI/CD
- LobsodiumUtil `unpad` and `fromBase64` native implementations use a nasty hack to support shared native sourceset. The hack either needs to be removed and replaced with another solution or additional safeguards need to be added.






















