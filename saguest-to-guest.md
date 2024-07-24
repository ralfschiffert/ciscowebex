## Overview

In the context of guest-to-guest Webex meetings, guests are defined as non-licensed users. Guests are often used in applications that provide some form of expert services. For example, a doctor may talk to a patient or a customer to a support agent, or a plumber may have a video conference with a homeowner to pre-consult the work. Common to these use-cases, is the idea that none of the participating parties are permanently licensed as hosts for Webex meetings.

In the past, these apps were built on top of space-backed meetings where a space was established between the two parties to conduct the meeting. This approach used to work, albeit with limited functionality. For example, recording the meeting, providing meeting transcripts, or patching in a translator was impossible. In addition, meetings between only guests, are expressly denied under our terms and conditions, which in all cases mandates a licensed user to sponsor the meeting.

With the advent of the Webex Suite Meeting Platform (WSMP) the approach to guest-to-guest meetings is changing. The space-backed meetings are no longer available and instead replaced by regular Webex Meetings licensed by a guest-to-guest Service App. The proven and successful meeting’s backend now powers all meeting functionality. The new attendee experience is a full-featured meeting with all the standard meeting controls like recording, transcription, etc., available even when only guests participate. The meetings infrastructure licenses the guest-to-guest meeting experience from the licensed Service App, which only counts the number of simultaneous meetings. The following sections provide additional background information and developer guidance on the development of guest-to-guest Service Apps.

## Guest-to-Guest Workflow In a Nutshell

To develop a guest-to-guest Service App you need to do the following:

1. As a developer, get a trial subscription from `Cisco Commerce Workspace` (CCW) or a [sandbox](/docs/developer-sandbox-guide), which is configured for guest-to-guest meetings. Cisco preconfigures the sandbox with a guest-to-guest meeting site as part of the sandbox setup. You can find the name of the guest-to-guest site by logging in to your sandbox’s Control Hub as an admin and loading the [meeting site list](https://admin.webex.com/meetings/site-list).
1. With your sandbox developer account, log on to [developer.webex.com](https://developer.webex.com) and [create a Service App](https://developer-portal.int-first-general1.ciscospark.com/my-apps/new/service-app) with this mandatory and additionally any desired scopes: `guest-meeting:rw`
<div><Callout type="info">If you want the Service App also to create guests, for example to launch your SDK app with the guest, you must additionally select the scopes - `guest-issuer:write` and `guest-issuer:read` ([see this guide for Service App guest creation.)](https://developer.webex.com/docs/sa-guest-management). Generally, you would employ Service App guest creation only in cases where you need to provide some persistence for the guest. For example, if the doctor communicates with the patient over time in a space.</Callout></div>
1. Submit your app for admin approval. This action makes your Service App visible to an admin in Control Hub in your sandbox organization.
1. As an admin in [Control Hub](https://admin.webex.com), authorize your Service App to manage your guest-to-guest site and meetings. This can be done in Control Hub [here](https://admin.webex.com/apps/service-apps). Your Service App must be tied to only one available guest-to-guest site. For now, Control Hub enforces a 1 to 1 mapping between a service app with the guest `meeting:rw` scope and a guest meeting site.
<div><Callout type="info"> In the Sandbox Environment, the Service App is licensed for up to 10 simultaneous meetings. Generally each meeting may have up to 400 participants</Callout></div>
1. As a developer, retrieve the access and refresh tokens for the service app in your Service App details page on developer.webex.com. The tokens should be safely stored.
<div><Calloout type="info">For a more automated way of being informed about a Service App authorization and subsequent token retrieval, please see [here](https://developer.webex.com/docs/service-apps)</Calloout></div>
1. With the access token in hand, schedule a meeting via the [meetings API](/docs/api/v1/meetings/create-a-meeting). The meeting `siteUrl` must be the guest-to-guest site and should be autoconfigured during the Service App licensing and authorization process.
1. Create [join links](/docs/api/v1/meetings/join-a-meeting) for the guests. When the scheduled time comes around, the guests can use the join link to attend the meeting. You may need to have one guest use the `startLink` while the other guest gets the `attendeeLink`. The guest with the `startLink` will be treated as host.
Your guests, that are not individually licensed for Webex Meetings can now experience the full meeting experience. 
<div><Callout type="info">The guest-to-guest meeting is not limited to 2 guests. If desired many guests and also guests and license hosts can meeting in the guest-to-guest Service App scheduled meeting.</Callout></div>


## Meetings without a host role joining
Sometimes, you may not want to grant host privileges to and attendee. Instead, both parties use the `attendeeLink` to join and start the meeting. To achieve this behavior, you need to do the following:

1. In Control Hub make sure you have enabled the setting `They can join the meeting` under Services -> Meeting -> [Select your guest site] -> All -> Security -> Webex Meeting Security. This allows an attendee to join a meeting without the host.
1. The Service App [meetingPreferences](https://developer-portal.int-first-general1.ciscospark.com/docs/api/v1/meeting-preferences/get-meeting-preference-details) should include the following settings.
```json
...
schedulingOptions": {"enabledJoinBeforeHost": true,"joinBeforeHostMinutes": 5,"enabledAutoShareRecording": false,"enabledWebexAssistantByDefault": false},
...
```
You can retrieve the Service App `meetingPreferences` using the Service App access token.
1. When scheduling the meeting via the [create a meeting API](https://developer.webex.com/docs/api/v1/meetings/create-a-meeting) with the Service App access token, please set the following field `"unlockedMeetingJoinSecurity" : "allowJoin”`




### Persistent guests

If you give your Service App the `guest-issuer:read` and `guest-issuer:write` scopes, you can use it to create two or more guests. You can do this with a `POST` to `/guests/token` and the `Service App’s` access token as the Bearer token. Details can be found [here](https://developer.webex.com/docs/sa-guest-management). You would then, launch the [SDK](/docs/SDKs/android) with the guest token and let them join the meeting.

The persistent guest approach is also suitable if you want the guests to chat outside a meeting in the Webex App or to reach parity with previous space-backed workflows.

Here is a comparison with the previous workflow

| Space-backed meeting | Service App guest-to-guest meeting | 
| ---------------- | ------------------- |
| 2 guests are created via guest-issuer app type | Service App creates 2 guests |
| Bot creates a space | Service App creates a space |
| Bot adds 2 people to space | Service App adds two people to space |
| Guests join space and meet (adhoc) | Service App schedules meeting in space and guests can meet* |

*Adhoc meetings in spaces can be facilitated by setting the `roomId` and `adHoc=true` in the [create a meeting API call](https://developer-portal.int-first-general1.ciscospark.com/docs/api/v1/meetings/create-a-meeting). 
 

## Guest Experience

When guests click the join link, they can join the meeting either via an installed Webex native client or Webex browser. If you want host control over a meeting, a join-link with host control can be generated, i.e. [startLink](https://developer-portal.int-first-general1.ciscospark.com/docs/api/v1/meetings/join-a-meeting). This allows a guest to have full meeting controls, including the ability to manage the recording functions in the meeting, add additional guests dynamically, and more.

The guests can also be joined via our Webex SDKs to the meeting, usually from an Android or iOS device. If you want to use the SDKs you will have to create guests via the Service App and use the guest to join the meeting. More details can be found in the section [Join the Meeting by Entering a Password or Captcha](https://github.com/webex/webex-js-sdk/wiki/Migration-to-improved-meetings-associated-with-a-space)

The following sections bring more color to the flow above. They work from the assumption that you had no previous exposure to Service Apps or guest-to-guest meetings.

## Service App Background Information

### What are Service Apps
Service Apps represent an application type in the Webex developer program. They are recommended for apps that are mission-critical, long-lived and they are often needed for some kind of admin functionality. We see them most often used for ongoing provisioning or reporting systems. Service Apps are at the intersection of two other application types: `bots` and `integrations`.

They borrow from bots their independent nature. A machine account is created in the authorizer’s organization when a Service App is authorized. The developer retrieves the token for each organization’s machine account as an individual access and refresh token pair. All API calls made with the Service App tokens are made on behalf of this machine account. Since the machine account is not managed directly, it cannot be accidentally deleted, or manipulated in other ways that would make the app stop working. From `integrations`, they borrow the concept of scopes as well as the need to be authorized; both are implemented as oAuth clients.

Scopes determine the API calls that the Service App is allowed to make. Service App authorization happens in the Control Hub GUI and must be done by a Full Admin. You can think of Service Apps as bots with scopes. Bot tokens also have a long lifetime and represent machine accounts.

### Choosing a Bot or a Service App

When confronted with the decision between a `bot` or a Service App as your application type, you should decide with the *least privilege principle*. If a `bot` can get you all the functions you need, you should use the `bot` application type. However, `bots` have the following limitations:

- They cannot be compliance officers.
- They cannot be admins.
- They cannot see messages that are not addressed to them. 
- They are unaware of meetings or media (audio/video) (the reason why we cannot use them for our guest-to-guest meetings).

If you need any kind of admin functionality or any of the capabilities just mentioned you need to use a Service App like we do for our guest-to-guest flow. For now, Service Apps always have an admin role.

## Service Apps as Guest-to-Guest Meeting Facilitators

In the guest-to-guest meeting scenario, we are using the Service App as the meeting orchestrator for guest-to-guest meetings. If you think back about what makes Service Apps special, our design decision may be immediately apparent. The app needs to run on an ongoing basis, and we cannot risk the app stop working because a user leaves or changes their password. The app also needs access to meeting functionality to schedule, monitor and report on meetings. The alternative application type would have been a user-authorized `integration` with the danger that the user either leaves or is accidentally being deleted or changes their password, all of which would render the tokens invalid.

### Service Apps as Guest Creator

Guests are temporary, unnamed, and unlicensed users. Guests are used where a named user license doesn’t make sense or where an interaction in Webex is tightly time-bound. For example, when a customer launches the Webex meetings or chat widget from a company website, this customer most likely doesn’t need to interact through Webex on an ongoing basis. Instead, they should be supplied with a temporary identity that allows them to interact with the agent, which will be discarded after the exchange.

Guests in the past were created through a different app type called the Guest Issuer. However, we are changing the guest creation and now let Service Apps create guests. The biggest advantage to guest creation via Service Apps is that the flow is now inherently admin-approved. Guest creation via guest issuer is possible by any developer in an organization, without admin approval. With the old Guest Issuer flow, the guest organizations are not admin-managed, and admins had no insight into which guests were created in the org. With the new approach, guests can only be created after an admin approves the Service App in the Control Hub. Admins can also query the number of guests created by the Service App.

<div><Callout type="warning">Guest Issuer guests are considered deprecated and will be removed in the future</Callout></div>

### Guest-To-Guest Subscriptions, Guest Sites, and Licenses

We offer a new subscription for `guest-to-guest` meetings and, with it, a new license. Remember, guest-to-guest meetings or, more broadly, meetings without the sponsor of a license were and are a terms and conditions violation. Now, the Service App takes on the role of the license sponsor. The Service App does not need to participate in the meeting. It does need to schedule the meeting and may create the guests that ultimately join the meeting. In contrast to a regular user, the Service App can sponsor as many simultaneous meetings as are allowed by the subscription. When procuring the subscription, the partner orders a certain number of concurrent meetings.

For example, Let’s say the partner ordered `150` meetings with the subscription. This means the Service App can sponsor up to `150` meetings in the `inProgress` state at the same time; in other words, it can host up to `150` simultaneous meetings. When there are `150` meetings ongoing and a guest tries to start the 151st meeting, by clicking the join link or joining via the SDK, a license error is issued. Each meeting can have up to 400 participants joining.

Service Apps in the guest-to-guest flow are bonded during the authorization process to a specially configured and marked site, the guest-to-guest site. The site is set up in the organization when an admin activates the subscription via the provisioning wizard. The admin determines and assigns the site name, which, like any other site setup, must be globally unique. The necessary configuration for this guest-to-guest site is done by our backend system. Only the Service App is allowed as a user on this site. Guests join guest-to-guest meetings, but are not part of the guest-to-guest site.

### Webhook Notification

The Service App developer’s organization and the organization where the guest-to-guest meetings happen are not necessarily the same. In fact, for most applications, the developer and the authorizing admin are in separate organizations. The developers need to be informed of the authorization of the Service App by the admin, for them to retrieve the tokens. To that end, we provide a webbook with the resource `serviceApp` and the event type `authorized` and `deauthorized`. When the admin authorizes the Service App a webhook is sent to the registered `targetURL`. The webhook contains information about the Service App like the scopes granted and also the list of guest-to-guest sites the Service App was authorized for. With the webhook, the developer knows they can retrieve the tokens to call the APIs. They also know for which Webex sites these tokens can be used to schedule meetings. When an admin deauthorizes a Service App the tokens get invalidated. The developer will receive a `serviceApp:deauthorized` webhook.


## Conclusion

At this point, you should now be well on your way with the new Webex Suite Meeting Platforms way to create guest-to-guest meetings. You should add your own unique application functionality, whether it is reporting, transcription or AI. Once your users are satisfied with your app, you may consider submitting it to Webex App Hub for broader consumption by the larger Webex user base.

