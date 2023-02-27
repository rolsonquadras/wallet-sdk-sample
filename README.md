# wallet-sdk-sample
Wallet SDK Sample


## SDK Setup

### KMS 
```kotlin
import dev.trustbloc.wallet.sdk.localkms.Localkms

// Implement KMS Store - here is the sample implementation
class KmsStore(context: Context) : Store {

    private val preferences = context.getSharedPreferences("KMSLocalStore", Context.MODE_PRIVATE)

    @RequiresApi(Build.VERSION_CODES.O)
    override fun get(keysetID: String?): Result {
        val localResult  = Result()
        val keyVal  = preferences.getString(keysetID, null)
        if(keyVal == null ) {
            localResult.found = false
            return localResult
        }
        localResult.found = true
        localResult.key = Base64.getDecoder().decode(keyVal)
        return localResult
    }

    @RequiresApi(Build.VERSION_CODES.O)
    override fun put(keySetID: String?, key: ByteArray?) {
        try{
            val editor = preferences.edit()
            editor.putString(keySetID, Base64.getEncoder().encodeToString(key))
            editor.apply()
        } catch(e: java.lang.Exception) {
            println(e)
        }
    }

}


// Instantiate LocalKMS from SDK by passing the KMS store
val kms = Localkms.newKMS(kmsStore())
```

##### Swift (iOS)

```swift
import Walletsdk

// Implement KMS Store - here is the sample implementation
public class kmsStore: NSObject, LocalkmsStoreProtocol{
    
    public func put(_ keysetID: String?, key: Data?) throws {
        UserDefaults.standard.set(key, forKey: keysetID!)
        return
    }

    public func get(_ keysetID: String?) throws -> LocalkmsResult {
        let localResult = LocalkmsResult.init()
        let keyVal = UserDefaults.standard.data(forKey: keysetID!)
        if (keyVal == nil){
            localResult.found = false
            return localResult
        }
        localResult.found = true
        localResult.key = keyVal
        return localResult
    }
    
    public func delete(_ keysetID: String?) throws {
        UserDefaults.standard.removeObject(forKey: keysetID!)
        return
    }

}


// Instantiate LocalKMS from SDK by passing the KMS store
let kms = LocalkmsNewKMS(kmsStore(), nil)
```

### DID Resolver
#### Kotlin (Android)

```kotlin
import dev.trustbloc.wallet.sdk.did.Resolver

// Setup
val didResolver = Resolver("")
```

#### Swift (iOS)

```swift
import Walletsdk

// Setup
let didResolver = DidNewResolver("", nil)
```


## DID Creation
### With Key Writer

The Keys used for DID documents are created for you automatically by the key writer.

#### Examples

##### Kotlin (Android)

```kotlin
import dev.trustbloc.wallet.sdk.api.CreateDIDOpts
import dev.trustbloc.wallet.sdk.did.Creator

// Instantiate DID Creator from SDK by passing the KMS
val didCreator = Creator(kms as KeyWriter)

// Add keyType (P-384) to Create DID Opts
val createDIDOpts = CreateDIDOpts()
createDIDOpts.keyType = "ECDSAP384IEEEP1363"

// Create DID API
val doc = didCreator.create("jwk", createDIDOpts) // Create a new did:jwk doc
didDocID = doc.id() // Save this DID
```

##### Swift (iOS)

```swift
import Walletsdk

// Instantiate DID Creator from SDK by passing the KMS
let didCreator = DidNewCreatorWithKeyWriter(kms, nil)

// Add keyType (P-384) to Create DID Opts
let apiCreate = ApiCreateDIDOpts.init()
apiCreate.keyType = "ECDSAP384IEEEP1363"

// Create DID API
let didDoc = didCreator.create("jwk", apiCreate) // Create a new did:jwk doc
didDocID = didDoc.id_(nil) // Save this DID
```

## Issuance Flow

#### Kotlin (Android)

```kotlin
import dev.trustbloc.wallet.sdk.did.Resolver
import dev.trustbloc.wallet.sdk.openid4ci.Interaction
import dev.trustbloc.wallet.sdk.openid4ci.ClientConfig
import dev.trustbloc.wallet.sdk.openid4ci.CredentialRequestOpts

// Setup and Instantiate Issuance Interaction object
val activityLogger = mem.ActivityLogger() // Optional, but useful for tracking credential activities - Validate the requirements
val cfg = ClientConfig("ClientID", kms.crypto, didResolver, activityLogger)
val interaction = Interaction("<Issuance_URL>", cfg)

// Start Issuance process
interaction.authorize() // Ignore the return here

// Resolve the DID created in previous step
val didDoc = didResolver.Resolve("<did-generated-in-previous-step>")) 

// Call Request credential method
val credentials = interaction.requestCredential(CredentialRequestOpts("), didDoc.assertionMethod()) // Store this credential
val issuerURI = interaction.issuerURI() // Store this Issuer URI
```

#### Swift (iOS)

```swift
import Walletsdk

// Setup and Instantiate Issuance Interaction object
let activityLogger = MemNewActivityLogger() // Optional, but useful for tracking credential activities - Validate the requirements
let cfg =  Openid4ciClientConfig(clientID: "ClientID", kms?.getCrypto(), didRes: didResolver, activityLogger: activityLogger)
let interaction = Openid4ciNewInteraction("<Issuance_URL>", cfg, nil)

// Start Issuance process
interaction.authorize() // Ignore the return here

// Resolve the DID created in previous step
val didDoc = didResolver.Resolve("<did-generated-in-previous-step>")) 

// Call Request credential method
let credentials = interaction.requestCredential(Openid4ciNewCredentialRequestOpts(""), didDocResolution.assertionMethod()) // Store this credential
let issuerURI = interaction.issuerURI() // Store this Issuer URI
```


## Presentation Flow
