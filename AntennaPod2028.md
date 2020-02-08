# AntennaPod 2028

```diff
 app/src/main/AndroidManifest.xml                    | 13 +++++++------
 .../antennapod/activity/PreferenceActivity.java     |  1 +
 2 files changed, 8 insertions(+), 6 deletions(-)

diff --git a/app/src/main/AndroidManifest.xml b/app/src/main/AndroidManifest.xml
index 0b8bf09ec..9194636a7 100644
--- a/app/src/main/AndroidManifest.xml
+++ b/app/src/main/AndroidManifest.xml
@@ -43,7 +43,7 @@
 
         <activity
             android:name=".activity.MainActivity"
-            android:configChanges="keyboardHidden|orientation"
+            android:configChanges="keyboardHidden|orientation|screenSize"
             android:launchMode="singleTask"
             android:label="@string/app_name">
             <intent-filter>
@@ -81,7 +81,7 @@
 
         <activity
             android:name=".activity.PreferenceActivityGingerbread"
-            android:configChanges="keyboardHidden|orientation"
+            android:configChanges="keyboardHidden|orientation|screenSize"
             android:label="@string/settings_label">
             <meta-data
                 android:name="android.support.PARENT_ACTIVITY"
@@ -90,7 +90,7 @@
 
         <activity
             android:name=".activity.PreferenceActivity"
-            android:configChanges="keyboardHidden|orientation"
+            android:configChanges="keyboardHidden|orientation|screenSize"
             android:label="@string/settings_label">
             <meta-data
                 android:name="android.support.PARENT_ACTIVITY"
@@ -146,6 +146,7 @@
         </activity>
         <activity
             android:name=".activity.AboutActivity"
+            android:configChanges="keyboardHidden|orientation|screenSize"
             android:label="@string/about_pref">
             <meta-data
                 android:name="android.support.PARENT_ACTIVITY"
@@ -160,12 +161,12 @@
         </activity>
         <activity
             android:name=".activity.OpmlImportFromPathActivity"
-            android:configChanges="keyboardHidden|orientation"
+            android:configChanges="keyboardHidden|orientation|screenSize"
             android:label="@string/opml_import_label">
         </activity>
         <activity
             android:name=".activity.OpmlImportFromIntentActivity"
-            android:configChanges="keyboardHidden|orientation"
+            android:configChanges="keyboardHidden|orientation|screenSize"
             android:label="@string/opml_import_label">
             <intent-filter>
                 <action android:name="android.intent.action.VIEW"/>
@@ -217,7 +218,7 @@
 
         <activity
             android:name=".activity.OnlineFeedViewActivity"
-            android:configChanges="orientation"
+            android:configChanges="orientation|screenSize"
             android:label="@string/add_feed_label">
             <meta-data
                 android:name="android.support.PARENT_ACTIVITY"
diff --git a/app/src/main/java/de/danoeh/antennapod/activity/PreferenceActivity.java b/app/src/main/java/de/danoeh/antennapod/activity/PreferenceActivity.java
index ba22a42b4..4e482fb2f 100644
--- a/app/src/main/java/de/danoeh/antennapod/activity/PreferenceActivity.java
+++ b/app/src/main/java/de/danoeh/antennapod/activity/PreferenceActivity.java
@@ -103,6 +103,7 @@ public boolean onOptionsItemSelected(MenuItem item) {
         @Override
         public void onCreate(Bundle savedInstanceState) {
             super.onCreate(savedInstanceState);
+            setRetainInstance(true);
             addPreferencesFromResource(R.xml.preferences);
             PreferenceActivity activity = instance.get();
             if(activity != null && activity.preferenceController != null) {


```
