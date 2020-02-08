# Flym

```diff
 .../fred/feedex/fragment/EntryFragment.java   | 23 +++++++++++++++++++
 src/net/fred/feedex/view/EntryView.java       | 19 ++++++++++-----
 2 files changed, 36 insertions(+), 6 deletions(-)

diff --git a/src/net/fred/feedex/fragment/EntryFragment.java b/src/net/fred/feedex/fragment/EntryFragment.java
index 4f48330c9..6dbc6e8b9 100644
--- a/src/net/fred/feedex/fragment/EntryFragment.java
+++ b/src/net/fred/feedex/fragment/EntryFragment.java
@@ -68,6 +68,7 @@
     private static final String STATE_CURRENT_PAGER_POS = "STATE_CURRENT_PAGER_POS";
     private static final String STATE_ENTRIES_IDS = "STATE_ENTRIES_IDS";
     private static final String STATE_INITIAL_ENTRY_ID = "STATE_INITIAL_ENTRY_ID";
+    private static final String STATE_SCROLL_PERCENTAGE = "STATE_SCROLL_PERCENTAGE";
 
     private int mTitlePos = -1, mDatePos, mMobilizedHtmlPos, mAbstractPos, mLinkPos, mIsFavoritePos, mIsReadPos, mEnclosurePos, mAuthorPos, mFeedNamePos, mFeedUrlPos, mFeedIconPos;
 
@@ -85,6 +86,7 @@
 
     private class EntryPagerAdapter extends PagerAdapter {
 
+        private float mScrollPercentage = 0;
         private SparseArray<EntryView> mEntryViews = new SparseArray<EntryView>();
 
         public EntryPagerAdapter() {
@@ -117,6 +119,19 @@ public boolean isViewFromObject(View view, Object object) {
             return view == ((View) object);
         }
 
+        public float getScrollPercentage() {
+            EntryView view = mEntryViews.get(mCurrentPagerPos);
+            if (view != null) {
+                return view.getScrollPercentage();
+            }
+
+            return 0;
+        }
+
+        public void setScrollPercentage(float scrollPercentage) {
+            mScrollPercentage = scrollPercentage;
+        }
+
         public void displayEntry(int pagerPos, Cursor newCursor, boolean forceUpdate) {
             EntryView view = mEntryViews.get(pagerPos);
             if (view != null) {
@@ -142,6 +157,12 @@ public void displayEntry(int pagerPos, Cursor newCursor, boolean forceUpdate) {
                     String title = newCursor.getString(mTitlePos);
                     String enclosure = newCursor.getString(mEnclosurePos);
 
+                    // Set the saved scroll position (not saved by the view itself due to the ViewPager)
+                    if (mScrollPercentage != 0 && pagerPos == mCurrentPagerPos) {
+                        view.setScrollPercentage(mScrollPercentage);
+                        mScrollPercentage = 0;
+                    }
+
                     view.setHtml(mEntriesIds[pagerPos], title, link, contentText, enclosure, author, timestamp, mPreferFullText);
                     view.setTag(newCursor);
 
@@ -215,6 +236,7 @@ public void onClick(View view) {
             mCurrentPagerPos = savedInstanceState.getInt(STATE_CURRENT_PAGER_POS);
             mEntryPager.getAdapter().notifyDataSetChanged();
             mEntryPager.setCurrentItem(mCurrentPagerPos);
+            mEntryPagerAdapter.setScrollPercentage(savedInstanceState.getFloat(STATE_SCROLL_PERCENTAGE));
         }
 
         mEntryPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
@@ -245,6 +267,7 @@ public void onSaveInstanceState(Bundle outState) {
         outState.putLongArray(STATE_ENTRIES_IDS, mEntriesIds);
         outState.putLong(STATE_INITIAL_ENTRY_ID, mInitialEntryId);
         outState.putInt(STATE_CURRENT_PAGER_POS, mCurrentPagerPos);
+        outState.putFloat(STATE_SCROLL_PERCENTAGE, mEntryPagerAdapter.getScrollPercentage());
 
         super.onSaveInstanceState(outState);
     }
diff --git a/src/net/fred/feedex/view/EntryView.java b/src/net/fred/feedex/view/EntryView.java
index 7c68f1f43..f3660255e 100644
--- a/src/net/fred/feedex/view/EntryView.java
+++ b/src/net/fred/feedex/view/EntryView.java
@@ -141,12 +141,7 @@ public EntryView(Context context, AttributeSet attrs, int defStyle) {
     protected Parcelable onSaveInstanceState() {
         Bundle bundle = new Bundle();
         bundle.putParcelable("superInstanceState", super.onSaveInstanceState());
-
-        float positionTopView = getTop();
-        float contentHeight = getContentHeight();
-        float currentScrollPosition = getScrollY();
-
-        bundle.putFloat(STATE_SCROLL_PERCENTAGE, (currentScrollPosition - positionTopView) / contentHeight);
+        bundle.putFloat(STATE_SCROLL_PERCENTAGE, getScrollPercentage());
 
         return bundle;
     }
@@ -158,6 +153,18 @@ protected void onRestoreInstanceState(Parcelable state) {
         super.onRestoreInstanceState(bundle.getParcelable("superInstanceState"));
     }
 
+    public float getScrollPercentage() {
+        float positionTopView = getTop();
+        float contentHeight = getContentHeight();
+        float currentScrollPosition = getScrollY();
+
+        return (currentScrollPosition - positionTopView) / contentHeight;
+    }
+
+    public void setScrollPercentage(float scrollPercentage) {
+        mScrollPercentage = scrollPercentage;
+    }
+
     public void setListener(OnActionListener listener) {
         mListener = listener;
     }


```
