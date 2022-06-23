
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

During the course of the workshop, the above diagram will evolve with additional component. 