# Simple Solitaire 3


```diff

app/build.gradle                              |   2 +-
 app/src/main/assets/changelog.html            |  47 +++++
 .../solitaire/ui/about/ChangeLogFragment.java |  81 ++++++++-
 .../solitaire/ui/about/LicenseFragment.java   |  30 +++-
 .../main/res/layout/content_about_tab2.xml    |   5 +-
 .../main/res/layout/content_about_tab3.xml    | 161 +-----------------
 6 files changed, 161 insertions(+), 165 deletions(-)
 create mode 100644 app/src/main/assets/changelog.html

diff --git a/app/build.gradle b/app/build.gradle
index 3be4bf6f..0d53b152 100644
--- a/app/build.gradle
+++ b/app/build.gradle
@@ -8,7 +8,7 @@ android {
         applicationId "de.tobiasbielefeld.solitaire"
         minSdkVersion 11
         targetSdkVersion 25
-        versionCode 16
+        versionCode 17
         versionName "2.0.2"
     }
     buildTypes {
diff --git a/app/src/main/assets/changelog.html b/app/src/main/assets/changelog.html
new file mode 100644
index 00000000..803c2ef3
--- /dev/null
+++ b/app/src/main/assets/changelog.html
@@ -0,0 +1,47 @@
+<h3>Ver. 2.0.2</h3>
+<p>
+    More bug fixes
+</p>
+
+<h3>Ver. 2.0.1</h3>
+<p>
+    Some bug fixes
+</p>
+
+<h3>Ver. 2.0</h3>
+<p>
+    Added more games: Freecell, Yukon, Spider, Simple Simon and Golf!<br>
+    Added a new basic card set with larger icon<br>
+    Changed the settings and added more options<br>
+    Also improved the about screen<br>
+</p>
+
+<h3>Ver. 1.4</h3>
+<p>
+    Fixed an error when deleting high scores<br>
+    Added an option to set screen orientation<br>
+</p>
+
+<h3>Ver. 1.3</h3>
+<p>
+    Moved the about button to the settings<br>
+    Some visual changes<br>
+    Game is now licensed under GPL 3.0!<br>
+</p>
+
+<h3>Ver. 1.2</h3>
+<p>
+    Added 2 new background colors<br>
+    Changed the auto complete question to a button<br>
+    Drawable updates for foundation and stock<br>
+</p>
+
+<h3>Ver. 1.1</h3>
+<p>
+    Improved settings
+</p>
+
+<h3>Ver. 1.0</h3>
+<p>
+    Initial Release
+</p>
\ No newline at end of file
diff --git a/app/src/main/java/de/tobiasbielefeld/solitaire/ui/about/ChangeLogFragment.java b/app/src/main/java/de/tobiasbielefeld/solitaire/ui/about/ChangeLogFragment.java
index a55a21b7..7cdf28f9 100644
--- a/app/src/main/java/de/tobiasbielefeld/solitaire/ui/about/ChangeLogFragment.java
+++ b/app/src/main/java/de/tobiasbielefeld/solitaire/ui/about/ChangeLogFragment.java
@@ -20,9 +20,17 @@
 
 import android.os.Bundle;
 import android.support.v4.app.Fragment;
+import android.text.Html;
 import android.view.LayoutInflater;
 import android.view.View;
 import android.view.ViewGroup;
+import android.widget.ScrollView;
+import android.widget.TextView;
+
+import java.io.BufferedReader;
+import java.io.IOException;
+import java.io.InputStream;
+import java.io.InputStreamReader;
 
 import de.tobiasbielefeld.solitaire.R;
 
@@ -32,13 +40,80 @@
 
 public class ChangeLogFragment extends Fragment implements View.OnClickListener {
 
+    ScrollView scrollView;
+    TextView textView;
+
     @Override
     public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
-        return inflater.inflate(R.layout.content_about_tab3, container, false);
+        View view = inflater.inflate(R.layout.content_about_tab3, container, false);
+
+        textView = (TextView) view.findViewById(R.id.aboutTab3Changelog);
+        scrollView = (ScrollView) view.findViewById(R.id.aboutTab3Scrollview);
+
+        //show the beautiful gpl license from the license.html in the assets folder
+        try {
+            InputStream is = getActivity().getAssets().open("changelog.html");
+            textView.setText(Html.fromHtml(new String(getStringFromInputStream(is))));
+        } catch (IOException ignored) {}
+
+        //this answer helped me to retain the scroll position on orientation change: http://stackoverflow.com/a/15686638/7016229
+        if (savedInstanceState!=null) {
+            final int firstVisibleCharacterOffset = savedInstanceState.getInt("SCROLL_POSITION", 0);
+
+            scrollView.post(new Runnable() {
+                public void run() {
+                    int firstVisibleLineOffset = textView.getLayout().getLineForOffset(firstVisibleCharacterOffset);
+                    int pixelOffset = textView.getLayout().getLineTop(firstVisibleLineOffset);
+                    scrollView.scrollTo(0, pixelOffset);
+                }
+            });
+        }
+
+        return view;
+    }
+
+    //this answer helped me to retain the scroll position on orientation change: http://stackoverflow.com/a/15686638/7016229
+    public void onSaveInstanceState(Bundle outState) {
+        super.onSaveInstanceState(outState);
+        int firstVisibleLineOffset = textView.getLayout().getLineForVertical(scrollView.getScrollY());
+        int firstVisibleCharacterOffset = textView.getLayout().getLineStart(firstVisibleLineOffset);
+        outState.putInt("SCROLL_POSITION", firstVisibleCharacterOffset);
     }
 
     @Override
-    public void onClick(View view) {
-        //nothing!!
+    public void onClick(View v) {
+        //nothing
+    }
+
+    /*
+     * Solution from StackOverflow to read a html file, found here:
+     * https://stackoverflow.com/questions/2436385/android-getting-from-a-uri-to-an-inputstream-to-a-byte-array
+     */
+    private byte[] getStringFromInputStream(InputStream is) {
+
+        BufferedReader br = null;
+        StringBuilder sb = new StringBuilder();
+        byte[] bReturn = new byte[0];
+        String line;
+
+        try {
+            br = new BufferedReader(new InputStreamReader(is, "UTF-8"));
+            while ((line = br.readLine()) != null) {
+                sb.append(line).append(" ");
+            }
+            String sContent = sb.toString();
+            bReturn = sContent.getBytes();
+        }
+        catch (IOException ignored) {
+
+        } finally {
+            if (br != null) {
+                try {
+                    br.close();
+                }
+                catch (IOException ignored) { }
+            }
+        }
+        return bReturn;
     }
 }
\ No newline at end of file
diff --git a/app/src/main/java/de/tobiasbielefeld/solitaire/ui/about/LicenseFragment.java b/app/src/main/java/de/tobiasbielefeld/solitaire/ui/about/LicenseFragment.java
index c66dfc15..b06fbe43 100644
--- a/app/src/main/java/de/tobiasbielefeld/solitaire/ui/about/LicenseFragment.java
+++ b/app/src/main/java/de/tobiasbielefeld/solitaire/ui/about/LicenseFragment.java
@@ -24,6 +24,7 @@
 import android.view.LayoutInflater;
 import android.view.View;
 import android.view.ViewGroup;
+import android.widget.ScrollView;
 import android.widget.TextView;
 
 import java.io.BufferedReader;
@@ -39,21 +40,46 @@
 
 public class LicenseFragment extends Fragment implements View.OnClickListener {
 
+    ScrollView scrollView;
+    TextView licenseText;
+
     @Override
     public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
         View view = inflater.inflate(R.layout.content_about_tab2, container, false);
 
-        TextView textViewGPLLicense = (TextView) view.findViewById(R.id.aboutLicenseText);          //my app license
+        licenseText = (TextView) view.findViewById(R.id.aboutTab2LicenseText);          //my app license
+        scrollView = (ScrollView) view.findViewById(R.id.aboutTab2Scrollview);
 
         //show the beautiful gpl license from the license.html in the assets folder
         try {
             InputStream is = getActivity().getAssets().open("license.html");
-            textViewGPLLicense.setText(Html.fromHtml(new String(getStringFromInputStream(is))));
+            licenseText.setText(Html.fromHtml(new String(getStringFromInputStream(is))));
         } catch (IOException ignored) {}
 
+        //this answer helped me to retain the scroll position on orientation change: http://stackoverflow.com/a/15686638/7016229
+        if (savedInstanceState!=null) {
+            final int firstVisibleCharacterOffset = savedInstanceState.getInt("SCROLL_POSITION", 0);
+
+            scrollView.post(new Runnable() {
+                public void run() {
+                    int firstVisibleLineOffset = licenseText.getLayout().getLineForOffset(firstVisibleCharacterOffset);
+                    int pixelOffset = licenseText.getLayout().getLineTop(firstVisibleLineOffset);
+                    scrollView.scrollTo(0, pixelOffset);
+                }
+            });
+        }
+
         return view;
     }
 
+    //this answer helped me to retain the scroll position on orientation change: http://stackoverflow.com/a/15686638/7016229
+    public void onSaveInstanceState(Bundle outState) {
+        super.onSaveInstanceState(outState);
+        int firstVisibleLineOffset = licenseText.getLayout().getLineForVertical(scrollView.getScrollY());
+        int firstVisibleCharacterOffset = licenseText.getLayout().getLineStart(firstVisibleLineOffset);
+        outState.putInt("SCROLL_POSITION", firstVisibleCharacterOffset);
+    }
+
     @Override
     public void onClick(View v) {
        //nothing
diff --git a/app/src/main/res/layout/content_about_tab2.xml b/app/src/main/res/layout/content_about_tab2.xml
index 755c92b4..7e5f56cc 100644
--- a/app/src/main/res/layout/content_about_tab2.xml
+++ b/app/src/main/res/layout/content_about_tab2.xml
@@ -1,7 +1,8 @@
 <?xml version="1.0" encoding="utf-8"?>
 <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
     android:layout_width="match_parent"
-    android:layout_height="wrap_content">
+    android:layout_height="wrap_content"
+    android:id="@+id/aboutTab2Scrollview">
 
     <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
         android:layout_width="match_parent"
@@ -23,7 +24,7 @@
             android:textStyle="normal|bold" />
 
         <TextView
-            android:id="@+id/aboutLicenseText"
+            android:id="@+id/aboutTab2LicenseText"
             android:layout_width="wrap_content"
             android:layout_height="wrap_content" />
     </LinearLayout>
diff --git a/app/src/main/res/layout/content_about_tab3.xml b/app/src/main/res/layout/content_about_tab3.xml
index fb6b0a02..d7a6f87e 100644
--- a/app/src/main/res/layout/content_about_tab3.xml
+++ b/app/src/main/res/layout/content_about_tab3.xml
@@ -1,7 +1,8 @@
 <?xml version="1.0" encoding="utf-8"?>
 <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
     android:layout_width="match_parent"
-    android:layout_height="wrap_content">
+    android:layout_height="wrap_content"
+    android:id="@+id/aboutTab3Scrollview">
 
     <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
         android:layout_width="match_parent"
@@ -13,163 +14,9 @@
         <TextView
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
-            android:text="Ver. 2.0.2"
+            android:text="Loading..."
             android:textAppearance="?android:attr/textAppearanceSmall"
-            android:textStyle="normal|bold" />
+            android:id="@+id/aboutTab3Changelog" />
 
-        <TextView
-            android:layout_width="match_parent"
-            android:layout_height="wrap_content"
-            android:layout_marginBottom="20dp"
-            android:text="More bug fixes"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Ver. 2.0.1"
-            android:textAppearance="?android:attr/textAppearanceSmall"
-            android:textStyle="normal|bold" />
-
-        <TextView
-            android:layout_width="match_parent"
-            android:layout_height="wrap_content"
-            android:layout_marginBottom="20dp"
-            android:text="Some bug fixes"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Ver. 2.0"
-            android:textAppearance="?android:attr/textAppearanceSmall"
-            android:textStyle="normal|bold" />
-
-        <TextView
-            android:layout_width="match_parent"
-            android:layout_height="wrap_content"
-            android:text="Added more games: Freecell, Yukon, Spider, Simple Simon and Golf!"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="match_parent"
-            android:layout_height="wrap_content"
-            android:text="Added a new basic card set with larger icons"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="match_parent"
-            android:layout_height="wrap_content"
-            android:text="Changed the settings and added more options"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="match_parent"
-            android:layout_height="wrap_content"
-            android:layout_marginBottom="20dp"
-            android:text="Also improved the about screen"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Ver. 1.4"
-            android:textAppearance="?android:attr/textAppearanceSmall"
-            android:textStyle="normal|bold" />
-
-        <TextView
-            android:layout_width="match_parent"
-            android:layout_height="wrap_content"
-            android:text="Fixed an error when deleting high scores"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="match_parent"
-            android:layout_height="wrap_content"
-            android:layout_marginBottom="20dp"
-            android:text="Added an option to set screen orientation"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="match_parent"
-            android:layout_height="wrap_content"
-            android:text="Ver. 1.3"
-            android:textAppearance="?android:attr/textAppearanceSmall"
-            android:textStyle="normal|bold" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Moved the about button to the settings"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Some visual changes"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:layout_marginBottom="20dp"
-            android:text="Game is now licensed under GPL 3.0!"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Ver. 1.2"
-            android:textAppearance="?android:attr/textAppearanceSmall"
-            android:textStyle="normal|bold" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Added 2 new background colors"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Changed the auto complete question to a button"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:layout_marginBottom="20dp"
-            android:text="Drawable updates for foundation and stock"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Ver. 1.1"
-            android:textAppearance="?android:attr/textAppearanceSmall"
-            android:textStyle="normal|bold" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:layout_marginBottom="20dp"
-            android:text="Improved settings"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:text="Ver. 1.0"
-            android:textAppearance="?android:attr/textAppearanceSmall"
-            android:textStyle="normal|bold" />
-
-        <TextView
-            android:layout_width="wrap_content"
-            android:layout_height="wrap_content"
-            android:layout_marginBottom="20dp"
-            android:text="Initial Release"
-            android:textAppearance="?android:attr/textAppearanceSmall" />
     </LinearLayout>
 </ScrollView>
 
 ```
