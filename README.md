# Approximate Geolocation Explainer

A proposal to add approximate location to Geolocation API.

## Motivation

Sharing precise location information puts the user's privacy at risk as it can
reveal sensitive information about the user's personal life such as home
address, workplace, or place of worship.
However, users may still wish to share location information to enable a more
localized experience or facilitate a transaction.
Approximate information about the user's location (for instance, a postal code)
has a lower privacy risk and is typically sufficient for most applications.
Extending Geolocation API to support approximate location would empower users to
protect their location privacy and enable sites to request safer defaults when
precise location is not needed.

In some jurisdictions it is illegal to collect precise geolocation data for
specific purposes, where "precise" is defined by a maximum accuracy radius.
Web browsers can facilitate compliance by providing API support for approximate
geolocation data that meets these requirements.

* The [California Privacy Rights Act](https://www.caprivacy.org/cpra-text/#1798.140(w))
defines precise geolocation as having an accuracy radius of 1,850 feet (564
meters) or less.
* The [Connecticut Data Privacy Act](https://www.cga.ct.gov/2022/act/pa/pdf/2022PA-00015-R00SB-00006-PA.pdf)
defines precise geolocation as 1,750 feet (533 meters) or less.
* The [Utah Consumer Privacy Act](https://le.utah.gov/~2022/bills/static/SB0227.html)
defines precise geolocation as 1,750 feet (533 meters) or less.
* The [Virginia Consumer Data Protection Act](https://law.lis.virginia.gov/vacodefull/title59.1/chapter53/)
defines precise geolocation as 1,750 feet (533 meters) or less.

Mobile operating systems already provide user controls for approximate location.

* iOS 14 introduced a Precise Location app setting which may be turned off to
use approximate location.
When precise location is off, the app receives an approximate location estimate
with accuracy between 2 kilometers and 10 kilometers based on the population
density of the user's location.
* On Android 12 and later, users choose to grant approximate or precise location
permissions.
When an app has the approximate location permission and not the precise location
permission, it receives an approximate location estimate with a minimum accuracy
of 2 kilometers.

# Proposal

This proposal introduces the concepts of approximate and precise location data.
For the purpose of this proposal, precise location data is location data with an
accuracy radius less or equal to than some bound, and all other location data is
approximate.
This proposal recommends adopting a bound of at least 2 kilometers.

**New permission.**
A new approximate geolocation permission is introduced and the existing
geolocation permission becomes the precise geolocation permission.
A site is allowed to access geolocation if it has either the approximate or the
precise permission.
A site that does not have the precise location permission must not receive
precise location data.
A corresponding policy-control feature is introduced.

**Generate approximate location data.**
To generate an approximate position estimate the browser should prefer to use
the system's approximate location source if available, but may acquire a precise
position estimate and apply a coarsening algorithm.
The coarsening algorithm must ensure it is not possible to infer the user's
precise location from an approximate position estimate.

As an example of a coarsening algorithm, see [LocationFudger](https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/services/core/java/com/android/server/location/fudger/LocationFudger.java)
which is used by the platform location API on Android.
This algorithm uses two techniques: snap-to-grid and random offsets.
Snap-to-grid creates a many-to-one mapping of precise locations to approximate
locations within a local area.
Random offsets ensure precise location information cannot be inferred when
crossing a grid boundary.

**Request approximate location.**
When calling `getCurrentPosition` or `watchPosition`, the caller may pass a
parameter to request approximate location mode.
If the caller requests approximate location mode it must only receive
approximate location data.

# Example

```js
function getCurrentApproximatePosition() {
  return new Promise((resolve, reject) => {
    if ('accuracyModes' in navigator.geolocation &&
        navigator.geolocation.accuracyModes.includes('approximate')) {
      navigator.geolocation.getCurrentPosition(
          resolve, reject, {accuracyMode: 'approximate'});
    } else {
      reject(new Error('Approximate location not available'));
    }
  };
}
```

# Potential specification changes

## Introduction

The introduction will be updated to introduce the concepts of precise and
approximate location and define the accuracy bound for precise location.

## PositionOptions dictionary

Section 7 defines the [`PositionOptions`](https://www.w3.org/TR/geolocation/#position_options_interface)
dictionary.
Callers of `getCurrentPosition` and `watchPosition` can pass a `PositionOptions`
parameter to modify the position request.

```webidl
dictionary PositionOptions {
  boolean enableHighAccuracy = false;
  [Clamp] unsigned long timeout = 0xFFFFFFFF;
  [Clamp] unsigned long maximumAge = 0;

  // New
  AccuracyMode accuracyMode = "default";
};

enum AccuracyMode {
  // The default accuracy mode.
  "default",

  // Request high accuracy location.
  "high",

  // Require approximate location.
  "approximate"
}
```

`PositionOptions` will be extended to add an `accuracyMode` member.
Callers can pass `"approximate"` to request approximate location or `"high"` to
request high accuracy.
If `accuracyMode` is not set, it defaults to `"default"`.

## Capability detection

Section 6 defines the [`Geolocation`](https://www.w3.org/TR/geolocation/#geolocation_interface)
interface.

```webidl
partial interface Geolocation {
  [SameObject] readonly attribute FrozenArray<AccuracyMode> accuracyModes;
}
```

A site must be able to detect whether the implementation will honor
`accuracyMode`.
A new attribute `accuracyModes` contains a list of supported modes.
If the list is `undefined` or the mode is not in the list, the mode will not be
honored.

`accuracyModes` must always include `"default"` and `"high"`.

If `accuracyModes` includes `"approximate"` (indicating that the implementation
supports approximate accuracy mode) but the implementation is not able to
generate an approximate position estimate then `getCurrentPosition` and
`watchPosition` must return `POSITION_UNAVAILABLE` when the `PositionOptions`
parameter requires approximate location.

## Permissions

Section 3.1 defines a powerful feature `"geolocation"`. This will be updated to
have a custom
[PermissionDescriptor](https://w3c.github.io/permissions/#dom-permissiondescriptor)
with an `accuracy` [aspect](https://w3c.github.io/permissions/#dfn-aspects):

```webidl
dictionary GeolocationPermissionDescriptor : PermissionDescriptor {
  AccuracyMode accuracyMode = "default";
}
```

`"high"` will be [stronger
than](https://w3c.github.io/permissions/#ref-for-dfn-stronger-than-1)
`"default"`, which will be stronger than `"approximate"`, so that if the user
denies access to approximate location then the User Agent denies access to
location at all, while if the user grants access to precise location, also
approximate location is granted.

The additional `"default"` value is needed for backwards compatibility, to
ensure that websites for which the user only granted access to coarse location
can still use `navigator.permissions.query({ name: "geolocation" })` and get
back `"granted"`.

## Permissions policy

Section 11. defines a [policy-controlled
feature](https://www.w3.org/TR/permissions-policy/#policy-controlled-feature)
`"geolocation"`. It will be updated to define an additional policy-controlled
feature `"geolocation-approximate"`, also with a default value of `"self"`, that
will only allow approximate location. The `"geolocation"` feature will imply the
`"geolocation-approximate"` feature.

## Request a position

In Section 6.5, the [request a
position](https://www.w3.org/TR/geolocation/#dfn-request-a-position) algorithm
requests permission to use `"geolocation"`. It will be updated to allow for
three different prompt cases:

1. If the website doesn't set `accuracyMode="approximate"`, the user will be
   prompted to choose between approximate and precise location.
2. If the website sets `accuracyMode="approximate"`, the user will only be
   prompted for approximate location.
3. If the website has already been granted approximate location (as result of a
   request with `accuracyMode="approximate"`) and now requests precise
   location, the user will be prompted to upgrade their choice from approximate
   to precise location.

## Acquire a position

In Section 6.6, the [acquire a position](https://www.w3.org/TR/geolocation/#dfn-acquire-a-position)
algorithm says "try to acquire position data from the underlying system,
optionally taking into consideration the value of `options.enableHighAccuracy`
during acquisition".
It will be updated to also consider the location accuracy mode.
Additionally, the algorithm will be updated to handle acquired position
estimates that do not satisfy the accuracy bound.

# Alternatives considered

## Reusing `enableHighAccuracy`

A boolean option `enableHighAccuracy` is already specified and its default value
is `false`.
As an alternative to approximate location as an opt-in mode, provide approximate
location by default and only provide precise location when high accuracy is
requested.

This was rejected because changing the behavior would degrade location quality
for existing applications.
`enableHighAccuracy` is specified as a hint that may be ignored by the
implementation.
Historically, the high accuracy hint indicates the implementation should prefer
accuracy over speed; specifically, if the system has GPS capabilities it should
wait for a GPS fix instead of returning a less precise result.
This tradeoff is not necessary on modern systems as system location providers
are able to generate a precise estimate quickly without waiting for GPS.
As a result, the behavior on most systems is not affected by the
`enableHighAccuracy` option and many applications use the default value
regardless of whether high accuracy is needed.

We could consider treating `enableHighAccuracy=true` the same as
`accuracyMode="high"`. However, that would have a small backwards
compatibility issue with `navigator.permissions.query({ name: "geolocation" })`,
since it could result in different prompting behaviours (while querying the
permission state would only return one state).

## A separate `geolocation-approximate` powerful feature

Instead of customizing `geolocation`'s PermissionDescriptor, we could add a
separate `geolocation-approximate` powerful feature. However, this was rejected
because of a backwards compatibility problem, since
`navigator.permissions.query({ name: "geolocation" })` must return `"granted"`
even if the user granted approximate geolocation only.
