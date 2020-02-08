# Loop Habit Tracker 239


```diff

 app/src/main/res/layout/about.xml            | 1 +
 app/src/main/res/layout/show_habit_inner.xml | 1 +
 2 files changed, 2 insertions(+)

diff --git a/app/src/main/res/layout/about.xml b/app/src/main/res/layout/about.xml
index 0d391a70..ea0d8899 100644
--- a/app/src/main/res/layout/about.xml
+++ b/app/src/main/res/layout/about.xml
@@ -31,6 +31,7 @@
         style="@style/Toolbar"/>
 
     <ScrollView
+        android:id="@+id/scrollView"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:layout_below="@id/toolbar">
diff --git a/app/src/main/res/layout/show_habit_inner.xml b/app/src/main/res/layout/show_habit_inner.xml
index 67b96a76..d73760ca 100644
--- a/app/src/main/res/layout/show_habit_inner.xml
+++ b/app/src/main/res/layout/show_habit_inner.xml
@@ -21,6 +21,7 @@
 <ScrollView
     xmlns:android="http://schemas.android.com/apk/res/android"
     xmlns:tools="http://schemas.android.com/tools"
+    android:id="@+id/scrollView"
     android:layout_width="fill_parent"
     android:layout_height="wrap_content"
     android:layout_below="@id/toolbar"
```
