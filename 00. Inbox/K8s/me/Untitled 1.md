```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '#3CB371', 'primaryTextColor': '#fff', 'primaryBorderColor': '#2F4F4F', 'lineColor': '#4682B4', 'secondaryColor': '#ADD8E6', 'tertiaryColor': '#fff'}}}%%
flowchart TD
    subgraph "3-Tier AWS Architecture (IaC: Terraform & ECS Fargate)"
        direction LR
        
        %% BẮT ĐẦU: VPC Subgraph
        subgraph "VPC 10.0.0.0/16 (Multi-AZ)"
            direction TB
            
            %% BẮT ĐẦU: Presentation_Tier (Kiểm tra kỹ dấu ']')
            subgraph Presentation_Tier [Presentation Tier (Public Subnets)]
                A[$(fa:cloud) IGW]
                B[$(fa:sitemap) ALB]
            end
            
            subgraph Application_Tier [Application Tier (Private Subnets)]
                C[$(fa:cubes) ECS Cluster]
                D[$(fa:cube) ECS Service (Fargate Tasks)]
                E[$(fa:sitemap) NAT Gateway]
                
                C --> D
            end
            
            subgraph Database_Tier [Database Tier (Isolated Subnets)]
                F[$(fa:database) RDS (e.g., PostgreSQL/MySQL)]
            end
        end
        %% KẾT THÚC: VPC Subgraph

        G[$(fa:file-archive) ECR Registry]

        %% Các luồng kết nối
        A --> B : Internet Access
        
        B -- HTTPS:443 (Load Balancer Listener) --> D : HTTP:8080 (Target Group)
        
        D -- Private Network --> F : DB Port (e.g., 5432)
        
        D --> E : Egress Traffic
        
        E --> G : Pull Container Image
        E --> A : Outbound Internet (Updates)

    end
```

```mermaid
flowchart TD
    subgraph "3-Tier AWS Architecture (IaC: Terraform & ECS Fargate)"
        direction LR
        
        %% Bắt đầu VPC Subgraph
        subgraph "VPC 10.0.0.0/16 (Multi-AZ)"
            direction TB
            
            %% DÒNG CẦN KIỂM TRA: Phải có ']' ngay sau '(Public Subnets)'
            subgraph Presentation_Tier [Presentation Tier (Public Subnets)]
                A[$(fa:cloud) IGW]
                B[$(fa:sitemap) ALB]
            end
            
            subgraph Application_Tier [Application Tier (Private Subnets)]
                C[$(fa:cubes) ECS Cluster]
                D[$(fa:cube) ECS Service (Fargate Tasks)]
                E[$(fa:sitemap) NAT Gateway]
                
                C --> D
            end
            
            subgraph Database_Tier [Database Tier (Isolated Subnets)]
                F[$(fa:database) RDS (e.g., PostgreSQL/MySQL)]
            end
        end
        %% Kết thúc VPC Subgraph

        G[$(fa:file-archive) ECR Registry]

        %% Các luồng kết nối
        A --> B : Internet Access
        
        B -- HTTPS:443 (Load Balancer Listener) --> D : HTTP:8080 (Target Group)
        
        D -- Private Network --> F : DB Port (e.g., 5432)
        
        D --> E : Egress Traffic (Internet Access)
        
        E --> G : Pull Container Image
        E --> A : Outbound Internet

    end
```

```mermaid
flowchart TD
    subgraph "3-Tier AWS Architecture (Terraform & ECS Fargate)"
        direction LR
        
        subgraph VPC ["VPC 10.0.0.0/16 (Multi-AZ)"]
            direction TB
            
            %% LƯU Ý: ĐÃ SỬ DỤNG CÚ PHÁP CHUỖI ĐƠN GIẢN CHO TIÊU ĐỀ SUBGRAPH
            subgraph Presentation_Tier [Presentation Tier (Public Subnets)]
                IGW(IGW - Internet Gateway)
                ALB(ALB - Application Load Balancer)
            end
            
            subgraph Application_Tier [Application Tier (Private Subnets)]
                ECS_C(ECS Cluster)
                ECS_S(ECS Service / Fargate Tasks)
                NAT(NAT Gateway)
                
                ECS_C --> ECS_S
            end
            
            subgraph Database_Tier [Database Tier (Isolated Subnets)]
                RDS(RDS Database)
            end
        end

        ECR(ECR - Container Registry)

        %% Các luồng kết nối
        IGW --> ALB : Internet Access (HTTPS:443)
        
        ALB -- HTTPS:443 --> ECS_S : HTTP:8080 (Target Group)
        
        ECS_S -- Private Network --> RDS : DB Port
        
        ECS_S --> NAT : Egress Traffic
        
        NAT --> ECR : Pull Container Image
        NAT --> IGW : Outbound Internet

    end
```

