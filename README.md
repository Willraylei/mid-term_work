# NotePad

This is an AndroidStudio rebuild of google SDK sample NotePad

## 部分功能及截图和相关代码如下
1. 1 Notelist中增加条目显示时间戳功能
截图如下:<br>
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%204.31.29%20PM.png)<br>
关键代码如下:<br>
                
        private final void updateNote(String text, String title) {

        // Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, System.currentTimeMillis());
        SimpleDateFormat format = new SimpleDateFormat("dd/MM/yyyy HH:mm:ss");
        String t=format.format(new Date());
        // If the action is to insert a new note, this creates an initial title for it.
        if (mState == STATE_INSERT) {

            // If no title was provided as an argument, create one from the note text.
            if (title == null) {
  
                // Get the note's length
                int length = text.length();

                // Sets the title by getting a substring of the text that is 31 characters long
                // or the number of characters in the note plus one, whichever is smaller.
                title = text.substring(0, Math.min(30, length));
  
                // If the resulting length is more than 30 characters, chops off any
                // trailing spaces
                if (length > 30) {
                    int lastSpace = title.lastIndexOf(' ');
                    if (lastSpace > 0) {
                        title = title.substring(0, lastSpace);
                    }
                }
            }
            // In the values map, sets the value of the title

            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE,t);
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,t);
        } else if (title != null) {
            // In the values map, sets the value of the title
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);

        }

        // This puts the desired notes text into the map.
        values.put(NotePad.Notes.COLUMN_NAME_NOTE, text);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,t);
        values.put(NotePad.Notes.COLOR,1);
        /*
         * Updates the provider with the new values in the map. The ListView is updated
         * automatically. The provider sets this up by setting the notification URI for
         * query Cursor objects to the incoming URI. The content resolver is thus
         * automatically notified when the Cursor for the URI changes, and the UI is
         * updated.
         * Note: This is being done on the UI thread. It will block the thread until the
         * update completes. In a sample app, going against a simple provider based on a
         * local database, the block will be momentary, but in a real app you should use
         * android.content.AsyncQueryHandler or android.os.AsyncTask.
         */
        getContentResolver().update(
                mUri,    // The URI for the record to update.
                values,  // The map of column names and new values to apply to them.
                null,    // No selection criteria are used, so no where columns are necessary.
                null     // No where columns are used, so no where arguments are necessary.
            );}

1. 2添加笔记查询内容,根据标题查询
截图如下:<br>
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%204.36.51%20PM.png)<br>
关键代码如下:<br>
     
     protected void onResume()
    {
        super.onResume();;
        listView=(ListView) findViewById(R.id.mainlist);
        editText=(EditText) findViewById(R.id.mainedit);
       Button button= (Button) findViewById(R.id.addbutton);
              button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivity(new Intent(Intent.ACTION_INSERT, getIntent().getData()));
            }
        });
          listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
              @Override
              public void onItemClick(AdapterView<?> parent, View view, int position, long id) {

                  // Constructs a new URI from the incoming URI and the row ID
                  onListItemClick(parent,view,position,id);}
          });

        setlistView();
        editText.addTextChangedListener(textWatcher);


     }

    private  TextWatcher textWatcher=new TextWatcher() {
        @Override
        public void beforeTextChanged(CharSequence s, int start, int count, int after) {

        }

        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {

            Intent intent = getIntent();

            // If there is no data associated with the Intent, sets the data to the default URI, which
            // accesses a list of notes.
            if (intent.getData() == null) {
                intent.setData(NotePad.Notes.CONTENT_URI);
            }

        /*
         * Sets the callback for context menu activation for the ListView. The listener is set
         * to be this Activity. The effect is that context menus are enabled for items in the
         * ListView, and the context menu is handled by a method in NotesList.
         */
            listView.setOnCreateContextMenuListener(NotesList.this);

        /* Performs a managed query. The Activity handles closing and requerying the cursor
         * when needed.
         *
         * Please see the introductory note about performing provider operations on the UI thread.
         */
            cursor = managedQuery(
                    getIntent().getData(),                                                           
                    // Use the default content URI for the provider.
                    PROJECTION,                                                                      
                    // Return the note ID and title for each note.
                    NotePad.Notes.COLUMN_NAME_TITLE+"   "+"like ?",                                  
                    // No where clause, return all records.
                    new String[]{"%"+editText.getText().toString()+"%"},                             
                    // No where clause, therefore no where column values.
                    null // Use the default sort order.
            );
            displayView();
        }
        @Override
        public void afterTextChanged(Editable s) {}};
       
2. 笔记本的更多细小的功能如(增加,删除,拷贝粘贴等对笔记本的基本操作和对用户比较方便的操作),进行了一定的UI美化,让界面看上去不会太突兀.
如下图:<br>
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%205.53.12%20PM.png)
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%205.53.30%20PM.png)<br>
如下图:<br>
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%204.31.05%20PM.png)
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%204.37.31%20PM.png)<br>



