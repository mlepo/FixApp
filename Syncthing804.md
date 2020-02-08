# Syncthing 804

```diff
 .../activities/DeviceActivity.java            | 117 ++++++++++++++----
 .../activities/FolderActivity.java            |  81 +++++++++---
 .../activities/MainActivity.java              |  35 +++++-
 .../fragments/DrawerFragment.java             |   9 +-
 .../dialog/KeepVersionsDialogFragment.java    |  13 ++
 5 files changed, 205 insertions(+), 50 deletions(-)

diff --git a/src/main/java/com/nutomic/syncthingandroid/activities/DeviceActivity.java b/src/main/java/com/nutomic/syncthingandroid/activities/DeviceActivity.java
index 41acda8ff..6087168e6 100644
--- a/src/main/java/com/nutomic/syncthingandroid/activities/DeviceActivity.java
+++ b/src/main/java/com/nutomic/syncthingandroid/activities/DeviceActivity.java
@@ -1,5 +1,6 @@
 package com.nutomic.syncthingandroid.activities;
 
+import android.app.Dialog;
 import android.content.Context;
 import android.content.DialogInterface;
 import android.content.Intent;
@@ -15,6 +16,7 @@
 import android.view.Menu;
 import android.view.MenuItem;
 import android.view.View;
+import android.widget.AdapterView;
 import android.widget.CompoundButton;
 import android.widget.EditText;
 import android.widget.TextView;
@@ -53,6 +55,9 @@
             "com.nutomic.syncthingandroid.activities.DeviceActivity.IS_CREATE";
 
     private static final String TAG = "DeviceSettingsFragment";
+    private static final String IS_SHOWING_DISCARD_DIALOG = "DISCARD_FOLDER_DIALOG_STATE";
+    private static final String IS_SHOWING_COMPRESSION_DIALOG = "COMPRESSION_FOLDER_DIALOG_STATE";
+    private static final String IS_SHOWING_DELETE_DIALOG = "DELETE_FOLDER_DIALOG_STATE";
 
     public static final List<String> DYNAMIC_ADDRESS = Collections.singletonList("dynamic");
 
@@ -82,11 +87,14 @@
 
     private boolean mDeviceNeedsToUpdate;
 
+    private Dialog mDeleteDialog;
+    private Dialog mDiscardDialog;
+    private Dialog mCompressionDialog;
+
     private final DialogInterface.OnClickListener mCompressionEntrySelectedListener = new DialogInterface.OnClickListener() {
         @Override
         public void onClick(DialogInterface dialog, int which) {
             dialog.dismiss();
-
             Compression compression = Compression.fromIndex(which);
             // Don't pop the restart dialog unless the value is actually different.
             if (compression != Compression.fromValue(DeviceActivity.this, mDevice.compression)) {
@@ -165,11 +173,14 @@ public void onCreate(Bundle savedInstanceState) {
         mQrButton.setOnClickListener(this);
         mCompressionContainer.setOnClickListener(this);
 
-        if (mIsCreateMode) {
-            if (savedInstanceState != null) {
+        if (savedInstanceState != null){
+            if (mDevice == null) {
                 mDevice = new Gson().fromJson(savedInstanceState.getString("device"), Device.class);
             }
-            if (mDevice == null) {
+            restoreDialogStates(savedInstanceState);
+        }
+        if (mIsCreateMode) {
+           if (mDevice == null) {
                 initDevice();
             }
         }
@@ -178,6 +189,22 @@ public void onCreate(Bundle savedInstanceState) {
         }
     }
 
+    private void restoreDialogStates(Bundle savedInstanceState) {
+        if (savedInstanceState.getBoolean(IS_SHOWING_COMPRESSION_DIALOG)){
+            showCompressionDialog();
+        }
+
+        if (savedInstanceState.getBoolean(IS_SHOWING_DELETE_DIALOG)){
+            showDeleteDialog();
+        }
+
+        if (mIsCreateMode){
+            if (savedInstanceState.getBoolean(IS_SHOWING_DISCARD_DIALOG)){
+                showDiscardDialog();
+            }
+        }
+    }
+
     @Override
     public void onDestroy() {
         super.onDestroy();
@@ -207,6 +234,22 @@ public void onPause() {
     public void onSaveInstanceState(Bundle outState) {
         super.onSaveInstanceState(outState);
         outState.putString("device", new Gson().toJson(mDevice));
+        if (mIsCreateMode){
+            outState.putBoolean(IS_SHOWING_DISCARD_DIALOG, mDiscardDialog != null && mDiscardDialog.isShowing());
+            if(mDiscardDialog != null){
+                mDiscardDialog.cancel();
+            }
+        }
+
+        outState.putBoolean(IS_SHOWING_COMPRESSION_DIALOG, mCompressionDialog != null && mCompressionDialog.isShowing());
+        if(mCompressionDialog != null){
+            mCompressionDialog.cancel();
+        }
+
+        outState.putBoolean(IS_SHOWING_DELETE_DIALOG, mDeleteDialog != null && mDeleteDialog.isShowing());
+        if (mDeleteDialog != null) {
+            mDeleteDialog.cancel();
+        }
     }
 
     public void onServiceConnected() {
@@ -302,14 +345,7 @@ public boolean onOptionsItemSelected(MenuItem item) {
                 shareDeviceId(this, mDevice.deviceID);
                 return true;
             case R.id.remove:
-                new AlertDialog.Builder(this)
-                        .setMessage(R.string.remove_device_confirm)
-                        .setPositiveButton(android.R.string.yes, (dialogInterface, i) -> {
-                            getApi().removeDevice(mDevice.deviceID);
-                            finish();
-                        })
-                        .setNegativeButton(android.R.string.no, null)
-                        .show();
+               showDeleteDialog();
                 return true;
             case android.R.id.home:
                 onBackPressed();
@@ -319,6 +355,23 @@ public boolean onOptionsItemSelected(MenuItem item) {
         }
     }
 
+
+    private void showDeleteDialog(){
+        mDeleteDialog = createDeleteDialog();
+        mDeleteDialog.show();
+    }
+
+    private Dialog createDeleteDialog(){
+        return new android.app.AlertDialog.Builder(this)
+                .setMessage(R.string.remove_device_confirm)
+                .setPositiveButton(android.R.string.yes, (dialogInterface, i) -> {
+                    getApi().removeDevice(mDevice.deviceID);
+                    finish();
+                })
+                .setNegativeButton(android.R.string.no, null)
+                .create();
+    }
+
     /**
      * Receives value of scanned QR code and sets it as device ID.
      */
@@ -376,12 +429,7 @@ private String displayableAddresses() {
     @Override
     public void onClick(View v) {
         if (v.equals(mCompressionContainer)) {
-            new AlertDialog.Builder(this)
-                    .setTitle(R.string.compression)
-                    .setSingleChoiceItems(R.array.compress_entries,
-                            Compression.fromValue(this, mDevice.compression).getIndex(),
-                            mCompressionEntrySelectedListener)
-                    .show();
+            showCompressionDialog();
         } else if (v.equals(mQrButton)){
             IntentIntegrator integrator = new IntentIntegrator(DeviceActivity.this);
             integrator.initiateScan();
@@ -390,6 +438,20 @@ public void onClick(View v) {
         }
     }
 
+    private void showCompressionDialog(){
+        mCompressionDialog = createCompressionDialog();
+        mCompressionDialog.show();
+    }
+
+    private Dialog createCompressionDialog(){
+        return new AlertDialog.Builder(this)
+                .setTitle(R.string.compression)
+                .setSingleChoiceItems(R.array.compress_entries,
+                        Compression.fromValue(this, mDevice.compression).getIndex(),
+                        mCompressionEntrySelectedListener)
+                .create();
+    }
+
     /**
      * Shares the given device ID via Intent. Must be called from an Activity.
      */
@@ -405,14 +467,23 @@ private void shareDeviceId(Context context, String id) {
     @Override
     public void onBackPressed() {
         if (mIsCreateMode) {
-            new AlertDialog.Builder(this)
-                    .setMessage(R.string.dialog_discard_changes)
-                    .setPositiveButton(android.R.string.ok, (dialog, which) -> finish())
-                    .setNegativeButton(android.R.string.cancel, null)
-                    .show();
+            showDiscardDialog();
         }
         else {
             super.onBackPressed();
         }
     }
+
+    private void showDiscardDialog(){
+        mDiscardDialog = createDiscardDialog();
+        mDiscardDialog.show();
+    }
+
+    private Dialog createDiscardDialog() {
+        return new android.app.AlertDialog.Builder(this)
+                .setMessage(R.string.dialog_discard_changes)
+                .setPositiveButton(android.R.string.ok, (dialog, which) -> finish())
+                .setNegativeButton(android.R.string.cancel, null)
+                .create();
+    }
 }
diff --git a/src/main/java/com/nutomic/syncthingandroid/activities/FolderActivity.java b/src/main/java/com/nutomic/syncthingandroid/activities/FolderActivity.java
index b3ddbc3e8..b710259c6 100644
--- a/src/main/java/com/nutomic/syncthingandroid/activities/FolderActivity.java
+++ b/src/main/java/com/nutomic/syncthingandroid/activities/FolderActivity.java
@@ -2,6 +2,8 @@
 
 import android.app.Activity;
 import android.app.AlertDialog;
+import android.app.Dialog;
+import android.content.DialogInterface;
 import android.content.Intent;
 import android.os.Build;
 import android.os.Bundle;
@@ -56,11 +58,14 @@
     public static final String EXTRA_DEVICE_ID =
             "com.nutomic.syncthingandroid.activities.FolderActivity.DEVICE_ID";
 
+
     private static final int DIRECTORY_REQUEST_CODE = 234;
 
     private static final String TAG = "EditFolderFragment";
 
     public static final String KEEP_VERSIONS_DIALOG_TAG = "KeepVersionsDialogFragment";
+    private static final String IS_SHOWING_DELETE_DIALOG = "DELETE_FOLDER_DIALOG_STATE";
+    private static final String IS_SHOW_DISCARD_DIALOG = "DISCARD_FOLDER_DIALOG_STATE";
 
     private Folder mFolder;
 
@@ -74,6 +79,9 @@
     private boolean mIsCreateMode;
     private boolean mFolderNeedsToUpdate;
 
+    private Dialog mDeleteDialog;
+    private Dialog mDiscardDialog;
+
     private final KeepVersionsDialogFragment mKeepVersionsDialogFragment = new KeepVersionsDialogFragment();
 
     private final TextWatcher mTextWatcher = new TextWatcherAdapter() {
@@ -158,6 +166,9 @@ public void onCreate(Bundle savedInstanceState) {
         if (mIsCreateMode) {
             if (savedInstanceState != null) {
                 mFolder = new Gson().fromJson(savedInstanceState.getString("folder"), Folder.class);
+                if (savedInstanceState.getBoolean(IS_SHOW_DISCARD_DIALOG)){
+                    showDiscardDialog();
+                }
             }
             if (mFolder == null) {
                 initFolder();
@@ -168,6 +179,12 @@ public void onCreate(Bundle savedInstanceState) {
         else {
             prepareEditMode();
         }
+
+        if (savedInstanceState != null){
+            if (savedInstanceState.getBoolean(IS_SHOWING_DELETE_DIALOG)){
+                showDeleteDialog();
+            }
+        }
     }
 
     @Override
@@ -192,15 +209,25 @@ public void onPause() {
         }
     }
 
-    /**
-     * Save current settings in case we are in create mode and they aren't yet stored in the config.
-     */
     @Override
-    public void onSaveInstanceState(Bundle outState) {
+    protected void onSaveInstanceState(Bundle outState) {
         super.onSaveInstanceState(outState);
-        outState.putString("folder", new Gson().toJson(mFolder));
+        outState.putBoolean(IS_SHOWING_DELETE_DIALOG, mDeleteDialog != null && mDeleteDialog.isShowing());
+        if (mDeleteDialog != null) {
+            mDeleteDialog.cancel();
+        }
+
+        if (mIsCreateMode){
+            outState.putBoolean(IS_SHOW_DISCARD_DIALOG, mDiscardDialog != null && mDiscardDialog.isShowing());
+            if(mDiscardDialog != null){
+                mDiscardDialog.cancel();
+            }
+        }
     }
 
+    /**
+     * Save current settings in case we are in create mode and they aren't yet stored in the config.
+     */
     @Override
     public void onServiceConnected() {
         getService().registerOnApiChangeListener(this);
@@ -309,14 +336,7 @@ public boolean onOptionsItemSelected(MenuItem item) {
                 finish();
                 return true;
             case R.id.remove:
-                new AlertDialog.Builder(this)
-                        .setMessage(R.string.remove_folder_confirm)
-                        .setPositiveButton(android.R.string.yes, (dialogInterface, i) -> {
-                            getApi().removeFolder(mFolder.id);
-                            finish();
-                        })
-                        .setNegativeButton(android.R.string.no, null)
-                        .show();
+                showDeleteDialog();
                 return true;
             case android.R.id.home:
                 onBackPressed();
@@ -326,6 +346,22 @@ public boolean onOptionsItemSelected(MenuItem item) {
         }
     }
 
+    private void showDeleteDialog(){
+        mDeleteDialog = createDeleteDialog();
+        mDeleteDialog.show();
+    }
+
+    private Dialog createDeleteDialog(){
+        return new AlertDialog.Builder(this)
+                .setMessage(R.string.remove_folder_confirm)
+                .setPositiveButton(android.R.string.yes, (dialogInterface, i) -> {
+                    getApi().removeFolder(mFolder.id);
+                    finish();
+                })
+                .setNegativeButton(android.R.string.no, null)
+                .create();
+    }
+
     @Override
     public void onActivityResult(int requestCode, int resultCode, Intent data) {
         if (resultCode == Activity.RESULT_OK && requestCode == DIRECTORY_REQUEST_CODE) {
@@ -389,14 +425,23 @@ private void updateFolder() {
     @Override
     public void onBackPressed() {
         if (mIsCreateMode) {
-            new AlertDialog.Builder(this)
-                    .setMessage(R.string.dialog_discard_changes)
-                    .setPositiveButton(android.R.string.ok, (dialog, which) -> finish())
-                    .setNegativeButton(android.R.string.cancel, null)
-                    .show();
+            showDiscardDialog();
         }
         else {
             super.onBackPressed();
         }
     }
+
+    private void showDiscardDialog(){
+        mDiscardDialog = createDiscardDialog();
+        mDiscardDialog.show();
+    }
+
+    private Dialog createDiscardDialog() {
+        return new AlertDialog.Builder(this)
+                .setMessage(R.string.dialog_discard_changes)
+                .setPositiveButton(android.R.string.ok, (dialog, which) -> finish())
+                .setNegativeButton(android.R.string.cancel, null)
+                .create();
+    }
 }
diff --git a/src/main/java/com/nutomic/syncthingandroid/activities/MainActivity.java b/src/main/java/com/nutomic/syncthingandroid/activities/MainActivity.java
index 5bd911ec9..b7fd07140 100644
--- a/src/main/java/com/nutomic/syncthingandroid/activities/MainActivity.java
+++ b/src/main/java/com/nutomic/syncthingandroid/activities/MainActivity.java
@@ -4,6 +4,7 @@
 import android.annotation.TargetApi;
 import android.app.Activity;
 import android.app.AlertDialog;
+import android.app.Dialog;
 import android.content.ComponentName;
 import android.content.Context;
 import android.content.DialogInterface;
@@ -58,6 +59,8 @@
         implements SyncthingService.OnApiChangeListener {
 
     private static final String TAG = "MainActivity";
+    private static final String IS_SHOWING_RESTART_DIALOG = "RESTART_DIALOG_STATE";
+    private static final String BATTERY_DIALOG_DISMISSED = "BATTERY_DIALOG_STATE";
 
     /**
      * Time after first start when usage reporting dialog should be shown.
@@ -68,6 +71,9 @@
 
     private AlertDialog mDisabledDialog;
     private AlertDialog mBatteryOptimizationsDialog;
+    private Dialog mRestartDialog;
+
+    private boolean mBatteryOptimizationDialogDismissed;
 
     private ViewPager mViewPager;
 
@@ -115,7 +121,8 @@ private void showBatteryOptimizationDialogIfNecessary() {
         boolean dontShowAgain = sp.getBoolean("battery_optimization_dont_show_again", false);
         if (dontShowAgain || mBatteryOptimizationsDialog != null ||
                 Build.VERSION.SDK_INT < Build.VERSION_CODES.M ||
-                pm.isIgnoringBatteryOptimizations(getPackageName())) {
+                pm.isIgnoringBatteryOptimizations(getPackageName()) ||
+                mBatteryOptimizationDialogDismissed) {
             return;
         }
 
@@ -127,10 +134,11 @@ private void showBatteryOptimizationDialogIfNecessary() {
                     intent.setData(Uri.parse("package:" + getPackageName()));
                     startActivity(intent);
                 })
-                .setNeutralButton(R.string.dialog_disable_battery_optimization_later, null)
+                .setNeutralButton(R.string.dialog_disable_battery_optimization_later, (d, i) -> mBatteryOptimizationDialogDismissed = true)
                 .setNegativeButton(R.string.dialog_disable_battery_optimization_dont_show_again, (d, i) -> {
                     sp.edit().putBoolean("battery_optimization_dont_show_again", true).apply();
                 })
+                .setOnCancelListener(d -> mBatteryOptimizationDialogDismissed = true)
                 .show();
     }
 
@@ -212,6 +220,10 @@ public void onCreate(Bundle savedInstanceState) {
             mDrawerFragment = (DrawerFragment) fm.getFragment(
                     savedInstanceState, DrawerFragment.class.getName());
             mViewPager.setCurrentItem(savedInstanceState.getInt("currentTab"));
+            if (savedInstanceState.getBoolean(IS_SHOWING_RESTART_DIALOG)){
+                showRestartDialog();
+            }
+            mBatteryOptimizationDialogDismissed = savedInstanceState.getBoolean(BATTERY_DIALOG_DISMISSED);
         } else {
             mFolderListFragment = new FolderListFragment();
             mDeviceListFragment = new DeviceListFragment();
@@ -262,6 +274,11 @@ protected void onSaveInstanceState(Bundle outState) {
             fm.putFragment(outState, DeviceListFragment.class.getName(), mDeviceListFragment);
             fm.putFragment(outState, DrawerFragment.class.getName(), mDrawerFragment);
             outState.putInt("currentTab", mViewPager.getCurrentItem());
+            outState.putBoolean(BATTERY_DIALOG_DISMISSED, mBatteryOptimizationsDialog == null || !mBatteryOptimizationsDialog.isShowing());
+            outState.putBoolean(IS_SHOWING_RESTART_DIALOG, mRestartDialog != null && mRestartDialog.isShowing());
+            if (mRestartDialog != null){
+                mRestartDialog.cancel();
+            }
         }
     }
 
@@ -301,6 +318,20 @@ private void showDisabledDialog() {
         mDisabledDialog.setCanceledOnTouchOutside(false);
     }
 
+    public void showRestartDialog(){
+        mRestartDialog = createRestartDialog();
+        mRestartDialog.show();
+    }
+
+    private Dialog createRestartDialog(){
+        return  new AlertDialog.Builder(this)
+                .setMessage(R.string.dialog_confirm_restart)
+                .setPositiveButton(android.R.string.yes, (dialogInterface, i1) -> this.startService(new Intent(this, SyncthingService.class)
+                        .setAction(SyncthingService.ACTION_RESTART)))
+                .setNegativeButton(android.R.string.no, null)
+                .create();
+    }
+
     @Override
      public boolean onOptionsItemSelected(MenuItem item) {
         return mDrawerToggle.onOptionsItemSelected(item) || super.onOptionsItemSelected(item);
diff --git a/src/main/java/com/nutomic/syncthingandroid/fragments/DrawerFragment.java b/src/main/java/com/nutomic/syncthingandroid/fragments/DrawerFragment.java
index 9c3f5dcfe..9029ed529 100644
--- a/src/main/java/com/nutomic/syncthingandroid/fragments/DrawerFragment.java
+++ b/src/main/java/com/nutomic/syncthingandroid/fragments/DrawerFragment.java
@@ -1,6 +1,7 @@
 package com.nutomic.syncthingandroid.fragments;
 
 import android.app.AlertDialog;
+import android.app.Dialog;
 import android.content.Intent;
 import android.os.Bundle;
 import android.support.v4.app.Fragment;
@@ -40,7 +41,6 @@
     private TextView mUpload;
     private TextView mAnnounceServer;
     private TextView mVersion;
-
     private TextView mExitButton;
 
     private Timer mTimer;
@@ -212,13 +212,8 @@ public void onClick(View v) {
                 mActivity.closeDrawer();
                 break;
             case R.id.drawerActionRestart:
+                mActivity.showRestartDialog();
                 mActivity.closeDrawer();
-                new AlertDialog.Builder(getContext())
-                        .setMessage(R.string.dialog_confirm_restart)
-                        .setPositiveButton(android.R.string.yes, (dialogInterface, i1) -> getContext().startService(new Intent(getContext(), SyncthingService.class)
-                                .setAction(SyncthingService.ACTION_RESTART)))
-                        .setNegativeButton(android.R.string.no, null)
-                        .show();
                 break;
             case R.id.drawerActionExit:
                 mActivity.stopService(new Intent(mActivity, SyncthingService.class));
diff --git a/src/main/java/com/nutomic/syncthingandroid/fragments/dialog/KeepVersionsDialogFragment.java b/src/main/java/com/nutomic/syncthingandroid/fragments/dialog/KeepVersionsDialogFragment.java
index dba37a817..5e1971c0c 100644
--- a/src/main/java/com/nutomic/syncthingandroid/fragments/dialog/KeepVersionsDialogFragment.java
+++ b/src/main/java/com/nutomic/syncthingandroid/fragments/dialog/KeepVersionsDialogFragment.java
@@ -6,6 +6,7 @@
 import android.os.Bundle;
 import android.support.annotation.NonNull;
 import android.support.v7.app.AlertDialog;
+import android.util.Log;
 import android.widget.FrameLayout.LayoutParams;
 import android.widget.NumberPicker;
 
@@ -16,6 +17,8 @@
 
 public class KeepVersionsDialogFragment extends DialogFragment {
 
+    private static String KEEP_VERSION_VALUE = "KEEP_VERSION_KEY";
+
     private OnValueChangeListener mOnValueChangeListener;
 
     private NumberPicker mNumberPickerView;
@@ -38,6 +41,10 @@ public void onClick(DialogInterface dialog, int which) {
     @NonNull
     @Override
     public Dialog onCreateDialog(Bundle savedInstanceState) {
+        if (savedInstanceState != null) {
+            setValue(savedInstanceState.getInt(KEEP_VERSION_VALUE));
+        }
+
         mNumberPickerView = createNumberPicker();
         return new AlertDialog.Builder(getActivity())
                 .setTitle(R.string.keep_versions)
@@ -47,6 +54,12 @@ public Dialog onCreateDialog(Bundle savedInstanceState) {
                 .create();
     }
 
+    @Override
+    public void onSaveInstanceState(Bundle outState) {
+        super.onSaveInstanceState(outState);
+        outState.putInt(KEEP_VERSION_VALUE, mNumberPickerView.getValue());
+    }
+
     public void setOnValueChangeListener(OnValueChangeListener onValueChangeListener) {
         mOnValueChangeListener = onValueChangeListener;
     }


```
