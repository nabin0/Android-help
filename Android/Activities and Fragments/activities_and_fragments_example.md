# Activity and fragments

### MainActivity

``` 
package com.nabin.activitiesandfragments;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentContainerView;
import androidx.fragment.app.FragmentManager;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import org.w3c.dom.Text;

public class MainActivity extends AppCompatActivity {
    // Views
    private Button button, show, hide;
    private TextView textView;
    private FragmentContainerView containerView;

    //Vars
    public static final String EXTRA_NAME = "com.nabin.activitiesandfragments.name";
    public static final int REQUEST_CODE = 101;

    // This is to handle the start activity for result, which is a method to start a activity
    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if(REQUEST_CODE == requestCode){
            if(resultCode == RESULT_OK){
                assert data != null;
                String s = data.getStringExtra(NewActivity.EXTRA_RES);
                textView.setText(s);
            }
        }
    }

    @Override
    public void onBackPressed() { // Onbackpress remove fragments or pop from the back stack
        super.onBackPressed();

        FragmentManager manager = getSupportFragmentManager();
        Fragment fragment = manager.findFragmentById(R.id.fragmentContainerView);

        if(manager.getBackStackEntryCount() > 0){
            manager.popBackStack();
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = findViewById(R.id.newActivityBtn);
        show = findViewById(R.id.show);
        hide = findViewById(R.id.hide);
        textView = findViewById(R.id.returned_data);

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(MainActivity.this, NewActivity.class);
                intent.putExtra(EXTRA_NAME, "This is the extra data");
                startActivityForResult(intent,REQUEST_CODE);
            }
        });

        // Handle configuration changes
        if(savedInstanceState == null){
            getSupportFragmentManager().beginTransaction()
                    .setReorderingAllowed(true)
                    .add(R.id.fragmentContainerView, new Fragment1())
                    .commit();
        }

        show.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Fragment2 fragment2 = new Fragment2();
                FragmentManager manager = getSupportFragmentManager();
                    
                    // Check condition for not placing the same fragment twice in stack
                    boolean poppedFrag = manager.popBackStackImmediate("frag2",0);

                    if(!poppedFrag && manager.findFragmentByTag("frag2") == null){
                        getSupportFragmentManager().beginTransaction()
                                .setReorderingAllowed(true)
                                .replace(R.id.fragmentContainerView, new Fragment2(), "frag2")
                                .addToBackStack("frag1")
                                .commit();
                    }
            }
        });

        hide.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Fragment2 fragment = (Fragment2) getSupportFragmentManager().findFragmentById(R.id.fragmentContainerView);

                if(fragment != null){
                    getSupportFragmentManager().popBackStack();
                }
            }
        });
    }
}

```

### NewActivity.java

```
package com.nabin.activitiesandfragments;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.os.Bundle;
import android.widget.TextView;

public class NewActivity extends AppCompatActivity {

    public static final String EXTRA_RES = "resextra";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_new);

        TextView textView = findViewById(R.id.textView);

        String data = getIntent().getStringExtra(MainActivity.EXTRA_NAME);

        textView.setText(data);

        // Return the result to the parent activity
        Intent resIntent = new Intent();
        resIntent.putExtra(EXTRA_RES,"result sent return");
        setResult(RESULT_OK, resIntent);

    }



    @Override
    protected void onPause() {
        super.onPause();
            // Works here also
//        Intent resIntent = new Intent();
//        resIntent.putExtra(EXTRA_RES,"result sent return");
//        setResult(RESULT_OK, resIntent);
    }

}

```

### Fragments (Two Fragments are used in this example the example code is like below for fragment)
```
package com.nabin.activitiesandfragments;

import android.os.Bundle;
import androidx.fragment.app.Fragment;

import android.view.LayoutInflater;
public class Fragment1 extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_1, container, false);
    }
}
```