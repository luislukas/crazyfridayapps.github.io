---
title: App Links and Instant apps - How to
layout: post
date: '2020-02-27 21:44:43 +0000'
categories: android
---

We've recently helped one of our clients to build an [Instant app](https://developer.android.com/topic/google-play-instant/getting-started/instant-enabled-app-bundle) to drive more conversion and enhance the user experience. Another of the requirements was to add deep-linking to it so that when a marketing campaing is launch their customers could benefit from the new native journey. Following are the steps to achieve all this as well as our pre-requisites. 

#### Starting point
* In our hands we have an already _published_ app with the old _apk_ format.
* The client manages the certificate and has not been enrolled into the [App signing program by Google Play](https://support.google.com/googleplay/android-developer/answer/7384423)
* The client has its own domain (we'll need this later on to validate the deep-links)
* We don't have different apps per market but a single one for everybody.
* The current source code hasn't been modularized yet, it sits under the _app_ main module.
* Our use case for the _Instant app_ won't go further 10Mb and we won't be using any banned permission. [Here](https://developer.android.com/topic/google-play-instant/overview) you can read more about it.

Our first step it's to convince the client to enroll into the _App signing program_. One of the requisites for Instant Apps it's that they need to be published using the [App bundle](https://developer.android.com/platform/technology/app-bundle) format which already has a huge amount of advantatges over the old _apk_ format. Also, enrolling into the _App signing program_ doesn't mean that you loose control over the certificate - you still need to sign the artifact with it, it's just that it becomes an _upload_ certificate. 
> Tip: You don't need to migrate your already released app from _apk_ to _aab_, they are independent artifacts as we are going to see in a bit.

#### Getting your hands _dirty_
* Add `<dist:module dist:instant="true" />` inside the `AndroidManifest.xml` file under the `<manifest>` node. It should look like:

{% highlight xml %}
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:dist="http://schemas.android.com/apk/distribution"
    package="some.package.appid">
	<dist:module dist:instant="true" />
	<application></application>
</manifest>  
{% endhighlight %}

* If you haven't added any of the `play-services-xyz` dependencies yet and you plan to use some common parts of the code _but_ you want to know if the user is in the _Instant app_ or the _installable_ one for analytics purpose for example, add the following dependency in the `build.gradle :`

{% highlight xml %}
implementation 'com.google.android.gms:play-services-instantapps:17.0.0'
{% endhighlight %}

Now you have access to the following `InstantApps` methods: 
{% highlight kotlin %}
//true if we are in the Instant App, otherwise false
val isInstant: Boolean = isInstantApp(context)

//Display the default dialog to prompt the user to install the app
val playStoreInstallable = Intent(Intent.ACTION_MAIN)
            .addCategory(Intent.CATEGORY_DEFAULT)
            .setPackage("some.package.appid")
        InstantApps.showInstallPrompt(
            this,
            postInstall, REQUEST_CODE, /* referrer= */ null
        )
{% endhighlight %}

* _Instant apps_ require an already published _app_. The relation between both of them is as follow: 
If you are using an _Instant app_ and you accept to install the full app, the system will handle this as an _Update_ of the app - hence the _installable app_ needs to have **greater** `versionCode` than the _Instant app_. In practice, our _installable app_ had a `versionCode` of `100` and the _Instant app_ of `10`, giving us quite a large buffer.
* This changes don't need to go into your main _Installable app_ so we played a bit with gradle `flavours` to achieve this.
* Android Studio allows you to run the app as an _Instant app_ just tick the checkbox under `Run > Edit Configurations > Deploy as instant app`

#### Publishing the _Instant app_
This part is quite straight forward:
* Enroll in the _App signing program_ if you are not already - You will have to provide the 
* Generate the `aab` file with AS for example: `Build > Generate signed Bundle > Android app Bundle > Keystore details > release`
* Log into the [Google Play Console](play.google.com/apps/publish/), select the app and  find: `Release Management > Android Instant Apps`
* Under this section you'll find the same release approach as the `apk` - do some _internal testing_ or publish a _Beta_. Once you've done your testing just publish to production. 

> It can take up to four days for the _Instant app_ to be visible so be patient. Also you should have the setting _enabled_ within the _Google Play Store_: In your device Play Store go to Menu > Settings > Google Play Instant > Enable Upgrade web links

#### Adding App Links for Instant Apps
_App links_ just map _links_ with your _app content_ ie. if the user taps in [www.mywebsite.com/showMeOffers](www.mywebsite.com/showMeOffers) then our app will launch and display a native result rather than a website. This happens though only if the user has our app _installed_. The point of adding _App Links_ to _Instant Apps_ is that the user doesn't have the app installed in the device **but** is offered a native experience rather than a web based experience.
To create _App Links_ we just need to:
*
The developer documentation [here](https://developer.android.com/training/app-links/instant-app-links) explains quite well how to do this but in a nutshell: