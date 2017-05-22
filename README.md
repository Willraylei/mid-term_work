# NotePad

This is an AndroidStudio rebuild of google SDK sample NotePad

## Notepad功能,截图和相关代码如下
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

1. 2 添加笔记查询内容,根据标题查询
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
     
2. 1 在进行笔记本内容编辑时,会显示编辑的相关提示

    
          // Modifies the window title for the Activity according to the current Activity state.
            if (mState == STATE_EDIT) {
                // Set the title of the Activity to include the note title
                int colTitleIndex = mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE);
                String title = mCursor.getString(colTitleIndex);
                Resources res = getResources();
                String text = String.format(res.getString(R.string.title_edit), title);
                setTitle(text);
            // Sets the title to "create" for inserts
            } else if (mState == STATE_INSERT) {
                setTitle(getText(R.string.title_create));
            }

2. 2 在NoteList页面进行选择即SelectItemMenu


       /**
        * This method is called when a menu item is selected. Android passes in the selected item.
        * The switch statement in this method calls the appropriate method to perform the action the
        * user chose.
        *
        * @param item The selected MenuItem
        * @return True to indicate that the item was processed, and no further work is necessary. False
        * to proceed to further processing as indicated in the MenuItem object.
        */
        @Override
        public boolean onOptionsItemSelected(MenuItem item) {
        // Handle all of the possible menu actions.
        switch (item.getItemId()) {
        case R.id.menu_save:
            String text = mText.getText().toString();
            updateNote(text, null);
            finish();
            break;
        case R.id.menu_delete:
            deleteNote();
            finish();
            break;
        case R.id.menu_revert:
            cancelNote();
            break;
        }
        return super.onOptionsItemSelected(item); }

2. 3 笔记本的粘贴功能(在粘贴前首先要进行copy)

        
       //BEGIN_INCLUDE(paste)
       /**
       * A helper method that replaces the note's data with the contents of the clipboard.
       */
       private final void performPaste() {

        // Gets a handle to the Clipboard Manager
        ClipboardManager clipboard = (ClipboardManager)
                getSystemService(Context.CLIPBOARD_SERVICE);

        // Gets a content resolver instance
        ContentResolver cr = getContentResolver();

        // Gets the clipboard data from the clipboard
        ClipData clip = clipboard.getPrimaryClip();
        if (clip != null) {

            String text=null;
            String title=null;

            // Gets the first item from the clipboard data
            ClipData.Item item = clip.getItemAt(0);

            // Tries to get the item's contents as a URI pointing to a note
            Uri uri = item.getUri();

            // Tests to see that the item actually is an URI, and that the URI
            // is a content URI pointing to a provider whose MIME type is the same
            // as the MIME type supported by the Note pad provider.
            if (uri != null && NotePad.Notes.CONTENT_ITEM_TYPE.equals(cr.getType(uri))) {

                // The clipboard holds a reference to data with a note MIME type. This copies it.
                Cursor orig = cr.query(
                        uri,            // URI for the content provider
                        PROJECTION,     // Get the columns referred to in the projection
                        null,           // No selection variables
                        null,           // No selection variables, so no criteria are needed
                        null            // Use the default sort order
                );

                // If the Cursor is not null, and it contains at least one record
                // (moveToFirst() returns true), then this gets the note data from it.
                if (orig != null) {
                    if (orig.moveToFirst()) {
                        int colNoteIndex = mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE);
                        int colTitleIndex = mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE);
                        text = orig.getString(colNoteIndex);
                        title = orig.getString(colTitleIndex);
                    }

                    // Closes the cursor.
                    orig.close();
                }
            }

            // If the contents of the clipboard wasn't a reference to a note, then
            // this converts whatever it is to text.
            if (text == null) {
                text = item.coerceToText(this).toString();
            }

            // Updates the current note with the retrieved title and text.
            updateNote(text, title);
        }
        }
        //END_INCLUDE(paste)
        
2. 4 删除笔记

        /**
       * This helper method cancels the work done on a note.  It deletes the note if it was
       * newly created, or reverts to the original text of the note i
       */
       private final void cancelNote() {
        if (mCursor != null) {
            if (mState == STATE_EDIT) {
                // Put the original note text back into the database
                mCursor.close();
                mCursor = null;
                ContentValues values = new ContentValues();
                values.put(NotePad.Notes.COLUMN_NAME_NOTE, mOriginalContent);
                getContentResolver().update(mUri, values, null, null);
            } else if (mState == STATE_INSERT) {
                // We inserted an empty note, make sure to delete it
                deleteNote();
            }
        }
        setResult(RESULT_CANCELED);
        finish();
        }

       /**
       * Take care of deleting a note.  Simply deletes the entry.
       */
       private final void deleteNote() {
        if (mCursor != null) {
            mCursor.close();
            mCursor = null;
            getContentResolver().delete(mUri, null, null);
            mText.setText("");
        } }
        

2. 5 笔记本的更多细小的功能如(增加,删除,拷贝粘贴等对笔记本的基本操作和对用户比较方便的操作),进行了一定的UI美化,让界面看上去不会太突兀.
如下图:<br>
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%205.53.12%20PM.png)
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%205.53.30%20PM.png)<br>
如下图:<br>
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%204.31.05%20PM.png)
![image](https://github.com/Willraylei/mid-term_work/blob/master/Screen%20Shot%202017-05-21%20at%204.37.31%20PM.png)<br>



