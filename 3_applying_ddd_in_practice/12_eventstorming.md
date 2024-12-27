# Eventstorming  

![event-storming](https://github.com/user-attachments/assets/c38c1aa6-04de-45b2-9946-75047741b5e8)  

An eventstorming session is a tool to draft a model describing a business domain process. It helps to share domain knowlegde and align mental models of the business, and to ultimately make better technical design decisions by identifying domain events, commands, aggregates and eventually bounded contexts.  

1. **unsctructured exploration** - we write down any domain events related to the business domain being explored that comes to mind
2. **timelines** - organize the events in chronological order starting with the happy path and expand from it 
3. **pain points** - write any bottleneck, manual step requiring automation or missing domain knowledge
4. **pivotal events** - identify significant events indicating change in context or phase
5. **commands** - write down the system operations and related actors that triggered the domain events or flow of events
6. **policies** - commands with no actor will be identified as *automation policies*
7. **read models** - identify the view of data the actors used to make a decision to execute a command 
8. **external systems** - identify systems that are outside the exploration domain, they either execute commands (input) or be notified about events (output)
9. **aggregates** - start identifying aggregates, they receive commands and produce events
10. **bounded contexts** - finally look for related aggregates which may form natural candidates for bounded context boundaries

