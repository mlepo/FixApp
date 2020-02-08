# Sms Backup+ 878


```diff

 .../zegoggles/smssync/activity/Dialogs.java   | 29 +++++++++++++++++--
 1 file changed, 26 insertions(+), 3 deletions(-)

diff --git a/app/src/main/java/com/zegoggles/smssync/activity/Dialogs.java b/app/src/main/java/com/zegoggles/smssync/activity/Dialogs.java
index 0d15cafc..49e9bf64 100644
--- a/app/src/main/java/com/zegoggles/smssync/activity/Dialogs.java
+++ b/app/src/main/java/com/zegoggles/smssync/activity/Dialogs.java
@@ -23,6 +23,7 @@
 import android.content.DialogInterface.OnClickListener;
 import android.content.Intent;
 import android.net.Uri;
+import android.os.Build;
 import android.os.Bundle;
 import android.support.annotation.NonNull;
 import android.support.annotation.Nullable;
@@ -146,18 +147,33 @@ public Dialog onCreateDialog(Bundle savedInstanceState) {
     }
 
     public static class About extends BaseFragment {
+        private static final String SCROLL_POSITION = "scrollPosition";
+        private static final String ABOUT_HTML = "file:///android_asset/about.html";
+        private WebView webView;
+
         @Override @NonNull @SuppressLint("InflateParams")
-        public Dialog onCreateDialog(Bundle savedInstanceState) {
+        public Dialog onCreateDialog(@Nullable Bundle savedInstanceState) {
             final View contentView = getActivity().getLayoutInflater().inflate(R.layout.about_dialog, null, false);
-            final WebView webView = (WebView) contentView.findViewById(R.id.about_content);
+            webView = (WebView) contentView.findViewById(R.id.about_content);
+            final float scrollPosition = savedInstanceState == null ? 0f : savedInstanceState.getFloat(SCROLL_POSITION);
+
             webView.setWebViewClient(new WebViewClient() {
                 @Override @SuppressWarnings("deprecation")
                 public boolean shouldOverrideUrlLoading(WebView view, String url) {
                     startActivity(new Intent(Intent.ACTION_VIEW).setData(Uri.parse(url)));
                     return true;
                 }
+
+                @Override @SuppressWarnings("deprecation") public void onPageFinished(WebView view, String url) {
+                    super.onPageFinished(view, url);
+                    if (scrollPosition > 0 &&
+                        ABOUT_HTML.equals(url) &&
+                        Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
+                        view.setScrollY((int) (view.getContentHeight() * view.getScale() * scrollPosition));
+                    }
+                }
             });
-            webView.loadUrl("file:///android_asset/about.html");
+            webView.loadUrl(ABOUT_HTML);
             return new AlertDialog.Builder(getContext())
                 .setPositiveButton(ok, null)
                 .setIcon(R.drawable.ic_sms_backup)
@@ -165,6 +181,13 @@ public boolean shouldOverrideUrlLoading(WebView view, String url) {
                 .setView(contentView)
                 .create();
         }
+
+        @Override @SuppressWarnings("deprecation")
+        public void onSaveInstanceState(Bundle outState) {
+            super.onSaveInstanceState(outState);
+            final float position = webView.getScrollY() / (webView.getContentHeight() * webView.getScale());
+            outState.putFloat(SCROLL_POSITION, position);
+        }
     }
 
     public static class Reset extends BaseFragment {
     
  ```
