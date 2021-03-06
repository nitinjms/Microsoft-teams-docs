# Creating deep links to a Microsoft Teams tab

For every tab, Microsoft Teams adds a 'Copy link to tab' menu action.  This generates a deep link that points to this tab, which users can share.

>**Note:** This deeplink is in a different format than the one you can generate yourself

You can also enable team members to create and share links to items _within_ your tab - such as an individual task within a tab that contains a task list.  When clicked, the link will navigate to your tab, which focuses on the specific item.  To implement this, you add a 'copy link' action to each item, in whatever way best suits your UI.  When the user takes this action, you call `shareDeepLink()` that displays a dialog containing a link that the user can copy to the clipboard.  When you make this call, you also pass in an ID for your item, which you get back in the [context](getusercontext.md) when the link is followed and your tab is reloaded.

Further, you can generate deeplinks programmatically, using the format specified below.  You may want to use these in [bot](bots.md) and [connector](connectors.md) messages that inform users about changes to your tab, or to items within it. 

>**Note:** Deep links only work properly if the tab was configured using the v0.4 or greater library and thus has an entity ID. Deep links to tabs without entity IDs will still navigate to the tab but not be able to provide the sub-entity ID to the tab.

## Showing a dialog that contains a deep link to an item within your tab

To show a dialog that contains a deep link to an item within your tab, call: `microsoftTeams.shareDeepLink({ subEntityId: <subEntityId>, subEntityLabel: <subEntityLabel>, subEntityWebUrl: <subEntityWebUrl> })`

The fields to provide are:
* `subEntityId` - a unique identifier for the item within your tab which you are deep linking to.
* `subEntityLabel` - a label for the item to use for displaying the deep link.
* `subEntityWebUrl` - an optional field with a fallback URL to navigate the user to, if the client does not support rendering the tab.

## Generating a deep link to your tab (for use in a bot or connector message)

The format for a deep link is as follows:
`https://teams.microsoft.com/l/entity/<appId>/<entityId>?webUrl=<entityWebUrl>&label=<entityLabel>&context=<context>`

The query parameters are:
* `appId` - the ID from your manifest.  For example, "fe4a8eba-2a31-4737-8e33-e5fae6fee194"
* `entityId` - the ID for the item in the tab, which you provided when [configuring the tab](createconfigpage.md).  For example, "tasklist123"
* `entityWebUrl` or `subEntityWebUrl` - an optional field with the fallback URL to navigate the user to, if the client does not support rendering the tab.  For example, "https://tasklist.example.com/123" or "https://tasklist.example.com/list123/task456"
* `entityLabel` or `subEntityLabel` - a label for the item in/within your tab, to use when displaying the deep link. For example, "Task List 123" or "Task 456"
* `context` - this is a JSON object containing the following fields.
    * `subEntityId` - an ID for the item _within_ the tab.  For example, "task456"
    * `canvasUrl` - the URL to load in the tab.  This should be the same as the `contentUrl` you provided when [configuring the tab](createconfigpage.md).  Note that this field is  required but not currently used - it is reserved for future use.  For example, "https://tab.tasklist.example.com/123"
    * `channelId` - the Microsoft Teams channel ID.  Note that you can get this from the tab [context](getusercontext.md).  For example, "19:cbe3683f25094106b826c9cada3afbe0@thread.skype"

>**Note:** Make sure `appId`, `entityId`, `entityWebUrl`, `subEntityWebUrl`, `entityLabel`, `subEntityLabel`, and `context` are all URI Encoded

Examples:

* Link to the tab itself: `https://teams.microsoft.com/l/entity/fe4a8eba-2a31-4737-8e33-e5fae6fee194/tasklist123?webUrl=https://tasklist.example.com/123&label=Task List 123&context={"canvasUrl": "https://tab.tasklist.example.com/123","channelId": "19:cbe3683f25094106b826c9cada3afbe0@thread.skype"}`

* Link to a task item within the tab: `https://teams.microsoft.com/l/entity/fe4a8eba-2a31-4737-8e33-e5fae6fee194/tasklist123?webUrl=https://tasklist.example.com/123/456&label=Task 456&context={"subEntityId": "task456","canvasUrl": "https://tab.tasklist.example.com/123","channelId": "19:cbe3683f25094106b826c9cada3afbe0@thread.skype"}`

## Consuming a deep link from a tab

When navigating to a deep link, Microsoft Teams simply navigates to the tab and provides a mechanism via the Microsoft Teams Tab library to retrieve the sub-entity id (if it exists).

The [`microsoftTeams.getContext`](jslibrary.md#getcontextcallback-context-contextcontext--void-void) call returns a context which will have the `subEntityId` field if the tab was navigated to via deep link.
