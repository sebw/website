
## Diagram 

A drawings is better than a thousands words! Never forget this event if your drawing skills did not evolve since kindergarden :wink:

``` mermaid
graph LR
  A(user) --> B{extranet};
  B --> C[load balancer];
  C --> D[CMS];
  C --> F(DNS)
  D --> E[DB]

  subgraph intranet
    C
    D
    E
    F
  end
```
