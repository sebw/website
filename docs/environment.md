
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

## Scenario
Considering the above environment, the following teams will have to coordinate themselves in order to support the application through it life. 

| Responsibility | Description | Team | 
|----------------|-------------|------|
| Provisioning | Deploying a physical or virtual machine machine | Infrastructure |
| OS deployment  | Installing the OS on a machine | Infrastructure |
| OS life cycle  | Upgrading & patching the OS on a machine   | Infrastructure |
| Security alerts | Alerting the teams about security issues | Security |
| App deployment | Installing the application on a machine | Application |
| App life cycle | Upgrading & patching the application | Application |
| Incident Mgmt | Creating, dispatching and following up | ITSM + relevant team |
| Problem Mgmt | Creating, dispatching and following up | ITSM + relevant team |
| Change Mgmt | Creating, dispatching and following  | ITSM + relevant team |

