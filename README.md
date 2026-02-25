# my-home-environment

```mermaid
graph TD
    subgraph External [" "]
        direction TB
        ExtTitle["<b>外部ネットワーク / インターネット</b>"]
        Internet((Internet))
        ExtTitle --- Internet
    end
    subgraph Tailnet ["Tailscale ネットワーク"]
        direction TB
        
        Server["<b>ノートPCサーバー (踏み台)</b><br/>OS: Ubuntu Server<br/>CPU: Core i3 7th Gen<br/>Storage: 1TB SSD"]
        
        MainPC["<b>メインPC</b><br/>OS: Windows 11 Pro<br/>CPU: Ryzen 7 7800X3D<br/>GPU: Radeon RX 7800XT<br/>Storage: 2TB SSD"]
        
        AIPC["<b>AI PC</b><br/>OS: Windows 11 Pro<br/>CPU: Ryzen 9 5950X<br/>GPU: RTX 3060 (12GB)<br/>Storage: 1TB SSD + 1TB HDD"]

        MainPC --- Server
        AIPC --- Server
    end

    Server <--> Internet
    
    style Server fill:#f96,stroke:#333,stroke-width:2px
    style Tailnet fill:#e1f5fe,stroke:#01579b,stroke-dasharray: 5 5
    style MainPC fill:#fff,stroke:#333
    style AIPC fill:#fff,stroke:#333
