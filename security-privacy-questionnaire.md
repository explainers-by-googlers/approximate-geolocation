# Security and Privacy Questionnaire

## What information does this feature expose, and for what purposes?

This feature allows websites to request approximate, instead of precise,
geolocation via the Geolocation API. It also allows users to grant access to
approximate location only, even if the website requested access to precise
location. This has the purpose of reducing the amount of user data exposed to
websites.

## Do features in your specification expose the minimum amount of information necessary to implement the intended functionality?

Yes.

## Do the features in your specification expose personal information, personally-identifiable information (PII), or information derived from either?

Yes, approximate location.

## How do the features in your specification deal with sensitive information?

Same as the Geolocation API.

## Does data exposed by your specification carry related but distinct information that may not be obvious to users?

It might not be obvious to users what approximate location exactly means and
what its properties are (in fact, this might depend on the platform) and it
might not be clear to users that a website might, joining approximate location
with other information, be able to infer a more precise location.

## Do the features in your specification introduce state that persists across browsing sessions?

Yes, it introduces a new permission `"geolocation-approximate"`.

## Do the features in your specification expose information about the underlying platform to origins?

If the implementation uses the underlying device's functionality to compute an
approximate location, this might expose information about the underlying
platform.

## Do the features in your specification expose information about the underlying platform to origins?

Depending on the implementation, this can result in origin triggering an
approximate location request to the underlying platform.

## Do features in this specification enable access to device sensors?

Yes, to geolocation related sensors, but not any more than the Geolocation API
already does.

## Do features in this specification enable new script execution/loading mechanisms?

No.

## Do features in this specification allow an origin to access other devices?

No.

## Do features in this specification allow an origin some measure of control over a user agent’s native UI?

Nothing else than triggering permission prompts.

## What temporary identifiers do the features in this specification create or expose to the web?

Approximate location.

## How does this specification distinguish between behavior in first-party and third-party contexts?

Approximate location is by default only available in first-party contexts.
Access to third-party contexts is controlled by the `"geolocation"`
policy-controlled feature. An additional policy-controlled feature
`"geolocation-approximate"` allows delegating access to approximate location
only to cross-origin subframes.

## How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?

Nothing special, same as the Geolocation API.

##  Does this specification have both "Security Considerations" and "Privacy Considerations" sections?

This is an incremental change of the Geolocation API, which already has Security
and Privacy Considerations. The privacy considerations are enlarged to warn
about the possibility of reconstructing a more precise location via refinement
attacks.

## Do features in your specification enable origins to downgrade default security protections?

No.

## What happens when a document that uses your feature is kept alive in BFCache (instead of getting destroyed) after navigation, and potentially gets reused on future navigations back to the document?

Same as the Geolocation API.

## What happens when a document that uses your feature gets disconnected?

Same as the Geolocation API.

## Does your spec define when and how new kinds of errors should be raised?

No, there are no new kinds of errors.

## Does your feature allow sites to learn about the user’s use of assistive technology?

No.

## What should this questionnaire have asked?

Nothing else.
