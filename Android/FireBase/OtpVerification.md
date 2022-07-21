# MainActivity

```
package com.nabin.otpverification;

import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.AppCompatButton;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import com.hbb20.CountryCodePicker;

public class MainActivity extends AppCompatActivity {

    EditText phoneNo;
    CountryCodePicker countryCodePicker;
    AppCompatButton getOtpBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        phoneNo = findViewById(R.id.phoneNoEditText);
        countryCodePicker = findViewById(R.id.countryCodePicker);
        countryCodePicker.registerCarrierNumberEditText(phoneNo);
        getOtpBtn = findViewById(R.id.getOtp);

        Toast.makeText(this, phoneNo.getText().toString(), Toast.LENGTH_SHORT).show();

        getOtpBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(MainActivity.this,countryCodePicker.getFullNumberWithPlus().replace(" ", "").toString() , Toast.LENGTH_SHORT).show();
                Intent intent = new Intent(getApplicationContext(), ProcessOtpActivity.class);
                intent.putExtra("phone_no", countryCodePicker.getFullNumberWithPlus().replace(" ", ""));
                startActivity(intent);
            }
        });
    }
}
```

# ProcessOtpActivity
```
package com.nabin.otpverification;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.AppCompatButton;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.FirebaseException;
import com.google.firebase.auth.AuthResult;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.PhoneAuthCredential;
import com.google.firebase.auth.PhoneAuthOptions;
import com.google.firebase.auth.PhoneAuthProvider;

import java.util.concurrent.TimeUnit;

public class ProcessOtpActivity extends AppCompatActivity {
    FirebaseAuth mAuth;
    EditText otpHolder;
    AppCompatButton btn;

    String otpId;
    String PhoneNo;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_process_otp);
        otpHolder = findViewById(R.id.otpHolder);
        btn = findViewById(R.id.gotoDashboard);
        mAuth = FirebaseAuth.getInstance();

        PhoneNo = getIntent().getStringExtra("phone_no");

        processOTP();

        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                String val = otpHolder.getText().toString();
                if(val.isEmpty()){
                    Toast.makeText(ProcessOtpActivity.this, "Otp cannot be empty", Toast.LENGTH_SHORT).show();
                }else if(val.length() != 6){
                    Toast.makeText(ProcessOtpActivity.this, "Otp is invalid must be of 6 character", Toast.LENGTH_SHORT).show();
                }else{
                    signInWithAuthCredentials(PhoneAuthProvider.getCredential(otpId, otpHolder.getText().toString()));
                }
            }
        });
    }

    private void processOTP() {
        PhoneAuthOptions options = PhoneAuthOptions.newBuilder(mAuth)
                .setPhoneNumber(PhoneNo)
                .setTimeout(60L, TimeUnit.SECONDS)
                .setActivity(this)
                .setCallbacks(new PhoneAuthProvider.OnVerificationStateChangedCallbacks() {
                    @Override
                    public void onVerificationCompleted(@NonNull PhoneAuthCredential phoneAuthCredential) {
                        signInWithAuthCredentials(phoneAuthCredential);
                    }

                    @Override
                    public void onVerificationFailed(@NonNull FirebaseException e) {
                        Toast.makeText(getApplicationContext(), "Failed " + e, Toast.LENGTH_SHORT).show();
                    }

                    @Override
                    public void onCodeSent(@NonNull String s, @NonNull PhoneAuthProvider.ForceResendingToken forceResendingToken) {
                        otpId = s;
                        Toast.makeText(getApplicationContext(), s, Toast.LENGTH_SHORT).show();
                    }
                })
                .build();

        PhoneAuthProvider.verifyPhoneNumber(options);

    }

    private void signInWithAuthCredentials(PhoneAuthCredential phoneAuthCredential){
        mAuth.signInWithCredential(phoneAuthCredential).addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
            @Override
            public void onComplete(@NonNull Task<AuthResult> task) {
                if(task.isSuccessful()){
                    Intent intent = new Intent(getApplicationContext(), DashboardActivity.class);
                    startActivity(intent);
                }else{
                    Toast.makeText(ProcessOtpActivity.this, "Not successfull", Toast.LENGTH_SHORT).show();
                }
            }
        });
    }


}
```

# DashBoard Activity
```
package com.nabin.otpverification;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;

public class DashboardActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dashboard);
    }
}
```


# MainActivityXml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    tools:context=".MainActivity">

    <com.hbb20.CountryCodePicker
        android:id="@+id/countryCodePicker"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"/>

    <EditText
        android:id="@+id/phoneNoEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"/>

    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/getOtp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="get otp"/>


</LinearLayout>
```

# processOtp
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ProcessOtpActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Insert the opt received below"
        app:layout_constraintBottom_toTopOf="@+id/otpHolder"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <EditText
        android:id="@+id/otpHolder"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        android:hint="OTP HERE"
        app:layout_constraintVertical_bias="0.431" />

    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/gotoDashboard"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Go to dashboard"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.498"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/otpHolder"
        app:layout_constraintVertical_bias="0.173" />


</androidx.constraintlayout.widget.ConstraintLayout>
```

# DashBoardxml
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".DashboardActivity">

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="172dp"
        android:layout_marginBottom="356dp"
        android:text="Button"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```