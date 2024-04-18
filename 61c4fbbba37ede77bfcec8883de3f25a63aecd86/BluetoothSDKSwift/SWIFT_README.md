# Swift Bluetooth SDK
This SDK provides an interface for interacting with iBoxen via Bluetooth communication.

To instansiate the SDK you need a `serviceId` string.
To interact with the locker you need a `payload` which contains the data and keys, these are provided by the client application.

### Methods
#### .getPeripherals
```swift
getPeripherals(_ completion: @escaping (Result<[String], Error>) -> Void)
```
#### .open
```swift
open(_ payload: AccessPayload, _ completion: @escaping (Result<Void, Error>) -> Void)
```
#### .isDoorsClosed
```swift
isDoorsClosed(_ payload: SensePayload, _ completion: @escaping (Result<Bool, Error>) -> Void)
```

### Errors
#### List of errors that could be returned from all the methods.
* Type: `BluetoothInactiveError` Description: `Bluetooth is inactive`<br>
* Type: `BluetoothMissingPermission` Description: `Missing bluetooth permission`<br>
* Type: `OperationInProgressError` Description: `Scan is already in progress` | `Payload execution already in progress`<br>
* Type: `TimeoutError` Description: `Operation timed out`<br>

#### List of errors that could be returned from `open`.
* Type: `NoCandidatesError` Description: `Found no candidates`<br>
Explanation: Happens if a locker was not found nearby.
* Type: `GenericError` Description: `Generic error`<br>
Explanation: Happens if locker could not give a response.


### Available payload structs
`AccessPayload` is used as argument for `open`<br>
`SensePayload` is used as argument for `isDoorsClosed`
```swift
public struct AccessPayload: Codable {
    public let payload: String
    public let id: String
    
    public init(payload: String, id: String) {
        self.payload = payload
        self.id = id
    }
}

public struct SensePayload: Codable {
    public let payload: String
    public let doorOpenValue: String?
    public let id: String

    public init(payload: String, doorOpenValue: String?, id: String) {
        self.payload = payload
        self.doorOpenValue = doorOpenValue
        self.id = id
    }
}

public struct Payloads: Codable {
    public let access: AccessPayload
    public let sense: SensePayload
    
    public init(access: AccessPayload, sense: SensePayload) {
        self.access = access
        self.sense = sense
    }
}
```

### Example implementation
```swift
import Foundation
import SwiftUI
import iBoxenInterfaceSwift

struct ContentView: View {
    @State private var payloads: Payloads?
    @State private var iBoxenSDK = iBoxenInterface(serviceId: "your-service-id-here")
    
    var body: some View {
        VStack {
                    Button("Get parcel payloads") {
                        // Get payloads from iboxen server
                        self.getParcelPayloads { result in
                            switch result {
                                case .success(let data):
                                    let decoder = JSONDecoder()
                                    do {
                                        // Decode response data into the 'Payloads' struct
                                        payloads = try decoder.decode(Payloads.self, from: data)
                                    } catch {
                                        print(error)
                                    }
                                case .failure(let error):
                                    print(error)
                            }
                        }
                    }
                    .padding().border(.blue)
            
                    Button("Get peripherals") {
                        self.getPeripherals()
                    }
                    .padding().border(.blue)
            
                    Button("Open door") {
                        self.open()
                    }
                    .padding().border(.blue).disabled(payloads == nil)
            
                    Button("Check if all doors are closed") {
                        self.isDoorsClosed()
                    }
                    .padding().border(.blue).disabled(payloads == nil)
                }
        .padding()
    }

    func getPeripherals() {
        iBoxenSDK.getPeripherals { result in
            switch result {
                case .success(let foundPeripherals):
                    // Handle success
                case .failure(let error):
                    // Handle error
            }
        }
    }
    
    func open() {
        iBoxenSDK.open(payloads!.access) { result in
            switch result {
                case .success(_):
                    // Handle success
                case .failure(let error):
                    // Handle error
            }
        }
    }
    
    func isDoorsClosed() {
        iBoxenSDK.isDoorsClosed(payloads!.sense) { result in
            switch result {
                case .success(let bool):
                    // Handle success
                case .failure(let error):
                    // Handle error
            }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}


```

## Permission requirements
The app's Info.plist must contain an `NSBluetoothAlwaysUsageDescription` key with a non-empty string as the value.

## Network requirements
The SDK pushes its own logs to the iBoxen server, the logs are used to troubleshoot and to improve the SDK.
Logs server endpoints are:

* staging: https://logger.iboxen-staging.se
* production: https://logger.qlocxiboxen.com
<br>
<br>
