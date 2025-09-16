flowchart LR
  %% High-level AWS and Snowflake layout with VPC routing details

  subgraph AWS[Amazon Web Services (AWS) Account]
    direction LR

    subgraph VPC[VPC]
      direction LR

      subgraph Private[Private Subnets (Multi-AZ)]
        direction TB
        L1[Lambda Function (ENI in Private Subnet AZ-a)]
        L2[Lambda Function (ENI in Private Subnet AZ-b)]
        SG[Security Group\nEgress: TCP 443 to Snowflake]
        RTp[Private Route Table\n0.0.0.0/0 -> NAT Gateway]
      end

      subgraph Public[Public Subnets (Multi-AZ)]
        direction TB
        NAT[NAT Gateway (Static EIP)]
        IGW[Internet Gateway]
        RTpub[Public Route Table\n0.0.0.0/0 -> Internet Gateway]
      end

      SM[AWS Secrets Manager\n(Snowflake creds)]
      CW[Amazon CloudWatch Logs]
      note_vpce[Optional: VPC Interface Endpoints\nfor Secrets Manager & CloudWatch]
    end
  end

  subgraph SNOW[Snowflake (Region)]
    direction TB
    Svc[Snowflake Service Endpoint\n(HTTPS 443)]
    NP[Account-level Network Policy\nAllow: NAT EIP\nDeny: All others]
  end

  %% Connectivity and data paths
  L1 -. Read credentials .-> SM
  L2 -. Read credentials .-> SM
  L1 -- HTTPS 443 --> NAT
  L2 -- HTTPS 443 --> NAT
  NAT -- Egress --> IGW
  IGW -- HTTPS 443 --> Svc
  NP -. attached to .-> Svc

  %% Associations
  L1 --- SG
  L2 --- SG
  Private --- RTp
  Public --- RTpub
