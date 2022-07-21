## Headset and charging broadcast 

# MainActivity
```
package com.nabin.broadcastreceiver;

import androidx.appcompat.app.AppCompatActivity;
import androidx.localbroadcastmanager.content.LocalBroadcastManager;

import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends AppCompatActivity {
    public static final String ACTION_CUSTOM_BROADCAST = BuildConfig.APPLICATION_ID + ".ACTION_CUSTOM_BROADCAST";

    // Instance of the custom receiver
    private MyCustomReceiver customReceiver = new MyCustomReceiver();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Intent Filter to Check the Action that receiver have to receive
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(Intent.ACTION_POWER_CONNECTED);
        intentFilter.addAction(Intent.ACTION_POWER_DISCONNECTED);
        intentFilter.addAction(Intent.ACTION_HEADSET_PLUG);

        // Register system broadcast receiver
        this.registerReceiver(customReceiver, intentFilter);

        // Register Local broadcast receiver
        LocalBroadcastManager.getInstance(this).registerReceiver(customReceiver, new IntentFilter(ACTION_CUSTOM_BROADCAST));

        findViewById(R.id.sendBroadcast).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent customBroadcastIntent = new Intent(ACTION_CUSTOM_BROADCAST);
                LocalBroadcastManager.getInstance(MainActivity.this).sendBroadcast(customBroadcastIntent);
            }
        });
    }

    @Override
    protected void onDestroy() {
        //Unregister all broadcast receivers
        this.unregisterReceiver(customReceiver);
        LocalBroadcastManager.getInstance(this).unregisterReceiver(customReceiver);
        super.onDestroy();
    }
}
```

# CustomReceiver
```
package com.nabin.broadcastreceiver;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class MyCustomReceiver extends BroadcastReceiver {
    private boolean headsetPluggedIn = false;

    @Override
    public void onReceive(Context context, Intent intent) { // Receives the System broadcast in this method
        String intentAction = intent.getAction();
        if (intentAction != null) {
            String toastMessage = "Unknown intent action.";
            switch (intentAction) {
                case Intent.ACTION_POWER_CONNECTED:
                    toastMessage = "Power connected";
                    break;
                case Intent.ACTION_POWER_DISCONNECTED:
                    toastMessage = "Power disconnected";
                    break;
                case Intent.ACTION_HEADSET_PLUG:
                    int headSetState = intent.getIntExtra("state", -1);

                    if (headsetPluggedIn && headSetState == 0) {
                        toastMessage = "Headset unplugged";
                        headsetPluggedIn = false;
                    } else if (!headsetPluggedIn && headSetState == 1) {
                        toastMessage = "Headset Plugged In";
                        headsetPluggedIn = true;
                    }
                    break;

                case MainActivity.ACTION_CUSTOM_BROADCAST:
                    toastMessage = "Custom broadcast received";
                    break;
            }
            Toast.makeText(context, toastMessage, Toast.LENGTH_SHORT).show();
        }
    }
}
```