[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Storage**](storage-documentation) » PostgreSQL Storage

Currently, Snowplow events are stored in PostgreSQL in a single events table. The full table definition is given in the [Github repo][postgres-atomic-def]. 

[postgres-atomic-def]: https://github.com/snowplow/snowplow/blob/master/4-storage/postgres-storage/sql/atomic-def.sql
[avro]: http://avro.apache.org/