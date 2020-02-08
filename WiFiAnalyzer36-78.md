# WiFiAnalyzer 36 / 78

```diff

 app/src/main/AndroidManifest.xml | 2 ++
 1 file changed, 2 insertions(+)

diff --git a/app/src/main/AndroidManifest.xml b/app/src/main/AndroidManifest.xml
index 447a7700..ca26ab35 100644
--- a/app/src/main/AndroidManifest.xml
+++ b/app/src/main/AndroidManifest.xml
@@ -40,6 +40,7 @@
         </activity>
         <activity
             android:name=".settings.SettingActivity"
+            android:configChanges="keyboardHidden|orientation|screenSize"
             android:label="@string/action_settings"
             android:launchMode="singleTask"
             android:parentActivityName=".MainActivity">
@@ -49,6 +50,7 @@
         </activity>
         <activity
             android:name=".about.AboutActivity"
+            android:configChanges="keyboardHidden|orientation|screenSize"
             android:label="@string/action_about"
             android:launchMode="singleTask"
             android:parentActivityName=".MainActivity">

```
