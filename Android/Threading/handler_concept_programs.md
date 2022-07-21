# MainActivity

```
package com.myapp.threaddemo;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;

import android.os.AsyncTask;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ProgressBar;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MyTag";
    private static final String MESSAGE_KEY = "MyMessage";
    private Button startBtn, clearBtn;
    private ProgressBar progressBar;
    private EditText editText;
    private TextView textView;
    DownloadThread thread;
    private boolean mTaskActive;
    private MyTask myTask;




    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        init();
        mTaskActive = false;
        mProgressBar(false);
        Log.d(TAG, "onCreate: Thread id : " + Thread.currentThread().getId());

//        thread = new DownloadThread(MainActivity.this);
//        thread.start();

        // On click listeners
        startBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mProgressBar(true);
                Log.d(TAG, "Started Downloading ---");

                // ------ Background Task using Handler --------
//                for (String song : PlayList.songs) {
//                    Message message = Message.obtain();
//                    message.obj = song;
//                    thread.mHandler.sendMessage(message);
//                }


                // Using Async Task
                if(mTaskActive && myTask != null){
                    myTask.cancel(true);
                    mTaskActive = false;
                    myTask = null;
                    startBtn.setText("Start");
                }else{
                    myTask = new MyTask();
                    myTask.execute("A", "B","C","D");
                    mTaskActive = true;
                    startBtn.setText("stop");
                }



            }
        });

        clearBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                textView.setText("");
                mProgressBar(false);
            }
        });
    }

    private void init() {
        startBtn = findViewById(R.id.startDemo);
        clearBtn = findViewById(R.id.clearText);
        progressBar = findViewById(R.id.progressBar);
        textView = findViewById(R.id.textView);
        editText = findViewById(R.id.inputText);
    }

    public void mProgressBar(boolean active) {
        if (active) {
            progressBar.setVisibility(View.VISIBLE);
        } else {
            progressBar.setVisibility(View.INVISIBLE);
        }
    }

    public void log(String st) {
        Log.i(TAG, "log: " + st);
        textView.append(st + "\n");
    }

     class MyTask extends AsyncTask<String, String, String> {
        @Override
        protected String doInBackground(String... strings) {
            for (String s :
                    strings) {
                if(!mTaskActive){
                    log("Task is cancelled");
                    break;
                }
                publishProgress(s);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                Log.d(TAG, "doInBackground: " + s);
            }
            return "Result";
        }

         @Override
         protected void onProgressUpdate(String... values) {
             log(values[0]);
         }

         @Override
         protected void onPostExecute(String s) {
            log("Download complete");
             mProgressBar(false);
         }

         @Override
         protected void onPreExecute() {
             log("Starting to download..");
         }

         @Override
         protected void onCancelled() {
             mProgressBar(false);
             log("Download cancelled!!!");
         }
     }
}
```

# DownloadThread.java
```
package com.myapp.threaddemo;

import android.os.Looper;

public class DownloadThread extends Thread{
    public DownloadHandler mHandler;
    private final MainActivity mainActivity;

    public DownloadThread(MainActivity mainActivity) {
        this.mainActivity = mainActivity ;
    }

    @Override
    public void run() {
        Looper.prepare();
        mHandler = new DownloadHandler(mainActivity);
        Looper.loop();
    }
}

```


# DownloadHandler
```
package com.myapp.threaddemo;


import android.os.Handler;
import android.os.Message;
import android.util.Log;

import androidx.annotation.NonNull;

public class DownloadHandler extends Handler {

    private static final String TAG = "MyTag";
    private final MainActivity mainActivity;

    public DownloadHandler(MainActivity mainActivity) {
        this.mainActivity = mainActivity;
    }

    @Override
    public void handleMessage(@NonNull Message msg) {
        String song = msg.obj.toString();
        download(song);
    }

    private void download(String songName){
        Log.d(TAG, "runnig in " + Thread.currentThread().getId());
        try {
            Thread.sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        mainActivity.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                mainActivity.mProgressBar(false);
                mainActivity.log("Download complete");
            }
        });
        Log.d(TAG, "download: Song  downloaded : " + songName);
    }
}

```


# PlayList
```
package com.myapp.threaddemo;

public class PlayList {
    public static String[] songs = {"Believer Imagine dragons", "The chainsmokers song", "Closer song The chainsmoker", "Touch song"};
}

```

