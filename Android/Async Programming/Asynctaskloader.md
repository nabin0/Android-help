# MainActivity

```
package com.nabin.asynctaskloadergooglebookapi;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.loader.app.LoaderManager;
import androidx.loader.content.AsyncTaskLoader;
import androidx.loader.content.Loader;

import android.content.Context;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.os.AsyncTask;
import android.os.Bundle;
import android.view.View;
import android.view.inputmethod.InputMethodManager;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import java.lang.ref.WeakReference;

public class MainActivity extends AppCompatActivity implements LoaderManager.LoaderCallbacks<String> {

    private EditText mBookInput;
    private TextView mTitleText, mAuthorText;
    private Button mSearchBookBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initializing views
        mBookInput = findViewById(R.id.searchEditText);
        mTitleText = findViewById(R.id.titleText);
        mAuthorText = findViewById(R.id.autherTextView);
        mSearchBookBtn = findViewById(R.id.searchBtn);

        //
        if(getSupportLoaderManager().getLoader(0) != null){
            getSupportLoaderManager().initLoader(0, null, this);
        }


        mSearchBookBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String queryString = mBookInput.getText().toString();

                // Hide Keyboard
                InputMethodManager inputMethodManager = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
                if (inputMethodManager != null) {
                    inputMethodManager.hideSoftInputFromWindow(mSearchBookBtn.getWindowToken(),
                            InputMethodManager.HIDE_NOT_ALWAYS);
                }

                //Check Network state and input value
                ConnectivityManager connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
                NetworkInfo networkInfo = null;
                if (connectivityManager != null) {
                    networkInfo = connectivityManager.getActiveNetworkInfo();
                }

                if (networkInfo != null && networkInfo.isConnected() && queryString.length() != 0) {
                    Bundle bundle = new Bundle();
                    bundle.putString("queryString", queryString);

                    getSupportLoaderManager().restartLoader(0, bundle, MainActivity.this);
                    mTitleText.setText("Loading...");
                    mAuthorText.setText("");
                } else {
                    if (queryString.length() == 0) {
                        mTitleText.setText("Please Enter Valid Input");
                        mAuthorText.setText("");
                    } else {
                        mTitleText.setText("No Network Connection");
                        mAuthorText.setText("");
                    }
                }

            }
        });

    }

    @NonNull
    @Override
    public Loader<String> onCreateLoader(int id, @Nullable Bundle args) {
        String queryString = "";

        if (args != null) {
            queryString = args.getString("queryString");
        }
        return new BookLoader(this, queryString);
    }

    @Override
    public void onLoadFinished(@NonNull Loader<String> loader, String data) {
        try {
            JSONObject jsonObject = new JSONObject(data);
            JSONArray itemsArray = jsonObject.getJSONArray("items");

            int i = 0;
            String title = null;
            String authors = null;

            while (i < itemsArray.length() && (authors == null && title == null)) {
                JSONObject book = itemsArray.getJSONObject(i);
                JSONObject volumeInfo = book.getJSONObject("volumeInfo");

                try {
                    title = volumeInfo.getString("title");
                    authors = volumeInfo.getString("authors");
                } catch (Exception e) {
                    e.printStackTrace();
                }

                i++;
            }

            if (title != null && authors != null) {
                mTitleText.setText(title);
                mAuthorText.setText(authors);
            } else {
                mTitleText.setText("NO RESULT FOUND!!!");
                mAuthorText.setText("");
            }

        } catch (JSONException e) {
            mTitleText.setText("NO RESULT FOUND!!!");
            mAuthorText.setText("");
            e.printStackTrace();
        }

    }

    @Override
    public void onLoaderReset(@NonNull Loader<String> loader) {

    }

    public static class BookLoader extends AsyncTaskLoader<String> {
        private String mQueryString;

        public BookLoader(@NonNull Context context, String queryString) {
            super(context);
            this.mQueryString = queryString;
        }

        @Nullable
        @Override
        public String loadInBackground() {
            return NetworkUtils.getBookInfo(mQueryString);
        }

        @Override
        protected void onStartLoading() {
            super.onStartLoading();
            forceLoad();
        }
    }
}
```


# Networl Util class

```
package com.nabin.asynctaskloadergooglebookapi;

import android.net.Uri;
import android.util.Log;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class NetworkUtils {
    private static final String LOG_TAG = NetworkUtils.class.getSimpleName();

    // Base url for Books API.
    public static final String BASE_BOOK_URL = "https://www.googleapis.com/books/v1/volumes?";
    // Parameter for search string
    public static final String QUERY_PARAM = "q";
    // Parameter That limits search result
    public static final String MAX_RESULTS = "maxResults";
    // Parameter To filter by print type
    public static final String PRINT_TYPE = "printType";

    static String getBookInfo(String queryString) {
        HttpURLConnection urlConnection = null;
        BufferedReader reader = null;
        String bookJSONString = null;

        try {
            //Combine all parts to make a URI
            Uri builtURI = Uri.parse(BASE_BOOK_URL).buildUpon()
                    .appendQueryParameter(QUERY_PARAM, queryString)
                    .appendQueryParameter(MAX_RESULTS, "10")
                    .appendQueryParameter(PRINT_TYPE, "books")
                    .build();

            // Build the URL form URI
            URL requestURL = new URL(builtURI.toString());

            // Open Connection and make request
            urlConnection = (HttpURLConnection) requestURL.openConnection();
            urlConnection.setRequestMethod("GET");
            urlConnection.connect();

            // Get Input stream
            InputStream inputStream = urlConnection.getInputStream();

            //Create a buffered readerFrom input stream
            reader = new BufferedReader(new InputStreamReader(inputStream));

            //Stringbuilder to hold the incomming respnese
            StringBuilder stringBuilder = new StringBuilder();

            // Store all the response in a string
            String line;
            while((line = reader.readLine()) != null){
                stringBuilder.append(line);
                stringBuilder.append("\n"); // Not compulsory
            }

            if (stringBuilder.length() == 0){
                return null;
            }

            bookJSONString = stringBuilder.toString();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if(urlConnection != null){
                urlConnection.disconnect();
            }
            if(reader != null){
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        return bookJSONString;
    }
}

```