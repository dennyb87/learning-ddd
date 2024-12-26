## Model translation  

A bounded context is the boundary of a model, a ubiquitous language. If in a communication across bounded context one of the two cannot conform, a model translation is required e.g. anticorruption layer, or open-host service. Model translation can be either *stateless*, which happens on the fly, or *stateful*, a more complex translation requiring a database.  

# Stateless model translation  
### Synchronous  

In a synchronous stateless translation the bounded context which owns the translation implements a proxy.  

```mermaid
flowchart LR
    bc1["bounded context 1"]--"request"-->proxy
    proxy--"transformed response"-->bc1
    proxy--"transformed request"-->bc2["bounded context 2"]
    bc2--"response"-->proxy
```

These proxy, which sometimes are offloaded to external components such an *API gateway*, are integration specific bounded contexts and are mainly in charge to transform models for consumption. These are often called *interchange contexts*.  

### Asynchronous  

Similarly, in an asynchronous communication, we implement a *message proxy*. This is essential for open-host service, where you don't want to leak bounded context implementation but rather convert domain events to a published language.  

```mermaid  
flowchart LR
    subgraph Upstream context
        aggregate-->de@{ shape: das, label: "domain events" }-->ohs["open-host service"]
    end
    ohs-->pl@{ shape: das, label: "published language" }
    pl-->downstream
```

# Stateful model translation  

A use case is combining multiple fine-grained messages into a single message. To track incoming data and process it, the translation logic requires its own persistent logic.  

```mermaid
flowchart LR
    req1@{ shape: notch-rect, label: "request 1" }
    req2@{ shape: notch-rect, label: "request 2" }
    req3@{ shape: notch-rect, label: "request 3" }
    input@{ shape: das, label: "input" }
    req1-->input
    req2-->input
    req3-->input
    aggregator["proxy/aggregator"]
    input-->aggregator
    aggregator-->db[(storage)]
    aggregator-->batch

    subgraph batch
        req-1@{ shape: notch-rect, label: "request 1" }
        req-2@{ shape: notch-rect, label: "request 2" }
        req-3@{ shape: notch-rect, label: "request 3" }
    end

    batch-->output@{ shape: das, label: "output" }
```


# Integrating aggregates  

One way for aggregates to communicate with the rest of the system is via domain events. A common mistake is to publishing the event from the aggregate it self.  

```python
class Campaign:
    ...
    def deactivate(self):
        self.is_active = False
        event = CampaignDeactivated(self.uid)
        self.events.append(event)
        self.message_bus.publish(event)
```

The event will be dispatched before the aggregate's new state is committed. A subscriber would receive the notification that the campaign is deactivated contradicting the campaign's state.  

## Outbox  

This pattern ansure reliable publishing of events as follows:  

* aggregate's state and events are committed in the same transaction
* a message relay fetches events from the database
* the message relay publishes the events to a message bus
* the relay marks the events as published or deletes them

```mermaid
flowchart LR
    application-->transaction
    subgraph transaction
        as@{ shape: doc, label: "aggregate state"}
        ot@{ shape: doc, label: "outbox table"}
    end

    transaction-->db[(database)]
    
    mr[message relay]--"query unpublished events"-->db
    mr--"publish"-->mb@{ shape: das, label: "message bus" }
```

The message relay can fetch unpublished events in a **pull** manner (*polling publisher*), so quering unpublished events directly, or in a **push** fashion (*transaction log tailing*) where the relay it's called directly by a database's insert or update hook.  

# Saga  

It's business process that spans multiple aggregates. It listens to events emmitted by components and issues relevant commands accordingly to other components. It's responsible to issue compensating actions if any of the steps fails. Let's examine a scenario where when a new campaign is activated the ad material gets sent to the publisher, which can either accept and publish or reject the material.  

```mermaid
flowchart LR
    ca[campaign aggregate]--"CampaignActivated"-->saga[campaign publishing saga]
    saga--"track confirmation/rejection"-->ca

    saga--"SubmitAd"-->ad-publishing
    ad-publishing--"PublishingConfirmed/Rejected"-->saga
```

```python
class CampaignPublishingSaga:
    ...
    @classmethod
    def campaign_activated(cls, event: "CampaignActivated"):
        campaign = event.campaign
        ad_material = campaign.generate_ad_material()
        PublishingService.submit_ad(campaign.uid, ad_material)

    @classmethod
    def publishing_confirmed(cls, event: "PublishingConfirmed"):
        campaign = event.campaign
        campaign.track_confirmation(event)
        CampaignRepo.persist(campaing)
    
    @classmethod
    def publishing_rejected(cls, event: "PublishingRejected"):
        campaign = event.campaign
        campaign.track_rejection(event)
        CampaignRepo.persist(campaign)
```

This is a stateless saga, but in cases where state management is required we can implement saga as an event-sourced aggregate.  

# Process manager  

While our saga implementation was simply matching events to commands. The process manager is a more complex process that maintains the state of a sequence and determines the next step.  

```mermaid
flowchart LR
    node((" "))
    application--"start new booking process"-->pm["trip booking<br>(process manager)"]<--->node
    pm<-->db[(state)]
    node-->fr[flight routing]
    node-->fb[flight booking]
    node-->hb[hotel booking]
    node-->at[approval tracking]
```

Booking a business trip starts with choosing a flight route, where the employee needs to approve it. When the flight is booked, it's time to book the hotel, and if none is available for the required dates the flight tickets have to be cancelled and so on...  
