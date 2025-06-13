---
title: "OAuth 2.0 Refresh Token and Consent Expiration"
abbrev: "OAuth RT/Consent Expiration"
category: info

docname: draft-watson-rt-expiration-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: oauth
keyword:
 - oauth
 - refresh token
 - consent
 - token endpoint
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Nicholas Watson
    organization: Google, LLC
    email: nwatson@google.com

normative:
  RFC6749:

informative:


--- abstract

This specification extends OAuth 2.0 [RFC6749] by adding new token endpoint
response parameters to specify refresh token expiration and user consent expiration.


--- middle

# Introduction

RFC6749 defines the OAuth 2.0 protocol, part of which is the ability for a client to receive
a refresh token that may be repeatedly exchanged for
more access tokens. OAuth 2.0 does not contain any normative language around
expiration or lack thereof for refresh tokens, mentioning only that they are
"typically long-lasting".

In the years since the publication of OAuth 2.0, in response to changing security
and privacy landscapes, many authorization servers have
begun to issue shorter-lived refresh tokens for two main reasons:

* The authorization server or user may decide that the access being granted is too
sensitive to allow indefinite access (e.g. mail or health data).
* The authorization server enforces a maximum duration that refresh tokens may be
held without rotation. [OAuth 2.1 Sec 4.3.1]

Clients may wish to implement special handling for expiring refresh tokens. For example,
if the user has granted expiring access, the client may notify the user that they will
need to reauthorize access before a certain date to avoid interruption of service.

# Requirements Notation and Conventions

{::boilerplate bcp14-tagged}

# Terminology

"Resource owner" and "user" are used interchangeably to refer to the entity
capable of granting access to a protected resource.

"Client", "application", and "relying party" are used interchangeably to refer
to the application making protected resource requests on behalf of the
resource owner and with its authorization.

# Concepts

There are two mechanisms that can affect refresh token expiration.

## Consent expiration

When granting consent for an application to access their data, the user may opt
to time-limit that consent, especially if the data is sensitive or they aren't
sure how long they'll continue using the application. The authorization server
itself may also impose mandatory limits on consent duration.

## Refresh token rotation

Authorization servers implementing refresh token rotation may wish to define a
maximum amount of time clients can hold a refresh token without rotating it.
Beyond the security benefit provided by expiring credentials, this also provides
a convenient mechanism for authorization servers to change refresh token keys
without having to accept old credentials forever.

# Refresh token expiration

[OPTION 1]

The refresh token MUST expire no later than the user consent
expires. It MAY expire earlier if the authorization server also enforces a
maximum duration between refresh token rotations.

If the user renews their consent, the authorization server SHOULD
update the expiration time of existing refresh tokens. The authorization server
MUST NOT accept expired refresh tokens, even if it has no way to update the
expiration time of existing refresh tokens.

[OPTION 2]

The refresh token SHOULD expire no later than the user consent
expires. It MAY expire earlier if the authorization server also enforces a
maximum duration between refresh token rotations. Attempting to exchange a refresh
token after user consent has expired MUST result in a failure.

If the user renews their consent, unexpired refresh tokens MUST become usable again.

===

TODO: Which option? See "Security Considerations".

# Token endpoint response

This specification introduces two new response parameters.

## Successful response

    refresh_token_expires_in
          The lifetime in seconds of the refresh token. For example, the value
          "604800" denotes that the refresh token will expire in one week from
          the time the response was generated. [If option 1, this value SHALL
          NOT exceed the value in consent_expires_in.]

    consent_expires_in
          The lifetime in seconds of the user's consent. For example, the value
          "2629800" denotes that the consent will expire in one month from the
          time the response was generated. This value MAY exceed that of
          refresh_token_expires_in.

TODO: use RFC3339 durations?

TODO: How to indicate "valid until revoked"? Omit field, couple with metadata? 0? -1?

## Error response

The existing invalid_grant error code already explicitly covers token expiration
and should be sufficient. Upon receiving this error code the client SHOULD start
a new authorization grant flow.

## Example

Suppose an authorization server enforces that refresh tokens must be rotated at
least once every 7 days, and a user has granted consent to an application for
access for 30 days. Initially, all refresh tokens will be valid for 7 days. An
exchange 7 days after initial authorization will result in the following
response values:

    refresh_token_expires_in: "604800"  // 7 days
    consent_expires_in: "1987200"  // 23 days

An exchange 28 days after initial authorization will result in the following
response values:

    [OPTION 1]
    refresh_token_expires_in: "172800"  // 2 days
    consent_expires_in: "172800"  // 2 days

    [OPTION 2]
    refresh_token_expires_in: "604800"  // 7 days
    consent_expires_in: "172800"  // 2 days

# Update to Authorization Server Metadata

Support for the expiring refresh tokens SHOULD be declared in the
OAuth 2.0 Authorization Server Metadata [RFC8414] with the following
metadata:

    refresh_token_expiration_types
        OPTIONAL. JSON array of supported expiration types. The possible values
        are "consent" and "credential".

TODO: If absence of expiration_time in token endpoint response means that the RT is valid
forever then the AS must declare refresh_token_expiration_types in metadata to
indicate to the client that it's aware of this spec.

# Security Considerations

[OPTION 1]
While it is possible to allow refresh token expiration to exceed
that of user consent expiration if the authorization server checks both
timestamps when validating a refresh token, this is a potentially dangerous
source of bugs in systems with complicated user consent models. By requiring
refresh tokens to expire no later than user consent expires, there is less risk
of bugs that accidentally provide data access to the client beyond the term of
the user's consent.

[OPTION 2]
While it would be less error-prone to require refresh tokens to expire no later
than the user's consent, it may pose an issue for authorization servers that
statelessly encode expiration time into the refresh token, as they'll have no
way to extend the lifetime of these tokens if the user renews their consent. As
long as authorization servers check both expiration timestamps when validating
refresh tokens, it's safe to have longer refresh token expiration.


# IANA Considerations

TODO


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

