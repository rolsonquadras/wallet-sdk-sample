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

### Linked Data(LD) Document Loaded
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
didDocID = didDoc.id() // Save this DID
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

// Resolve the DID created in previous step; or if you have diDoc then ignore this step
val didDoc = didResolver.Resolve("<did-generated-in-previous-step>")) 

// Call Request credential method
let credentials = interaction.requestCredential(Openid4ciNewCredentialRequestOpts(""), didDoc.assertionMethod()) // Store this credential
let issuerURI = interaction.issuerURI() // Store this Issuer URI
```


## Presentation Flow (In Progress)

#### Kotlin (Android)

```kotlin
import dev.trustbloc.wallet.sdk.localkms.Localkms
import dev.trustbloc.wallet.sdk.localkms.MemKMSStore
import dev.trustbloc.wallet.sdk.did.Resolver
import dev.trustbloc.wallet.sdk.did.Creator
import dev.trustbloc.wallet.sdk.ld.DocLoader
import dev.trustbloc.wallet.sdk.localkms
import dev.trustbloc.wallet.sdk.openid4vp
import dev.trustbloc.wallet.sdk.ld
import dev.trustbloc.wallet.sdk.credential
import dev.trustbloc.wallet.sdk.openid4ci.mem
import dev.trustbloc.wallet.sdk.openid4vp.ClientConfig
import dev.trustbloc.wallet.sdk.openid4vp.Interaction

// Setup
val activityLogger = mem.ActivityLogger() // Optional, but useful for tracking credential activities - Validate the requirements
val cfg = ClientConfig(kms, kms.getCrypto(), didResolver, documentLoader, activityLogger)

// Going through the flow
val interaction = openid4vp.Interaction(""<Presentation_URL>"", cfg)
val query = interaction.getQuery()
val inquirer = credential.Inquirer(docLoader)
val issuedCredentials = api.VerifiableCredentialsArray() // Would need some actual credentials for this to actually work

// Use this code to display the list of VCs to select which of them to send.
val matchedRequirements = inquirer.getSubmissionRequirements(query, credentials) 
val matchedRequirement = matchedRequirements.atIndex(0) // Usually we will have one requirement
val requirementDesc = matchedRequirement.descriptorAtIndex(0) // Usually requirement will contain one descriptor
val selectedVCs = api.VerifiableCredentialsArray()
selectedVCs.add(requirementDesc.matchedVCs.atIndex(0)) // Users should select one VC for each descriptor from the matched list and confirm that they want to share it

val verifiablePres = inquirer.Query(query, credential.CredentialsOpt(selectedVCs))
interaction.presentCredential(verifiablePres, didDocResolution.assertionMethod())
// Consider checking the activity log at some point after the interaction
```

#### Swift (iOS)

```swift
import Walletsdk

// Setup
let memKMSStore = LocalkmsNewMemKMSStore()
let kms = LocalkmsNewKMS(memKMSStore, nil)
let didResolver = DidNewResolver("", nil)
let didCreator = DidNewCreatorWithKeyWriter(kms, nil)
let documentLoader = LdNewDocLoader()
let activityLogger = MemNewActivityLogger() // Optional, but useful for tracking credential activities
let clientConfig = Openid4vpClientConfig(keyHandleReader: kms, crypto: kms.getCrypto(), didResolver: didResolver, ldDocumentLoader: documentLoader, activityLogger: activityLogger)

// Going through the flow
let interaction = Openid4vpInteraction("YourAuthRequestURIHere", config: clientConfig)
let query = interaction.getQuery()
let inquirer = CredentialNewInquirer(docLoader)
let issuedCredentials = ApiVerifiableCredentialsArray() // Would need some actual credentials for this to actually work

// Use this code to display the list of VCs to select which of them to send.
let matchedRequirements = inquirer.getSubmissionRequirements(query, credentials) 
let matchedRequirement = matchedRequirements.atIndex(0) // Usually we will have one requirement
let requirementDesc = matchedRequirement.descriptorAtIndex(0) // Usually requirement will contain one descriptor
let selectedVCs = ApiVerifiableCredentialsArray()
selectedVCs.add(requirementDesc.matchedVCs.atIndex(0)) // Users should select one VC for each descriptor from the matched list and confirm that they want to share it

let verifiablePres = inquirer.Query(query, CredentialNewCredentialsOpt(selectedVCs))
let credentials = interaction.presentCredential(verifiablePres, doc.assertionMethod())
// Consider checking the activity log at some point after the interaction
```
