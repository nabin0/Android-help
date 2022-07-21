# MainActivity

```
package com.nabin.sqlitecontactsdatabase;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.util.Log;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        DatabaseHelper db = new DatabaseHelper(this);

        //Insert contacts
        /*
        db.insertContacts(new ContactModel("adam", "4576"));
        db.insertContacts(new ContactModel("deepti", "4543"));
        db.insertContacts(new ContactModel("ram", "8978"));
        db.insertContacts(new ContactModel("sukracharya", "234"));
        */

        //Fetch contacts
        ArrayList<ContactModel> list = new ArrayList<>();
        list = db.fetchAllContacts();

        //Update contact
        ContactModel contactModel = new ContactModel(1, "John snow", "0000000");
        db.updateContact(contactModel);

        //Delete contact
        db.deleteContact(4);
        db.deleteContact(5);
        db.deleteContact(6);
        db.deleteContact(7);
        db.deleteContact(8);

        //Log all contacts
        for (ContactModel c : list) {
            Log.d("MyTag", "Contact id: " + c.getId() + " Contact Name : " + c.getName() + " Phone no : " + c.getContact_no());
        }
    }

}
```

# DatabaseHelper
### Can create parameters file whe we have many parameters
```
package com.nabin.sqlitecontactsdatabase;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import androidx.annotation.Nullable;

import java.net.IDN;
import java.util.ArrayList;

public class DatabaseHelper extends SQLiteOpenHelper {
    public static final String DATABASE_NAME = "contact_database";
    public static final int DATABASE_VERSION = 1;
    public static final String TABLE_CONTACTS = "contacts";
    public static final String KEY_ID = "contact_id";
    public static final String KEY_NAME = "contact_name";
    public static final String KEY_CONTACT_NUMBER = "contact_number";

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
    
    //Runs once when database is created
    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        sqLiteDatabase.execSQL("CREATE TABLE " + TABLE_CONTACTS + "(" + KEY_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
                KEY_NAME + " TEXT, " + KEY_CONTACT_NUMBER + " TEXT" + ")");
    }

    //When database schema is updated
    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {
        sqLiteDatabase.execSQL("DROP TABLE IF EXISTS " + TABLE_CONTACTS);
        onCreate(sqLiteDatabase);
    }

    public void insertContacts(ContactModel contact) {
        SQLiteDatabase db = this.getWritableDatabase();

        ContentValues contentValues = new ContentValues();
        contentValues.put(KEY_NAME, contact.getName());
        contentValues.put(KEY_CONTACT_NUMBER, contact.getContact_no());

        db.insert(TABLE_CONTACTS, null, contentValues);
        db.close();
    }

    public ArrayList<ContactModel> fetchAllContacts() {
        SQLiteDatabase db = this.getReadableDatabase();
        String query = "SELECT * FROM " + TABLE_CONTACTS;
        Cursor cursor = db.rawQuery(query, null);

        ArrayList<ContactModel> contacts = new ArrayList<>();
        while (cursor.moveToNext()) {
            ContactModel model = new ContactModel(cursor.getInt(0),
                    cursor.getString(1),
                    cursor.getString(2));

            contacts.add(model);
        }
        db.close();
        return contacts;
    }

    public void updateContact(ContactModel model) {
        SQLiteDatabase db = this.getWritableDatabase();
        ContentValues cv = new ContentValues();
        cv.put(KEY_NAME, model.getName());
        cv.put(KEY_CONTACT_NUMBER, model.getContact_no());
        db.update(TABLE_CONTACTS, cv, KEY_ID + "= " + model.id, null);
        db.close();
    }

    public void deleteContact(int id) {
        SQLiteDatabase db = this.getWritableDatabase();
        db.delete(TABLE_CONTACTS, KEY_ID + " = ?", new String[]{String.valueOf(id)});
        db.close();
    }
}

```

# Modelclass

```
package com.nabin.sqlitecontactsdatabase;

public class ContactModel {
    int id;
    String name, contact_no;

    public ContactModel(int id, String name, String contact_no) {
        this.id = id;
        this.name = name;
        this.contact_no = contact_no;
    }

    public ContactModel(String name, String contact_no) {
        this.name = name;
        this.contact_no = contact_no;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getContact_no() {
        return contact_no;
    }

    public void setContact_no(String contact_no) {
        this.contact_no = contact_no;
    }
}

```