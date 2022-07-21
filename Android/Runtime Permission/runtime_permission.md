# Runtime Permission Demo

```
package com.nabin.runtimepermission;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;

import android.Manifest;
import android.content.DialogInterface;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {


    // Can use list for multiple permission
    private final List<String> permissions = new ArrayList<>();
    private Button requestPermissionBtn;
    private final int STORAGE_PERMISSION_REQUEST_CODE = 80;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        requestPermissionBtn = findViewById(R.id.requestPermission);

        requestPermissionBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    //For Multiple Permissions do if statement check and insert the denied permission into arraylist
                    if (checkSelfPermission(Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                        permissions.add(Manifest.permission.READ_EXTERNAL_STORAGE);
                        requestStoragePermission();
                    } else {
                        Toast.makeText(MainActivity.this, "Storage Permission Already Granted", Toast.LENGTH_SHORT).show();
                    }
                } else {
                    Toast.makeText(MainActivity.this, "Storage Permission Granted At Install Time.", Toast.LENGTH_SHORT).show();
                }
            }
        });

    }

    private void requestStoragePermission() {
        if(ActivityCompat.shouldShowRequestPermissionRationale(this,Manifest.permission.READ_EXTERNAL_STORAGE)){
            new AlertDialog.Builder(this)
                    .setTitle("External Storage Permission Requeired.")
                    .setMessage("Storage permission required to fetch media.")
                    .setPositiveButton("YES", new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            ActivityCompat.requestPermissions(MainActivity.this, permissions.toArray(new String[permissions.size()]), STORAGE_PERMISSION_REQUEST_CODE);
                        }
                    }).setNegativeButton("NO", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    Toast.makeText(MainActivity.this, "Permission Denied. Some Features might not be available.", Toast.LENGTH_SHORT).show();
                }
            }).show();
        }else{
            ActivityCompat.requestPermissions(MainActivity.this, permissions.toArray(new String[permissions.size()]), STORAGE_PERMISSION_REQUEST_CODE);
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        if(requestCode == STORAGE_PERMISSION_REQUEST_CODE){
            if(grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                Toast.makeText(this, "Permission Granted :: On RequestPermissionResult", Toast.LENGTH_SHORT).show();
            }else{
                Toast.makeText(this, "Permission Denied :: On RequestPermissionResult ", Toast.LENGTH_SHORT).show();
            }
        }
    }
}

```