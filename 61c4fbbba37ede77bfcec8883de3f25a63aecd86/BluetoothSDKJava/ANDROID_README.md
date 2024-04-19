# Android Bluetooth SDK
The SDK is instantiated requiring `Context`, `BluetoothManager` and `BluetoothAdapter` objects.

The data/keys used to interact with the locker are provided by the client application.

### Example implementation
```java
package com.appexampleiboxensdk;

import android.Manifest;
import android.app.Activity;
import android.bluetooth.BluetoothAdapter;
import android.bluetooth.BluetoothManager;
import androidx.core.app.ActivityCompat;
import android.content.Context;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import com.qlocxiboxen.sdk.iBoxenInterface;

public class MyJavaClass extends Activity {
    private Context ctx;
    private iBoxenInterface ibInterface;
    private static final int REQUEST_CODE_PERMISSIONS = 213;

    private String[] requiredPermissions = new String[] {
            Manifest.permission.BLUETOOTH,
            Manifest.permission.BLUETOOTH_SCAN,
            Manifest.permission.BLUETOOTH_CONNECT,
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.ACCESS_COARSE_LOCATION,
            Manifest.permission.INTERNET
    };


    MyJavaClass(Context context) {
        ctx = context;
    }

    public void initSDK() {
        ActivityCompat.requestPermissions(
                getCurrentActivity(),
                requiredPermissions,
                REQUEST_CODE_PERMISSIONS);

        BluetoothManager mBluetoothManager = (BluetoothManager) ctx.getSystemService(ctx.BLUETOOTH_SERVICE);
        BluetoothAdapter mBluetoothAdapter = bm.getAdapter();

        String serviceId = "<your service id>";

        iBoxenInterface.Environment stage = iBoxenInterface.Environment.staging; // decide which environment to use

        ibInterface = new iBoxenInterface(ctx,
                serviceId,
                mBluetoothManager,
                mBluetoothAdapter,
                iBoxenInterface.Environment.staging,
                new iBoxenInterface.Callbacks.iBoxenDeviceEventCallback() {
                    @Override
                    public void event(String eventName, String deviceName) {
                        // handle event
                    }
                });
    }

    public void getPeripherals() {
        ibInterface.getPeripherals(new iBoxenInterface.Callbacks.getPeripheralsCallback() {
            @Override
            public void peripheralNames(ArrayList<String> names) { }

            @Override
            public void error(iBoxenException exception) { }
        });
    }

    public void open(String payload) {
        ibInterface.openLock(payload, new iBoxenInterface.Callbacks.openLockCallback() {
            @Override
            public void success() { }

            @Override
            public void error(iBoxenException exception) { }
        });
    }

    public void sense(String payload) {
        ibInterface.getDoorsOpen(payload, new iBoxenInterface.Callbacks.senseCallback() {
            @Override
            public void success(String doorsStatus) { }

            @Override
            public void error(iBoxenException exception) { }
        });
    }

    public void connnect() {
        ibInterface.connect("9fb1575937e40581f90259e81432e7d22e73fffd0fea41f25ba885242fb5be04",
                new iBoxenInterface.Callbacks.connectCallback() {
                    @Override
                    public void success() { }

                    @Override
                    public void error(iBoxenException exception) { }
                });
    }
    
    public void disconnect() {
        ibInterface.disconnect(new iBoxenInterface.Callbacks.disconnectCallback() {
            @Override
            public void success() { }

            @Override
            public void error(iBoxenException exception) { }
        });
    }
}

```

### Example snippets

Check if location services are enabled:

```java
public Boolean locationServicesEnabled() {
    LocationManager locationManager = (LocationManager) ctx.getSystemService(Context.LOCATION_SERVICE);

    Boolean enabled = locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER);

    return enabled;
}
```

Check if Android Bluetooth is enabled:

```java
public Boolen bluetoothEnabled() {
    BluetoothAdapter mBluetoothAdapter;
    BluetoothManager mBluetoothManager = (BluetoothManager) ctx.getSystemService(ctx.BLUETOOTH_SERVICE);

    mBluetoothAdapter = (BluetoothAdapter) mBluetoothManager.getAdapter();

    return mBluetoothAdapter.isEnabled();
}
```

### Permission requirements

Requires the app to have the following permissions in manifest & allowed by user:

```xml
<!-- To be able to use Bluetooth -->
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"/>
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>

<!-- To be able to push SDK logs to the server -->
<uses-permission android:name="android.permission.INTERNET"/>

<!-- To be able to get location information/bluetooth devices -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

### Network requirements
The SDK pushes its own logs to the iBoxen server, the logs are used to troubleshoot and to improve the SDK.
Logs server endpoints are:

staging: https://logger.iboxen-staging.se

production: https://logger.qlocxiboxen.com