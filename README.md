# FCL

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.portto.fcl/fcl/badge.svg?style=flat)](https://maven-badges.herokuapp.com/maven-central/com.portto.fcl/fcl)
![Github Action](https://github.com/portto/fcl-android/actions/workflows/ci.yml/badge.svg)

The Flow Client Library (FCL) is a package used to interact with user wallets and the Flow
blockchain. When using FCL for authentication, DApps are able to support all FCL-compatible wallets
on Flow and their users without any custom integrations or changes needed to the DApp code.

Requirements
-------
- Min SDK 21+

Download
-------
In `build.gradle` of app module, include this dependency

```kotlin
implementation("com.portto.fcl:fcl:x.y.z")
```

Initialization
-------
To initialize FCL, use the following function:

```kotlin
Fcl.init(
    env = Network.TESTNET,
    supportedWallets = getWalletList(isMainnet),
    appDetail = AppDetail(),
)
```

- env: The network on which the app was built. i.e. `Network.TESTNET` or `Network.MAINNET`
- supportedWallets: A list of **wallet providers** supported by the app. See **Wallet Discovery**
  for more info.
- appDetail(Optional): The information of the app.

Wallet Discovery
-------
Knowing all the wallets available to users on a blockchain can be challenging. FCL's Discovery
mechanism relieves much of the burden of integrating with Flow compatible wallets and lets
developers focus on building their DApp and providing as many options as possible to their users.

When you initialize FCL, it's required to provide a list of wallet providers. The following is an
example of adding `Blocto` and `Dapper` as wallet providers.

```kotlin
listOf(Blocto.getInstance(BLOCTO_TESTNET_APP_ID), Dapper)
```

### Current Wallet Providers

- [Blocto][1] (Fully supported)
- [Dapper Wallet][2] (Support only authn for now)

Once you've initialized FCL, you may use Discovery directly from `Discovery UI` or `Config`.

### Discovery UI

The simplest way to integrate Wallet Discovery is to utilize Discovery UI.

```kotlin
showConnectWalletDialog {
    // authentication
}
```

### Config

If you want to create your own authentication UI, the wallet providers are exposed as a list which
can be retrieved from
`Fcl.config`.

```kotlin
// Create a custom UI by supplying wallet provider list as data
val providerList = Fcl.config.supportedWallets
// Once you get the user's selected provider, you need to add it to config
Fcl.config.put(Config.Option.SelectedWalletProvider(provider))
```

Authentication
-------
In order to interact with Flow blockchain, the current user's information is required. The user's
account address can be easily retrieved by calling `Fcl.authenticate()`.

```kotlin
when (val result = Fcl.authenticate()) {
    is Result.Success -> {
        val address = result.value
    }
    is Result.Failure -> {
        // Handle error
    }
}
```

Proof of Authentication
-------
A common desire that application developers have is to be able to prove that a user controls an
on-chain account. Proving ownership of an on-chain account is a way to authenticate a user with an
application backend. Fortunately, FCL provides a way to achieve this.

### Authenticating a user using `AccountProof`

By supplying `AccountProofData` to authenticate, the signatures will be acquired and set to the
current user in `config`
.

```kotlin
when (val result = Fcl.authenticate(accountProofData)) {
    is Result.Success -> {
        val address = result.value
    }
    is Result.Failure -> {
        // Handle error
    }
}


```

Verify `AccountProof`
-------
If you've acquired account proof signed data from `authenticate`, you may retrieve the signatures
from `config`.

```kotlin
val accountProofData = Fcl.currentUser?.accountProofData
```

Then you can verify the signatures by calling `Fcl.verifyAccountProof()`.

```kotlin
when (val result = Fcl.verifyAccountProof(FLOW_APP_IDENTIFIER, accountProofData)) {
    is Result.Success -> {
        val isValid = reuslt.value
    }
    is Result.Failure -> {
        // Handle error
    }
}
```

[Learn more about account proof][3]

Sign a message
-------
Cryptographic signatures are a key part of the blockchain. They are used to prove ownership of an
address without exposing its private key. While primarily used for signing transactions,
cryptographic signatures can also be used to sign arbitrary messages.

To sign a message, simply call `Fcl.signUserMessage`.

```kotlin
when (val result = Fcl.signUserMessage(message)) {
    is Result.Success -> {
        val userSignatures = result.value
    }
    is Result.Failure -> {
        // Handle error
    }
}
```

Verify user signatures
-------
```kotlin
when (val result = Fcl.verifyUserSignatures(userMessage, userSignatures)) {
    is Result.Success -> {
        val isValid = reuslt.value
    }
    is Result.Failure -> {
        // Handle error 
    }
}
```

Blockchain Interactions
-------
- `Fcl.query()`: Send arbitrary Cadence scripts to the chain and receive back decoded values
- `Fcl.mutate()`: Send arbitrary transactions with your own signatures or via a user's wallet to
  perform state changes on chain

[Learn more about on-chain interactions][4]

Support
-------
Discuss FCL with the community on the [forum][5].

License
-------
See the [LICENSE][6] file for details.
    
[1]: https://portto.com
[2]: https://www.meetdapper.com
[3]: https://developers.flow.com/tools/fcl-js/reference/proving-authentication#authenticating-a-user-using-account-proof
[4]: https://docs.onflow.org/fcl/reference/api/#on-chain-interactions
[5]: https://forum.onflow.org/c/developer-tools/flow-fcl/22
[6]: https://github.com/portto/fcl-android/blob/main/LICENSE
