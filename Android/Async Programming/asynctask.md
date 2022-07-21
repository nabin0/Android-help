# MainActivity

```
package com.nabin.asyncprogramming;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.loader.content.AsyncTaskLoader;

import android.app.LoaderManager;
import android.content.Context;
import android.content.Loader;
import android.os.AsyncTask;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.ProgressBar;
import android.widget.TextView;

import org.w3c.dom.Text;

import java.lang.ref.WeakReference;
import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MyTag";

    public static final String TEXT_STATE = "currentText";
    public static final String PROGRESS_VALUE = "currentProgress";

    TextView showProgressValue;
    Button button;
    ProgressBar progressBar;
    private boolean taskIsRunning;
    MyAsyncTask obj;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);



        progressBar = findViewById(R.id.progressBar);
        showProgressValue = findViewById(R.id.showProgressText);
        button = findViewById(R.id.asyncTaskBtn);


        if(savedInstanceState != null){
            showProgressValue.setText(savedInstanceState.getString(TEXT_STATE));
            progressBar.setProgress(Integer.parseInt(savedInstanceState.getString(PROGRESS_VALUE)));
        }

        // AsyncTask Driver Code
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                if (taskIsRunning && obj != null) {
                    taskIsRunning = false;
                    obj.cancel(true);
                } else {

                    obj = new MyAsyncTask();
                    taskIsRunning = true;
                    obj.execute("String 1", "String 2");
                }
            }
        });


    }

    @Override
    protected void onSaveInstanceState(@NonNull Bundle outState) {
        super.onSaveInstanceState(outState);

        // Save the state

        outState.putString(TEXT_STATE, showProgressValue.getText().toString());
        outState.putString(PROGRESS_VALUE, String.valueOf(progressBar.getProgress()));
    }

    // Asynctask Class (input, progress , result)
    class MyAsyncTask extends AsyncTask<String, Integer, Integer> {

        @Override
        protected Integer doInBackground(String... strings) {

            int progress = 0;

            publishProgress(0);
            int val = 0;
            for (int i = 0; i < 5; i++) {
                if (isCancelled()) {
                    break;
                }
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                val = (100 / 5) * (i + 1);

                Log.d(TAG, "doInBackground: " + val);
                publishProgress(val);
            }

            return val;
        }

        @Override
        protected void onProgressUpdate(Integer... values) {
            Log.d(TAG, "onProgressUpdate: " + values[0]);
            progressBar.setProgress(values[0]);
            showProgressValue.setText(values[0] + "%");
        }

        @Override
        protected void onPostExecute(Integer integer) {
            progressBar.setProgress(integer);
            showProgressValue.setText("Task Completed");
        }

        @Override
        protected void onCancelled() {
            showProgressValue.setText("Task Cancelled");
        }
    }
}
```