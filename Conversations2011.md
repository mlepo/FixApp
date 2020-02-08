# Conversations 2011

```diff
 .../ui/ConversationActivity.java              | 33 ++++++++++++-----
 .../ui/ConversationFragment.java              | 35 +++++++++++++++++--
 2 files changed, 57 insertions(+), 11 deletions(-)

diff --git a/src/main/java/eu/siacs/conversations/ui/ConversationActivity.java b/src/main/java/eu/siacs/conversations/ui/ConversationActivity.java
index e891f00b87..2c8b27b0ec 100644
--- a/src/main/java/eu/siacs/conversations/ui/ConversationActivity.java
+++ b/src/main/java/eu/siacs/conversations/ui/ConversationActivity.java
@@ -67,8 +67,6 @@
 import eu.siacs.conversations.xmpp.jid.InvalidJidException;
 import eu.siacs.conversations.xmpp.jid.Jid;
 
-import static eu.siacs.conversations.crypto.axolotl.AxolotlService.AxolotlCapability.MISSING_PRESENCE;
-
 public class ConversationActivity extends XmppActivity
 	implements OnAccountUpdate, OnConversationUpdate, OnRosterUpdate, OnUpdateBlocklist, XmppConnectionService.OnShowErrorToast {
 
@@ -94,9 +92,12 @@
 	private static final String STATE_OPEN_CONVERSATION = "state_open_conversation";
 	private static final String STATE_PANEL_OPEN = "state_panel_open";
 	private static final String STATE_PENDING_URI = "state_pending_uri";
+	private static final String STATE_FIRST_VISIBLE = "first_visible";
+	private static final String STATE_OFFSET_FROM_TOP = "offset_from_top";
 
-	private String mOpenConverstaion = null;
+	private String mOpenConversation = null;
 	private boolean mPanelOpen = true;
+	private Pair<Integer,Integer> mScrollPosition = null;
 	final private List<Uri> mPendingImageUris = new ArrayList<>();
 	final private List<Uri> mPendingFileUris = new ArrayList<>();
 	private Uri mPendingGeoUri = null;
@@ -172,8 +173,16 @@ public boolean isConversationsOverviewVisable() {
 	protected void onCreate(final Bundle savedInstanceState) {
 		super.onCreate(savedInstanceState);
 		if (savedInstanceState != null) {
-			mOpenConverstaion = savedInstanceState.getString(STATE_OPEN_CONVERSATION, null);
+			mOpenConversation = savedInstanceState.getString(STATE_OPEN_CONVERSATION, null);
 			mPanelOpen = savedInstanceState.getBoolean(STATE_PANEL_OPEN, true);
+			int pos = savedInstanceState.getInt(STATE_FIRST_VISIBLE, -1);
+			int offset = savedInstanceState.getInt(STATE_OFFSET_FROM_TOP, 1);
+			if (pos >= 0 && offset <= 0) {
+				Log.d(Config.LOGTAG,"retrieved scroll position from instanceState "+pos+":"+offset);
+				mScrollPosition = new Pair<>(pos,offset);
+			} else {
+				mScrollPosition = null;
+			}
 			String pending = savedInstanceState.getString(STATE_PENDING_URI, null);
 			if (pending != null) {
 				mPendingImageUris.clear();
@@ -1081,7 +1090,7 @@ private boolean openConversationByIndex(int index) {
 	@Override
 	protected void onNewIntent(final Intent intent) {
 		if (intent != null && ACTION_VIEW_CONVERSATION.equals(intent.getAction())) {
-			mOpenConverstaion = null;
+			mOpenConversation = null;
 			if (xmppConnectionServiceBound) {
 				handleViewConversationIntent(intent);
 				intent.setAction(Intent.ACTION_MAIN);
@@ -1131,6 +1140,11 @@ public void onSaveInstanceState(final Bundle savedInstanceState) {
 		Conversation conversation = getSelectedConversation();
 		if (conversation != null) {
 			savedInstanceState.putString(STATE_OPEN_CONVERSATION, conversation.getUuid());
+			Pair<Integer,Integer> scrollPosition = mConversationFragment.getScrollPosition();
+			if (scrollPosition != null) {
+				savedInstanceState.putInt(STATE_FIRST_VISIBLE, scrollPosition.first);
+				savedInstanceState.putInt(STATE_OFFSET_FROM_TOP, scrollPosition.second);
+			}
 		} else {
 			savedInstanceState.remove(STATE_OPEN_CONVERSATION);
 		}
@@ -1190,7 +1204,7 @@ void onBackendConnected() {
 				}
 				finish();
 			}
-		} else if (selectConversationByUuid(mOpenConverstaion)) {
+		} else if (selectConversationByUuid(mOpenConversation)) {
 			if (mPanelOpen) {
 				showConversationsOverview();
 			} else {
@@ -1199,8 +1213,11 @@ void onBackendConnected() {
 					updateActionBarTitle(true);
 				}
 			}
-			this.mConversationFragment.reInit(getSelectedConversation());
-			mOpenConverstaion = null;
+			if (this.mConversationFragment.reInit(getSelectedConversation())) {
+				Log.d(Config.LOGTAG,"setting scroll position on fragment");
+				this.mConversationFragment.setScrollPosition(mScrollPosition);
+			}
+			mOpenConversation = null;
 		} else if (intent != null && ACTION_VIEW_CONVERSATION.equals(intent.getAction())) {
 			clearPending();
 			handleViewConversationIntent(intent);
diff --git a/src/main/java/eu/siacs/conversations/ui/ConversationFragment.java b/src/main/java/eu/siacs/conversations/ui/ConversationFragment.java
index edee551bb5..80b5ff82cf 100644
--- a/src/main/java/eu/siacs/conversations/ui/ConversationFragment.java
+++ b/src/main/java/eu/siacs/conversations/ui/ConversationFragment.java
@@ -12,6 +12,8 @@
 import android.os.Bundle;
 import android.os.Handler;
 import android.text.InputType;
+import android.util.Log;
+import android.util.Pair;
 import android.view.ContextMenu;
 import android.view.ContextMenu.ContextMenuInfo;
 import android.view.Gravity;
@@ -153,7 +155,11 @@ public void run() {
 									View v = messagesView.getChildAt(0);
 									final int pxOffset = (v == null) ? 0 : v.getTop();
 									ConversationFragment.this.conversation.populateWithMessages(ConversationFragment.this.messageList);
-									updateStatusMessages();
+									try {
+										updateStatusMessages();
+									} catch (IllegalStateException e) {
+										Log.d(Config.LOGTAG,"caught illegal state exception while updating status messages");
+									}
 									messageListAdapter.notifyDataSetChanged();
 									int pos = Math.max(getIndexOf(uuid,messageList),0);
 									messagesView.setSelectionFromTop(pos, pxOffset);
@@ -210,6 +216,28 @@ private int getIndexOf(String uuid, List<Message> messages) {
 		}
 		return -1;
 	}
+
+	public Pair<Integer,Integer> getScrollPosition() {
+		if (this.messagesView.getCount() == 0 ||
+				this.messagesView.getLastVisiblePosition() == this.messagesView.getCount() - 1) {
+			return null;
+		} else {
+			final int pos = messagesView.getFirstVisiblePosition();
+			final View view = messagesView.getChildAt(0);
+			if (view == null) {
+				return null;
+			} else {
+				return new Pair<>(pos, view.getTop());
+			}
+		}
+	}
+
+	public void setScrollPosition(Pair<Integer,Integer> scrollPosition) {
+		if (scrollPosition != null) {
+			this.messagesView.setSelectionFromTop(scrollPosition.first, scrollPosition.second);
+		}
+	}
+
 	protected OnClickListener clickToDecryptListener = new OnClickListener() {
 
 		@Override
@@ -736,9 +764,9 @@ private void updateChatState(final Conversation conversation, final String msg)
 		}
 	}
 
-	public void reInit(Conversation conversation) {
+	public boolean reInit(Conversation conversation) {
 		if (conversation == null) {
-			return;
+			return false;
 		}
 		this.activity = (ConversationActivity) getActivity();
 		setupIme();
@@ -774,6 +802,7 @@ public void reInit(Conversation conversation) {
 				pos = i < 0 ? bottom : i;
 			}
 			messagesView.setSelection(pos);
+			return pos == bottom;
 		}
 	}

```
