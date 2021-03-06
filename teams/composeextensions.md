# Compose extensions (preview)

>**Note:** Compose extensions are available in [Public Developer Preview](publicpreview.md) only. Additionally, many features in this document are under construction and are subject to change.

Compose extensions are a powerful new way for users to engage with your app within Microsoft Teams. With this capability, users can query for information from your service and post that information, in the form of rich cards, right into the channel conversation.

![Example of compose extension card](images/ComposeExtension/CEExample.png)

How would you use compose extensions? Here are a few possibilities:

* Work items and bugs
* Customer support tickets
* Usage charts and reports
* Images and media content
* Sales opportunities and leads

## Adding a compose extension to your app

Building a compose extension involves implementing familiar Microsoft Teams developer-platform concepts like bot APIs, rich cards, and tabs.

At its core, a compose extension is a cloud-hosted service that listens to user requests and responds with either structured data (cards) or rich media.  You integrate your service with Microsoft Teams via the Bot Framework APIs, which establishes the protocol for receiving and replying to user requests.

![Compose extension message flow diagram](images/ComposeExtension/CEFlow.png)

### Register in the Bot Framework

If you haven't done so already, you must first register a bot with the Microsoft Bot Framework. Visit [this link](https://msdn.microsoft.com/en-us/microsoft-teams/botscreate) for instructions.  The Microsoft app ID and callback endpoints for your bot, as defined there, will be used in your compose extension to receive and respond to user requests.  Remember to enable the Microsoft Teams channel for your bot.

Record your bot's app ID—you will need to supply this value in your app manifest.

### Update your app manifest

As with bots and tabs, you update the [manifest](schema.md) of your app to include the compose extension properties. These properties govern how your compose extension appears and behaves in the Microsoft Teams client. Compose extensions are supported beginning with v1.0 of the manifest.

#### Declare your compose extension

To add a compose extension, simply include a new top-level JSON structure in your manifest with the `composeExtensions` property.  Currently, you are limited to creating a single compose extension for your app.

The extension definition is an object that has the following structure:

| Property name | Purpose | Required? |
|---|---|---|
| `botId` | The unique Microsoft app ID for the bot as registered with the Bot Framework. This should typically be the same as the ID for your overall Teams app. | Yes |
| `scopes` | Array declaring whether this extension can be added to `personal` or `team` scopes. | Yes |
| `commands` | Array of commands that this compose extension supports. This is currently limited to one command. | Yes |

#### Define commands

Your compose extension should declare one command, which will appear when the user selects your app from the `...` option in the chat window. 

![Compose extension UI pop-up, showing the search option](images/ComposeExtension/CESearch.png)

In the app manifest, your command item is an object with the following structure:

| Property name | Purpose | Required? |
|---|---|---|
| `id` | Unique ID that you assign to this command.  The user request will include this ID. | Yes |
| `title` | Command name.  This value appears in the UI. | Yes |
| `description` | Help text indicating what this command does. This value appears in the UI. | Yes |
| `initialRun` | If set to true, indicates this command should be executed as soon as the user chooses this command in the UI. | No |
| `parameters` | List of parameters. | Yes |
| `parameter.name` | The name of the parameter.  This is sent to your service in the user request. | Yes |
| `parameter.description` | Describes this parameter’s purposes or example of the value that should be provided. This value appears in the UI. | Yes |
| `parameter.title` | Short user-friendly parameter title or label. | Yes |

#### Complete app manifest example

```json
{
  "$schema": "https://statics.teams.microsoft.com/sdk/v1.0/manifest/MicrosoftTeams.schema.json",
  "manifestVersion": "1.0",
  "version": "1.0",
  "id": "57a3c29f-1fc5-4d97-a142-35bb662b7b23",
  "packageName": "com.microsoft.teams.samples.bing",
  "developer": {
    "name": "John Developer",
    "websiteUrl": "http://bingbotservice.azurewebsites.net/",
    "privacyUrl": "http://bingbotservice.azurewebsites.net/privacy",
    "termsOfUseUrl": "http://bingbotservice.azurewebsites.net/termsofuse"
  },
  "name": {
    "short": "Bing",
    "full": "Bing"
  },
  "description": {
    "short": "Find Bing search results",
    "full": "Find Bing search results and share them with your team members."
  },
  "icons": {
    "outline": "https://pbs.twimg.com/profile_images/aceaeaojo18201/zlVaaEoG.jpg",
    "color": "https://pbs.twimg.com/profile_images/aceaeaojo18201/zlVaaEoG.jpg"
  },
  "accentColor": "#ff6a00",
  "composeExtensions": [
    {
      "botId": "57a3c29f-1fc5-4d97-a142-35bb662b7b23",
      "scopes": [
        "personal"
      ],
      "commands": [{
          "id": "searchCmd",
          "description": "Search Bing for information on the web",
          "title": "Search",
          "initialRun": "true",
          "parameters": [{
            "name": "searchKeyword",
            "description": "Enter your search keywords",
            "title": "Keywords"
          }]
        }
      ]
    }
  ],
  "permissions": [
    "identity",
    "messageTeamMembers"
  ],
  "validDomains": [
    "bingbotservice.azurewebsites.net",
    "*.bingbotservice.azurewebsites.net"
  ]
}
```
 
### Test via sideloading

You can test your compose extension by sideloading your app.  To do this, navigate to a team and its Apps dashboard.  Choose the "Sideload an app" link in the bottom right of the page.  Browse to the .zip file containing your apps manifest.

If the manifest is loaded correctly, your app will appear in the list of that team's installed apps. 

To invoke the compose extension, navigate to any of your chats or channels.  Choose the "..." below the message compose box.  Your compose extension’s commands should now appear in the list within the dialog.  Choose any command to bring up the search page.

>**Note:** Your app appears in channels only if you declare the `team` scope. Similarly, it appears in your chats only if it supports the `personal` scope.

## Receive and respond to queries

Now that you have your app up and running, it’s time to add some functionality.  Every request to your compose extension is done via an `Activity` that is posted to your callback URL.  The request contains information about the user command, such as ID and parameter values.  The request also supplies metadata about the context in which your extension was invoked, including user and tenant ID, along with chat ID or channel and team IDs.

### Receiving user requests

When a user performs a query, Microsoft Teams sends your service a standard Bot Framework activity.  It should perform its logic for activity with type `invoke`.

In addition to the standard bot activity properties, the payload contains the following relevant request metadata:

|Property name|Purpose|
|---|---|
|`type`| Must be `invoke`. |
|`name`| Type of command that is issued to your service.  For now, the only valid value is `composeExtension/query`. |
|`from.id`| ID of the user that sent the request. |
|`from.name`| Name of the user that sent the request. |
|`channelData.tenant.id`| Azure Active Directory tenant ID. |
|`channelData.channel.id`| If the request was made in a channel, this is the channel ID. |
|`channelData.team.id`| If the request was made in a channel, this is the team ID. |
|`clientInfo`| Additional metadata about the user’s client, such as locale/language and the type of client. |


The request parameters itself are found in the value object, which includes the following properties:

| Property name | Purpose |
|---|---|
| `commandId` | The name of the command invoked by the user, matching one of declared commands in the app manifest. |
| `parameters` | Array of parameters. Each parameter object contains the parameter name, along with the parameter value provided by the user. |
| `queryOptions` | Pagination parameters:<br>`skip`: skip count for this query<br>`count`: number of elements to return |

>**Note:** You should authenticate any request to your service. See [Messages, cards, and actions](https://msdn.microsoft.com/en-us/microsoft-teams/botsmessages) for more detailed documentation on receiving messages from the Bot Framework.

#### Request example

```json
{
  "name": "composeExtension/query",
  "value": {
    "commandId": "searchCmd",
    "parameters": [
      {
        "name": "searchKeywords",
        "value": "Toronto"
      }
    ],
    "queryOptions": {
      "skip": 0,
      "count": 25
    }
  },
  "type": "invoke",
  "timestamp": "2017-05-01T15:45:51.876Z",
  "id": "f:622749630322482883",
  "channelId": "msteams",
  "serviceUrl": "https://smba.trafficmanager.net/amer-client-ss.msg/",
  "from": {
    "id": "29:1C7dbRrC_5yzN1RGtZIrcWT0xz88KPGP9sxdpVpV8sODlgPHeQE9RqQ02hnpuKzy6zZ-AaZx6swUOMj_Dsdse3TQ4sIaeebbFBF-VgjJy_nY",
    "name": "Larry Jin"
  },
  "conversation": {
    "id": "19:skypespaces_8198cfe0dd2647ae91930f0974768a40@thread.skype"
  },
  "recipient": {
    "id": "28:b4922ea1-5315-4fd0-9b21-d941ab06e39f",
    "name": "TheComposeExtensionDev"
  },
  "entities": [
    {
      "locale": "en-US",
      "country": "US",
      "platform": "Web",
      "type": "clientInfo"
    }
  ]
}
```

### Responding to user requests

When the user performs a query, Microsoft Teams issues a synchronous HTTP request to your service.  At that point, your code has 5 seconds to provide an HTTP response to the request.  During this time, your service can perform additional lookup, or any other business logic needed to serve the request.

Your service should respond with the results matching the user query.  The response must indicate an HTTP status code of `200 OK` and a valid application/json object with the following body:

|Property name|Purpose|
|---|---|
|`composeExtension`|Top-level response envelope.|
|`composeExtension.type`|Should be of value `result`.|
|`composeExtension.attachmentLayout`|`list`: list of card objects containing thumbnail, title, and text fields. This is currently the only supported layout type, with more coming soon.|
|`composeExtension.attachments`|Array of valid bot attachment objects.  Currently the following types are supported:<br>`application/vnd.microsoft.card.thumbnail` <br>`application/vnd.microsoft.card.hero`<br>`application/vnd/microsoft.teams.card.o365connector`|

#### Response card types and previews

We support the following attachment types:

* Thumbnail card
* Hero card
* Office 365 Connector card

For full documentation on the thumbnail and hero card types, see [Messages, cards, and actions](https://msdn.microsoft.com/en-us/microsoft-teams/botsmessages). For additional documentation regarding the Office 365 Connector card, see [Actionable message card reference](https://docs.microsoft.com/en-us/outlook/actionable-messages/card-reference) in the Outlook Dev Center.

The result list is displayed in the Microsoft Teams UI with a preview of each item.  The preview is generated in one of two ways:

* Using the `preview` property within the `attachment` object.
* Extracted from the basic `title`, `text`, and `image` properties of the attachment.  These are used only if the `preview` property is not set and these properties are available.

#### Response example

```json
{
  "composeExtension":{
    "type":"result",
    "channelData":{
    },
    "attachmentLayout":"list",
    "attachments":[
      {
        "contentType":"application/vnd.microsoft.teams.card.o365connector",
        "content":{
          "sections":[
            {
              "activityTitle":"[85069]: Create a cool app",
              "activityImage":"https://<ImageUrl1>"
            },
            {
              "title":"Details",
              "facts":[
                {
                  "name":"Assigned to:",
                  "value":"[Larry Brown](mailto:larryb@example.com)"
                },
                {
                  "name":"State:",
                  "value":"Active"
                }
              ]
            }
          ]
        },
        "preview":{
        "contentType":"application/vnd.microsoft.card.thumbnail",
        "content":{
            "title":"85069: Create a cool app",
            "images":[
              {
              "url":"https://<ImageUrl1>"
              }
            ]
          }
        }
      }
    ]
  }
}
```

### Default query

When the user first brings up the compose extension dialog, Microsoft Teams issues a "default" query.  Your service can respond to this query with a set of prepopulated results.  This can be useful for displaying, for instance, recently viewed items, favorites, or any other information that is not dependent on user input.

The default query has the same structure as any regular user query, except with a parameter `initialRun` whose Boolean value is `true`.

#### Request example

```json
{
  "type": "invoke",
  "name": "composeExtension/query",
  "value": {
    "commandId": "searchCmd",
    "parameters": [
      {
        "name": "initialRun",
        "value": "true"
      }
    ],
    "queryOptions": {
      "skip": 0,
      "count": 25
    }
  },
...
}
```

## Identifying the user

Every request to your services includes the obfuscated ID of the user that performed the request, as well as the display name.

```json
"from": {
  "id": "29:1C7dbRrC_5yzN1RGtZIrcWT0xz88KPGP9sxdpVpV8sODlgPHeQE9RqQ02hnpuKzy6zZ-AaZx6swUOMj_Dsdse3TQ4sIaeebbFBF-VgjJy_nY",
  "name": "Larry Jin"
},
```

The `id` value is guaranteed to be that of the authenticated Teams user.  It can be used as a key to look up credentials or any cached state in your service. In addition, each request contains the user’s Azure Active Directory tenant ID, which can be used to identify the user’s organization. If applicable, the request also contains the team and channel IDs from which the request originated.

## Authentication

If your service requires user authentication, you need to sign in the user before he or she can use the compose extension. If you have written a bot or a tab that signs in the user, this section should be familiar.

The sequence is as follows:

1.	User issues a query, or the default query is automatically sent to your service.
2.	Your service checks whether the user has first authenticated by inspecting the Teams user ID.
3.	If the user has not authenticated, send back a `login` action including the authentication URL.
4.	The Microsoft Teams client launches a pop-up window hosting your webpage using the given authentication URL.
5.	After the user signs in, you should close your window and send an "authentication code" to the Teams client.
6.	The Teams client then reissues the query to your service, which includes the authentication code passed in step 5.

Your service should verify that the authentication code received in step 6 matches the one from step 5.  This ensures that a malicious user does not try to spoof or compromise the sign-in flow.  This effectively "closes the loop" to finish the secure authentication sequence.

### Responding with a sign-in action

To prompt an unauthenticated user to sign in, respond with a suggested action that includes the authentication URL.

#### Response example

```json
{
  "composeExtension":{
    "type":"auth",
    "channelData":{
    },
    "suggestedActions":[
      {
        "type": "openApp",
        "value": "https://example.com/auth",
        "title": "Sign in to this app"
      }
    ]
  }
}
```

>**Note:** For the sign-in experience to be hosted in a Teams pop-up, the domain portion of the URL must be in your app’s list of valid domains.

### Starting the sign-in flow

Your sign-in experience should be responsive and fit within a popup window. It should integrate with the [Microsoft Teams JavaScript library](jslibrary.md), which uses message passing.

As with other embedded experiences running inside Microsoft Teams, your code inside the window needs to first call `microsoftTeams.initialize()`.  If your code performs an OAuth flow, you should pass the Teams user ID into your window, which also then gets passed to the OAuth sign-in URL.

### Completing the sign-in flow

When the sign-in request completes and redirects back to your page, it should perform the following steps:

1.	Generate a security code. (This can be a random number.) You need to cache this code on your service, along with the credentials obtained through the sign-in flow (e.g. OAuth 2.0 tokens)
2.	Call `microsoftTeams.authentication.notifySuccess` and pass the security code.

At this point, the window closes and control is passed to the Teams client. The client now reissues the original user query, along with the security code in the `authenticationCode` property. Your code should use the security code to look up the credentials stored earlier to complete the authentication sequence. Your service can now complete the user request.

#### Reissued request example

```json
{
    "name": "composeExtension/query",
    "value": {
        "commandId": "insertWiki",
        "parameters": [{
            "name": "searchKeyword",
            "value": "lakers"
        }],
        "authenticationCode": "12345",
        "queryOptions": {
            "skip": 0,
            "count": 25
        }
    },
    "type": "invoke",
    "timestamp": "2017-04-26T05:18:25.629Z",
    "entities": [{
        "locale": "en-US",
        "country": "US",
        "platform": "Web",
        "type": "clientInfo"
    }],
    "text": "",
    "attachments": [],
    "address": {
        "id": "f:7638210432489287768",
        "channelId": "msteams",
        "user": {
            "id": "29:1A5TJWHkbOwSyu_L9Ktk9QFI1d_kBOEPeNEeO1INscpKHzHTvWfiau5AX_6y3SuiOby-r73dzHJ17HipUWqGPgw"
        },
        "conversation": {
            "id": "19:7705841b240044b297123ad7f9c99217@thread.skype"
        },
        "bot": {
            "id": "28:c073afa8-7e77-4f92-b3e7-aa589e952a3e",
            "name": "maotestbot2"
        },
        "serviceUrl": "https://smba.trafficmanager.net/amer-client-ss.msg/",
        "useAuth": true
    },
    "source": "msteams"
}
```

## SDK support

### .NET

To receive and handle queries with the Bot Builder SDK for .NET, you can check for the `invoke` action type on the incoming activity and then use the helper method in the NuGet package [Microsoft.Bot.Connector.Teams](https://www.nuget.org/packages/Microsoft.Bot.Connector.Teams) to determine whether it's a compose extension activity.

#### Example code

```csharp
public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
{
    if (activity.Type == ActivityTypes.Invoke) // Received an invoke
    {
        if (activity.IsComposeExtensionQuery())
        {
            // This is the response object that will get sent back to the compose extension request.
            ComposeExtensionResponse invokeResponse = null;

            // This helper method gets the query as an object.
            var query = activity.GetComposeExtensionQueryData();

            if (query.CommandId != null && query.Parameters != null && query.Parameters.Count > 0)
            {
                // query.Parameters has the parameters sent by client
                var results = new ComposeExtensionResult()
                {
                    AttachmentLayout = "list",
                    Type = "result",
                    Attachments = new List<ComposeExtensionAttachment>(),
                };
                invokeResponse.ComposeExtension = results;
            }

            // Return the response
            return Request.CreateResponse<ComposeExtensionResponse>(HttpStatusCode.OK, invokeResponse);
        } else
        {
            // Handle other types of Invoke activities here.
        }
    } else {
      // Failure case catch-all.
      var response = Request.CreateResponse(HttpStatusCode.BadRequest);
      response.Content = new StringContent("Invalid request! This API supports only compose extension requests. Check your query and try again");
      return response;
    }
}
```

### Node.js

The [Teams Extensions](https://www.npmjs.com/package/botbuilder-teams) for the Bot Builder SDK for Node.js provide helper objects and methods to simplify receiving, processing, and responding to compose extension requests.

#### Example code

```js
require('dotenv').config();

import * as restify from 'restify';
import * as builder from 'botbuilder';
import * as teamBuilder from 'botbuilder-teams';

class App {
    run() {
        const server = restify.createServer();
        let teamChatConnector = new teamBuilder.TeamsChatConnector({
            appId: process.env.MICROSOFT_APP_ID,
            appPassword: process.env.MICROSOFT_APP_PASSWORD
        });

        // Command ID must match what's defined in manifest
        teamChatConnector.onQuery('<%= commandId %>',
            (event: builder.IEvent,
            query: teamBuilder.ComposeExtensionQuery,
            callback: (err: Error, result: teamBuilder.IComposeExtensionResponse, statusCode: number) => void) => {
                // Check for initialRun; i.e., when you should return default results
                // if (query.parameters[0].name === 'initialRun') {}

                // Check query.queryOptions.count and query.queryOptions.skip for paging

                // Return auth response
                // let response = teamBuilder.ComposeExtensionResponse.auth().actions([
                //     builder.CardAction.openUrl(null, 'https://authUrl', 'Please sign in')
                // ]).toResponse();

                // Return config response
                // let response = teamBuilder.ComposeExtensionResponse.config().actions([
                //     builder.CardAction.openUrl(null, 'https://configUrl', 'Please sign in')
                // ]).toResponse();

                // Return result response
                let response = teamBuilder.ComposeExtensionResponse.result('list').attachments([
                    new builder.ThumbnailCard()
                        .title('Test thumbnail card')
                        .text('This is a test thumbnail card')
                        .images([new builder.CardImage().url('https://bot-framework.azureedge.net/bot-icons-v1/bot-framework-default-9.png')])
                        .toAttachment()
                ]).toResponse();
                callback(null, response, 200);
            });
        server.post('/api/composeExtension', teamChatConnector.listen());
        server.listen(process.env.PORT, () => console.log(`listening to port:` + process.env.PORT));
    }
}

const app = new App();
app.run();
```
