# OLTP vs OLAP  

The *OnLine Transactional Processing Data* (**OLTP**) are operational models are built around busines domain entities, implementing their lifecycles and orchestrating their interactions, they serve operational systems, hence are optimized for real-time business transactions.  

The *OnLine Analytical Processing Data* (**OLAP**) are analytical models designed to provide insights into the system, the performance of business activities and how operations can be optimized to achieve a greater value. This pattern focuses on business activities by modelling **fact tables** and **dimension tables**.  

## Fact tables  

| id  | case_id | timestamp           | agent | customer | status |
| --- | ------- | ------------------- | ----- | -------- | ------ |
| 1   | 13      | 2024-09-01T10:00:00 | 1     | 345      | 1      |
| 3   | 13      | 2024-09-01T10:30:00 | 1     | 345      | 1      |
| 7   | 13      | 2024-09-01T11:00:00 | 1     | 345      | 2      |

Fact tables represent business events that have already happened. Fact records are never deleted or modified (append only). While *OLTP* is concerned with precise data to handle business transactions, *OLAP* is more interesed in aggregate data. The analysts decides the appropriate level of granularity (see *Fact_CaseStatus* has a snapshot every 30 minutes).  

## Dimension tables  

```mermaid
erDiagram
    Fact_SolvedCases {
        foreign_key agent
        foreign_key customer
        datetime opened
        datetime closed
    }
    Dim_Customers {
        string first_name
        string last_name
    }
    Dim_Agents {
        string name
        foreign_key team
        foreign_key role
    }
    Dim_Customers ||--|{Fact_SolvedCases: ""
    Dim_Agents ||--|{Fact_SolvedCases: ""
```

While fact tables represents business processs or actions, dimensions' tables are designed to describe facts' attributes. Querying patterns of analytical models are much more difficult to predict, therefore high normalization is required to support flexible querying.  

## Analytical models  

![star_snowflake_schema](https://github.com/user-attachments/assets/99e7f24a-87d6-4661-872d-4ec53a05f886)  

We have the star schema on the right, based on the many-to-one relationship between facts and dimenstions. While on the left we have the snowflake schema, essentially the same but with higher normalization, which allows for more flexibility but requires more complex joins and so more computational power.  

# Analytical data management platforms  
## Data warehouse  

```mermaid
flowchart LR

    subgraph data sources
        opsa[(operational system A)]
        opsb[(operational system B)]
        logs[(logs)]
    end

    opsa--"extract"-->trans3[transform]--"load"-->mart[(marketing data mart)]
    mart-->a2[analysis]
    mart-->rep2[reports]

    opsb--"extract"-->trans1[transform]--"load"-->dw
    logs--"extract"-->trans2[transform]--"load"-->dw

    dw[(data warehouse)]
    
    dw-->a1[analysis]
    dw-->rep1[reports]

    dw--"extract"-->trans4[transform]--"load"-->sdmart[(support desk data mart)]
    sdmart-->a3[analysis]
    sdmart-->rep3[reports]

    subgraph users
        a1
        rep1
        a2
        rep2
        a3
        rep3
    end
```

The data warehouse architecture is based on extract-transform-load scripts (**ETL**). The data can come from various sources and it's transformed into an enterprise-wide model that supposedly address all the different analytical use cases. When a specific analytical need that can't be served by the enterprise-wide model arise, it's possible to use a database to hold a specific model, these are called **data marts**.  

## Data lake  

```mermaid
flowchart LR
    subgraph data sources
        opsa[(operational system A)]
        opsb[(operational system B)]
        logs[(logs)]
    end

    opsa--"ingest"-->lake[(data lake)]
    opsb--"ingest"-->lake
    logs--"ingest"-->lake

    lake--"extract"-->transform--"load"-->dw

    dw[(data warehouse)]
    
    dw-->a1[analysis]
    dw-->rep1[reports]

    subgraph users
        a1
        rep1
    end
```

While data warehouses store structured data, a data lake is a centralized repository that allows you to store unstructured data at any scale. Both data warehouse and lakes tend to break under the weight of *big data*, leading to thousands of unmantainable ad-hoc ETL scripts. They also trespass the boundaries of the operational systems creating dependencies on their implementation details.  

## Data mesh  

```mermaid
flowchart LR
    subgraph bc2[marketing bounded context]
        direction LR
        subgraph aout2[analytical outputs]
            out3@{ shape: bow-rect, label: "port A" }
            out4@{ shape: bow-rect, label: "port B" }
        end

        ui2-->be2[backend]-->od2[(operational data)]
        api2-->be2

        od2--"ETL"-->am2[(analysis model)]

        am2-->node2((" "))-->out3
        node2-->out4

        subgraph oin2[operational interfaces]
            ui2[UI]
            api2[API]
        end
    end
    subgraph bc1[payments bounded context]
        direction LR
        subgraph aout1[analytical outputs]
            out1@{ shape: bow-rect, label: "port A" }
            out2@{ shape: bow-rect, label: "port B" }
        end

        ui1-->be1[backend]-->od1[(operational data)]
        api1-->be1

        od1--"ETL"-->am1[(analysis model)]

        am1-->node((" "))-->out1
        node-->out2

        subgraph oin1[operational interfaces]
            ui1[UI]
            api1[API]
        end
    end
```

This kind of architecture in a way is domain-drive design for analytical data. The responsibility of generating the analytical data now belongs to the corresponding product team. By utilising a *data as a product* principle, the team ensures the analytical model addresses the needs of its consumers and serve the data thorugh well-defined output ports. This architecture enables autonomy by creating an ecosystem where product teams creates their own data products and consumes data products served by other teams (and potentially their own too). A federated governance body will be responsible to design rules to ensure interoperability and ecosystem thinking throughout the enterprise.  

