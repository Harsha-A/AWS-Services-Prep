
# Amazon EventBridge - Interview Questions & Answers

## What is Amazon EventBridge?

Amazon EventBridge is a serverless event bus service that enables you to build event-driven applications by connecting application data from your own apps, Software-as-a-Service (SaaS) applications, and AWS services. It simplifies the process of building event-driven architectures by providing a centralized platform for event ingestion, filtering, and routing to multiple targets.

***

## Basics

**1. What are the core components of Amazon EventBridge?**

The four core architectural components are:
- **Event Sources**: Generate events from AWS services, custom applications, or SaaS partners
- **Event Buses**: Centralized routing mechanism that receives events from sources
- **Rules**: Define filtering criteria and determine which events route to which targets
- **Targets**: AWS services or endpoints that process events (Lambda, SQS, SNS, Step Functions, etc.)

**2. What is an Event in EventBridge?**

An Event is a JSON object that represents a change in state or an update. Events have a specific structure with fields like source, detail-type, detail, time, and region. Events are immutable records of something that happened.

**3. What is an Event Bus?**

An Event Bus is a pipeline that receives events. EventBridge provides a default event bus for AWS service events, and you can create custom event buses for your application events or partner events. Event buses act as routers that apply rules to determine where events should be sent.

**4. What are the types of Event Buses?**

- **Default Event Bus**: Automatically receives events from AWS services
- **Custom Event Bus**: Created by users for application-specific events and cross-account event routing
- **Partner Event Bus**: Receives events from SaaS applications and third-party services

**5. What is an Event Pattern?**

An Event Pattern is a JSON structure that defines the filtering criteria for matching events. It specifies which events a rule should match based on event fields like source, detail-type, or custom attributes in the detail object.

***

## Rules and Targets

**6. What are Rules in EventBridge?**

Rules match incoming events based on event patterns or schedules and route them to one or more targets for processing. A single rule can send an event to multiple targets, which run in parallel.

**7. What are the two types of Rules?**

- **Event Pattern Rules**: Match events based on content and structure
- **Schedule Rules**: Trigger targets at regular intervals using cron or rate expressions

**8. How many targets can a single rule have?**

A single EventBridge rule can route events to up to 5 targets simultaneously. All targets receive the event in parallel.

**9. What AWS services can be EventBridge targets?**

Common targets include Lambda functions, SQS queues, SNS topics, Step Functions state machines, Kinesis streams, ECS tasks, Batch jobs, CodePipeline, CodeBuild, Systems Manager Automation, API Gateway, and EventBridge event buses (for cross-account or cross-region routing).

**10. Can you transform event data before sending to targets?**

Yes, EventBridge supports input transformers that allow you to customize the event payload before sending it to targets. You can extract specific fields, add static values, or restructure the JSON.

***

## Event Patterns and Filtering

**11. How do you write a basic event pattern?**

Event patterns use JSON matching. For example, to match EC2 instance state changes:
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running"]
  }
}
```

**12. What filtering operators does EventBridge support?**

EventBridge supports exact matching, prefix matching, suffix matching, anything-but matching, numeric comparisons (greater than, less than, between), exists checks, and IP address matching.

**13. What is content-based filtering?**

Content-based filtering allows you to match events based on the values within the event payload, not just metadata. This enables precise routing based on business logic embedded in the event details.

**14. How do you match on multiple values?**

Use an array in the event pattern. For example, to match multiple instance states:
```json
{
  "detail": {
    "state": ["running", "stopped", "terminated"]
  }
}
```

**15. What is the anything-but filter?**

The anything-but filter matches events that do NOT contain specified values. For example, to match all states except "pending":
```json
{
  "detail": {
    "state": [{"anything-but": "pending"}]
  }
}
```

***

## Schema Registry

**16. What is the EventBridge Schema Registry?**

The Schema Registry is a collection of event schemas that helps you discover, create, and manage event structures. It provides code bindings for popular programming languages to work with strongly-typed events.

**17. What are the types of schema registries?**

- **AWS Event Schema Registry**: Contains built-in schemas for AWS service events
- **Discovered Schema Registry**: Automatically inferred schemas from events on your event bus
- **Custom Schema Registry**: User-created schemas for application events

**18. What is Schema Discovery?**

Schema Discovery automatically infers schemas from events flowing through your event bus. When enabled, EventBridge analyzes events and creates schema definitions, including those from cross-account events.

**19. What are the benefits of using schemas?**

Schemas provide event structure documentation, enable code generation for type-safe event handling, facilitate event validation, improve discoverability of available events, and help maintain consistency across event producers and consumers.

**20. How do you enable Schema Discovery?**

Navigate to the EventBridge console, select your event bus, and enable "Start discovery" on the Schemas page. Discovery can incur costs after the first 5 million processed events per month.

***

## Use Cases and Integration

**21. What are common use cases for EventBridge?**

- Application integration and microservices communication
- Real-time data processing pipelines
- Scheduled task automation
- Security and compliance monitoring
- Multi-account and multi-region event routing
- SaaS integration and partner event consumption
- Infrastructure monitoring and alerting

**22. How do you use EventBridge for microservices?**

Microservices publish domain events to a custom event bus. Other services subscribe to relevant events through rules. This decouples services, improves scalability, and enables independent deployment.

**23. How does EventBridge integrate with AWS Step Functions?**

EventBridge rules can trigger Step Functions state machine executions. You can pass event data as input to the state machine, enabling complex workflow orchestration based on events.

**24. How do you implement cross-account event routing?**

Create a custom event bus in the target account, configure resource-based policies to allow source accounts to publish events, create rules in source accounts that target the cross-account event bus, and set up rules in the target account to route events to targets.

**25. How does EventBridge work with SaaS applications?**

EventBridge integrates with partner SaaS applications (Zendesk, Datadog, PagerDuty, etc.) through partner event sources. After authorization, partner events flow directly into your EventBridge event bus for processing.

***

## Scheduling

**26. How do you create scheduled events?**

Use schedule expressions with either rate-based (rate(5 minutes)) or cron-based (cron(0 12 * * ? *)) syntax. Schedule-based rules trigger targets at specified intervals without requiring an event source.

**27. What is the difference between rate and cron expressions?**

Rate expressions trigger at regular intervals (every X minutes/hours/days). Cron expressions provide fine-grained scheduling control (specific times, days of week, months) using the Unix cron format.

**28. What is the minimum schedule frequency?**

The minimum rate expression is 1 minute. EventBridge Scheduler (a related service) supports schedules down to 1-minute granularity with more advanced features.

**29. Can you pause and resume scheduled rules?**

Yes, you can disable rules to pause them and enable rules to resume scheduling. Disabled rules do not trigger targets or incur charges for event processing.

**30. How do you handle timezone considerations?**

Cron expressions in EventBridge use UTC timezone by default. Consider UTC offsets when scheduling time-sensitive operations, or use EventBridge Scheduler which supports timezone specifications.

***

## Advanced Features

**31. What is an Archive in EventBridge?**

Archives store events for replay purposes. You can archive all events or use filtering patterns to archive specific events. Archives enable event replay for debugging, testing, or disaster recovery.

**32. What is Event Replay?**

Event Replay allows you to reprocess archived events by sending them back through your event bus. This is useful for recovering from processing failures, testing new rules, or backfilling data.

**33. What is Dead Letter Queue (DLQ) support?**

EventBridge supports configuring DLQs for targets. If a target fails to process an event after retry attempts, EventBridge sends the event to a specified SQS queue for later analysis and reprocessing.

**34. How does retry logic work in EventBridge?**

EventBridge automatically retries failed event deliveries for up to 24 hours with exponential backoff. You can configure maximum retry attempts and DLQs to handle persistent failures.

**35. What is Input Transformation?**

Input Transformation customizes event payloads before sending to targets. You can extract specific fields, combine multiple fields, add static text, or restructure JSON to match target requirements.

***

## Security and Permissions

**36. How do you secure EventBridge?**

Use IAM policies to control who can create, modify, and delete resources. Apply resource-based policies on event buses for cross-account access. Encrypt events with AWS KMS. Use VPC endpoints for private connectivity. Enable CloudTrail logging for audit trails.

**37. What IAM permissions are required to publish events?**

The `events:PutEvents` permission allows publishing events to an event bus. You can scope this to specific event buses using resource ARNs in IAM policies.

**38. How do you enable cross-account event publishing?**

Create a resource-based policy on the target event bus that grants `events:PutEvents` permission to the source account. The source account must also have IAM permissions to publish to the target bus.

**39. Can you encrypt events?**

Yes, EventBridge integrates with AWS KMS for server-side encryption. You can specify a KMS key when creating an event bus to encrypt all events at rest.

**40. How do you audit EventBridge activity?**

Enable AWS CloudTrail to log all EventBridge API calls. CloudTrail captures rule creation, modification, deletion, and PutEvents calls for security auditing and compliance.

***

## Performance and Limits

**41. What are the service quotas for EventBridge?**

Default quotas include: 5 targets per rule, 300 rules per event bus (adjustable), 10,000 PutEvents requests per second per account (adjustable), and 256 KB maximum event size.

**42. What factors affect EventBridge performance?**

Event pattern complexity, number of rules evaluated per event, target service rate limits and throttling, payload size, and concurrent executions all impact performance.

**43. How do you optimize event pattern matching?**

Write precise event patterns that match only necessary events. Include account and region filters. Use content filters efficiently. Avoid overly broad patterns that match unnecessary events.

**44. What is the maximum event size?**

The maximum event size is 256 KB. Events exceeding this limit are rejected. Consider storing large payloads in S3 and passing references in events.

**45. How do you handle high-volume event processing?**

Use Lambda for scalable compute, implement proper error handling and retries, leverage SQS queues as buffers for rate-limited targets, monitor CloudWatch metrics for throttling, and request quota increases if needed.

***

## Design Best Practices

**46. What are best practices for event design?**

Use meaningful and descriptive event types. Include sufficient context in the detail field. Maintain backward compatibility when evolving schemas. Use schema registry for validation. Keep events small and focused on state changes.

**47. How do you avoid infinite event loops?**

Write precise event patterns that don't match events generated by your own actions. For example, if a Lambda function modifies S3 objects, ensure the rule pattern doesn't trigger on those modifications. Implement circuit breaker patterns.

**48. When should you use multiple event buses?**

Use separate event buses for different environments (dev, staging, production), different applications or teams, isolation of third-party partner events, and multi-tenant architectures requiring logical separation.

**49. How do you version events?**

Include version information in the detail-type field or within the detail object. Maintain backward compatibility for consumers. Use schema registry versioning to track changes. Consider semantic versioning (v1, v2, etc.).

**50. What monitoring should you implement?**

Monitor CloudWatch metrics for invocations, failed invocations, throttled events, and DLQ messages. Set up alarms for failure rates exceeding thresholds. Use CloudWatch Logs Insights for event analysis. Enable X-Ray tracing for distributed tracing.

***

## Comparison with Other Services

**51. EventBridge vs. SNS?**

EventBridge provides advanced event filtering, content-based routing, schema registry, and built-in integrations with 90+ AWS services. SNS offers simple pub-sub messaging with topic-based fanout and push notifications. Use EventBridge for event-driven architectures and SNS for notification delivery.

**52. EventBridge vs. SQS?**

EventBridge routes events to multiple targets based on patterns. SQS provides reliable message queuing with delivery guarantees. They're complementary: EventBridge can send events to SQS queues for buffering and rate limiting.

**53. EventBridge vs. Kinesis?**

EventBridge handles discrete events with pattern matching and routing. Kinesis processes continuous data streams with real-time analytics. Use EventBridge for event-driven workflows and Kinesis for streaming data pipelines.

**54. EventBridge vs. EventBridge Pipes?**

EventBridge Pipes is a point-to-point integration for connecting event sources to targets with optional filtering and enrichment. EventBridge provides full event bus functionality with multiple targets, archives, and replay. Use Pipes for simple integrations and EventBridge for complex routing.

**55. When should you use EventBridge Scheduler vs. EventBridge scheduled rules?**

EventBridge Scheduler offers more flexibility with one-time schedules, flexible time windows, timezone support, and higher scale (millions of schedules). Use Scheduler for application-level scheduling needs and EventBridge rules for simpler recurring schedules.

***
