---
weight: 10
title: Overview
layout: redirect
---

The new Notifications 2.0 API allows applications or microservices to receive and process notifications generated by the use of the {{< product-c8y-iot >}} REST or MQTT API (device management and other platform API) in a reliable manner.
Currently, device management notifications equivalent to what is covered by realtime notifications are available, but delivered with at-least-once semantics over a new protocol.
In the future, further platform events will be included as 2.0 notifications.

This new notification API supersedes realtime notifications, as it provides for stronger delivery semantics and ordering.
It is also intended to be simpler to use than the Bayeaux protocol used with realtime notifications.

In the {{< product-c8y-iot >}} 10.11 release, Notifications 2.0 needs to be enabled in the platform as it depends on the new Messaging Service that currently is optional.
See the *Messaging Service - Installation & operations guide* on how to do that.
Ingress via load balancers also needs to allow ingress for the new WebSocket endpoint that is enabled by default in {{< product-c8y-iot >}} 10.11, which might affect custom deployments that manage their own ingress.

To receive notifications over the 2.0 protocol, an application or microservice must subscribe to notifications, either for notifications about a particular managed object or in a wider context that is scoped to a tenant.

For managed objects, an object's global identifier must be used to subscribe in the managed object ("mo") context in order to receive notifications.
There can be multiple subscriptions for the same subscribed object, with different filters or just to fan out to multiple interested consumer parties.
This subscribed object is known as the source object for the `notification/event` and it is referenced in the notifications delivered to subscribers.
Subscriptions are set up using the [subscription method](https://{{<domain-c8y>}}/api/{{< c8y-current-version >}}/#section/#subscriptions) of the Notifications 2.0 API.
This API requires the calling user to be an authenticated {{< product-c8y-iot >}} user and to have the new ROLE_NOTIFICATION_2_ADMIN role.

When subscribing to notifications, a filter for notifications can be specified which determines which APIs (alarms, events, measurements, managed objects or any combination of these) to filter by.
It is also possible to filter by presence of a specific JSON fragment or "fragment type".
When matched, either the whole notification content is forwarded or one or more fragments can be specified to be copied to the consumer.

In order to receive subscribed notifications, a consumer application or microservice must obtain an authorization token that provides proof that the holder is allowed to receive subscribed notifications.
This token is in the form of a string conforming to the JWT (JSON Web Token) standard that is obtained from the [token method](https://{{<domain-c8y>}}/api/{{< c8y-current-version >}}/#section/#tokens) of the Notifications 2.0 API.
This API requires the calling user to be an authenticated {{< product-c8y-iot >}} user and to have the new ROLE_NOTIFICATION_2_ADMIN role.

See the [{{< openapi >}}](https://{{<domain-c8y>}}/api/{{< c8y-current-version >}}/#section/#tokens) for both the Notification 2.0 Subscription and Token API.

Once subscribed, notifications are persisted and available to be consumed using a new WebSocket-based protocol.
This protocol implements a reliable delivery with at-least-once semantics.
The underlying Messaging Service will repeatedly attempt to deliver a notification until that notification is acknowledged as being received and processed by the consuming application or microservice.
Notification order is preserved from the point of view of a device sending in REST and MQTT API requests.
The protocol is text-based and described in detail in the next section.

An additional context for subscribing to and receiving notifications, in addition to the managed object context ("mo") for events related to known managed objects, is the tenant context ("tenant").
Creations of managed objects, which generate a new object identifier that can act as a source for notifications are reported in the tenant context.
This allows an application to discover a new managed object which it can then choose to subscribe to in the managed object context.
It is also possible to subscribe to all alarms that are generated in the tenant context.
See the [{{< openapi >}}](https://{{<domain-c8y>}}/api/{{< c8y-current-version >}}/#section/#subscriptions) on how to subscribe to these notifications, additionally filtering the notification of interest.
For the protocol consumer, both managed object creations and alarms subscribed under the tenant context are reported in the same way.
There is no distinction between the two contexts for consumers and notification ordering is maintained between the two contexts.

For Java developers, the API and the protocol have been wrapped up as an open Java API and a sample WebSocket client application.
There is a sample Java microservice available in the [cumulocity-examples repository](https://github.com/SoftwareAG/cumulocity-examples/tree/develop/hello-world-notification-microservice) so developers do not need to code to the following protocol specification directly.