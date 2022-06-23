
## Diagram 

Never forget that a drawings is better than a thousands words! Even if your drawing skills did not evolve since kindergarden :wink:

``` mermaid
graph LR
  A(browser) --> B{network};
  AA(mobile) --> B;
  B --> C[load balancer];
  C --> D[CMS];
  C --> F(DNS)
  D --> E[DB]

  subgraph  
    C
    subgraph app tier
      D
    end
    subgraph data tier
      E
    end
    F
  end

  subgraph client
  A
  AA
  end


```

The above diagram represents a typical 3-tier architecture that you might find within 99% of organizations:

- browser as the client (also called to presentation) tier
- CMS as the application (also called business logic) tier
- DB as the data tier

