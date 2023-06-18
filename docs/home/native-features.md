?> **Note:** Stay tuned for new feature instructions as we update and expand this page.

# Integrating Native Features

One of the best ways to upgrade your progressive web app is to take advantage of web capabilities to integrate with the user's operating system. Modern web technology has enabled a whole host of ways to make your PWA behave more like a native application and interact seamlessly with the OS.

This article will showcase how to enable various native functionality for your progressive web app.

## Shortcuts 

Application shortcuts allow your user to directly navigate to certain parts of your progressive web app directly from the operating system. When shortcuts are enabled, they can be accessed through your application's context menu. In Windows, the context menu can be opened by right-clicking on your app's icon.

A context menu with shortcuts will look like this:

<div class="docs-image">
   <img src="assets/home/native-features/shortcuts.png" alt="The shortcuts menu open in Windows." width=600>
</div>

Clicking a specific shortcut will take you directly to the associated content within your PWA. For example, in the screenshot above, clicking "PWA Starter" would open the PWABuilder Documentation PWA and navigate directly to the documentation for the PWA Starter.

### How to Implement Shortcuts

All that's necessary to add shortcuts to your progressive web app is adding a valid `shortcuts` field to your web app manifest. The `shortcuts` field accepts a list of `shortcut` objects and a very basic example would look like this: 

```json
"shortcuts": [
  {
    "name": "News Feed",
    "url": "/feed",
    "description": "Noteworthy news from today."
  },
  {
    "name": "New Post",
    "url": "/post",
    "description": "Create a new post."
  }
]
```

These shortcut objects just have a `name`, a `url` to navigate to, and a brief `description` of the shortcut.

There are a few extra fields you can add to spruce up your shortcut. Another example:

```json
"shortcuts": [
  {
    "name": "News Feed",
    "short_name": "Feed",
    "url": "/feed",
    "description": "Noteworthy news from today.",
    "icons": [
      {
        "src": "assets/icons/news.png",
        "type": "image/png",
        "purpose": "any"
      }
    ]
  }
]
```

In this example, we've added a few extra fields to our shortcut. `short_name` provides an alternate name to display for your shortcut if there is limited display space, while `icons` allows you to specify a custom icon to display for your shortcut in the context menu. For shortcuts, the array you provide must include an icon that is 96x96.

## Window Controls Overlay

!> Window Controls Overlay will only work with certain browsers, such as Edge and Chrome.

Enabling the window controls overlay feature allows you to customize the space next to the window controls (minimize, close, etc.) with CSS and Javascript. This can give you more control over how your app is presented in a native context after it is installed.

Enabling window controls overlay gives you access to this space: 

<div class="docs-image">
   <img src="assets/home/native-features/display-spec-wco.png" alt="The shortcuts menu open in Windows.">
</div>

The area to the right of the controls becomes customizable.

### How to Enable and Use Window Controls Overlay

To enable window controls overlay, set the value of `display_override` in your web app manifest like this:

```json
{
    “display_override”: [“window-controls-overlay”],
    "display": "standalone"
}
```

As you can see, we've also set the `display` value. This ensures that our application has a fallback to use in instances where window controls overlay is unavailable.

### Customizing With CSS

Now that the area in the title bar area is available, we can make use of CSS environment variables to make use of it. Some key variables:

* `titlebar-area-x`: the distance from the left of the viewport where the title bar area appears
* `titlebar-area-y`: the distance from the top of the viewport where the title bar area appears
* `titlebar-area-width`: the width of the title bar area
* `titlebar-area-height`: the height of the title bar area

Some example CSS to see how these could be used: 

```CSS
.titleBar {
    position: fixed;
    left: env(titlebar-area-x, 0);
    top: env(titlebar-area-y, 0);
    width: env(titlebar-area-width, 100%);
    height: env(titlebar-area-height, 40px);
    -webkit-app-region: drag;
    app-region: drag;
}
```

Take note of the `-webkit-app-region` and `app-region` values being set to `drag`. This allows our title bar drag behavior to work properly when window controls overlay is enabled.

## Web Share API 

The Web Share API allows your app to use the operating system's native share dialog to share content (files, links) to and from your progressive web app.

On Windows, the dialog for sharing from your progressive web app looks like this:

<div class="docs-image">
   <img src="assets/home/native-features/pwinter-share.jpg" alt="The native share dialog open on Windows.">
</div>

Making use of the Web Share API will ease the process of sharing to and from your PWA, and allows apps that make use of files to interact more cleanly with the OS.

### How to Share *From* Your PWA

!> Using the Web Share API requires `HTTPS`. Your progressive web app should be using `HTTPS` anyway to properly enable your service worker.

The easiest use case for sharing from your progressive web app is with a web link. It can be implemented with a single function:

```js
async function shareLink(shareTitle, shareText, link) {
    const shareData = {
        title: shareTitle,
        text: shareText,
        url: link,
    };
    try {
        await navigator.share(shareData);
    } catch (e) {
        console.error(e);
    }
}
```

All you have to do is make a call to `navigator.share()` and pass in the desired share data. We are passing a title to display, some text to add more detail, and the actual link to share itself.

You can also share files with the Web Share API. The code to share files is very similar to sharing a link:

```js
async function shareFiles(filesArray, shareTitle, shareText) {
    if (navigator.canShare && navigator.canShare({ files: filesArray })) {
        try {
            await navigator.share({
                files: filesArray,
                title: shareTitle,
                text: shareText
            });
        } catch (error) {
            console.log('Sharing failed', error);
        }
    } else {
        console.log(`System doesn't support sharing.`);
    }
};
```

The only primary change here is that we are making a call to `navigator.canShare()` to confirm that the file types we are trying to share are compatible with the Web Share API. You can find a list of supported file types [here.](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/share#shareable_file_types)

### How to Share *To* Your PWA

You can also enable your progressive web app to receive shared files from the native operating system. This is enabled in your PWA's web app manifest by adding the `share_target` member.

#### Adding Share Target To Your Manifest

You can add the `share_target` field to your Manifest like this:

```json
"share_target": {
    "action": "/share-action/",
    "method": "GET",
    "enctype": "application/x-www-form-urlencoded",
    "params": {
      "title": "title",
      "text": "text"
    }
}
```

The key field here is `action`. This allows you to set a specific URL that will open and handle a shared link of this type. If you want to execute functionality based on a shared link, you can have this page parse a link and determine how to handle the shared data.

Also, you can use the `params` field to specify which data will be parsed from the shared URL. For our above snippet, an example URL could look like:

```
/share-action/?title=example+share+title&text=example+share+text
```

#### Accessing Shared Data
To handle data shared through the Web Share API (`title` and `text` from the example above), you can add an event listener to the `action` URL that you specified for your share target:

```js
window.addEventListener('DOMContentLoaded', () => {
  const parsedUrl = new URL(window.location);
  const sharedTitle = parsedUrl.searchParams.get('title');
  const sharedText = parsedUrl.searchParams.get('text');

  // Do something with the parsed data
});
```

Once you have parsed the data from the share, you can do whatever you would like with it.

#### Receiving Share Data Sent With POST

In the above example, we handled data shared with `GET`, but you can also use the Web Share API to handle `POST` requests. Unlike `GET`, data shared with `POST` is handled by your service worker.

First, you need to specify in the `share_target` field of your Web Manifest that your application accepts POST requests for sharing data:

```json
"share_target": {
    "action": "/post-action-url/",
    "method": "POST",
    "enctype": "application/x-www-form-urlencoded",
    "params": {
      "title": "title",
      "text": "text"
    }
}
```

Next, you need to add handling for POST requests in a `fetch` handler in your service worker:

```js
self.addEventListener('fetch', (event) => {
  // Check if we are handling the proper request
  if(event.request.url.endsWith("/post-action-url/") && event.request.method === 'POST') {
    // Parse parameters from POST into proper URL Search Params format
    const params = new URLSearchParams(Array.from(event.request.url.searchParams.entries()));
    // Redirect to our page for handling shared data, forward the params
    return event.respondWith(Response.redirect(`/share-target/?${params.toString()}`, 303));
  }
});
```

Once you have redirected to your share target, you can handle the data just like you would with `GET`.

## Badging

If your progressive web app is installed to the OS, the Badging API allows you to display a notification badge on your PWA's taskbar icon. You can use this functionality to inform the user when new content is available or requires their attention. This can help to keep the engagement up for your progressive web app - bringing users back to view new content.

Here's what a badge for a progressive web app will look like on Windows:

<div class="docs-image">
   <img src="assets/home/native-features/badging-task-bar.png" alt="Image of a progressive web app with a notification badge displayed on Windows.">
</div>

### Displaying and Clearing Badges

Using the Badging API only takes a few lines of code:

```js
if ('setAppBadge' in navigator) {
  navigator.setAppBadge(1);
}
```

In this code snippet, we are just checking if the `setAppBadge` feature is available and then making a call to `navigator.setAppBadge(1)` to display a badge with a value of one.

To clear the app badge, use this function:

```js
navigator.clearAppBadge();
```

Or you can make another call to `setAppBadge()` with a value of zero:

```js
navigator.setAppBadge(0);
```

The Badging API can be used from within your progressive web app, or it's service worker. A common use case for the Badging API is setting the badge in response to a `push` event, which can be handled with a listener in your service worker.

Badging is often used in conjunction with the Notifications API to inform users when new content is available. The next section will take you through how make use of notifications for your progressive web app.

## Push Notifications

If you want a more direct way to notify users of content in your progressive web app, you can make use of the Notifications API. If the user gives permission for your app to send notifications, your app will be able to send a pop-up notification that displays on the operating system regardless of whether your app is currently running.

A notification displayed on Windows would look something like this:

<div class="docs-image">
   <img src="assets/home/native-features/notifications-action-center.png" alt="A push notification being sent by Edge for a progressive web app.">
</div>

### Displaying a Push Notification

Notifications are often displayed in response to `push` event from the back end. A service worker can listen for this event and then display a notification when a push is received. 

?> **Note** For simplicity sake, this guidance will focus on handling `push` from the front end, but you can learn more about the Push API [here.](https://microsoft.github.io/win-student-devs/#/30DaysOfPWA/advanced-capabilities/07?id=how-to-start-1)

### Requesting Permission

Before your progressive web app can display notifications, you need to request permission to display them. 

The Notification API has a function to request permission from the user:

```js
Notification.requestPermission();
```

If the user grants permission when prompted, your app will then be able to send notifications to the user. You only need to request permission to display notifications once to send notifications from your app indefinitely. The user can revoke this permission in their browser's settings.

How and when you choose to request permission is up to you. However, it is often recommended to request permission in response to a user action, such as a button click. You can take a look at an [example](/home/pwa-intro?id=trigger-a-notification) in the PWA Overview documentation.

### Adding A Push Listener to our Service Worker

Once we have permission to display notifications, we need a way to actually display them. We can add a `push` event listener to our service worker to handle push events and then display a notification:

```js
self.addEventListener('push', (event) => {
  event.waitUntil(
    self.registration.showNotification('Notification Title', {
      body: 'Notification Body Text',
      icon: 'custom-notification-icon.png',
    });
  );
});
```

In this snippet we are making a call to `showNotification` and passing in a `title` argument and an object containing further specifications for our notification. In this case, we are sending body text and a custom icon for our notification.

?> **Note** You can view a list of data you can pass to your Notification [here.](https://developer.mozilla.org/en-US/docs/Web/API/notification)

### Handling Notification Clicks

Now that we can display notifications, we have to add logic for handling when those notifications are clicked by the user.

We can add a listener for the `notificationclick` event to our service worker:

```js
self.addEventListener('notificationclick', (event) => {
    event.notification.close(); 
    var fullPath = self.location.origin + event.notification.data.path; 
    clients.openWindow(fullPath); 
});
```

First, we call `notification.close` to remove the notification.

Next, if we included the `path` field with our notification data, we can append that path our origin path and make a call to `clients.openWindow`. This will launch our progressive web app and open it at our desired location.

## Learn More

More native integrations documentation will be added here over time, but if you want more guidance on how to further integrate and upgrade your progressive web app, check out the [30 Days of PWA series.](https://microsoft.github.io/win-student-devs/#/30DaysOfPWA/README)