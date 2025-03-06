# Problem 2: Building Castle In The Cloud

## Architecture Diagram

```
                                     ┌─────────────────┐
                                     │  AWS CloudFront │
                                     └────────┬────────┘
                                              │
                                     ┌────────▼────────┐
                                     │   AWS WAF/Shield│
                                     └────────┬────────┘
                                              │
┌─────────────────┐               ┌───────────▼──────────┐
│ Route 53        │───────────────►  Application Load    │
└─────────────────┘               │  Balancer            │
                                  └───────────┬──────────┘
                                              │
                ┌──────────────────┬──────────┴──────────┬───────────────────┐
                │                  │                     │                   │
        ┌───────▼─────┐     ┌──────▼───────┐    ┌───────▼─────┐     ┌───────▼─────┐
        │ ECS Cluster │     │ ECS Cluster  │    │ ECS Cluster │     │ ECS Cluster │
        │ Auth Service│     │ Trading API  │    │ Market Data │     │ Wallet      │
        └───────┬─────┘     └──────┬───────┘    └───────┬─────┘     └───────┬─────┘
                │                  │                    │                   │
                └──────────┬───────┴────────────┬───────┴───────────────────┘
                           │                    │
                ┌──────────▼──────────┐ ┌───────▼────────┐     ┌─────────────────┐
                │ Amazon RDS Aurora   │ │ Amazon         │     │ Amazon          │
                │ (Multi-AZ)          │ │ ElastiCache    │◄────┤ EventBridge     │
                └──────────┬──────────┘ └───────┬────────┘     └─────────────────┘
                           │                    │                       ▲
                           │                    │                       │
                ┌──────────▼──────────┐ ┌───────▼────────┐     ┌────────┴────────┐
                │ Amazon DynamoDB     │ │ Amazon Kinesis │     │ AWS Lambda      │
                │ (Global Tables)     │ │ Data Streams   │     │ (Event Handlers)│
                └─────────────────────┘ └───────┬────────┘     └─────────────────┘
                                                │
                                       ┌────────▼────────┐
                                       │ Amazon S3       │
                                       │ (Data Lake)     │
                                       └─────────────────┘
```

## Key Components and Their Roles

### Frontend and Content Delivery

- **AWS CloudFront**: Global CDN for low-latency static asset delivery
- **AWS WAF & Shield**: Protection against DDoS attacks and malicious traffic
- **Route 53**: DNS routing with health checks and failover capabilities
- **Application Load Balancer**: Distributes traffic across availability zones

### Microservices (ECS with Fargate)

1. **Authentication Service**: User registration, login, MFA, and session management
2. **Trading API Service**: Order placement, matching engine, and trade execution
3. **Market Data Service**: Real-time price feeds and order book updates
4. **Wallet Service**: Cryptocurrency wallet management and balance tracking

### Data Storage

- **Amazon RDS Aurora (Multi-AZ)**: Relational database for user accounts and financial data
- **Amazon DynamoDB (Global Tables)**: NoSQL database for order books and real-time market data
- **Amazon ElastiCache (Redis)**: In-memory caching for session data and order matching
- **Amazon S3**: Object storage for logs, backups, and data lake

### Event Processing

- **Amazon Kinesis Data Streams**: Real-time data streaming for market data and order events
- **Amazon EventBridge**: Event bus for decoupled communication between services
- **AWS Lambda**: Serverless functions for event processing and background tasks

## Justification of Technology Choices and Alternatives

- **ECS with Fargate over EKS**: 
  - Chosen: Reduced operational overhead while maintaining scalability
  - Alternative: EKS would offer more customization but requires more maintenance

- **Aurora over standard RDS**: 
  - Chosen: 5x performance of MySQL with automated failover
  - Alternative: Standard RDS would be cheaper but lacks the performance and HA features

- **DynamoDB over MongoDB**: 
  - Chosen: Single-digit millisecond response times with auto-scaling
  - Alternative: MongoDB would offer more flexible schema but lacks AWS integration

- **ElastiCache over self-managed Redis**: 
  - Chosen: Sub-millisecond latency with managed failover
  - Alternative: Self-managed Redis clusters would allow more customization but increase operational burden

- **Kinesis over Kafka**: 
  - Chosen: Managed service with high throughput for real-time processing
  - Alternative: Kafka on MSK would have better ecosystem but higher operational complexity

## Scaling Strategy

### Current Setup (500 RPS)

- ECS services with auto-scaling based on CPU/memory utilization
- DynamoDB provisioned throughput for predictable performance
- ElastiCache Redis cluster with read replicas

### Future Scaling

- Horizontal scaling of ECS tasks based on traffic patterns
- Database sharding by user ID or asset class
- Regional expansion with global database replication
- Performance optimization with custom algorithms and caching
- Multi-region deployment for lower latency and disaster recovery
- Increase DynamoDB capacity units for higher throughput
- Implement more aggressive caching strategies at edge locations

