# DNS66

```diff
 .../dns66/main/WhitelistFragment.java         | 110 +++++++++++-------
 .../main/res/layout/activity_whitelist.xml    |   2 +-
 app/src/main/res/layout/whitelist_row.xml     |   4 +
 3 files changed, 72 insertions(+), 44 deletions(-)

diff --git a/app/src/main/java/org/jak_linux/dns66/main/WhitelistFragment.java b/app/src/main/java/org/jak_linux/dns66/main/WhitelistFragment.java
index 293fe3c..4dce1c4 100644
--- a/app/src/main/java/org/jak_linux/dns66/main/WhitelistFragment.java
+++ b/app/src/main/java/org/jak_linux/dns66/main/WhitelistFragment.java
@@ -11,17 +11,19 @@
 import android.content.pm.ApplicationInfo;
 import android.content.pm.PackageManager;
 import android.graphics.drawable.Drawable;
+import android.media.Image;
 import android.os.AsyncTask;
 import android.os.Bundle;
 import android.support.v4.app.Fragment;
 import android.support.v4.widget.SwipeRefreshLayout;
+import android.support.v7.widget.DividerItemDecoration;
+import android.support.v7.widget.LinearLayoutManager;
+import android.support.v7.widget.RecyclerView;
 import android.view.LayoutInflater;
 import android.view.View;
 import android.view.ViewGroup;
-import android.widget.ArrayAdapter;
 import android.widget.CompoundButton;
 import android.widget.ImageView;
-import android.widget.ListView;
 import android.widget.Switch;
 import android.widget.TextView;
 
@@ -44,7 +46,7 @@
 
     private static final String TAG = "Whitelist";
     private AppListGenerator appListGenerator;
-    private ListView appList;
+    private RecyclerView appList;
     private SwipeRefreshLayout swipeRefresh;
 
     @Override
@@ -54,7 +56,16 @@ public View onCreateView(LayoutInflater inflater, ViewGroup container,
 
         View rootView = inflater.inflate(R.layout.activity_whitelist, container, false);
 
-        appList = (ListView) rootView.findViewById(R.id.list);
+        appList = (RecyclerView) rootView.findViewById(R.id.list);
+        appList.setHasFixedSize(true);
+
+        RecyclerView.LayoutManager layoutManager = new LinearLayoutManager(getContext());
+        appList.setLayoutManager(layoutManager);
+
+        DividerItemDecoration dividerItemDecoration = new DividerItemDecoration(appList.getContext(),
+                DividerItemDecoration.VERTICAL);
+        appList.addItemDecoration(dividerItemDecoration);
+
 
         swipeRefresh = (SwipeRefreshLayout) rootView.findViewById(R.id.swiperefresh);
         swipeRefresh.setOnRefreshListener(
@@ -87,35 +98,35 @@ public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
         return rootView;
     }
 
-    private class AppListAdapter extends ArrayAdapter<ListEntry> {
-        AppListAdapter(Context context, int layout, List<ListEntry> list) {
-            super(context, layout, list);
+    private class AppListAdapter extends RecyclerView.Adapter<AppListAdapter.ViewHolder> {
+
+        public ArrayList<ListEntry> list;
+
+        public AppListAdapter(ArrayList<ListEntry> list) {
+            this.list = list;
         }
 
         @Override
-        public View getView(int position, View convertView, final ViewGroup parent) {
-            // Check if an existing view is being reused, otherwise inflate the view
-            if (convertView == null)
-                convertView = LayoutInflater.from(getContext()).inflate(R.layout.whitelist_row, parent, false);
-
-            final ListEntry entry = getItem(position);
+        public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
+            return new ViewHolder(LayoutInflater.from(getContext()).inflate(R.layout.whitelist_row, parent, false));
+        }
 
-            final ImageView iconView = (ImageView) convertView.findViewById(R.id.app_icon);
+        @Override
+        public void onBindViewHolder(final ViewHolder holder, int position) {
+            final ListEntry entry = list.get(position);
 
-            AsyncTask<ListEntry, Void, Drawable> task = (AsyncTask<ListEntry, Void, Drawable>) convertView.getTag();
-            if (task != null)
-                task.cancel(true);
+            if (holder.task != null)
+                holder.task.cancel(true);
 
-            task = null;
+            holder.task = null;
             final Drawable icon = entry.getIcon();
             if (icon != null) {
-                iconView.setImageDrawable(icon);
-                iconView.setVisibility(View.VISIBLE);
-                convertView.setTag(null);
+                holder.icon.setImageDrawable(icon);
+                holder.icon.setVisibility(View.VISIBLE);
             } else {
-                iconView.setVisibility(View.INVISIBLE);
+                holder.icon.setVisibility(View.INVISIBLE);
 
-                task = new AsyncTask<ListEntry, Void, Drawable>() {
+                holder.task = new AsyncTask<ListEntry, Void, Drawable>() {
                     @Override
                     protected Drawable doInBackground(ListEntry... entries) {
                         return entries[0].loadIcon(getContext().getPackageManager());
@@ -124,27 +135,20 @@ protected Drawable doInBackground(ListEntry... entries) {
                     @Override
                     protected void onPostExecute(Drawable drawable) {
                         if (!isCancelled()) {
-                            iconView.setImageDrawable(drawable);
-                            iconView.setVisibility(View.VISIBLE);
+                            holder.icon.setImageDrawable(drawable);
+                            holder.icon.setVisibility(View.VISIBLE);
                         }
                         super.onPostExecute(drawable);
                     }
                 };
-                convertView.setTag(task);
 
-                task.execute(entry);
+                holder.task.execute(entry);
             }
 
-            TextView textView = (TextView) convertView.findViewById(R.id.name);
-            textView.setText(entry.getLabel());
-
-
-            TextView details = (TextView) convertView.findViewById(R.id.details);
-            details.setText(entry.getPackageName());
 
-            final Switch checkBox = (Switch) convertView.findViewById(R.id.checkbox);
-
-            checkBox.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
+            holder.name.setText(entry.getLabel());
+            holder.details.setText(entry.getPackageName());
+            holder.whitelistSwitch.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
                 @Override
                 public void onCheckedChanged(CompoundButton compoundButton, boolean checked) {
                     /* No change, do nothing */
@@ -159,17 +163,37 @@ public void onCheckedChanged(CompoundButton compoundButton, boolean checked) {
                 }
             });
 
-            View layout = convertView.findViewById(R.id.entry);
-            layout.setOnClickListener(new View.OnClickListener() {
+
+            holder.itemView.setOnClickListener(new View.OnClickListener() {
                 @Override
                 public void onClick(View view) {
-                    checkBox.setChecked(!checkBox.isChecked());
+                    holder.whitelistSwitch.setChecked(!holder.whitelistSwitch.isChecked());
                 }
             });
 
-            checkBox.setChecked(MainActivity.config.whitelist.items.contains(entry.getPackageName()));
+            holder.whitelistSwitch.setChecked(MainActivity.config.whitelist.items.contains(entry.getPackageName()));
+
+        }
+
+        @Override
+        public int getItemCount() {
+            return list.size();
+        }
+
+        public class ViewHolder extends RecyclerView.ViewHolder {
+            public ViewHolder(View itemView) {
+                super(itemView);
+                icon = (ImageView) itemView.findViewById(R.id.app_icon);
+                name = (TextView) itemView.findViewById(R.id.name);
+                details = (TextView) itemView.findViewById(R.id.details);
+                whitelistSwitch = (Switch) itemView.findViewById(R.id.checkbox);
+            }
 
-            return convertView;
+            ImageView icon;
+            TextView name;
+            TextView details;
+            Switch whitelistSwitch;
+            AsyncTask<ListEntry, Void, Drawable> task;
         }
     }
 
@@ -184,7 +208,7 @@ protected AppListAdapter doInBackground(Void... params) {
 
             Collections.sort(info, new ApplicationInfo.DisplayNameComparator(pm));
 
-            final List<ListEntry> entries = new ArrayList<>();
+            final ArrayList<ListEntry> entries = new ArrayList<>();
             for (ApplicationInfo appInfo : info) {
                 if (!appInfo.packageName.equals(BuildConfig.APPLICATION_ID) && (MainActivity.config.whitelist.showSystemApps || (appInfo.flags & ApplicationInfo.FLAG_SYSTEM) == 0))
                     entries.add(new ListEntry(
@@ -194,7 +218,7 @@ protected AppListAdapter doInBackground(Void... params) {
             }
 
 
-            return new AppListAdapter(getContext(), R.layout.whitelist_row, entries);
+            return new AppListAdapter(entries);
         }
 
         @Override
diff --git a/app/src/main/res/layout/activity_whitelist.xml b/app/src/main/res/layout/activity_whitelist.xml
index b9aa0d2..ca85e3c 100644
--- a/app/src/main/res/layout/activity_whitelist.xml
+++ b/app/src/main/res/layout/activity_whitelist.xml
@@ -26,7 +26,7 @@
         android:layout_above="@+id/switch_show_system_apps"
         android:layout_height="match_parent">
 
-        <ListView
+        <android.support.v7.widget.RecyclerView
             android:id="@+id/list"
             android:layout_width="match_parent"
             android:layout_height="match_parent" />
diff --git a/app/src/main/res/layout/whitelist_row.xml b/app/src/main/res/layout/whitelist_row.xml
index a416b54..ccb0e71 100644
--- a/app/src/main/res/layout/whitelist_row.xml
+++ b/app/src/main/res/layout/whitelist_row.xml
@@ -6,6 +6,9 @@
     android:paddingBottom="8dp"
     android:paddingLeft="@dimen/activity_horizontal_margin"
     android:paddingRight="@dimen/activity_horizontal_margin"
+    android:background="?android:attr/selectableItemBackground"
+    android:clickable="true"
+    android:focusable="true"
     android:paddingTop="8dp">
 
     <ImageView
@@ -40,6 +43,7 @@
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:layout_alignParentEnd="true"
+        android:focusable="false"
         android:clickable="false" />
 
 </RelativeLayout>
\ No newline at end of file

```
