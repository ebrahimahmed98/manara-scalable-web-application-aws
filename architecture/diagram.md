
flowchart TB
    User["Users"]
    R53["Amazon Route 53<br/>DNS Alias and Health Checks"]
    CF["Amazon CloudFront<br/>HTTPS and Caching"]
    WAF["AWS WAF<br/>Managed Rules"]
    ALB["Application Load Balancer"]

    subgraph VPC["Custom VPC - 10.0.0.0/16"]
        IGW["Internet Gateway"]
        NAT["NAT Gateway"]

        subgraph AZA["Availability Zone A"]
            PUBA["Public Subnet"]
            APPA["Private App Subnet"]
            EC2A["EC2 / Nginx"]
            DBA["Private DB Subnet A"]
            DBPrimary["RDS Primary - Proposed"]
        end

        subgraph AZB["Availability Zone B"]
            PUBB["Public Subnet"]
            APPB["Private App Subnet"]
            EC2B["EC2 - ASG Scale-out"]
            DBB["Private DB Subnet B"]
            DBStandby["RDS Standby - Proposed"]
        end

        SSM["Systems Manager VPC Endpoints"]
        S3EP["S3 Gateway Endpoint"]
    end

    CW["Amazon CloudWatch"]
    SNS["Amazon SNS Email Alerts"]

    User --> R53
    R53 --> CF
    CF --- WAF
    CF --> ALB
    ALB --> EC2A
    ALB -. "When scaled out" .-> EC2B

    EC2A --> DBPrimary
    EC2B -. "Application access" .-> DBPrimary
    DBPrimary <-. "Synchronous replication<br/>Multi-AZ proposal" .-> DBStandby

    EC2A --> NAT
    EC2B --> NAT
    EC2A --> SSM
    EC2B --> SSM
    EC2A --> S3EP
    EC2B --> S3EP

    EC2A -. "Metrics" .-> CW
    CW --> SNS

    classDef proposed fill:#fff0cc,stroke:#a66b00,color:#4d3200
    class R53,DBStandby,DBPrimary proposed
