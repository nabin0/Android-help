# RecyclerView 

### MainActivity.java
```
package com.nabin.recyclerviewandmenu;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.Toolbar;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import android.content.Intent;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.Toast;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity implements CustomAdapter.ItemClickListener {
    private RecyclerView recyclerView;

    public ArrayList<ModelClass> objects;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        recyclerView = findViewById(R.id.recyclerView);

        objects = new ArrayList<>();

        getDemoDataForRecyclerview();
        initRecyclerView();

    }

    private void getDemoDataForRecyclerview() {
        for(int i =0 ; i < 20; i++){
            objects.add(new ModelClass("Name "+ i, "12:"+i+"am", "Hello this is the \n Description no : " + i));
        }
    }

    void initRecyclerView(){
        LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
        recyclerView.setLayoutManager(linearLayoutManager);

        RecyclerItemDecorator decorator = new RecyclerItemDecorator(30);
        recyclerView.addItemDecoration(decorator);

        CustomAdapter customAdapter = new CustomAdapter(objects, this);
        recyclerView.setAdapter(customAdapter);
    }

    // Oncllick for recycler item
    @Override
    public void onItemClick(int positon) {
        Toast.makeText(this, "Position : " + positon, Toast.LENGTH_SHORT).show();
    }

    // To Inflate the Option layout
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    // Handle Option Onclick event
    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()){
            case R.id.settings:
                Intent intent = new Intent(this, SettingActivity.class);
                startActivity(intent);
                return true;
            case R.id.search:
                Toast.makeText(this, "Search func. will be added soon!!!", Toast.LENGTH_SHORT).show();
                return true;
            case R.id.favourite:
                Toast.makeText(this, "will be added soon!!!", Toast.LENGTH_SHORT).show();
                return true;
            default:
                Toast.makeText(this, "Can be Added soon", Toast.LENGTH_SHORT).show();
        }

        return super.onOptionsItemSelected(item);
    }
}

```

### Custom Adapter

```
package com.nabin.recyclerviewandmenu;

import android.view.Display;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import java.util.ArrayList;
import java.util.zip.Inflater;

public class CustomAdapter extends RecyclerView.Adapter<CustomAdapter.CustomViewHolder> {

    private ArrayList<ModelClass> objects;
    private ItemClickListener itemClickListener; //This is interface used to handle onclick event

    public CustomAdapter(ArrayList<ModelClass> objects, ItemClickListener itemClickListener) {
        this.objects = objects;
        this.itemClickListener = itemClickListener;
    }

    // This creates the view holder
    @NonNull
    @Override
    public CustomViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.recyclerview_item_layout, parent, false);
        return new CustomViewHolder(view, itemClickListener);
    }

    // Binding data to the view holder
    @Override
    public void onBindViewHolder(@NonNull CustomViewHolder holder, int position) {
        holder.name.setText(objects.get(position).getName());
        holder.time.setText(objects.get(position).getTime());
        holder.desc.setText(objects.get(position).getDesc());
    }

    // Return the size of data to be shown in the recycler view
    @Override
    public int getItemCount() {
        return objects.size();
    }

    // Custom holder class to access the item layout elements
    public static class CustomViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {
        private TextView name, time, desc;
        ItemClickListener itemClickListener;

        public CustomViewHolder(@NonNull View itemView, ItemClickListener itemClickListener) {
            super(itemView);

            name = itemView.findViewById(R.id.name);
            time = itemView.findViewById(R.id.time);
            desc = itemView.findViewById(R.id.desc);
            this.itemClickListener = itemClickListener;

            itemView.setOnClickListener(this);
        }

        @Override
        public void onClick(View v) {
            itemClickListener.onItemClick(getAdapterPosition());
        }
    }

    // Interface to handle click event
    public interface ItemClickListener{
        void onItemClick(int positon);
    }
}
```

### Model class

``` 
package com.nabin.recyclerviewandmenu;

public class ModelClass {
    private String name;
    private String time;
    private String desc;

    public ModelClass(String name, String time, String desc) {
        this.name = name;
        this.time = time;
        this.desc = desc;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getTime() {
        return time;
    }

    public void setTime(String time) {
        this.time = time;
    }

    public String getDesc() {
        return desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }
}
```

### RecyclerItem decorator
```
package com.nabin.recyclerviewandmenu;

import android.graphics.Rect;
import android.view.View;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

public class RecyclerItemDecorator extends RecyclerView.ItemDecoration {
    public final int verticalSpaceValue;

    public RecyclerItemDecorator(int verticalSpaceValue) {
        this.verticalSpaceValue = verticalSpaceValue;
    }

    @Override
    public void getItemOffsets(@NonNull Rect outRect, @NonNull View view, @NonNull RecyclerView parent, @NonNull RecyclerView.State state) {
        outRect.bottom = verticalSpaceValue;
    }
}

```

### Settings Activity (navigate from the menu)
```
package com.nabin.recyclerviewandmenu;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;

public class SettingActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_setting);
    }
}
```

# Menus and xml layouts

### Mainactivity layout

```
<?xml version="1.0" encoding="utf-8"?>

<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

        <com.google.android.material.appbar.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <androidx.appcompat.widget.Toolbar
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="?attr/colorPrimary"
                android:id="@+id/toolbar" >

            </androidx.appcompat.widget.Toolbar>

        </com.google.android.material.appbar.AppBarLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        
        />
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

### Menu layout

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".SettingActivity">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="setting"
        android:textSize="48sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

### Recycler item layout

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="5dp">

    <TextView
        android:id="@+id/desc"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:text="Hello this is \n desription"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/name" />

    <TextView
        android:id="@+id/name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Thomas Shelby"
        android:textStyle="bold"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/time"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="10:12 am"
        app:layout_constraintBottom_toBottomOf="@+id/name"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.945"
        app:layout_constraintStart_toEndOf="@+id/name" />
</androidx.constraintlayout.widget.ConstraintLayout>
```