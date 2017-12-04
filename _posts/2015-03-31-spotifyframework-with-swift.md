---
layout: post
title: "Spotify.framework with Swift"
tags: ["spotify", "swift", "ios"]
---

From Spotify's iOS SDK documentation:

> Important! The iOS SDK is currently a beta release; the content and functionality
> of future versions is likely to change significantly and without warning.

It's true. The API is changing substantially and without warning. Worse yet, the documentation isn't always updated. This post itself will probably be useless soon.

The two main problems that I had trying to log in and get a song to play were
that [Spotify's getting started guide](https://developer.spotify.com/technologies/spotify-ios-sdk/tutorial/) calls deprecated methods and is written in Objective-C, not Swift. These two problems combined made getting a song to play in Swift difficult because Objective-C will allow you to call methods marked as deprecated with a warning, whereas Swift will raise an error.

Some parameters marked as being type id in [Spotify's documentation](https://developer.spotify.com/ios-sdk-docs/Documents/index.html) actually had specific types specified in the Spotify.framework/Headers directory. Running into deprecated methods necessitated guessing which other methods do the same job now. Not being able to see any example usage of those methods and not knowing their true parameter types made my job hard.

After some frustration, I edited my copy of the Spotify framework to undeprecate the methods used in Spotify's getting started guide. It worked to get me logged in and playing tunes, but the hackery of it all left me feeling queezy. After putting in some more time, I figured out how to get the Spotify API working in Swift without undeprecating any methods.

#setup

Create a new project in Swift and follow [Spotify's getting started
guide](https://developer.spotify.com/technologies/spotify-ios-sdk/tutorial/) up
to but not including the section titled __Creating the Application Code__. They're straightforward. You need to register your app with Spotify to get keys to login to you account, run a small Sinatra server in Ruby on your local machine, and set up your Xcode project to include the Spotify framework and some other options.

Next, you need to link Spotify's Objective-C framework to your Swift app. To do
that, create a new Cocoa Touch class and make sure the language is set to
Objective-C. Leave it as a subclass of NSObject, though I don't think it
matters. Name it "__Spotify__" and continue. After you click create, __Xcode
will ask you if you want to configure an Objective-C bridging header. Click
Yes.__

Now delete _Spotify.h_ and _Spotify.m_. There should also exist a file whose name ends in -Bridging-Header.h. Open it and add the following import line:

{% highlight objective-c %}
#import <Spotify/Spotify.h>
{% endhighlight %}

#log in and tune out

While following Spotify's setup instructions, you created a "single view application" which means Xcode should have created a ViewController.swift file for you. If not, create one. Add these lines:

{% highlight swift %}
let kClientID = "<app-client-id>"
let kCallbackURL = "your-app-name://your-return-after-login-path"
let kTokenSwapURL = "http://localhost:1234/swap"
let kTokenRefreshURL = "http://localhost:1234/refresh"
{% endhighlight %}

Replace the client id and callback url values with your own. Assuming you're running the HTTP token service on your local machine, and left the port number its default 1234, the latter two values should be the same for you.

Add a button to your initial view controller and wire up an action between them.
Either storyboard it out or do it programmatically. My action's name was
_loginWithSpotify_. After filling in that action, and deleting the cruft that
Xcode prepopulates view controllers with, my __ViewController.swift__ looks like this:


{% highlight swift %}
import UIKit

class ViewController: UIViewController, SPTAuthViewDelegate, SPTAudioStreamingPlaybackDelegate {
    let kClientID = "<client-id>"
    let kCallbackURL = "my-spotify-app://return-after-login"
    let kTokenSwapURL = "http://localhost:1234/swap"
    let kTokenRefreshURL = "http://localhost:1234/refresh"

    var player: SPTAudioStreamingController?
    let spotifyAuthenticator = SPTAuth.defaultInstance()

    @IBAction func loginWithSpotify(sender: AnyObject) {
        spotifyAuthenticator.clientID = kClientID
        spotifyAuthenticator.requestedScopes = [SPTAuthStreamingScope]
        spotifyAuthenticator.redirectURL = NSURL(string: kCallbackURL)
        spotifyAuthenticator.tokenSwapURL = NSURL(string: kTokenSwapURL)
        spotifyAuthenticator.tokenRefreshURL = NSURL(string: kTokenRefreshURL)

        let spotifyAuthenticationViewController = SPTAuthViewController.authenticationViewController()
        spotifyAuthenticationViewController.delegate = self
        spotifyAuthenticationViewController.modalPresentationStyle = UIModalPresentationStyle.OverCurrentContext
        spotifyAuthenticationViewController.definesPresentationContext = true
        presentViewController(spotifyAuthenticationViewController, animated: false, completion: nil)
    }

    // SPTAuthViewDelegate protocol methods

    func authenticationViewController(authenticationViewController: SPTAuthViewController!, didLoginWithSession session: SPTSession!) {
        setupSpotifyPlayer()
        loginWithSpotifySession(session)
    }

    func authenticationViewControllerDidCancelLogin(authenticationViewController: SPTAuthViewController!) {
        println("login cancelled")
    }

    func authenticationViewController(authenticationViewController: SPTAuthViewController!, didFailToLogin error: NSError!) {
        println("login failed")
    }

    // SPTAudioStreamingPlaybackDelegate protocol methods

    private

    func setupSpotifyPlayer() {
        player = SPTAudioStreamingController(clientId: spotifyAuthenticator.clientID) // can also use kClientID; they're the same value
        player!.playbackDelegate = self
        player!.diskCache = SPTDiskCache(capacity: 1024 * 1024 * 64)
    }

    func loginWithSpotifySession(session: SPTSession) {
        player!.loginWithSession(session, callback: { (error: NSError!) in
            if error != nil {
                println("Couldn't login with session: \(error)")
                return
            }
            self.useLoggedInPermissions()
        })
    }

    func useLoggedInPermissions() {
        let spotifyURI = "spotify:track:1WJk986df8mpqpktoktlce"
        player!.playURIs([NSURL(string: spotifyURI)!], withOptions: nil, callback: nil)
    }
}

{% endhighlight %}

#done!

This is the simplest working example I could manage. If you run it, you should be able to log into your Spotify account and have some music start playing. Between this example and the help that a bit of dated documentation provides, you should have a solid springboard for a Spotify app in Swift. There are some things worth noting.

###SPTAudioStreamingController.playTrackProvider:callback:

Every getting started guide I've found uses playTrackProvider:callback: to start playing music, but it's deprecated and therefore flagged as a compile time error in Swift. It is only flagged as a warning in Objective-C projects, which is why Spotify's Demo projects will still run. The fix is to either use

{% highlight swift %}
SPTAudioStreamingController.playURIs:withOptions:callback
{% endhighlight %}

as I did above, or to expand the Spotify.Framework/Headers directory in your Xcode project and open up SPTAudioStreamingController.h for editing. Then find this line:

{% highlight objective-c %}
-(void)playTrackProvider:(id<SPTTrackProvider>)provider callback:(SPTErrorableOperationCallback)block DEPRECATED_ATTRIBUTE;
{% endhighlight -c %}

and remove __DEPRECATED_ATTRIBUTE__.

###Spotify URIs

Right click any playlist, album, or song in Spotify to get its URI. You can use your non-public playlists if you're logged in as well.
