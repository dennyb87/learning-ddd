Here we'll explore common application architecture patterns: layered architecture, ports & adapters, and CQRS.  

# Layered architecture  

This pattern organises the codebase into three main layers:  

1. **presentation layer** - where the interaction with consumers happens, the public interface
2. **business logic layer** - where the business logic lives, the heart of software
3. **data access layer** - the layer providing access to any persistence mechanisms

```mermaid
flowchart TD
    subgraph dal["Data access layer"]
        database
        message-bus
        object-storage
    end
    subgraph bll["Business logic layer"]
        entities
        rules
        processes
    end
    subgraph pl["presentation layer"]
        Web-UI
        CLI
        REST-API
    end
    pl-->bll-->dal
```

The communication happens in a top-down fashion, where for example the presentation layer references only the business logic layer and has no knowledge of the data access layer.  

## Service layer  

The pattern often comes with an extra *service layer* acting as a facade for the business logic layer. It exposes interfaces that match with public interface methods and implement the logic required for the layer underneath.  

* it can be reused for multiple public interfaces
* improves modularity
* decouples presentation and business logic layer
* ease testing

```python
class CampaignManagementService:
    ...
    @classmethod
    def create_campaign(cls, campaign_details):
        ...
    @classmethod
    def publish(cls, campaign, publishing_schedule):
        ...
    @classmethod
    def deactivate(cls, campaign):
        ...
```

```mermaid
flowchart TD
    subgraph dal["Data access layer"]
        database
        message-bus
        object-storage
    end
    subgraph bll["Business logic layer"]
        entities
        rules
        processes
    end
    subgraph sl["Service layer"]
        action-x
        action-y
        action-z
    end
    subgraph pl["Presentation layer"]
        Web-UI
        CLI
        REST-API
    end
    pl-->sl-->bll-->dal
```

# Port & Adapters  

![hexagonal_architecture](https://github.com/user-attachments/assets/d88cd221-6928-481a-83a9-3dd37d6bd38f)  

Also known as **hexagonal architecture** or **onion architecture**, this pattern places the business logic at the core. The outermost layer is the infrastructure layer, which contain integration with external components such user interface, databases etc.. and it implements **adapters** which will connect to the **ports** exposes by the bussiness logic layer, the innermost layer.  

```mermaid
flowchart TD
    subgraph bll["Business logic layer"]
        entities
        rules
        processes
    end
    subgraph il["Infrastructure layer"]
        database
        UI-framework
        message-bus
    end
    il-->bll-->il
```

An application layer could be placed in between acting as a facade for the business logic just like the service layer in the layered architecture.  
