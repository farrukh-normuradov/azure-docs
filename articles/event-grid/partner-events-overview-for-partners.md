---
title: Partner Events overview for system owners who desire to become partners
description: Provides an overview of the concepts and general steps to become a partner. 
ms.topic: conceptual
ms.date: 04/28/2022
---

# Partner Events overview for partners - Azure Event Grid (preview)
Event Grid's **Partner Events** allows customers to **subscribe to events** that originate in a registered system using the same mechanism they would use for any other event source on Azure, such as an Azure service. Those registered systems integrate with Event Grid are known as "partners". This feature also enables customers to **send events** to partner systems that support receiving and routing events to customer's solutions/endpoints in their platform. Typically, partners are software-as-a-service (SaaS) or [ERP](https://en.wikipedia.org/wiki/Enterprise_resource_planning) providers, but they might be corporate platforms wishing to make their events available to internal teams. They purposely integrate with Event Grid to realize end-to-end customer use cases that end on Azure (customers subscribe to events sent by partner) or end on a partner system (customers subscribe to Microsoft events sent by Azure Event Grid). Customers bank on Azure Event Grid to send events published by a partner to supported destinations such as webhooks, Azure Functions, Azure Event Hubs, or Azure Service Bus, to name a few. Customers also rely on Azure Event Grid to route events that originate in Microsoft services, such as Azure Storage, Outlook, Teams, or Azure AD, to partner systems where customer's solutions can react to them. With Partner Events, customers can build event-driven solutions across platforms and network boundaries to receive or send events reliably, securely and at a scale.

> [!NOTE]
> This is a conceptual article that's required reading before you decide to onboard as a partner to Azure Event Grid. For step-by-step instructions on how to onboard as an Event Grid partner using the Azure portal, see [How to onboard as an Event Grid partner (Azure portal)](onboard-partner.md). 

## Partner Events: How it works

As a partner, you create Event Grid resources that enable to you publish events to Azure Event Grid so that customers on Azure can subscribe to them. For most partners, for example SaaS providers, it's the only integration capability that they'll use.

You can also create Event Grid resources to receive events from Azure Event Grid. This use case is for those organizations that own or manage a platform that enables their customers to receive events by exposing endpoints. Some of those organizations are ERP systems that also have event routing capabilities within their platform, which sends the incoming Azure events to a customer application hosted on their platform. 

For either publishing events or receiving events, you create the same kind of Event Grid [resources](#resources-managed-by-partners) following these general steps. 

1. Communicate your interest in becoming a partner by sending an email to [GridPartner@microsoft.com](mailto:GridPartner@microsoft.com). Once you contact us, we'll guide you through the onboarding process and help your service get an entry card on our [Azure Event Grid gallery](https://portal.azure.com/#create/Microsoft.EventGridPartnerTopic) so that your service can be found on the Azure portal. 
2. Create a [partner registration](#partner-registration). This is a global resource and you usually need to create once.
3. Create a [partner namespace](#partner-namespace). This resource exposes an endpoint to which you can publish events to Azure. When creating the partner namespace, provide the partner registration you created. 
4. Customer authorizes you to create a partner resource, either a [partner topic](concepts.md#partner-topics) or a [partner destination](concepts.md#partner-destination), in customer's Azure subscription. 
5. Customer accesses your web page or executes a command, you define the user experience, to request either the flow of your events to Azure or the ability to receive Microsoft events into your system. In response to that request, you set up your system to do so with input from the customer. For example, the customer may have the option to select certain events from your system that should be forwarded to Azure.
6. According to customer's requirements, you create a partner topic or a partner destination under the customer's Azure subscription, resource group and with the name the customer provides to you. It's achieved by using channels. Create a [channel](#channel) of type `partner topic`, if the customer wants to receive your events on Azure, or `partner destination` if the customer wants to send events to your system. Channels are resources contained by partner namespaces.
7. Customer activates the partner topic or the partner destination that you created in their Azure subscription and resource group.
8. If you created a partner topic, start publishing events to your partner namespace. If you created a partner destination, expect events coming to your system endpoints defined in the partner definition.

    >[!NOTE]
    > You must [register the Azure Event Grid resource provider](subscribe-to-partner-events.md#register-the-event-grid-resource-provider) to every Azure subscription where you want create Event Grid resources. Otherwise, operations to create resources will fail.


## Why should I use Partner Events?
You may want to use the Partner Events feature if you've one or more of the following requirements.

### For partners as event publishers

- You want a mechanism to make your events available to your customers on Azure. Your users can filter and route those events by using partner topics and event subscriptions they own and manage. You could use other integration approaches such as [topics](custom-topics.md) and [domains](event-domains.md). However, those approaches wouldn't allow for a clean separation of resource ownership, management, and billing between you and your customer. The Partner Events feature also provides a more intuitive user experience that makes it easy to discover your service.
- You need a simple multi-tenant model where you publish events to a single regional endpoint, the namespace’s endpoint, to route the events to different customers.  
- You want to have visibility into metrics related to published events.
- You want to use [Cloud Events 1.0](https://cloudevents.io/) schema for your events.

### For partners as a subscriber

- You want your service to react to customer events that originate in Microsoft/Azure.
- You want your customer to react to Microsoft/Azure service events using their applications hosted by your platform. You use your platform's event routing capabilities to deliver events to the right customer solution.
- You want a simple model where your customers just select your service name as a destination without the need for them to know technical details like your platform endpoints.
- Your system/platform supports [Cloud Events 1.0](https://cloudevents.io/) schema.

## Resources managed by partners
As a partner, you manage the following types of resources.

### Partner registration
A registration holds general information related to a partner. A registration is required when creating a partner namespace. That is, you must have a partner registration to create the necessary Azure resources to integrate with Azure Event Grid. 

Registrations are global. That is, they aren't associated with a particular Azure region. You may create a single partner registration and use that when creating your partner namespaces.
  
### Channel
A Channel is a nested resource to a Partner Namespace. A channel has two main purposes:
  - It's the resource type that allows you to create partner resources on a customer's Azure subscription.  When you create a channel of type `partner topic`, a partner topic is created on a customer's Azure subscription. A partner topic is the customer's resource where events from a partner system. Similarly, when a channel of type `partner destination` is created, a partner destination is created on a customer's Azure subscription. Partner destinations are resources that represent a partner system endpoint to where events are delivered. A channel is the kind of resource, along with partner topics and partner destinations, that enable bi-directional event integration.
   
      A channel has the same lifecycle as its associated customer partner topic or destination. When a channel of type `partner topic` is deleted, for example, the associated customer's partner topic is deleted. Similarly, if the partner topic is deleted by the customer, the associated channel on your Azure subscription is deleted.
  - It's a resource that is used to route events. A channel of type ``partner topic`` is used to route events to a customer's partner topic. It supports two types of routing modes. 
      - **Channel name routing**. With this kind of routing, you publish events using an http header called `aeg-channel-name` where you provide the name of the channel to which events should be routed. As channels are a partner's representation of partner topics, the events routed to the channel show on the customer's parter topic. This kind of routing is a new capability not present in `event channels`, which support only source-based routing. Channel name routing enables more use cases than the source-based routing and it's the recommended routing mode to choose. For example, with channel name routing a customer can request events that originate in different event sources to land on a single partner topic.
      - **Source-based routing**. This routing approach is based on the value of the `source` context attribute in the event. Sources are mapped to channels and when an event comes with a source, say, of value "A" that event is routed to the partner topic associated to the channel that contains "A" in its source property.

      A channel of type ``partner destination`` is used to route events to a partner system. When creating a channel of this type, you provide your webhook URL where you receive the events published by Azure Event Grid. Once the channel is created, a customer can use the partner destination resource when creating an [event subscription](subscribe-through-portal.md) as the destination to deliver events to the partner system. Event Grid publishes events with the request including an http header `aeg-channel-name` too. Its value can be used to associate the incoming events with a specific user who in the first place requested the partner destination.
 
      A customer can use your partner destination to send your service any kind of events available to [Event Grid](overview.md).

### Partner namespace
A partner namespace is a regional resource that has an endpoint to publish events to Azure Event Grid. Partner namespaces contain either channels or event channels (legacy resource). You must create partner namespaces in regions where customers request partner topics or destinations because channels and their corresponding partner resources must reside in the same region. You can't have a channel in a given region with its related partner topic, for example, located in a different region. 

Partner namespaces contain either channels or event Channels. It's determined by the property **partner topic routing mode** in the namespace. If it's set to **Channel name header**, channels are the only type of resource that can be created under the namespace. If partner topic routing mode is set to **Source attribute in event**, then the namespace can only contain event channels. Mind that the decision of setting the right ``partner topic routing mode`` isn't a decision between choosing channel name or source-based routing. Channels support both. It's rather a decision between using the new type of routing resource, the channels, versus using a legacy resource, the event channels.

### Event channel

An Event channel is the resource that was first released with Partner Events to route incoming events to partner topics. Event channels only support source-based routing and they always represent a customer partner topic. 

>[!IMPORTANT]
>Event channels are being deprecated. Hence, it is advised that you use Channels.

## Verified partners

A verified partner is a partner organization whose identity has been validated by Microsoft. It's strongly encouraged that your organization gets verified. Customers seek to engage with partners that have been verified as such verification provides greater assurances that they're dealing with a legitimate organization. Once verified, you benefit from having a presence on the [Event Grid Gallery](https://portal.azure.com/#create/Microsoft.EventGridPartnerTopic) where customers can discover your service easily and have a first-party experience when subscribing to your events, for example.

## Customer's authorization to create partner topics and partner destinations

Customers authorize you to create partner topics or partner destinations on their Azure subscription. The authorization is granted for a given resource group in a customer Azure subscription and it's time bound. You must create the channel before the expiration date set by the customer. You should have documentation suggesting the customer an adequate window of time for configuring your system to send or receive events and to create the channel before the authorization expires. If you attempt to create a channel without authorization or after it has expired, the channel creation will fail and no resource will be created on the customer's Azure subscription. 

> [!NOTE]
> Event Grid will start **requiring authorizations to create partner topics or partner destinations** around June 15th, 2022. You should update your documentation asking your customers to grant you the authorization before you attempt to create a channel or an event channel.

>[!IMPORTANT]
> **A verified partner is not an authorized partner**. Even if a partner has been vetted by Microsoft, you still need to be authorized before you can create a partner topic or partner destination on the customer's Azure subscription. 

## Partner topic and partner destination activation

Customer activates the partner topic or destination you've created for them. At that point, the channel's activation status changes to **Activated**. Once a channel is activated, you can start publishing events to the partner namespace endpoint that contains the channel. 

### How do you automate the process to know when you can start publishing events for a given partner topic?

You have two options:
- Read (poll) the channel state periodically to check if the activation status has transitioned from **NeverActivated** to **Activated**. This operation can be computationally intensive.
- Create an [event subscription](subscribe-through-portal.md)  for the [Azure subscription](event-schema-subscriptions.md#available-event-types) or [resource group](event-schema-resource-groups.md#available-event-types) that contains the channel(s) you want to monitor. You'll receive `Microsoft.Resources.ResourceWriteSuccess` events whenever a channel is updated. You'll then need to read the state of the channel with the Azure Resource Manager ID provided in the event to ascertain that the update is related to a change in the activation status to **Activated**.

## References

  * [Swagger](https://github.com/ahamad-MS/azure-rest-api-specs/blob/master/specification/eventgrid/resource-manager/Microsoft.EventGrid/preview/2020-04-01-preview/EventGrid.json)
  * [ARM template](/azure/templates/microsoft.eventgrid/allversions)
  * [ARM template schema](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2020-04-01-preview/Microsoft.EventGrid.json)
  * [REST APIs](/azure/templates/microsoft.eventgrid/2020-04-01-preview/partnernamespaces)
  * [CLI extension](/cli/azure/eventgrid)

### SDKs
  * [.NET](https://www.nuget.org/packages/Microsoft.Azure.Management.EventGrid/5.3.1-preview)
  * [Python](https://pypi.org/project/azure-mgmt-eventgrid/3.0.0rc6/)
  * [Java](https://search.maven.org/artifact/com.microsoft.azure.eventgrid.v2020_04_01_preview/azure-mgmt-eventgrid/1.0.0-beta-3/jar)
  * [Ruby](https://rubygems.org/gems/azure_mgmt_event_grid/versions/0.19.0)
  * [JS](https://www.npmjs.com/package/@azure/arm-eventgrid/v/7.0.0)
  * [Go](https://github.com/Azure/azure-sdk-for-go)


## Next steps
- [How to onboard as an Event Grid partner (Azure portal)](onboard-partner.md)
- [Partner topics onboarding form](https://aka.ms/gridpartnerform)
- [Partner topics overview](partner-events-overview.md)
- [Auth0 partner topic](auth0-overview.md)
- [How to use the Auth0 partner topic](auth0-how-to.md)