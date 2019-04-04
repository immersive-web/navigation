# UA initiated immersive sessions

## Motivation

Current workflow to [request a WebXR session](https://www.w3.org/TR/webxr/#dom-xr-requestsession) requires user gesture activation. 2D Pages usually provide a button that users click or tap to trigger immersive mode.

Some scenarios like in-vr navigation require entering immersive mode automatically without user gesture activation. This necessitates a standard mechanism for UAs to grant immersive sessions.

## Design Goals

The goal of this repository is to discuss and design a simple API for UAs to grant immersive sessions without an explicit user activation in specific scenarios.

## Non Goals

Define how a Virtual Reality or Augmented Reality browser would work [in alignment with the WebXR spec](https://github.com/immersive-web/webxr/blob/master/explainer.md#non-goals)

## User Scenarios

1. Immersive navigation, allowing UAs to implement seamless transitions between interlinked VR and AR content. Use cases include:

    - Games that contains links to other games. Or links to friends' games.
    - Personal blogs that link to affiliated VR sites or your friends blogs.
    - Educational websites for a class, that link to the school's portal or other resources.
    - User generated content linking to one another. Like Rec Room, VR Chat or Super Craft.
    - Sponsored links. You have a VR world about photography and filmmaking that links to Photorama, B&H, GoPro's VR portals.
    - Content portals in general (directories of games, immersive videos, collections).
    - Personal storefronts that link to products hosted by eBay VR, Amazon VR. One could also easily incorporate product 3D models hosted externally in services the like of Sketchfab or Google Blocks.

2. In an AR browser, virtual objects might be fetched from different URLs and rendered in physical context. Content is expected to always present at load time.

3. Open in immersive mode. 2D windowed browsers might offer a way to open a link directly immersive mode, similar to "open in new tab" or "open in new page" actions offered today.

4. Intents, deep linking, PWA, kiosk mode. Native aplications might request the browser to open content directly in immersive mode.

5. Performance implications of single page applications. Having different web pages load is sort of beneficial for not having to deal with offloading / onloading new models and environments or making your own interstitial. You can avoid frame drops by letting the browser handle the transition, and VR web pages load quick enough.

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

In immersive mode the Browser UI is not always rendered and content controls all display real estate. During immersive navigation, UAs must guide and keep the user informed at all times to prevent spoofing and give the user control of displayed content. Possible mechanisms are:

- Interstitial. When navigation occurs the UA can present the user a screen to evaluate the destination URL, an experience description and a button to continue or return to previous site.
- Overlays. UAs control compositing and can always render information about the navigated page over content.
- Modal dialog. User can invoke the browser UI at all times to consult info about the current site. UAs UI can also be modal (similar to Oculus or Steam home spaces) and invoked with a reserved button or interaction that cannot be listened by content. To prevent spoofing, UAs could additionally display personal information that user can recognize and pages don't have access to (user generated doodles or environment choices)
- Fall back to windowed mode. Some UAs might decide to retain a more traditional flow and always fallback to windowed / 2D mode on navigation.

## Developer and library maintainers considerations.

The `sessiongranted` event is low friction and agnostic. It doesn't tie or prescribe a specific browser UX allowing experimentation and innovation in the space. Some browsers might decide to offer a traditional windowed workflow that fallback to 2D, others a more immersive experience. Content creators and library maintainers have to consider just two flows: User and UA initiated immersive sessions via user gesture and sessiongranted event respectively.

## Privacy considerations.

TO-DO

## Same-Origin vs. Cross-Origin Navigation

TO-DO

## References

https://github.com/immersive-web/webxr/blob/master/designdocs/navigation.md

https://github.com/immersive-web/webxr/issues/517

https://github.com/mrdoob/three.js/blob/dev/examples/js/vr/WebVR.js#L176

https://github.com/aframevr/aframe/blob/master/src/utils/device.js#L6

https://github.com/immersive-web/navigation/issues/1