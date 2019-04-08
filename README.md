# UA initiated immersive sessions

## Motivation

Current workflow to [request a WebXR session](https://www.w3.org/TR/webxr/#dom-xr-requestsession) requires user gesture activation. 2D pages usually provide a button that users click or tap to allow immersive mode, preventing pages to take over the display inadvertently without user's permission.

Some scenarios like in-vr navigation require pages to enter immersive mode immediately without loading a 2D page first. UAs need an aditional standard mechanism to explicitly grant immersive sessions. Browsers will have the resposibility to inform the user and ask for appropriate permissions outside of the context of a 2D page.

## Design Goals

The goal of this repository is to discuss and design a simple API for UAs to explicitly grant immersive sessions.

## Non Goals

Define how a Virtual Reality or Augmented Reality browser would work [in alignment with the WebXR spec](https://github.com/immersive-web/webxr/blob/master/explainer.md#non-goals)

While this discussion may make recommendations or define requirements for required behavior for informing users about targeted URLs for navigation, for example, the actual behavior and mechanism a browser should implement is not a goal of this proposal.

## User Scenarios

1. Immersive navigation, allowing UAs to implement seamless transitions between interlinked VR and AR content. Use cases include:

    - Games that contains links to other games. Or links to friends' games.
    - Personal blogs that link to affiliated VR sites or your friends blogs.
    - Educational websites for a class, that link to the school's portal or other resources.
    - User generated content linking to one another. Like Rec Room, VR Chat or Super Craft.
    - Sponsored links. You have a VR world about photography and filmmaking that links to Photorama, B&H, GoPro's VR portals.
    - Content portals in general (directories of games, immersive videos, collections).
    - Personal storefronts that link to products hosted by eBay VR, Amazon VR. One could also easily incorporate product 3D models hosted externally in services the like of Sketchfab or Google Blocks.

2. In an AR browser, a page will display virtual objects in physical context. Sites present 3D content immediately at load time without going through a 2D page workflow first.

3. Open in immersive mode. 2D windowed browsers might offer a way to open a link directly immersive mode, similar to "open in new tab" or "open in new window" actions offered today. This and other UA approaches could automatically grant permission to create an immersive session.

4. Kiosk mode. In public spaces like museums or trade shows one could setup an URL as `virtual homescreen`: A page that loads and enters immersive mode automatically when the browser starts.

5. Intents and deep linking. Native aplications might request a
browser to open content directly in immersive mode.

6. [PWA](https://developers.google.com/web/progressive-web-apps/) An installed progressive WebXR app will enter immersive mode at launch.

7. Performance implications of single page applications. Having different web pages load is sort of beneficial for not having to deal with offloading / onloading new models and environments or making your own interstitial. You can avoid frame drops by letting the browser handle the transition, and VR web pages load quick enough.

## API Proposal

UAs fire a `sessiongranted` event that allows content to enter immersive mode without user gesture activation:

```
navigator.xr.addEventListener('sessiongranted', function (evt) {
   // One could check for the type of session granted.
   // Events notifies of session creation after navigation, UA action, or requestSession.
   // The session object is provided as part of this event.
   if (evt.session.mode === 'immersive-vr') {
      // set up app state for immersive vr, if that's what the app wants
   } else {
      // notify user that this app only works in immersive vr mode, if desired
   }
}
```

This is similar to the [WebVR displayactivate event](https://immersive-web.github.io/webvr/spec/1.1/#dom-window-onvrdisplayactivate) that popular libraries and frameworks incorporate alongide user gesture activated flows. The UA always remains in control on when immersive sessions are granted and can also revoke them.

## Security considerations

In immersive mode the Browser UI may not be rendered and content controls all display real estate. The UA must steward users giving them information and control over displayed content at all times.

Spoofing is a potential risk during immersive navitagion. A malicious site could masquerade legitimate content, the browser UI or even the underling operating system interface without user noticing. Possible mitigaton mechanisms are:

- Interstitial. When navigation occurs the UA can present the user a screen to evaluate the destination URL, an experience description and a button to continue or return to previous site.
- Overlays. UAs control compositing and can always render information about the navigated page over content.
- Modal dialog. User can invoke the browser UI at all times to interrupt or consult info about the current site. A modal interface (similar to Oculus or Steam home spaces) could be showed on demand with a reserved button or interaction that cannot be listened by content. To prevent spoofing, UAs could additionally display personal information that user can recognize and pages don't have access to (user generated doodles or environment choices)
- Fall back to windowed mode. Some UAs might decide to retain a more traditional flow and always fallback to windowed / 2D mode on navigation.

## Developer and library maintainers considerations.

The `sessiongranted` event is low friction and agnostic. It doesn't tie or prescribe a specific browser UX allowing experimentation and innovation in the space. Some browsers might decide to offer a traditional windowed workflow that falls back to 2D, others a more immersive experience. Content creators and library maintainers have to consider just two flows: User and UA initiated immersive sessions via user gesture and sessiongranted event respectively.

## Privacy considerations.

### Exposing State About the Client or Previous Page

Varying the behavior based on client state (e.g., is the UA displaying multiple tabs) or the state of the previous page (e.g., whether the previous page had an active immersive session) leaks state about the user and user’s behavior that is not otherwise available to applications.

This may be acceptable for same-origin transitions, since the application could have already known the state. However, this is not acceptable for cross-origin transitions, including history and background documents.

## Same-Origin vs. Cross-Origin Navigation

For navigation scenarios (scenario #1), it may be relevant whether the target URL has the same origin as the existing page. Specifically, since the user has already interacted with the origin, it may be reasonable for implementations to be more permissive in such cases.

However, allowing one but not the other would lead to an [Inconsistent User Experience](https://github.com/immersive-web/webxr/blob/master/designdocs/navigation.md#inconsistent-user-experience).

## References

https://github.com/immersive-web/webxr/blob/master/designdocs/navigation.md

https://github.com/immersive-web/webxr/issues/517

https://github.com/mrdoob/three.js/blob/dev/examples/js/vr/WebVR.js#L176

https://github.com/aframevr/aframe/blob/master/src/utils/device.js#L6

https://github.com/immersive-web/navigation/issues/1