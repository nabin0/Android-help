# MainActivity

```

package com.nabin.mediastoreaudio;

import androidx.appcompat.app.AppCompatActivity;

import android.Manifest;
import android.content.Context;
import android.database.Cursor;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.provider.MediaStore;
import android.util.Log;
import android.view.View;

import java.util.ArrayList;
import java.util.List;

 public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    // Returns the list of Audio
    public List<AudioModel> getAllAudioFromDevice(final Context context){
        final List<AudioModel> tempAudioList = new ArrayList<>();

        Uri uri = MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
        String[] projection = {MediaStore.Audio.AudioColumns.DATA,
                MediaStore.Audio.AudioColumns.TITLE,
                MediaStore.Audio.AudioColumns.ALBUM,
                MediaStore.Audio.ArtistColumns.ARTIST
            };
        Cursor cursor = context.getContentResolver().query(uri,projection,null, null,null,null);

        if(cursor!= null){
            while (cursor.moveToNext()){
                AudioModel audioModel = new AudioModel();

                String path = cursor.getString(0);
                String name = cursor.getString(1);
                String album = cursor.getString(2);
                String artist = cursor.getString(3);

                //Set data to model object
                audioModel.setaPath(path);
                audioModel.setaName(name);
                audioModel.setaArtist(artist);
                audioModel.setaAlbum(album);

                Log.d("MYTAG", "\n" +"name: " + name + " album:: " + album +"\nPath:: " + path + "\nArtist :: "+artist);

                tempAudioList.add(audioModel);
            }
        }
        return tempAudioList;
    }

     public void fetchMusic(View view) {
         if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
             requestPermissions(new String[]{Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE}, 8);
         }
         getAllAudioFromDevice(this);
     }
 }

```

# Audio Model POJO

```
package com.nabin.mediastoreaudio;

public class AudioModel {
    private String aPath;
    private String aName;
    private String aAlbum;
    private String aArtist;

    public String getaPath() {
        return aPath;
    }

    public void setaPath(String aPath) {
        this.aPath = aPath;
    }

    public String getaName() {
        return aName;
    }

    public void setaName(String aName) {
        this.aName = aName;
    }

    public String getaAlbum() {
        return aAlbum;
    }

    public void setaAlbum(String aAlbum) {
        this.aAlbum = aAlbum;
    }

    public String getaArtist() {
        return aArtist;
    }

    public void setaArtist(String aArtist) {
        this.aArtist = aArtist;
    }
}

```