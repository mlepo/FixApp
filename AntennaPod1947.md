# AntennaPod 1947

```diff

 .../de/danoeh/antennapod/fragment/ItemDescriptionFragment.java  | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

diff --git a/app/src/main/java/de/danoeh/antennapod/fragment/ItemDescriptionFragment.java b/app/src/main/java/de/danoeh/antennapod/fragment/ItemDescriptionFragment.java
index 55d28cadb..94d61e075 100644
--- a/app/src/main/java/de/danoeh/antennapod/fragment/ItemDescriptionFragment.java
+++ b/app/src/main/java/de/danoeh/antennapod/fragment/ItemDescriptionFragment.java
@@ -348,7 +348,7 @@ private void savePreference() {
     }
 
     private boolean restoreFromPreference() {
-        if (!saveState) {
+        if (saveState) {
             Log.d(TAG, "Restoring from preferences");
             Activity activity = getActivity();
             if (activity != null) {
         
```
