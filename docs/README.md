# openloop-html-connect
**Your single HTML5 Creative -> OpenLoop -> All Media Owners -> Millions of Panels.**

HTML5 SDK for connecting OpenLoop with HTML Creatives with interfaces to simplify Media Owner integration for DOOH Dynamic Campaigns.

This library interface all you need for getting panel's information or campaign assets while the HTML5 creative is running on any Media Owner's panel.

@@TOC@@

# Installation
If you are using webpack or similar, install the library:

`npm install openloop-html-connect --save`

And require/import it on your project.

Or just copy the content on [index.js](index.js) and paste it on your creative.

# Usage
- **Download** or install this library from npm.
- **Implement your creative** using the library API.
	- **Import library** on your creative.
	- **Set defaults** for local testing using:
		- `setDefaultSyncPath`
		- `addDefaultFeed`
	- **Load** using `.load(success, error)`.
	- **Use getters** (inside sucess callback) to feed your creative using:
		- `getSyncPath`
		- `getFrameId`
		- `isLive`
		- `isDebug`
		- `feeds`
	- **onPlay**
		- Render / Play Videos / Trigger animations inside `onPlay` method.
	- *IMPORTANT*: Do not use getters outside the sucess callback of the load method.
- **Go live**
	- **Upload creative and content** through OpenLoop or OpenLoop API.
	- OpenLoop will take care about how to modify the Creative in order to make it works on any Media Owner's panel.
	- Your creative will runs on a Media Owner's panel and when using the library methods, those methods will retrieve the correct information depending on the Media Owner and the feeds attached by OpenLoop.

# Examples

## Creative using webpack
You can see a complete creative example on:

[examples/creative-example](examples/creative-example)

 This is an example using this library and webpack for generating an unique HTML with everything self contained.

## Quick sample
```javascript
// Import or copy library. (e.g.: ES6 / TS using webpack).
import openLoopConnect from 'openloop-html-connect';

// Defaults for local testing on dev environment:
openLoopConnect.setDefaultSyncPath('../../myAssetsFolderOnDevEnvironment/');
openLoopConnect.feeds.assets.addDefaultFeed('cloudy')
	.addItem('cloudy.jpg');
openLoopConnect.feeds.json.addDefaultFeed('weather', {
	panels: [
		{ id: 123, status: 'cloudy' },
		{ id: 456, status: 'sunny' }
	]
});

// Initialization.
let imageToDisplay;

openLoopConnect.load(function () {
	// On success parsing config file or loading setted defaults.
	try {
		// Your logic depending on frame_id and parsing of feed data.
		let customFeed = openLoopConnect.feeds.json.getFeed('weather');
		let frameId = parseInt(openLoopConnect.getFrameId());
		// The following parse is based on your own feed json structure
		let panelWeatherData = customFeed.panels.find(panel => panel.id === frameId);
		let currentWeather = panelWeatherData.status;
		// Your logic depending on feed data.
		let imageToDisplay = openLoopConnect.feeds.assets.getFeed(currentWeather)[0];
	} catch (e) {
		// Error on creative logic.
		// e.g.: Feed not found, frameId not found, panel not found, etc..
		fallBackToEmbeddedDefaults();
	}
}, function (e) {
	// On error loading/parsing Config file.
	fallBackToEmbeddedDefaults();
});

function fallBackToEmbeddedDefaults() {
	imageToDisplay = 'blob:embeddedImage';
}

openLoopConnect.onPlay(() => {
	// Place here all play of videos or trigger of animations.
	document.getElementById("myImageId").src = imageToDisplay;
	// Note: Assets getted from feeds.assets.getFeed, already contains syncPath as prefix.
});
```

# API

## load(success, error)
Use it with the `success` and `error` callbacks parameters.

- `success` - Called if:
	- OpenLoop config file was loaded successfully
	- Config file is not setted so the library successfully fallbacks to the setted defaults.
- `error` - Called if OpenLoop sets that a config file needs to be loaded but the library failed to load or parse the config file.

Using **getters** inside the `success` is **mandatory** and we strongly recommend using the `error` callback to fallback to your own defaulting strategy (see example above).

**Why?** - Resolves if it necessary to load the OpenLoop config file or it can use the default values. If the OpenLoop config file is loaded it waits until load, parsing and set of new values is complete.

## onPlay(listener)
The listener function received by the `onPlay` method should contain all the logic for playing videos or triggering animations.

This listener will be called always after `.load` and depending on the Media Owner should also be called after the player notifies the play action or otherwise instantly after `.load`. So by default (on dev environment) it will be called just after `.load` finishes.

**Why?** - Some Media Owners pre-loads the creative by loading the HTML without actually showing it, but then when they really switch to your creative they can let you know when you need to start playing videos or triggering animations. We unify the different MO's specific logic by providing this single entry point method for play videos or trigger animations.

## setDefaultPlayCallback(callbackName)
**Only for testing purposes** - Sets the default play callback that will be called by the player. (like `setDefaultPlayCallback('BoardSignPlay)`).

## getSyncPath()
Gets the current sync path.

You should use this path as suffix for any external asset you need to request as it may change depending on the Media Owner.

You do not need to use this if you are using `openLoopConnect.feeds.assets.getFeeds`, the assets retrieved by this method already prefix the file paths with this sync path.

If you are testing locally (without OpenLoop publishing process yet), this will automatically returns the content received on `setDefaultSyncPath`.

## setDefaultSyncPath(syncPath)
As all your requests should use `getSyncPath` as suffix, this methods allows you to set the current path while you are testing locally.

Your final creative can still calling this method as the default sync will not be used once OpenLoop process your creative.

## setSyncPath(syncPath)
You should **NEVER use this method** unless you are a Media Owner.

This method is reserved for Media Owner's players that can directly specify the sync path.

## getFrameId()
Gets the current frame id.

Depending on Media Owner there will be different ways to retrieve the frame id (BroadSignObject, query string, etc..).

This method provides you a single entry point to retrieve that.

While testing you can call your HTML with a query string with the frame_id.
```javascript
// accessing: myCampaign/index.html?frame_id=123
openLoopConnect.getFrameId() // will return "123".
```

## setDefaultFrameId(frameId)
Sets the default frame id that the `getFrameId` method should retrieve in case that there is no other place to retrieve the frameId (like query string).

## isLive()
Returns a boolean. True if it is accessing live (no offline sync).

## isDebug()
Returns a boolean. True if the debug flag is enabled.

While testing you can call your HTML with a query string with the debug flag.
```javascript
// accessing: myCampaign/index.html?frame_id=123&debug=1
openLoopConnect.isDebug() // will return true.
// ---
// accessing: myCampaign/index.html?frame_id=123
openLoopConnect.isDebug() // will return false
```
## getVersion()
Get the current version of this library.

## setDefaultConfigFile(filePath)
**Only for testing purposes** - Sets the OpenLoop's config file to be loaded by default. Please try to use default setters instead of this method so you don't need to deal with know the final structure of the OpenLoop's config file.

## getForceDefault()
Returns a boolean which is the Force Default flag of the campaign defined on OpenLoop.

This is a flag that can be easily setted on OpenLoop to switch the entire campaign for a default behaviour instanly.

## setDefaultForceDefault(forceDefault)
Sets the default value of the `getForceDefault`.

## getWidth()
Gets the panel's width in pixels.

**Note:** This data is not available for all Formats, please contact us before use it for a specific campaign.

## setDefaultWidth(width)
Sets the default value of the `getWidth`.

## getHeight()
Gets the panel's height in pixels.

**Note:** This data is not available for all Formats, please contact us before use it for a specific campaign.

## setDefaultHeight(height)
Sets the default value of the `getHeight`.

## getBackgroundColor()
Gets the panel's recommended background color in hexadecimal (like #000000).

## setDefaultBackgroundColor(bgColor)
Sets the default value of the `getBackgroundColor`.

## feeds
OpenLoop and OpenLoop API lets you set/upload these **types of feeds**:

- assets (images or videos)
- free texts
- ~~xml~~ (not used on HTML5)
- json

And for each **type of feed** you can have as many feeds as you want, each of them with a defined **feedId**.
Then for each feed you can have various items where the only important thing is the order.

So the full structure will be:

- assets (images or videos)
	- feed1
		- image1
		- image2
		- video1
	- feed2
		- image1
		- image2
		- video1
- free texts
	- feed1
		- text1
		- text2
- json
	- feed1
	- feed2

So you can say for example that the **second item** of the **assets** feed with id **cloudy** will be a video for a cloudy day and you can retrieve it like this.
```javascript
let videoSrc = openLoopConnect.feeds.assets.getFeed('cloudy')[1];
```

### feeds.json
#### feeds.json.getFeed(feedId)
Retrieves the json feed by id. Returns the object resulting on parsing the original JSON feed embedded by OpenLoop, or the feedObject defined by `feeds.json.addDefaultFeed`.

#### feeds.json.addDefaultFeed(feedId, feedObject)
Adds a default feed with the given id and the feedObject.

#### feeds.assets.addDefaultFeedFromFile(feedId, filePath)
Adds a default feed that on the `load` stage will load the feeds content from an external file.

**Note:** As this uses an AJAX call, this will only work when you have the creative hosted on a server (like webpack-dev-server) while testing locally.

### feeds.freeTexts
#### feeds.freeTexts.getFeed(feedId)
Retrieves the free texts feed by id. Returns an array of strings which are the free texts defined and embedded by OpenLoop, or the items defined by `feeds.freeTexts.addDefaultFeed` and `addItem`.

#### feeds.freeTexts.addDefaultFeed(feedId)
Adds a default feed with the given id. Then this feed can be filled using the `addItem(item)` method using chaining pattern like the following example.
```javascript
openLoopConnect.feeds.freeTexts.addDefaultFeed('cloudy')
	.addItem('Today is cloudy')
	.addItem('Today is overcast');
```
### feeds.assets
#### feeds.assets.getFeed(feedId)
Retrieves the assets feed by id. Returns an array of strings which are the asset's paths uploaded and embedded by OpenLoop, or the items defined by `feeds.assets.addDefaultFeed` and `addItem`.

*Note:* Those assets paths already contains the sync path as prefix of the asset path.

#### feeds.assets.addDefaultFeed(feedId)
Adds a default feed with the given id. Then this feed can be filled using the `addItem(item)` method using chaining pattern like the following example.
```javascript
openLoopConnect.feeds.assets.addDefaultFeed('cloudy')
	.addItem('cloudy.jpg')
	.addItem('cloudy.mp4');
```

## errors
The connect library also expose a set of custom error types that can be usefull for error handling on the creative like the following example.
```javascript
try {
	openLoopConnect.feeds.json.getFeed('weather')
} catch(e) {
	if(e instanceof openLoopConnect.errors.ResourceNotFoundError) {
		// do something if feed was not found.
	}
}
```

### errors.OpenLoopHTMLConnectError
All the errors that this library throws, inherits from this error type.

### errors.ResourceNotFoundError
Thrown when:
- `getFrameId` was called but is not setted on query string.
- `getFeed` was called but the requested feed was not found.

### errors.InvalidOperationError
Thrown when any getter was called outside the sucess callback of the `.load` method.

# Ajax calls
We **strongly disencourage** the usage of ajax calls because they will not work on an offline panel. Internally the SDK and OpenLoop use JSONP (using script tags) to send data to the creative.

If you want to attach API feeds (social network, traffic, weather, custom, etc..) so the creative can have that information, please add this feeds on the OpenLoop configuration and OpenLoop will make sure that the feeds will be attached to the **OpenLoop config file** and so the creative can load this data through this SDK library (see example above).

If despite this warning you still want to make ajax calls, please make sure that your campaign will not run offline and the panel's Media Owner allows you to make external calls.

# How it works
- The library contains some placeholders for parameters that can be defined by OpenLoop.
- OpenLoop process the Creative on the Publishing process, on this process it will search for this library inside your Creative and modify those placeholders with the correct information for the targeted Media Owner.
- The library API methods, internally checks the existence of some parameters that can be defined by OpenLoop.
- If those parameters are setted the library returns those values, if not, the library will fallback for the default values that you can also define on the development stage.
- Some parameters like feeds, are setted on the **OpenLoop config file**, this file will be automatically loaded defining the correct values for the feeds in the presence of a flag setted by OpenLoop on the publishing stage.

# Browser compatibility
This library was successfully tested on:
- IE 11
- Microsoft Edge 40
- Chrome 64
- Firefox 58

# Support
Please contact with [QDOT](http://q.media/).

We will be glad to support you on integrating with OpenLoop.
