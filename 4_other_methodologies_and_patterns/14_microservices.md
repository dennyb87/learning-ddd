# Microservices  

A **service** is a mechanism that enables access to one or more capabilities via a prescribed interface, where the interface is any mechanism for getting data in or out of the service. A well expressed interface should be enough to describe the service functionality.  

```mermaid
erDiagram
    CampaignPublishingService {
        publish(campaign_id) publishing_result
        pause(campaign_id) confirmation
        get_statistics(campaign_id) publishing_statistics
    }
```

Since a service is defined by its public interface, a **microservice** is a *service with a micro public interface*. This simplistic definition could make you think that limiting service interfaces to a single method would make it a perfect microservice. Even though each microservice would end up being much simpler reducing **local complexity**, the integration complexity between the resulting number of microservices would increase creating **global complexity**. So one needs to find the sweet spot, the global optima which balances both complexities.  

![local_global_complexity](https://github.com/user-attachments/assets/58535857-425d-49b3-8c10-4e71eb1a3656)

## Microservices as deep modules  

While microservices are strictly physical, modules can denote both logic and physical boundaries. Aside from that both concepts and their underlying design principles are the same. [A Philosophy of Software Design](https://blog.pragmaticengineer.com/a-philosophy-of-software-design-review/) proposes a simple yet powerful visual heuristic for evaluating a module's design: **depth**.  

![deep_module](https://github.com/user-attachments/assets/00d358a3-8ef8-4951-9b8c-3d3fd2f97aa3)  

According to this model effective modules (just like microservices) are deep: a simple interface encapsulating complex logic. If we decompose a monolith into services the cost of introducing a change goes down, and minimized when **decomposed into microservices**. However past the microservice threshold, the services will become more and more shallow, and as the interfaces increase so does the cost of integration.  

> **BBoM** stand for *Big Balls of Mud*

![granularity_and_cost_of_change](https://github.com/user-attachments/assets/714cf904-58e2-4fc7-aabd-8e9f424c5033)  

