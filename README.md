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
        val editor = preferences.edit()
        editor.putString(keySetID, Base64.getEncoder().encodeToString(key))
        editor.apply()

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

### Linked Data(LD) Document Loader
#### Kotlin (Android)

```kotlin
import dev.trustbloc.wallet.sdk.ld.DocLoader

// Setup
val documentLoader = DocLoader()
```

#### Swift (iOS)

```swift
import Walletsdk

// Setup
let documentLoader = LdNewDocLoader()
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
val didDoc = didCreator.create("jwk", createDIDOpts) // Create a new did:jwk doc
didDocID = didDoc.id() // Save this DID; sample ID did:jwk:eyJjcnYiOiJQLTM4NCIsImtpZCI6ImJRTnYzODh3UkpjNFhsZzJaUy03T0VJbVBUaDRhdF9kZDI5aXNwbjJZRlUiLCJrdHkiOiJFQyIsIngiOiJBSVc4NVBjTHg1TE85bmpUY2JyeFBoOW9zSkI3ekFTV0h1bHVPcFFyVkV0UGJ5QmJOd1FBYjNqcXVOWUJlR01jIiwieSI6IjJ1aXV2Njk1eVJoV2l5MXF4ZEVlend0eDQ1bVdBamZiNG94eVZuUWk4V1Z5NGc4WGM5ZngzWFR3TUNyZThacUwifQ
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

// Resolve the DID created in previous step; or if you have diDoc then ignore this step
val didDoc = didResolver.Resolve("<did-generated-in-previous-step>")) 

// Call Request credential method
val credentials = interaction.requestCredential(CredentialRequestOpts(""), didDoc.assertionMethod()) // Store this credential
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

// Resolve the DID created in previous step; or if you have diDoc then ignore this step
val didDoc = didResolver.Resolve("<did-generated-in-previous-step>")) 

// Call Request credential method
let credentials = interaction.requestCredential(Openid4ciNewCredentialRequestOpts(""), didDoc.assertionMethod()) // Store this credential
let issuerURI = interaction.issuerURI() // Store this Issuer URI
```


## Presentation Flow

#### Kotlin (Android)

```kotlin
import dev.trustbloc.wallet.sdk.openid4vp
import dev.trustbloc.wallet.sdk.credential
import dev.trustbloc.wallet.sdk.openid4vp.ClientConfig
import dev.trustbloc.wallet.sdk.openid4vp.Interaction

// Setup
val activityLogger = mem.ActivityLogger() // Optional, but useful for tracking credential activities - Validate the requirements
val cfg = ClientConfig(kms, kms.getCrypto(), didResolver, documentLoader, activityLogger)
val interaction = openid4vp.Interaction("<Presentation_URL>", cfg)
val inquirer = credential.Inquirer(docLoader)

// Get Presentation query
val query = interaction.getQuery()

// pass query and saved credentials to get matched VCs.
val matchedRequirements = inquirer.getSubmissionRequirements(query, savedCredentials) 

// Get presentation submission requirements 
val matchedRequirement = matchedRequirements.atIndex(0) // Usually we will have one requirement
val requirementDesc = matchedRequirement.descriptorAtIndex(0) // Usually requirement will contain one descriptor

// Select VCs to be sent to the verifier
val selectedVCs = api.VerifiableCredentialsArray()
selectedVCs.add(requirementDesc.matchedVCs.atIndex(0)) 

// Get the final presentation request
val verifiablePres = inquirer.Query(query, credential.CredentialsOpt(selectedVCs))

// Present credentials
interaction.presentCredential(verifiablePres, didDoc.assertionMethod())
```

#### Swift (iOS)

```swift
import Walletsdk

// Setup
let activityLogger = MemNewActivityLogger() // Optional, but useful for tracking credential activities
let clientConfig = Openid4vpClientConfig(keyHandleReader: kms, crypto: kms.getCrypto(), didResolver: didResolver, ldDocumentLoader: documentLoader, activityLogger: activityLogger)
let interaction = Openid4vpInteraction("<Presentation_URL>", config: clientConfig)
let inquirer = CredentialNewInquirer(docLoader)

// Get Presentation query
let query = interaction.getQuery()
let issuedCredentials = ApiVerifiableCredentialsArray() // Would need some actual credentials for this to actually work

// pass query and saved credentials to get matched VCs.
let matchedRequirements = inquirer.getSubmissionRequirements(query, credentials) 

// Get presentation submission requirements 
let matchedRequirement = matchedRequirements.atIndex(0) // Usually we will have one requirement
let requirementDesc = matchedRequirement.descriptorAtIndex(0) // Usually requirement will contain one descriptor

// Select VCs to be sent to the verifier
let selectedVCs = ApiVerifiableCredentialsArray()
selectedVCs.add(requirementDesc.matchedVCs.atIndex(0)) 

// Get the final presentation request
let verifiablePres = inquirer.Query(query, CredentialNewCredentialsOpt(selectedVCs))

// Present credentials
let credentials = interaction.presentCredential(verifiablePres, doc.assertionMethod())
```
