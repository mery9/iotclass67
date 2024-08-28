# Store data.

When IoT sensors send data to Kafka topics, Kafka partitions this data across the brokers to ensure fault tolerance and scalability. Each partition can be replicated across multiple brokers to prevent data loss in case of failures. Kafka also retains data for a configurable period, allowing the system to replay or reprocess data as needed.