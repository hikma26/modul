main.java
package com.example.c2_prak4_13020250206;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import androidx.activity.EdgeToEdge;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    private EditText txtStb, txtNama, txtAngkatan;
    private Button btnSimpan, btnTampil;
    private DbHelper dbHelper;
    private Mahasiswa mhs;
    private Intent intentEdit;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_main);

        dbHelper = new DbHelper(this);

        txtStb = findViewById(R.id.txt_stb);
        txtNama = findViewById(R.id.txt_nama);
        txtAngkatan = findViewById(R.id.txt_angkatan);

        btnSimpan = findViewById(R.id.btn_simpan);
        btnTampil = findViewById(R.id.btn_tampil);

        btnSimpan.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (intentEdit == null)
                    simpanData();
                else
                    editData();
            }
        });

        btnTampil.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                intentEdit = null;
                Intent intent = new Intent(getApplicationContext(), TampilActivity.class);
                startActivityForResult(intent, 1);
                dbHelper.close();
            }
        });
    }

    private void clearText() {
        txtStb.setText("");
        txtNama.setText("");
        txtAngkatan.setText("");
        intentEdit = null;
        txtStb.requestFocus();
    }

    private void simpanData(){
        mhs = new Mahasiswa(
                txtStb.getText().toString(),
                txtNama.getText().toString(),
                Integer.parseInt(txtAngkatan.getText().toString())
        );

        dbHelper.insertData(dbHelper.getWritableDatabase(), mhs);

        Toast.makeText(this, "Data tersimpan...", Toast.LENGTH_LONG).show();
        clearText();
    }

    private void editData(){
        mhs = new Mahasiswa(
                txtStb.getText().toString(),
                txtNama.getText().toString(),
                Integer.parseInt(txtAngkatan.getText().toString())
        );

        dbHelper.editData(dbHelper.getWritableDatabase(), mhs, intentEdit.getStringExtra("stb"));

        Toast.makeText(this, "Edit Data berhasil...", Toast.LENGTH_LONG).show();
        clearText();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == 1){
            intentEdit = data;
            txtStb.setText(data.getStringExtra("stb"));
            txtNama.setText(data.getStringExtra("nama"));
            txtAngkatan.setText(String.valueOf(data.getIntExtra("angkatan", 0)));
        }
    }
}
DbHelper.java
package com.example.c2_prak4_13020250206;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import java.util.ArrayList;

public class DbHelper extends SQLiteOpenHelper {

    private ArrayList<Mahasiswa> arrListMhs = new ArrayList<>();

    public DbHelper(Context context){
        super(context, "db_mhs", null, 1);
    }

    @Override
    public void onCreate(SQLiteDatabase sqliteDatabase) {
        String sql = "create table tb_mhs(stb text(11) primary key, nama text(50), angkatan text(4))";
        sqliteDatabase.execSQL(sql);
    }

    @Override
    public void onUpgrade(SQLiteDatabase sqliteDatabase, int i, int i1) {

    }

    public void insertData(SQLiteDatabase db, Mahasiswa mhs){
        ContentValues cv = new ContentValues();
        cv.put("stb", mhs.getStb());
        cv.put("nama", mhs.getNama());
        cv.put("angkatan", mhs.getAngkatan());
        db.insert("tb_mhs", null, cv);
    }

    public ArrayList<Mahasiswa> getArrListMhs(SQLiteDatabase db){
        arrListMhs.clear();
        Cursor cursor = db.rawQuery("select * from tb_mhs", null);
        if (cursor.getCount() > 0){
            cursor.moveToFirst();
            do{
                arrListMhs.add(new Mahasiswa(
                        cursor.getString(0),
                        cursor.getString(1),
                        cursor.getInt(2)
                ));
            }while(cursor.moveToNext());
        }
        return arrListMhs;
    }

    public void hapusData(SQLiteDatabase db, String stb){
        db.delete("tb_mhs", "stb=?", new String[]{stb});
    }

    public void editData(SQLiteDatabase db, Mahasiswa mhs, String stb){
        ContentValues cv = new ContentValues();
        cv.put("stb", mhs.getStb());
        cv.put("nama", mhs.getNama());
        cv.put("angkatan", mhs.getAngkatan());
        db.update("tb_mhs", cv, "stb=?", new String[]{stb});
    }
}
Mahasiswa.java
package com.example.c2_prak4_13020250206;

public class Mahasiswa {

    private String stb;
    private String nama;
    private int angkatan;

    public Mahasiswa(String stb, String nama, int angkatan) {
        this.stb = stb;
        this.nama = nama;
        this.angkatan = angkatan;
    }

    public String getStb() {
        return stb;
    }

    public String getNama() {
        return nama;
    }

    public int getAngkatan() {
        return angkatan;
    }
}
tampilan.java
package com.example.c2_prak4_13020250206;

import android.content.Intent;
import android.graphics.Color;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TableLayout;
import android.widget.TableRow;
import android.widget.TextView;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;

import java.util.ArrayList;

public class TampilActivity extends AppCompatActivity {
    private DbHelper dbHelper;
    private TableLayout tbMhs;
    private TableRow tr;
    private TextView col1, col2, col3;
    private Button btnTutup, btnEdit, btnHapus;
    private ArrayList<Mahasiswa> arrListMhs = new ArrayList<>();
    private String stb, nama;
    private int angkatan;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_tampil);

        dbHelper = new DbHelper(this);

        tbMhs = findViewById(R.id.tb_mahasiswa);
        btnTutup = findViewById(R.id.btn_tutup);
        btnEdit = findViewById(R.id.btn_edit);
        btnHapus = findViewById(R.id.btn_hapus);

        tampilTabelMhs();

        btnHapus.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                dbHelper.hapusData(dbHelper.getWritableDatabase(), stb);
                tampilTabelMhs();
            }
        });

        btnEdit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent();
                intent.putExtra("stb", stb);
                intent.putExtra("nama", nama);
                intent.putExtra("angkatan", angkatan);
                dbHelper.close();
                setResult(1, intent);
                finish();
            }
        });

        btnTutup.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                finish();
            }
        });
    }

    public void tampilTabelMhs(){
        tbMhs.removeAllViews();

        arrListMhs = dbHelper.getArrListMhs(dbHelper.getWritableDatabase());

        tr = new TableRow(this);
        col1 = new TextView(this);
        col2 = new TextView(this);
        col3 = new TextView(this);

        col1.setText("Stambuk");
        col2.setText("Nama Mahasiswa");
        col3.setText("Angkatan");

        col1.setWidth(200);
        col2.setWidth(300);
        col3.setWidth(150);

        tr.addView(col1);
        tr.addView(col2);
        tr.addView(col3);

        tbMhs.addView(tr);

        for(final Mahasiswa mhs : arrListMhs){
            tr = new TableRow(this);
            col1 = new TextView(this);
            col2 = new TextView(this);
            col3 = new TextView(this);

            col1.setText(mhs.getStb());
            col2.setText(mhs.getNama());
            col3.setText(String.valueOf(mhs.getAngkatan()));

            col1.setWidth(200);
            col2.setWidth(300);
            col3.setWidth(150);

            tr.addView(col1);
            tr.addView(col2);
            tr.addView(col3);

            tbMhs.addView(tr);

            tr.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    for(int i=0; i<tbMhs.getChildCount(); i++){
                        stb = mhs.getStb();
                        nama = mhs.getNama();
                        angkatan = mhs.getAngkatan();
                        if(tbMhs.getChildAt(i) == view)
                            tbMhs.getChildAt(i).setBackgroundColor(Color.LTGRAY);
                        else
                            tbMhs.getChildAt(i).setBackgroundColor(Color.WHITE);
                    }
                }
            });
        }
    }
}
tampilan
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp">

    <TableLayout
        android:id="@+id/tb_mahasiswa"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:stretchColumns="*">

        <TableRow>
            <TextView
                android:text="Stambuk"
                android:textStyle="bold"
                android:padding="6dp" />

            <TextView
                android:text="Nama Mahasiswa"
                android:textStyle="bold"
                android:padding="6dp" />

            <TextView
                android:text="Angkatan"
                android:textStyle="bold"
                android:padding="6dp" />
        </TableRow>

    </TableLayout>


    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:weightSum="3">

        <Button
            android:id="@+id/btn_tutup"
            android:text="TUTUP"
            android:layout_width="0dp"
            android:layout_height="48dp"
            android:layout_weight="1"
            android:layout_marginEnd="6dp"/>

        <Button
            android:id="@+id/btn_edit"
            android:text="EDIT"
            android:layout_width="0dp"
            android:layout_height="48dp"
            android:layout_weight="1"
            android:layout_marginEnd="6dp"
            android:layout_marginStart="6dp"/>

        <Button
            android:id="@+id/btn_hapus"
            android:text="HAPUS"
            android:layout_width="0dp"
            android:layout_height="48dp"
            android:layout_weight="1"
            android:layout_marginStart="6dp"/>
    </LinearLayout>

</LinearLayout>

activity
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp"
    android:gravity="bottom">


    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="vertical">

        <EditText
            android:id="@+id/txt_stb"
            android:hint="Masukkan Stambuk"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="12dp"/>

        <EditText
            android:id="@+id/txt_nama"
            android:hint="Masukkan Nama"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="12dp"/>

        <EditText
            android:id="@+id/txt_angkatan"
            android:hint="Masukkan Angkatan"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"/>

    </LinearLayout>


    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:weightSum="2">

        <Button
            android:id="@+id/btn_simpan"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="48dp"
            android:text="SIMPAN"
            android:layout_marginEnd="8dp"/>

        <Button
            android:id="@+id/btn_tampil"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="48dp"
            android:text="TAMPIL DATA"
            android:layout_marginStart="8dp"/>
    </LinearLayout>

</LinearLayout>

