# MTG Familiar 373

```diff
 .../mtgfam/fragments/HtmlDocFragment.java     | 51 +++++++++++++++++--
 1 file changed, 48 insertions(+), 3 deletions(-)

diff --git a/mobile/src/main/java/com/gelakinetic/mtgfam/fragments/HtmlDocFragment.java b/mobile/src/main/java/com/gelakinetic/mtgfam/fragments/HtmlDocFragment.java
index 15e4ad21..2ec9020f 100644
--- a/mobile/src/main/java/com/gelakinetic/mtgfam/fragments/HtmlDocFragment.java
+++ b/mobile/src/main/java/com/gelakinetic/mtgfam/fragments/HtmlDocFragment.java
@@ -56,6 +56,7 @@
  */
 public class HtmlDocFragment extends FamiliarFragment {
 
+    private static final String SCROLL_PCT = "scroll_pct";
     private String mName;
     private String mLastSearchTerm;
     private WebView mWebView;
@@ -89,6 +90,15 @@ public void onPageFinished(WebView view, String url) {
                 if (getActivity() != null) {
                     getActivity().runOnUiThread(() -> progressBar.setVisibility(View.GONE));
                 }
+                if (savedInstanceState != null && savedInstanceState.containsKey(SCROLL_PCT)) {
+                    // Delay the scrollTo to make it work
+                    view.postDelayed(() -> {
+                        float webViewSize = mWebView.getContentHeight() - mWebView.getTop();
+                        float positionInWV = webViewSize * savedInstanceState.getFloat(SCROLL_PCT);
+                        int positionY = Math.round(mWebView.getTop() + positionInWV);
+                        mWebView.scrollTo(0, positionY);
+                    }, 300);
+                }
             }
 
             @Override
@@ -158,11 +168,46 @@ private void startCardViewFrag(String name) {
         }
     }
 
+    /**
+     * Calculate the percentage of scroll progress in the actual web page content
+     *
+     * @return The percentage of scroll progress
+     */
+    private float calculateProgression() {
+        float positionTopView = mWebView.getTop();
+        float contentHeight = mWebView.getContentHeight();
+        float currentScrollPosition = mWebView.getScrollY();
+        return (currentScrollPosition - positionTopView) / contentHeight;
+    }
+
+    /**
+     * Save the scroll location before rotating or whatever
+     *
+     * @param outState Bundle in which to place your saved state.
+     */
+    @Override
+    public void onSaveInstanceState(@NonNull Bundle outState) {
+        try {
+            outState.putFloat(SCROLL_PCT, calculateProgression());
+        } catch (NullPointerException e) {
+            outState.putFloat(SCROLL_PCT, 0);
+        }
+        super.onSaveInstanceState(outState);
+    }
+
+    /**
+     * @return true, because the Html Doc Fragment overrides the search key
+     */
     @Override
     boolean canInterceptSearchKey() {
         return true;
     }
 
+    /**
+     * Show the dialog to search when the search key is pressed
+     *
+     * @return true, because the dialog was shown
+     */
     @Override
     public boolean onInterceptSearchKey() {
         showDialog();
@@ -189,21 +234,21 @@ private void showDialog() throws IllegalStateException {
     }
 
     /**
-     * @return Get the name of the webpage being displayed
+     * @return Get the name of the web page being displayed
      */
     public String getName() {
         return mName;
     }
 
     /**
-     * @return Get the last string that was searched on this webpage
+     * @return Get the last string that was searched on this web page
      */
     public String getLastSearchTerm() {
         return mLastSearchTerm;
     }
 
     /**
-     * Search the displayed webpage for the given term. If a search isn't in progress, start it.
+     * Search the displayed web page for the given term. If a search isn't in progress, start it.
      * Otherwise, find the next term
      *
      * @param searchTerm The term to search for, saved for later searches


```
