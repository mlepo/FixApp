# Loop Habit Tracker 241

```diff
 .../habits/list/ListHabitsScreen.java         | 52 ++++++++++++++++++-
 1 file changed, 50 insertions(+), 2 deletions(-)

diff --git a/app/src/main/java/org/isoron/uhabits/activities/habits/list/ListHabitsScreen.java b/app/src/main/java/org/isoron/uhabits/activities/habits/list/ListHabitsScreen.java
index 3fb92e81..1238daad 100644
--- a/app/src/main/java/org/isoron/uhabits/activities/habits/list/ListHabitsScreen.java
+++ b/app/src/main/java/org/isoron/uhabits/activities/habits/list/ListHabitsScreen.java
@@ -20,6 +20,7 @@
 package org.isoron.uhabits.activities.habits.list;
 
 import android.content.*;
+import android.net.*;
 import android.support.annotation.*;
 
 import org.isoron.uhabits.*;
@@ -31,24 +32,30 @@
 import org.isoron.uhabits.intents.*;
 import org.isoron.uhabits.io.*;
 import org.isoron.uhabits.models.*;
+import org.isoron.uhabits.utils.*;
 
 import java.io.*;
 
 import javax.inject.*;
 
+import static android.os.Build.VERSION.*;
+import static android.os.Build.VERSION_CODES.*;
+
 @ActivityScope
 public class ListHabitsScreen extends BaseScreen
     implements CommandRunner.Listener
 {
-    public static final int RESULT_BUG_REPORT = 4;
+    public static final int RESULT_IMPORT_DATA = 1;
 
     public static final int RESULT_EXPORT_CSV = 2;
 
     public static final int RESULT_EXPORT_DB = 3;
 
+    public static final int RESULT_BUG_REPORT = 4;
+
     public static final int RESULT_REPAIR_DB = 5;
 
-    public static final int RESULT_IMPORT_DATA = 1;
+    public static final int REQUEST_OPEN_DOCUMENT = 6;
 
     @Nullable
     private ListHabitsController controller;
@@ -128,6 +135,31 @@ public void onResult(int requestCode, int resultCode, Intent data)
     {
         if (controller == null) return;
 
+        if (requestCode == REQUEST_OPEN_DOCUMENT)
+        {
+            if(resultCode != BaseActivity.RESULT_OK) return;
+            try
+            {
+                // TODO: Make it async
+                // TODO: Remove temporary file at the end of operation
+                Uri uri = data.getData();
+                ContentResolver cr = activity.getContentResolver();
+                InputStream is = cr.openInputStream(uri);
+
+                File cacheDir = activity.getCacheDir();
+                File tempFile = File.createTempFile("import", "", cacheDir);
+
+                FileUtils.copy(is, tempFile);
+                controller.onImportData(tempFile);
+            }
+            catch (IOException e)
+            {
+                showMessage(R.string.could_not_import);
+                e.printStackTrace();
+                return;
+            }
+        }
+
         switch (resultCode)
         {
             case RESULT_IMPORT_DATA:
@@ -208,6 +240,21 @@ public void showHabitScreen(@NonNull Habit habit)
     }
 
     public void showImportScreen()
+    {
+        if (SDK_INT < KITKAT)
+        {
+            showImportScreenPreKitKat();
+            return;
+        }
+
+        Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
+        intent.addCategory(Intent.CATEGORY_OPENABLE);
+        intent.setType("*/*");
+
+        activity.startActivityForResult(intent, REQUEST_OPEN_DOCUMENT);
+    }
+
+    public void showImportScreenPreKitKat()
     {
         File dir = dirFinder.findStorageDir(null);
 
@@ -221,6 +268,7 @@ public void showImportScreen()
 
         if (controller != null)
             picker.setListener(file -> controller.onImportData(file));
+
         activity.showDialog(picker.getDialog());
     }
 


```
