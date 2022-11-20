# Midterm
1.增加时间戳
过程：
1.1修改NotesList.java中PROJECTION的内容，添加modif字段，使其在后面的搜索中才能从SQLite中读取修改时间的字段。
代码：
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //Extended:display time, color
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
1.2修改适配器内容，增加dataColumns中装配到ListView的内容，因此要同时增加一个文本框来存放时间。
代码：
final String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE , NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
int[] viewIDs = { android.R.id.text1, R.id.text2};
1.3修改layout文件夹中noteslist_item.xml的内容，增加一个textview组件，因为有两个组件，所以要相应的为他们添加一个布局。
代码：
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">


    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
        />
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:singleLine="true"
        />
</LinearLayout>
1.4修改NoteEditor.java中updateNote方法中的时间类型。
代码：
private final void updateNote(String text, String title) {

        // Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        Long now = Long.valueOf(System.currentTimeMillis());
        SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");
        Date d = new Date(now);
        String format = sf.format(d);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);
1.5图片展示
![BM`H$VY40YU6PFWG1BP4CN](https://user-images.githubusercontent.com/114241292/202895206-c1865fcb-57ed-4a96-872b-0730a69c5bd2.png)
2查询功能
2.1搜索组件在主页面的菜单选项中，在list_options_menu.xml布局文件中添加搜索功能。
代码：
    <item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_menu_search"
        android:title="@string/menu_search"
        android:showAsAction="always" />
2.2新建一个查找笔记内容的布局文件note_search.xml。
代码：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        />
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />
</LinearLayout>
2.3在NotesList.java中的onOptionsItemSelected方法中添加search查询的处理。
代码：
        case R.id.menu_search:
        //Find function
        //startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));
          Intent intent = new Intent(this, NoteSearch.class);
          this.startActivity(intent);
          return true;
2.4新建一个NoteSearch.java用于search功能的功能实现。
代码：
package com.example.android.notepad;
import android.app.Activity;
import android.content.Intent;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;
import android.widget.Toast;

public class NoteSearch extends Activity implements SearchView.OnQueryTextListener
{
    ListView listView;
    SQLiteDatabase sqLiteDatabase;
    /**
     * The columns needed by the cursor adapter
     */
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE//时间
    };

    public boolean onQueryTextSubmit(String query) {
        Toast.makeText(this, "you choose:"+query, Toast.LENGTH_SHORT).show();
        return false;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        SearchView searchView = findViewById(R.id.search_view);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listView = findViewById(R.id.list_view);
        sqLiteDatabase = new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
        //Set the searchview to display the search button
        searchView.setSubmitButtonEnabled(true);

        //Set the prompt text displayed by default in this searchview
        searchView.setQueryHint("search");
        searchView.setOnQueryTextListener(this);

    }
    public boolean onQueryTextChange(String string) {
        String selection1 = NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?";
        String[] selection2 = {"%"+string+"%","%"+string+"%"};
        Cursor cursor = sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION, // The columns to return from the query
                selection1, // The columns for the where clause
                selection2, // The values for the where clause
                null,          // don't group the rows
                null,          // don't filter by row groups
                NotePad.Notes.DEFAULT_SORT_ORDER // The sort order
        );
        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        } ;
        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = {
                android.R.id.text1,
                android.R.id.text2
        };
        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,         // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        // Sets the ListView's adapter to be the cursor adapter that was just created.
        listView.setAdapter(adapter);
        return true;
    }
}
2.5在清单文件AndroidManifest.xml里面注册NoteSearch。
代码：
        <activity android:name=".NoteSearch" android:label="@string/search_note" />
2.6图片展示
![4 `WIVWX$1 WW% })SAEO08](https://user-images.githubusercontent.com/114241292/202895306-75d06616-921e-41fb-8fb3-78d040e132d7.png)
3UI美化
3.1给NotesList换个主题，把黑色换成白色，在AndroidManifest.xml中NotesList的Activity中添加。
代码：
        <activity android:name="NotesList" android:label="@string/title_notes_list"
                  android:theme="@android:style/Theme.Holo.Light">
3.2在NotePad.java中添加:
代码：
        public static final String COLUMN_NAME_BACK_COLOR = "color";
3.3创建数据库表地方添加颜色的字段。
代码：
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + "   ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //color
                   + ");");
       }
3.4在NotePad.java中定义：
代码：
        public static final int DEFAULT_COLOR = 0; //white
        public static final int YELLOW_COLOR = 1; //yellow
        public static final int BLUE_COLOR = 2; //blue
        public static final int GREEN_COLOR = 3; //green
        public static final int RED_COLOR = 4; //red
3.5在NotePadProvider.java中添加对其相应的处理，
static中：
        sNotesProjectionMap.put(
                NotePad.Notes.COLUMN_NAME_BACK_COLOR,
                NotePad.Notes.COLUMN_NAME_BACK_COLOR);
insert中：

        // Create a new notepad. The background is white by default
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }
3.6自定义一个MyCursorAdapter.java继承SimpleCursorAdapter，将颜色填充到ListView。
代码：
package com.example.android.notepad;

import android.content.Context;
import android.database.Cursor;
import android.graphics.Color;
import android.view.View;
import android.widget.SimpleCursorAdapter;

public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //Get the color data corresponding to the note list from the cursor read from the database, and set the note color
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        /**
         * white 255 255 255
         * yellow 247 216 133
         * blue 165 202 237
         * green 161 214 174
         * red 244 149 133
         */
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
3.7在NotesList.java中的PROJECTION添加颜色项。
代码：
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //Extended:display time, color
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
3.8将NotesList.java中用的SimpleCursorAdapter改为使用MyCursorAdapter：
代码：
//        // Creates the backing adapter for the ListView.
//        SimpleCursorAdapter adapter
//            = new SimpleCursorAdapter(
//                      this,                             // The Context for the ListView
//                      R.layout.noteslist_item,          // Points to the XML for a list item
//                      cursor,                           // The cursor to get items from
//                      dataColumns,
//                      viewIDs
//              );
        //Modify to a custom adapter that can be filled with color. The custom code is in MyCursorAdapter.java
        MyCursorAdapter adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
3.9图片展示
![D(`{`9Z5FYUZZOWLSSC{X{4](https://user-images.githubusercontent.com/114241292/202895513-ec8ece85-c4d5-4801-bd93-2e9c939be280.png)

4更改背景颜色
4.1在editor_options_menu.xml中添加一个更改背景的功能选项。
代码：
    <item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/ic_menu_edit"
        android:showAsAction="always"/>
4.2在NoteEditor.java中找到onOptionsItemSelected方法，在菜单的switch中添加：
代码：
        //Change background color option
        case R.id.menu_color:
            changeColor();
            break;
4.3在NoteEditor.java中添加函数changeColor：
代码：
    //Jump to the activity changing the color and transfer the URI information to the new activity
    private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
4.4新建布局note_color.xml，垂直线性布局放置5个ImageButton，对选择颜色界面进行布局。
代码：
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorWhite"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorYellow"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorBlue"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorGreen"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorRed"
        android:onClick="red"/>
</LinearLayout>
4.5新建资源文件color.xml,添加所需颜色。
代码：
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <color name="colorWhite">#fff</color>
    <color name="colorYellow">#FFD885</color>
    <color name="colorBlue">#A5CAED</color>
    <color name="colorGreen">#A1D6AE</color>
    <color name="colorRed">#F49585</color>

</resources>
4.6创建NoteColor.java，用来选择颜色。代码见库。
4.7在清单文件AndroidManifest.xml里面注册NoteColor
代码：
        <!--Change background color-->
        <activity android:name="NoteColor"
            android:theme="@android:style/Theme.Holo.Light.Dialog"
            android:label="ChangeColor"
            android:windowSoftInputMode="stateVisible"/>
4.8图片展示
![9V54{R6{1}JQX0QGAUV~G Q](https://user-images.githubusercontent.com/114241292/202895666-6b10b32c-2dea-44ba-8d1e-38cd7492a3ee.png)
![SL{J9H%S_`G82NS08YA{%{T](https://user-images.githubusercontent.com/114241292/202895677-527e7a70-3dbc-4991-9d8d-33b817c5bccd.png)



