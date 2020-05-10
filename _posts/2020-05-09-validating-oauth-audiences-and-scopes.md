---
layout: post
title: "Why You Should Prefer OAuth Scope Validation Over Audience Validation"
author: "Joel Braun"
categories: journal
tags: [OAuth, Audiences, Scopes]
---

I often receive the question of how OAuth token audience validation should work in environments where multiple OAuth clients are calling multiple resource services. Generally, the audience or `aud` claim in OAuth represents the application to which the OAuth token was issued. This can be handy as an additional layer of token validation for certain types of applications (say you have a single, monolithic application architecture with no additional services). However, validating the audience claim quickly starts to increase in complexity when multiple clients and services enter the mix.

![audience validation](/assets/img/2020-05-09-validating-oauth-audiences-and-scopes/audience.svg "Audience validation diagram")


As the number of applications in a service ecosystem scale, requiring audience validation forces all applications to track and validate against all audiences, often individual clients, which have been introduced. As new OAuth clients (with their new audiences) are created, this rapidly becomes an unwieldy task. One erroneous approach to correct this issue is to [try issuing multiple audiences in tokens](https://github.com/IdentityServer/IdentityServer3/issues/1365).

There's a better alternative, which is to simply rely on scope validation. Rather than requiring each application accepting OAuth tokens to maintain knowledge of all possible clients, you can validate `scope` in a token instead. The `scope` array of values within a token represent the resources to which the token has access. This effectively inverts the dependency - rather than resources having to know what will call them, the IdP pre-determines which resources are allowed.

![scope validation](/assets/img/2020-05-09-validating-oauth-audiences-and-scopes/scope.svg "Scope validation diagram")

In practice, this proves to be a much better model as new client applications and resource services are added. Additionally, the list of scopes allowed for each client provides a effective form of 'documentation' for determining the resources upon which a particular client depends.

One of the common sources of this confusion is Microsoft's own JWT authorization libraries, both for ASP.NET and ASP.NET Core. Rather than providing scope validation by default, they provide audience validation. [A separate piece of middleware](https://github.com/IdentityServer/IdentityServer4.AccessTokenValidation/blob/master/src/AuthorizationPolicyExtensions.cs) is required to implement scope validation (but highly recommended!). 

Interestingly, the [OAuth 2 Authorization Framework RFC](https://tools.ietf.org/html/rfc6749) only mentions audience validation as a security mechanism for applications using less secure flows, such as implicit. All other references in the document use the term scope to describe the resource authorization property of an access token. 