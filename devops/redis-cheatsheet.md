# Redis Cheatsheet

Redis (Remote Dictionary Server) is an open-source, in-memory data structure store that can be used as a database, cache, message broker, and streaming engine. It supports various data structures and provides high performance with sub-millisecond latency.

## Overview

### Key Features
- **In-Memory Storage** - All data stored in RAM for ultra-fast access
- **Persistence** - Optional disk persistence with RDB snapshots and AOF logs
- **Data Structures** - Rich set of data types (strings, hashes, lists, sets, sorted sets, etc.)
- **Atomic Operations** - All operations are atomic by design
- **Pub/Sub** - Built-in publish/subscribe messaging
- **Lua Scripting** - Server-side scripting support
- **Transactions** - Multi-command transactions with MULTI/EXEC
- **Replication** - Master-slave replication
- **Clustering** - Horizontal scaling with Redis Cluster
- **High Availability** - Redis Sentinel for monitoring and failover
- **Streams** - Log-like data structure for event sourcing
- **Modules** - Extensibility through Redis modules

### Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Apps  │    │   Redis Server  │    │   Persistence   │
│                 │    │                 │    │                 │
│ • Applications  │◄──►│ • Event Loop    │◄──►│ • RDB Snapshots │
│ • Redis CLI     │    │ • Command Proc. │    │ • AOF Logs      │
│ • Libraries     │    │ • Memory Store  │    │ • Disk Storage  │
│ • Tools         │    │ • Pub/Sub       │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                       ┌─────────────────┐
                       │   Replication   │
                       │                 │
                       │ • Master/Slave  │
                       │ • Clustering    │
                       │ • Sentinel      │
                       └─────────────────┘
```

## Installation

### macOS Installation
```bash
# Using Homebrew
brew install redis

# Start Redis server
brew services start redis

# Or start manually
redis-server

# Start Redis server with config file
redis-server /usr/local/etc/redis.conf

# Connect to Redis
redis-cli

# Connect to specific host/port
redis-cli -h localhost -p 6379

# Connect with authentication
redis-cli -a password

# Stop Redis service
brew services stop redis
```

### Linux Installation
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server

# CentOS/RHEL
sudo yum install epel-release
sudo yum install redis
sudo systemctl start redis
sudo systemctl enable redis

# From source
wget http://download.redis.io/redis-stable.tar.gz
tar xzf redis-stable.tar.gz
cd redis-stable
make
make install

# Start Redis
redis-server

# Check Redis status
sudo systemctl status redis
```

### Docker Installation
```bash
# Run Redis container
docker run --name redis-server -p 6379:6379 -d redis:7-alpine

# Run with persistent data
docker run --name redis-server \
  -p 6379:6379 \
  -v redis_data:/data \
  -d redis:7-alpine redis-server --appendonly yes

# Connect to Redis in container
docker exec -it redis-server redis-cli

# Using Docker Compose
cat > docker-compose.yml << EOF
version: '3.8'
services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    environment:
      - REDIS_PASSWORD=mypassword

volumes:
  redis_data:
EOF

docker-compose up -d
```

## Basic Commands

### Connection and Server Commands
```bash
# Connect to Redis
redis-cli
redis-cli -h hostname -p port -a password

# Test connection
PING                    # Returns PONG

# Authenticate
AUTH password

# Select database (0-15 by default)
SELECT 1

# Get server info
INFO
INFO replication
INFO memory
INFO stats

# Get configuration
CONFIG GET *
CONFIG GET maxmemory

# Set configuration
CONFIG SET maxmemory 100mb

# Save configuration to file
CONFIG REWRITE

# Monitor commands
MONITOR

# Get Redis version
INFO server

# Shutdown Redis
SHUTDOWN
SHUTDOWN SAVE      # Save before shutdown
SHUTDOWN NOSAVE    # Don't save before shutdown

# Flush databases
FLUSHDB            # Flush current database
FLUSHALL           # Flush all databases

# Get database size
DBSIZE

# Get memory usage
MEMORY USAGE key
MEMORY STATS

# Client commands
CLIENT LIST
CLIENT INFO
CLIENT KILL ip:port
```

### Key Management
```bash
# Key operations
KEYS pattern           # List keys (avoid in production)
SCAN cursor [MATCH pattern] [COUNT count]  # Iterate keys safely
EXISTS key [key ...]   # Check if key exists
TYPE key              # Get key type
TTL key               # Get time to live in seconds
PTTL key              # Get time to live in milliseconds
EXPIRE key seconds    # Set expiration
EXPIREAT key timestamp # Set expiration at timestamp
PEXPIRE key milliseconds # Set expiration in milliseconds
PERSIST key           # Remove expiration
DEL key [key ...]     # Delete keys
UNLINK key [key ...]  # Async delete (non-blocking)
RENAME key newkey     # Rename key
RENAMENX key newkey   # Rename if new key doesn't exist
RANDOMKEY             # Get random key

# Examples
KEYS user:*           # Get all keys starting with "user:"
SCAN 0 MATCH user:* COUNT 10
EXISTS user:123
EXPIRE session:abc 3600  # Expire in 1 hour
DEL user:123 user:456
```

## Data Types and Commands

### Strings
```bash
# Basic string operations
SET key value [EX seconds] [PX milliseconds] [NX|XX]
GET key
GETSET key value      # Set and return old value
MSET key1 value1 key2 value2  # Set multiple
MGET key1 key2 key3   # Get multiple
SETNX key value       # Set if not exists
SETEX key seconds value # Set with expiration
PSETEX key milliseconds value # Set with expiration in ms

# String manipulation
APPEND key value      # Append to string
STRLEN key            # Get string length
GETRANGE key start end # Get substring
SETRANGE key offset value # Replace part of string

# Numeric operations
INCR key              # Increment by 1
DECR key              # Decrement by 1
INCRBY key increment  # Increment by amount
DECRBY key decrement  # Decrement by amount
INCRBYFLOAT key increment # Increment by float

# Bit operations
SETBIT key offset value
GETBIT key offset
BITCOUNT key [start end]
BITOP operation destkey key [key ...]

# Examples
SET user:123:name "John Doe"
SET counter 0
INCR counter
SET session:abc "token123" EX 3600
APPEND user:123:name " Jr."
SETBIT online_users 123 1
```

### Lists
```bash
# List operations (O(1) for head/tail, O(N) for middle)
LPUSH key element [element ...]    # Push to left (head)
RPUSH key element [element ...]    # Push to right (tail)
LPOP key [count]                   # Pop from left
RPOP key [count]                   # Pop from right
LLEN key                          # Get list length
LRANGE key start stop             # Get range of elements
LINDEX key index                  # Get element by index
LSET key index element            # Set element by index
LINSERT key BEFORE|AFTER pivot element # Insert element
LREM key count element            # Remove elements
LTRIM key start stop              # Trim list to range

# Blocking operations
BLPOP key [key ...] timeout       # Blocking left pop
BRPOP key [key ...] timeout       # Blocking right pop
BRPOPLPUSH source destination timeout # Blocking move

# Advanced operations
RPOPLPUSH source destination      # Move element between lists
LMOVE source destination LEFT|RIGHT LEFT|RIGHT
BLMOVE source destination LEFT|RIGHT LEFT|RIGHT timeout

# Examples
LPUSH tasks "task1" "task2" "task3"
RPOP tasks
LRANGE tasks 0 -1     # Get all elements
LLEN tasks
BLPOP tasks 30        # Wait up to 30 seconds for element
```

### Sets
```bash
# Set operations
SADD key member [member ...]      # Add members
SREM key member [member ...]      # Remove members
SMEMBERS key                      # Get all members
SISMEMBER key member              # Check if member exists
SCARD key                         # Get set size
SRANDMEMBER key [count]           # Get random member(s)
SPOP key [count]                  # Pop random member(s)
SMOVE source destination member   # Move member between sets

# Set operations between sets
SUNION key [key ...]              # Union of sets
SUNIONSTORE destination key [key ...] # Store union
SINTER key [key ...]              # Intersection of sets
SINTERSTORE destination key [key ...] # Store intersection
SDIFF key [key ...]               # Difference of sets
SDIFFSTORE destination key [key ...] # Store difference

# Scanning sets
SSCAN key cursor [MATCH pattern] [COUNT count]

# Examples
SADD users:online 123 456 789
SISMEMBER users:online 123
SUNION users:online users:premium
SINTER users:online users:active
SCARD users:online
```

### Sorted Sets (ZSets)
```bash
# Sorted set operations
ZADD key [NX|XX] [CH] [INCR] score member [score member ...]
ZREM key member [member ...]      # Remove members
ZCARD key                         # Get set size
ZCOUNT key min max               # Count members in score range
ZSCORE key member                # Get member score
ZRANK key member                 # Get member rank (0-based)
ZREVRANK key member              # Get reverse rank

# Range operations
ZRANGE key start stop [WITHSCORES] # Get by rank
ZREVRANGE key start stop [WITHSCORES] # Get by reverse rank
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
ZRANGEBYLEX key min max [LIMIT offset count]

# Score operations
ZINCRBY key increment member     # Increment member score
ZREMRANGEBYRANK key start stop  # Remove by rank range
ZREMRANGEBYSCORE key min max    # Remove by score range
ZREMRANGEBYLEX key min max      # Remove by lexicographical range

# Set operations
ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight ...] [AGGREGATE SUM|MIN|MAX]
ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight ...] [AGGREGATE SUM|MIN|MAX]

# Scanning
ZSCAN key cursor [MATCH pattern] [COUNT count]

# Examples
ZADD leaderboard 1000 player1 1500 player2 2000 player3
ZRANGE leaderboard 0 2 WITHSCORES
ZREVRANGE leaderboard 0 2 WITHSCORES
ZRANGEBYSCORE leaderboard 1000 2000
ZINCRBY leaderboard 100 player1
```

### Hashes
```bash
# Hash operations
HSET key field value [field value ...]  # Set field(s)
HGET key field                          # Get field value
HMSET key field value [field value ...] # Set multiple (deprecated)
HMGET key field [field ...]             # Get multiple fields
HGETALL key                             # Get all fields and values
HKEYS key                               # Get all field names
HVALS key                               # Get all values
HEXISTS key field                       # Check if field exists
HDEL key field [field ...]              # Delete fields
HLEN key                                # Get number of fields
HSTRLEN key field                       # Get field value length

# Numeric operations
HINCRBY key field increment             # Increment field by integer
HINCRBYFLOAT key field increment        # Increment field by float

# Conditional operations
HSETNX key field value                  # Set if field doesn't exist

# Scanning
HSCAN key cursor [MATCH pattern] [COUNT count]

# Examples
HSET user:123 name "John" age 30 email "john@example.com"
HGET user:123 name
HMGET user:123 name age
HGETALL user:123
HINCRBY user:123 age 1
HDEL user:123 email
```

### Streams
```bash
# Stream operations
XADD key ID field value [field value ...] # Add entry
XLEN key                                   # Get stream length
XRANGE key start end [COUNT count]         # Get entries by ID range
XREVRANGE key end start [COUNT count]      # Get entries in reverse
XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...]

# Consumer groups
XGROUP CREATE key groupname id [MKSTREAM]
XGROUP DESTROY key groupname
XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...]
XACK key group ID [ID ...]                # Acknowledge message
XPENDING key group [start end count] [consumer]

# Stream management
XDEL key ID [ID ...]                      # Delete entries
XTRIM key MAXLEN [~] count                # Trim stream

# Examples
XADD events * user 123 action login timestamp 1640995200
XRANGE events - +
XGROUP CREATE events processors 0
XREADGROUP GROUP processors consumer1 COUNT 1 STREAMS events >
```

### HyperLogLog
```bash
# HyperLogLog operations (probabilistic cardinality)
PFADD key element [element ...]    # Add elements
PFCOUNT key [key ...]             # Get approximate cardinality
PFMERGE destkey sourcekey [sourcekey ...] # Merge HLLs

# Examples
PFADD unique_visitors user1 user2 user3
PFCOUNT unique_visitors
```

### Geospatial
```bash
# Geospatial operations
GEOADD key longitude latitude member [longitude latitude member ...]
GEOPOS key member [member ...]     # Get positions
GEODIST key member1 member2 [unit] # Get distance
GEORADIUS key longitude latitude radius unit [WITHCOORD] [WITHDIST] [COUNT count]
GEORADIUSBYMEMBER key member radius unit [WITHCOORD] [WITHDIST] [COUNT count]
GEOHASH key member [member ...]    # Get geohash

# Examples
GEOADD locations -122.4194 37.7749 "San Francisco" -74.0059 40.7128 "New York"
GEODIST locations "San Francisco" "New York" km
GEORADIUS locations -122.4194 37.7749 100 km
```

## Advanced Features

### Transactions
```bash
# Transaction commands
MULTI                    # Start transaction
EXEC                     # Execute transaction
DISCARD                  # Discard transaction
WATCH key [key ...]      # Watch keys for changes
UNWATCH                  # Unwatch all keys

# Example transaction
MULTI
SET counter 0
INCR counter
GET counter
EXEC

# Optimistic locking example
WATCH counter
val = GET counter
MULTI
SET counter (val + 1)
EXEC
```

### Pub/Sub
```bash
# Publisher commands
PUBLISH channel message       # Publish message
PUBSUB CHANNELS [pattern]     # List channels
PUBSUB NUMSUB [channel ...]   # Get subscriber count
PUBSUB NUMPAT                 # Get pattern subscriber count

# Subscriber commands
SUBSCRIBE channel [channel ...] # Subscribe to channels
UNSUBSCRIBE [channel ...]      # Unsubscribe from channels
PSUBSCRIBE pattern [pattern ...] # Subscribe to patterns
PUNSUBSCRIBE [pattern ...]     # Unsubscribe from patterns

# Examples
# Terminal 1 (Subscriber)
SUBSCRIBE news sports

# Terminal 2 (Publisher)
PUBLISH news "Breaking news!"
PUBLISH sports "Game result!"

# Pattern subscription
PSUBSCRIBE news:*
PUBLISH news:tech "Tech news!"
PUBLISH news:sports "Sports news!"
```

### Lua Scripting
```bash
# Script commands
EVAL script numkeys key [key ...] arg [arg ...]
EVALSHA sha1 numkeys key [key ...] arg [arg ...]
SCRIPT LOAD script
SCRIPT EXISTS sha1 [sha1 ...]
SCRIPT FLUSH
SCRIPT KILL

# Example Lua script
EVAL "return redis.call('GET', KEYS[1])" 1 mykey

# Atomic increment with max value
local script = [[
  local current = redis.call('GET', KEYS[1]) or 0
  if tonumber(current) < tonumber(ARGV[1]) then
    return redis.call('INCR', KEYS[1])
  else
    return current
  end
]]
EVAL script 1 counter 100

# Load and execute script
SCRIPT LOAD "return redis.call('GET', KEYS[1])"
# Returns SHA1 hash
EVALSHA <sha1> 1 mykey
```

### Pipeline and Batch Operations
```bash
# Using redis-cli with pipeline
echo -e "SET key1 value1\nSET key2 value2\nGET key1\nGET key2" | redis-cli --pipe

# Mass insertion
cat data.txt | redis-cli --pipe

# Example data.txt format:
# SET key1 value1
# SET key2 value2
# SADD myset member1 member2
```

## Configuration and Persistence

### Configuration
```bash
# View configuration
CONFIG GET *
CONFIG GET save
CONFIG GET appendonly

# Set configuration
CONFIG SET maxmemory 100mb
CONFIG SET maxmemory-policy allkeys-lru
CONFIG SET save "900 1 300 10 60 10000"

# Save configuration to file
CONFIG REWRITE

# Common configuration options
CONFIG SET maxclients 10000
CONFIG SET timeout 300
CONFIG SET tcp-keepalive 60
CONFIG SET bind "127.0.0.1"
CONFIG SET port 6379
CONFIG SET requirepass "mypassword"
```

### Persistence
```bash
# RDB (Redis Database) snapshots
SAVE                     # Synchronous save (blocks server)
BGSAVE                   # Background save (non-blocking)
LASTSAVE                 # Get last save timestamp

# AOF (Append Only File)
BGREWRITEAOF            # Rewrite AOF file

# Configuration for persistence
CONFIG SET save "900 1 300 10 60 10000"  # RDB save conditions
CONFIG SET appendonly yes                  # Enable AOF
CONFIG SET appendfsync everysec           # AOF sync policy
CONFIG SET auto-aof-rewrite-percentage 100
CONFIG SET auto-aof-rewrite-min-size 64mb

# Check persistence status
INFO persistence
```

### Memory Management
```bash
# Memory commands
MEMORY USAGE key         # Get memory usage of key
MEMORY STATS             # Get memory statistics
MEMORY DOCTOR            # Memory usage analysis
MEMORY PURGE             # Purge memory

# Memory policies
CONFIG SET maxmemory-policy noeviction     # Don't evict, return errors
CONFIG SET maxmemory-policy allkeys-lru    # Evict least recently used
CONFIG SET maxmemory-policy allkeys-lfu    # Evict least frequently used
CONFIG SET maxmemory-policy allkeys-random # Evict random keys
CONFIG SET maxmemory-policy volatile-lru   # Evict LRU keys with expiration
CONFIG SET maxmemory-policy volatile-lfu   # Evict LFU keys with expiration
CONFIG SET maxmemory-policy volatile-random # Evict random keys with expiration
CONFIG SET maxmemory-policy volatile-ttl   # Evict keys with shortest TTL

# Get memory info
INFO memory
```

## Replication and High Availability

### Master-Slave Replication
```bash
# On slave server
REPLICAOF hostname port  # Set as replica of master
REPLICAOF NO ONE        # Promote slave to master

# Replication info
INFO replication

# Configuration
CONFIG SET replica-read-only yes
CONFIG SET replica-serve-stale-data yes

# Replication commands
PSYNC replicationid offset  # Partial resynchronization
```

### Redis Sentinel
```bash
# Start Sentinel
redis-sentinel /path/to/sentinel.conf

# Sentinel commands
SENTINEL masters
SENTINEL slaves mymaster
SENTINEL get-master-addr-by-name mymaster
SENTINEL failover mymaster
SENTINEL reset mymaster

# Example sentinel.conf
# sentinel monitor mymaster 127.0.0.1 6379 2
# sentinel auth-pass mymaster password
# sentinel down-after-milliseconds mymaster 5000
# sentinel failover-timeout mymaster 10000
# sentinel parallel-syncs mymaster 1
```

### Redis Cluster
```bash
# Create cluster
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1

# Cluster commands
CLUSTER NODES            # List cluster nodes
CLUSTER INFO             # Cluster information
CLUSTER SLOTS            # Show slot allocation
CLUSTER KEYSLOT key      # Get key's slot
CLUSTER COUNTKEYSINSLOT slot # Count keys in slot

# Add/remove nodes
redis-cli --cluster add-node new_host:new_port existing_host:existing_port
redis-cli --cluster del-node host:port node_id

# Reshard cluster
redis-cli --cluster reshard host:port

# Fix cluster
redis-cli --cluster fix host:port

# Check cluster
redis-cli --cluster check host:port
```

## Monitoring and Debugging

### Monitoring Commands
```bash
# Real-time monitoring
MONITOR                  # Monitor all commands

# Statistics
INFO stats
INFO commandstats
INFO clients
INFO memory
INFO replication
INFO persistence

# Slow log
SLOWLOG GET [count]      # Get slow queries
SLOWLOG LEN              # Get slow log length
SLOWLOG RESET            # Clear slow log

# Configuration for slow log
CONFIG SET slowlog-log-slower-than 10000  # Log queries slower than 10ms
CONFIG SET slowlog-max-len 128             # Keep last 128 slow queries

# Latency monitoring
LATENCY LATEST
LATENCY HISTORY event
LATENCY RESET [event ...]
LATENCY DOCTOR

# Client tracking
CLIENT TRACKING ON
CLIENT TRACKING OFF
CLIENT LIST
CLIENT INFO
CLIENT KILL TYPE normal
```

### Debugging
```bash
# Debug commands
DEBUG OBJECT key         # Get object encoding info
DEBUG SEGFAULT          # Crash Redis (for testing)
OBJECT ENCODING key     # Get value encoding
OBJECT IDLETIME key     # Get idle time
OBJECT REFCOUNT key     # Get reference count

# Memory debugging
MEMORY USAGE key
MEMORY MALLOC-STATS

# Benchmark
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
redis-benchmark -t set,get -n 100000 -q
```

## Redis CLI Tips and Tricks

### CLI Options
```bash
# Connect with options
redis-cli -h hostname -p port -a password -n database

# Execute single command
redis-cli GET mykey
redis-cli -h localhost -p 6379 SET mykey "value"

# Execute commands from file
redis-cli < commands.txt

# Raw output
redis-cli --raw GET mykey

# CSV output
redis-cli --csv LRANGE mylist 0 -1

# Latency mode
redis-cli --latency -h hostname -p port
redis-cli --latency-history -h hostname -p port

# Continuous stats
redis-cli --stat

# Memory optimization analyzer
redis-cli --memkeys

# Big keys finder
redis-cli --bigkeys

# Scan for patterns
redis-cli --scan --pattern "user:*"

# RDB analysis
redis-cli --rdb /path/to/dump.rdb
```

### Batch Operations
```bash
# Mass insertion
cat data.txt | redis-cli --pipe

# Generate test data
for i in {1..1000}; do echo "SET key$i value$i"; done | redis-cli --pipe

# Eval script from file
redis-cli EVAL "$(cat script.lua)" 0

# Execute multiple commands
redis-cli -c 'SET key1 value1' 'SET key2 value2' 'GET key1'
```

## Performance Optimization

### Best Practices
```bash
# Use appropriate data structures
# Strings for simple values
SET user:123:name "John"

# Hashes for objects
HSET user:123 name "John" age 30 email "john@example.com"

# Lists for ordered collections
LPUSH timeline:123 "event1" "event2"

# Sets for unique collections
SADD tags:123 "redis" "database" "cache"

# Sorted sets for ranked data
ZADD leaderboard 1000 player1 1500 player2

# Use pipelining for multiple commands
# Instead of:
# SET key1 value1
# SET key2 value2
# SET key3 value3
# Use:
echo -e "SET key1 value1\nSET key2 value2\nSET key3 value3" | redis-cli --pipe

# Use SCAN instead of KEYS
# Bad:
KEYS user:*
# Good:
SCAN 0 MATCH user:* COUNT 100

# Set expiration for cache data
SETEX cache:user:123 3600 "cached_data"

# Use appropriate eviction policy
CONFIG SET maxmemory-policy allkeys-lru

# Monitor slow queries
CONFIG SET slowlog-log-slower-than 10000
SLOWLOG GET 10
```

### Memory Optimization
```bash
# Use smaller data types when possible
# For small hashes, lists, sets - Redis uses optimized encodings
CONFIG SET hash-max-ziplist-entries 512
CONFIG SET hash-max-ziplist-value 64
CONFIG SET list-max-ziplist-size -2
CONFIG SET set-max-intset-entries 512
CONFIG SET zset-max-ziplist-entries 128
CONFIG SET zset-max-ziplist-value 64

# Check memory usage
MEMORY USAGE key
INFO memory

# Find memory-intensive keys
redis-cli --memkeys
redis-cli --bigkeys

# Optimize string memory
# Use integers when possible (stored as long)
SET counter 123

# For small objects, consider JSON strings instead of hashes
SET user:123 '{"name":"John","age":30}'
```

## Common Use Cases and Patterns

### Caching
```bash
# Simple cache with expiration
SETEX cache:user:123 3600 "user_data"
GET cache:user:123

# Cache-aside pattern
data = GET cache:key
if data is null:
    data = fetch_from_database()
    SETEX cache:key 3600 data
return data

# Write-through cache
SET cache:key data
write_to_database(data)

# Write-behind cache
SET cache:key data
# Async write to database
```

### Session Storage
```bash
# Store session data
HSET session:abc123 user_id 456 username "john" last_seen 1640995200
EXPIRE session:abc123 1800  # 30 minutes

# Get session data
HGETALL session:abc123

# Update session
HSET session:abc123 last_seen 1640995800
EXPIRE session:abc123 1800  # Reset expiration
```

### Rate Limiting
```bash
# Simple rate limiter
key = "rate_limit:" + user_id + ":" + current_minute
count = INCR key
if count == 1:
    EXPIRE key 60  # Reset after 1 minute
if count > rate_limit:
    return "Rate limit exceeded"

# Sliding window rate limiter with sorted set
ZREMRANGEBYSCORE rate_limit:user:123 0 (current_time - window_size)
ZADD rate_limit:user:123 current_time current_time
if ZCARD rate_limit:user:123 > rate_limit:
    return "Rate limit exceeded"
```

### Real-time Analytics
```bash
# Page views counter
INCR page_views:total
INCR page_views:$(date +%Y-%m-%d)
INCR page_views:$(date +%Y-%m-%d-%H)

# Unique visitors with HyperLogLog
PFADD unique_visitors:$(date +%Y-%m-%d) user_id
PFCOUNT unique_visitors:$(date +%Y-%m-%d)

# Top pages with sorted set
ZINCRBY popular_pages 1 "/homepage"
ZREVRANGE popular_pages 0 9 WITHSCORES  # Top 10 pages
```

### Message Queue
```bash
# Simple queue with lists
LPUSH queue:tasks task_data
BRPOP queue:tasks 30  # Block for up to 30 seconds

# Priority queue with sorted sets
ZADD priority_queue 1 "low_priority_task"
ZADD priority_queue 10 "high_priority_task"
ZPOPMAX priority_queue  # Get highest priority task

# Reliable queue with processing set
task = BRPOPLPUSH queue:tasks queue:processing 30
# Process task
LREM queue:processing 1 task  # Remove from processing when done
```

### Leaderboard
```bash
# Gaming leaderboard
ZADD leaderboard 1000 player1 1500 player2 2000 player3

# Get top 10 players
ZREVRANGE leaderboard 0 9 WITHSCORES

# Get player rank
ZREVRANK leaderboard player1

# Get players around a specific player
rank = ZREVRANK leaderboard target_player
ZREVRANGE leaderboard (rank-2) (rank+2) WITHSCORES

# Update score
ZINCRBY leaderboard 100 player1
```

### Geolocation
```bash
# Store locations
GEOADD drivers 37.7749 -122.4194 driver1 40.7128 -74.0059 driver2

# Find nearby drivers
GEORADIUS drivers 37.7849 -122.4094 5 km WITHCOORD WITHDIST

# Get distance between drivers
GEODIST drivers driver1 driver2 km
```

## Troubleshooting

### Common Issues
```bash
# Check Redis status
PING
INFO server

# Check memory usage
INFO memory
MEMORY STATS

# Check slow queries
SLOWLOG GET 10

# Check connections
INFO clients
CLIENT LIST

# Check replication
INFO replication

# Check persistence
INFO persistence
LASTSAVE

# Memory issues
CONFIG GET maxmemory
CONFIG GET maxmemory-policy
redis-cli --bigkeys

# Connection issues
CONFIG GET bind
CONFIG GET port
CONFIG GET protected-mode
CONFIG GET requirepass

# Performance issues
LATENCY LATEST
SLOWLOG GET
redis-benchmark
```

### Error Messages
```bash
# "MISCONF Redis is configured to save RDB snapshots"
# Solution: Fix disk space or disable saves
CONFIG SET save ""

# "READONLY You can't write against a read only replica"
# Solution: Write to master or promote replica
REPLICAOF NO ONE

# "OOM command not allowed when used memory > 'maxmemory'"
# Solution: Increase maxmemory or set eviction policy
CONFIG SET maxmemory-policy allkeys-lru

# "NOAUTH Authentication required"
# Solution: Authenticate
AUTH password

# "WRONGTYPE Operation against a key holding the wrong kind of value"
# Solution: Check key type and use appropriate commands
TYPE mykey
```

### Performance Tuning
```bash
# Optimize configuration
CONFIG SET maxmemory 1gb
CONFIG SET maxmemory-policy allkeys-lru
CONFIG SET tcp-keepalive 60
CONFIG SET timeout 300

# Optimize persistence
CONFIG SET save "900 1 300 10 60 10000"
CONFIG SET appendonly yes
CONFIG SET appendfsync everysec

# Optimize replication
CONFIG SET repl-diskless-sync yes
CONFIG SET repl-diskless-sync-delay 5

# Monitor and tune
MONITOR  # Watch commands in real-time
LATENCY DOCTOR  # Get latency analysis
redis-cli --latency-history
```

## Security

### Authentication and Authorization
```bash
# Set password
CONFIG SET requirepass "strong_password"

# Authenticate
AUTH strong_password

# Rename dangerous commands
CONFIG SET rename-command FLUSHDB ""
CONFIG SET rename-command FLUSHALL ""
CONFIG SET rename-command CONFIG "CONFIG_abc123"

# Disable commands
CONFIG SET rename-command DEBUG ""

# Use ACLs (Redis 6+)
ACL SETUSER alice on >password ~* +@all
ACL SETUSER bob on >password ~app:* +@read
ACL LIST
ACL DELUSER alice
```

### Network Security
```bash
# Bind to specific interfaces
CONFIG SET bind "127.0.0.1 192.168.1.100"

# Enable protected mode
CONFIG SET protected-mode yes

# Change default port
CONFIG SET port 6380

# Use TLS (Redis 6+)
CONFIG SET tls-port 6380
CONFIG SET port 0  # Disable non-TLS
```

---

*For more detailed information, visit the [official Redis documentation](https://redis.io/documentation)*

