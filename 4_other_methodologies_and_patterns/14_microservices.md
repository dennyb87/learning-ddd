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

# Microservices' boundaries  
### Bounded contexts  

```mermaid
flowchart TD
    subgraph ctx2[bounded context v2]
        subgraph support
            tel2[telephony]
            bill2[billing]
        end
        subgraph s2[sales]
            lead3[lead]
            crm2[CRM]
        end
        subgraph m2[marketing]
            opt2[optimization]
            lead4[lead]
        end
    end
    subgraph ctx1[bounded context v1]
        subgraph s1[sales]
            bill1[billing]
            lead2[lead]
            crm1[CRM]
        end
        subgraph m1[marketing]
            opt1[optimization]
            tel1[telephony]
            lead1[lead]
        end
    end
```

Both bounded contexts decompositions in the diagram are valid, but doesn't make them valid microservices, specially considering their wide functionalities. So although microservices are bounded contexts, not every bounded context is a valid microservice. The relationship between the two is not symmetrical.  

![decomposition_safe_area](https://github.com/user-attachments/assets/b16a7bcf-9168-49b7-83fd-e3e45735298b)  

Here we can observe a safe decomposition area, past the bounded context or microservices' threshold, it will result in a *BBoM* or a *distributed BBoM*.  

### Aggregates  

While a bounded context impose a limit on the widest valid boundary, the aggregate's boundary represents the narrowest boundary possible. Usually if the aggregate has a strong relationship with other business entities of its subdomain it will result in a shallow individual service.  

### Subdomains as a service  

Therefore a more balanced heuristic for designing microservices is to align the its boundary with the subdomain. The coherent nature of the usecase contained in a subdomain ensures the module's depth making it a safe heuristic that produces optimal solutions for the majority of microservices.  

![subdomain_as_services](https://github.com/user-attachments/assets/7a8c6f9d-e333-4ce4-b38c-7b80260ca9b4)  

Finally to compress microservices interfaces and making them deeper we can introduce either the **open-host service** or the **anticorruption layer** pattern. These will expose a restrained model designed around intergation needs, encapsulating implementation details and reducing global complexity.  
