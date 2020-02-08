# Simple Draw 95

```diff
 .../draw/activities/MainActivity.kt             | 17 +++++++++++++++++
 .../draw/models/MyParcelable.kt                 |  2 +-
 .../simplemobiletools/draw/views/MyCanvas.kt    |  6 +++---
 3 files changed, 21 insertions(+), 4 deletions(-)

diff --git a/app/src/main/kotlin/com/simplemobiletools/draw/activities/MainActivity.kt b/app/src/main/kotlin/com/simplemobiletools/draw/activities/MainActivity.kt
index 95fe147..7a47f50 100644
--- a/app/src/main/kotlin/com/simplemobiletools/draw/activities/MainActivity.kt
+++ b/app/src/main/kotlin/com/simplemobiletools/draw/activities/MainActivity.kt
@@ -37,6 +37,7 @@ import java.io.OutputStream
 class MainActivity : SimpleActivity(), CanvasListener {
     private val FOLDER_NAME = "images"
     private val FILE_NAME = "simple-draw.png"
+    private val BITMAP_PATH = "bitmap_path"
 
     private var defaultPath = ""
     private var defaultFilename = ""
@@ -47,6 +48,7 @@ class MainActivity : SimpleActivity(), CanvasListener {
     private var strokeWidth = 0f
     private var isEraserOn = false
     private var isImageCaptureIntent = false
+    private var lastBitmapPath = ""
 
     override fun onCreate(savedInstanceState: Bundle?) {
         super.onCreate(savedInstanceState)
@@ -208,6 +210,7 @@ class MainActivity : SimpleActivity(), CanvasListener {
             true
         }
         File(path).isImageSlow() -> {
+            lastBitmapPath = path
             my_canvas.drawBitmap(this, path)
             defaultExtension = JPG
             true
@@ -371,6 +374,7 @@ class MainActivity : SimpleActivity(), CanvasListener {
         my_canvas.clearCanvas()
         defaultExtension = PNG
         defaultPath = ""
+        lastBitmapPath = ""
     }
 
     private fun pickColor() {
@@ -404,6 +408,19 @@ class MainActivity : SimpleActivity(), CanvasListener {
         redo.beVisibleIf(visible)
     }
 
+    override fun onSaveInstanceState(outState: Bundle) {
+        super.onSaveInstanceState(outState)
+        outState.putString(BITMAP_PATH, lastBitmapPath)
+    }
+
+    override fun onRestoreInstanceState(savedInstanceState: Bundle) {
+        super.onRestoreInstanceState(savedInstanceState)
+        lastBitmapPath = savedInstanceState.getString(BITMAP_PATH)
+        if (lastBitmapPath.isNotEmpty()) {
+            openPath(lastBitmapPath)
+        }
+    }
+
     private var onStrokeWidthBarChangeListener: SeekBar.OnSeekBarChangeListener = object : SeekBar.OnSeekBarChangeListener {
         override fun onProgressChanged(seekBar: SeekBar, progress: Int, fromUser: Boolean) {
             my_canvas.setStrokeWidth(progress.toFloat())
diff --git a/app/src/main/kotlin/com/simplemobiletools/draw/models/MyParcelable.kt b/app/src/main/kotlin/com/simplemobiletools/draw/models/MyParcelable.kt
index f20a88c..62a5f80 100644
--- a/app/src/main/kotlin/com/simplemobiletools/draw/models/MyParcelable.kt
+++ b/app/src/main/kotlin/com/simplemobiletools/draw/models/MyParcelable.kt
@@ -15,7 +15,7 @@ internal class MyParcelable : View.BaseSavedState {
         for (i in 0 until size) {
             val key = parcel.readSerializable() as MyPath
             val paintOptions = PaintOptions(parcel.readInt(), parcel.readFloat(), parcel.readInt() == 1)
-            paths.put(key, paintOptions)
+            paths[key] = paintOptions
         }
     }
 
diff --git a/app/src/main/kotlin/com/simplemobiletools/draw/views/MyCanvas.kt b/app/src/main/kotlin/com/simplemobiletools/draw/views/MyCanvas.kt
index cfa866a..716f96f 100644
--- a/app/src/main/kotlin/com/simplemobiletools/draw/views/MyCanvas.kt
+++ b/app/src/main/kotlin/com/simplemobiletools/draw/views/MyCanvas.kt
@@ -27,9 +27,9 @@ class MyCanvas(context: Context, attrs: AttributeSet) : View(context, attrs) {
     var mBackgroundBitmap: Bitmap? = null
     var mListener: CanvasListener? = null
 
-    var mLastPaths = LinkedHashMap<MyPath, PaintOptions>()
-    var mLastBackgroundBitmap: Bitmap? = null
-    var mUndonePaths = LinkedHashMap<MyPath, PaintOptions>()
+    private var mLastPaths = LinkedHashMap<MyPath, PaintOptions>()
+    private var mLastBackgroundBitmap: Bitmap? = null
+    private var mUndonePaths = LinkedHashMap<MyPath, PaintOptions>()
 
     private var mPaint = Paint()
     private var mPath = MyPath()


```
