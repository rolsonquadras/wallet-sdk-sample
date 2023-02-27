# wallet-sdk-sample
Wallet SDK Sample


## DID Creation
### With Key Writer

The Keys used for DID documents are created for you automatically by the key writer.

#### Examples

##### Kotlin (Android)

```kotlin
import dev.trustbloc.wallet.sdk.api.CreateDIDOpts
import dev.trustbloc.wallet.sdk.did.Creator
import dev.trustbloc.wallet.sdk.localkms.Localkms
import dev.trustbloc.wallet.sdk.localkms.MemKMSStore

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



val kms = Localkms.newKMS(kmsStore())
val didCreator = Creator(kms as KeyWriter)

val createDIDOpts = CreateDIDOpts()
createDIDOpts.keyType = "ECDSAP384IEEEP1363"
val didDocResolution = didCreator.create("jwk", createDIDOpts) // Create a new did:jwk doc
didDocID = doc.id() // Save this DID
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

let kms = LocalkmsNewKMS(kmsStore(), nil)
let didCreator = DidNewCreatorWithKeyWriter(kms, nil)

let apiCreate = ApiCreateDIDOpts.init()
apiCreate.keyType = "ECDSAP384IEEEP1363"

let didDoc = didCreator.create("jwk", apiCreate) // Create a new did:jwk doc
didDocID = didDoc.id_(nil) // Save this DID


## Issuance Flow

## Presentation Flow
