# n8n Cheatsheet

## Installation & Setup

### Installation Methods
```bash
# NPM (Global)
npm install n8n -g

# NPX (No install required)
npx n8n

# Docker
docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n

# Docker with persistence
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n

# Docker Compose
version: '3.8'
services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=password
    volumes:
      - ~/.n8n:/home/node/.n8n
```

### Running n8n
```bash
# Start n8n
n8n start

# Start with custom port
n8n start --port 3000

# Start in tunnel mode (for webhooks)
n8n start --tunnel

# Start with specific workflow
n8n start --file workflow.json

# Execute workflow from CLI
n8n execute --file workflow.json

# Import workflow
n8n import:workflow --input workflow.json

# Export workflow
n8n export:workflow --id 1 --output workflow.json
```

## Environment Variables

### Basic Configuration
```bash
# Database
export DB_TYPE=postgresdb
export DB_POSTGRESDB_HOST=localhost
export DB_POSTGRESDB_PORT=5432
export DB_POSTGRESDB_DATABASE=n8n
export DB_POSTGRESDB_USER=n8n
export DB_POSTGRESDB_PASSWORD=password

# Basic Auth
export N8N_BASIC_AUTH_ACTIVE=true
export N8N_BASIC_AUTH_USER=admin
export N8N_BASIC_AUTH_PASSWORD=securepassword

# Webhook URL
export WEBHOOK_URL=https://your-domain.com

# Security
export N8N_JWT_AUTH_ACTIVE=true
export N8N_JWT_AUTH_HEADER=authorization
export N8N_ENCRYPTION_KEY=your-encryption-key

# Paths
export N8N_USER_FOLDER=/home/node/.n8n
export N8N_CUSTOM_EXTENSIONS=/home/node/.n8n/custom

# Logging
export N8N_LOG_LEVEL=info
export N8N_LOG_OUTPUT=console,file
export N8N_LOG_FILE_LOCATION=/var/log/n8n.log
```

### Advanced Configuration
```bash
# Performance
export EXECUTIONS_PROCESS=main
export EXECUTIONS_TIMEOUT=3600
export EXECUTIONS_TIMEOUT_MAX=86400
export EXECUTIONS_DATA_SAVE_ON_ERROR=all
export EXECUTIONS_DATA_SAVE_ON_SUCCESS=none

# Email (SMTP)
export N8N_EMAIL_MODE=smtp
export N8N_SMTP_HOST=smtp.gmail.com
export N8N_SMTP_PORT=587
export N8N_SMTP_USER=your-email@gmail.com
export N8N_SMTP_PASS=your-app-password
export N8N_SMTP_SSL=false

# Metrics
export N8N_METRICS=true
export N8N_METRICS_PREFIX=n8n_

# Disable certain features
export N8N_DISABLE_UI=false
export N8N_DISABLE_PRODUCTION_MAIN_PROCESS=false
```

## Core Concepts

### Workflow Structure
```json
{
  "name": "My Workflow",
  "nodes": [
    {
      "parameters": {},
      "name": "Start",
      "type": "n8n-nodes-base.start",
      "typeVersion": 1,
      "position": [240, 300]
    }
  ],
  "connections": {},
  "active": false,
  "settings": {
    "saveDataErrorExecution": "all",
    "saveDataSuccessExecution": "all",
    "saveManualExecutions": false
  },
  "staticData": {},
  "tags": [],
  "triggerCount": 0,
  "updatedAt": "2023-01-01T00:00:00.000Z",
  "versionId": "uuid"
}
```

### Node Types
- **Trigger Nodes**: Start workflows (Webhook, Cron, Manual)
- **Regular Nodes**: Process data (HTTP Request, Set, Function)
- **Credential Nodes**: Store authentication data

## Common Node Examples

### HTTP Request Node
```json
{
  "parameters": {
    "url": "https://api.example.com/users",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "httpBasicAuth",
    "options": {
      "response": {
        "response": {
          "responseFormat": "json"
        }
      }
    }
  },
  "name": "HTTP Request",
  "type": "n8n-nodes-base.httpRequest"
}
```

### Webhook Trigger
```json
{
  "parameters": {
    "httpMethod": "POST",
    "path": "webhook-path",
    "responseMode": "responseNode",
    "options": {
      "noResponseBody": true
    }
  },
  "name": "Webhook",
  "type": "n8n-nodes-base.webhook"
}
```

### Cron Trigger
```json
{
  "parameters": {
    "rule": {
      "interval": [
        {
          "field": "cronExpression",
          "expression": "0 9 * * 1-5"
        }
      ]
    }
  },
  "name": "Cron",
  "type": "n8n-nodes-base.cron"
}
```

### Function Node
```javascript
// Function node example
const items = $input.all();

for (const item of items) {
  // Process each item
  item.json.processedAt = new Date().toISOString();
  item.json.upperCaseName = item.json.name.toUpperCase();
}

return items;
```

### Set Node
```json
{
  "parameters": {
    "values": {
      "string": [
        {
          "name": "status",
          "value": "completed"
        }
      ],
      "number": [
        {
          "name": "count",
          "value": 1
        }
      ],
      "boolean": [
        {
          "name": "isActive",
          "value": true
        }
      ]
    }
  },
  "name": "Set",
  "type": "n8n-nodes-base.set"
}
```

## Expressions & Functions

### Basic Expressions
```javascript
// Access input data
{{ $json.fieldName }}
{{ $json["field-with-dashes"] }}

// Access node data
{{ $node["NodeName"].json.field }}
{{ $('NodeName').item.json.field }}

// Environment variables
{{ $env.API_KEY }}

// Current date/time
{{ $now }}
{{ $today }}

// Item index
{{ $itemIndex }}

// Workflow data
{{ $workflow.id }}
{{ $workflow.name }}
{{ $workflow.active }}

// Execution data
{{ $execution.id }}
{{ $execution.mode }}
{{ $execution.resumeUrl }}
```

### String Functions
```javascript
// String manipulation
{{ $json.name.toUpperCase() }}
{{ $json.email.toLowerCase() }}
{{ $json.text.trim() }}
{{ $json.text.substring(0, 10) }}
{{ $json.text.replace('old', 'new') }}
{{ $json.text.split(',') }}

// String checks
{{ $json.email.includes('@') }}
{{ $json.name.startsWith('John') }}
{{ $json.name.endsWith('Smith') }}

// String length
{{ $json.text.length }}
```

### Date Functions
```javascript
// Date formatting
{{ $now.format('YYYY-MM-DD') }}
{{ $now.format('HH:mm:ss') }}
{{ $now.format('MMMM Do YYYY, h:mm:ss a') }}

// Date manipulation
{{ $now.plus({ days: 1 }) }}
{{ $now.minus({ hours: 2 }) }}
{{ $now.startOf('day') }}
{{ $now.endOf('month') }}

// Date comparisons
{{ $now.isAfter('2023-01-01') }}
{{ $now.isBefore($json.deadline) }}
{{ $now.isSame($json.date, 'day') }}

// Parse dates
{{ DateTime.fromISO($json.dateString) }}
{{ DateTime.fromFormat($json.date, 'dd/MM/yyyy') }}
```

### Math Functions
```javascript
// Basic math
{{ $json.price * 1.2 }}
{{ Math.round($json.value) }}
{{ Math.floor($json.amount) }}
{{ Math.ceil($json.score) }}
{{ Math.abs($json.difference) }}

// Min/Max
{{ Math.min($json.a, $json.b, $json.c) }}
{{ Math.max($json.scores) }}

// Random
{{ Math.random() }}
{{ Math.floor(Math.random() * 100) }}
```

### Array Functions
```javascript
// Array operations
{{ $json.items.length }}
{{ $json.items.join(', ') }}
{{ $json.items.includes('target') }}
{{ $json.items.indexOf('item') }}

// Array methods
{{ $json.numbers.map(x => x * 2) }}
{{ $json.items.filter(item => item.active) }}
{{ $json.numbers.reduce((sum, num) => sum + num, 0) }}
{{ $json.items.find(item => item.id === 123) }}
{{ $json.items.sort((a, b) => a.name.localeCompare(b.name)) }}

// Array slicing
{{ $json.items.slice(0, 5) }}
{{ $json.items.slice(-3) }}
```

### Conditional Logic
```javascript
// Ternary operator
{{ $json.score > 80 ? 'Pass' : 'Fail' }}

// If function
{{ $if($json.active, 'Active', 'Inactive') }}

// Multiple conditions
{{ $json.score >= 90 ? 'A' : $json.score >= 80 ? 'B' : 'C' }}

// Null checks
{{ $json.field || 'default value' }}
{{ $json.field ?? 'default for null/undefined' }}
```

## Workflow Patterns

### Error Handling
```json
{
  "parameters": {
    "conditions": {
      "string": [
        {
          "value1": "{{ $json.error }}",
          "operation": "exists"
        }
      ]
    }
  },
  "name": "Check for Error",
  "type": "n8n-nodes-base.if"
}
```

### Data Transformation
```javascript
// Transform data structure
const items = $input.all();
const transformed = [];

for (const item of items) {
  transformed.push({
    json: {
      id: item.json.user_id,
      name: `${item.json.first_name} ${item.json.last_name}`,
      email: item.json.email_address,
      createdAt: new Date(item.json.created_timestamp).toISOString()
    }
  });
}

return transformed;
```

### Batch Processing
```javascript
// Process items in batches
const items = $input.all();
const batchSize = 10;
const batches = [];

for (let i = 0; i < items.length; i += batchSize) {
  const batch = items.slice(i, i + batchSize);
  batches.push({
    json: {
      items: batch.map(item => item.json),
      batchNumber: Math.floor(i / batchSize) + 1
    }
  });
}

return batches;
```

### Rate Limiting
```javascript
// Add delay between requests
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

const items = $input.all();
const processed = [];

for (let i = 0; i < items.length; i++) {
  const item = items[i];
  
  // Process item
  processed.push(item);
  
  // Wait before next item (except for last item)
  if (i < items.length - 1) {
    await delay(1000); // 1 second delay
  }
}

return processed;
```

## Common Integrations

### Slack
```json
{
  "parameters": {
    "authentication": "oAuth2",
    "select": "channel",
    "channelId": {
      "__rl": true,
      "value": "C1234567890",
      "mode": "list"
    },
    "text": "Hello from n8n!",
    "otherOptions": {
      "username": "n8n Bot"
    }
  },
  "name": "Slack",
  "type": "n8n-nodes-base.slack"
}
```

### Gmail
```json
{
  "parameters": {
    "operation": "send",
    "toEmail": "{{ $json.email }}",
    "subject": "Welcome!",
    "message": "Welcome to our platform!",
    "options": {
      "ccEmail": "admin@company.com",
      "attachments": "attachment"
    }
  },
  "name": "Gmail",
  "type": "n8n-nodes-base.gmail"
}
```

### Google Sheets
```json
{
  "parameters": {
    "operation": "append",
    "documentId": {
      "__rl": true,
      "value": "1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms",
      "mode": "list"
    },
    "sheetName": {
      "__rl": true,
      "value": "Sheet1",
      "mode": "list"
    },
    "columns": {
      "mappingMode": "defineBelow",
      "value": {
        "Name": "{{ $json.name }}",
        "Email": "{{ $json.email }}",
        "Date": "{{ $now.format('YYYY-MM-DD') }}"
      }
    }
  },
  "name": "Google Sheets",
  "type": "n8n-nodes-base.googleSheets"
}
```

### MySQL/PostgreSQL
```json
{
  "parameters": {
    "operation": "insert",
    "table": "users",
    "columns": "name,email,created_at",
    "additionalFields": {
      "mode": "defineBelow",
      "values": {
        "name": "{{ $json.name }}",
        "email": "{{ $json.email }}",
        "created_at": "{{ $now }}"
      }
    }
  },
  "name": "MySQL",
  "type": "n8n-nodes-base.mySql"
}
```

## CLI Commands

### Workflow Management
```bash
# List workflows
n8n list:workflow

# Export workflow
n8n export:workflow --id=1 --output=./backup/
n8n export:workflow --all --output=./backup/

# Import workflow
n8n import:workflow --input=./workflow.json
n8n import:workflow --separate --input=./workflows/

# Execute workflow
n8n execute --id=1
n8n execute --file=./workflow.json

# Update workflow
n8n update:workflow --id=1 --file=./updated-workflow.json
```

### Credential Management
```bash
# List credentials
n8n list:credential

# Export credentials
n8n export:credential --id=1 --output=./backup/
n8n export:credential --all --output=./backup/

# Import credentials
n8n import:credential --input=./credential.json
```

### Database Operations
```bash
# Reset database
n8n db:revert

# Migrate database
n8n db:migrate

# Database schema info
n8n db:info
```

## Docker & Production

### Docker Compose for Production
```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
      - WEBHOOK_URL=https://your-domain.com
      - GENERIC_TIMEZONE=UTC
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - postgres
    networks:
      - n8n-network

  postgres:
    image: postgres:15
    restart: unless-stopped
    environment:
      - POSTGRES_DB=n8n
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - n8n-network

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - n8n
    networks:
      - n8n-network

volumes:
  n8n_data:
  postgres_data:

networks:
  n8n-network:
    driver: bridge
```

### Nginx Configuration
```nginx
events {
    worker_connections 1024;
}

http {
    upstream n8n {
        server n8n:5678;
    }

    server {
        listen 80;
        server_name your-domain.com;
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name your-domain.com;

        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;

        location / {
            proxy_pass http://n8n;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # WebSocket support
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
    }
}
```

## Monitoring & Debugging

### Logging Configuration
```bash
# Environment variables for logging
export N8N_LOG_LEVEL=debug
export N8N_LOG_OUTPUT=console,file
export N8N_LOG_FILE_LOCATION=/var/log/n8n/
export N8N_LOG_FILE_COUNT_MAX=10
export N8N_LOG_FILE_SIZE_MAX=10485760
```

### Health Checks
```bash
# Health check endpoint
curl http://localhost:5678/healthz

# Metrics endpoint (if enabled)
curl http://localhost:5678/metrics
```

### Debugging Workflows
```javascript
// Debug function node
console.log('Debug info:', $json);
console.log('Node name:', $node.name);
console.log('Item index:', $itemIndex);
console.log('Workflow ID:', $workflow.id);

// Return debug data
return [{
  json: {
    debug: true,
    originalData: $json,
    nodeInfo: {
      name: $node.name,
      type: $node.type
    },
    executionInfo: {
      workflowId: $workflow.id,
      executionId: $execution.id,
      itemIndex: $itemIndex
    }
  }
}];
```

## Security Best Practices

### Authentication Setup
```bash
# Basic Auth
export N8N_BASIC_AUTH_ACTIVE=true
export N8N_BASIC_AUTH_USER=admin
export N8N_BASIC_AUTH_PASSWORD=strongpassword123

# JWT Auth
export N8N_JWT_AUTH_ACTIVE=true
export N8N_JWT_AUTH_HEADER=authorization

# Encryption
export N8N_ENCRYPTION_KEY=your-32-character-encryption-key
```

### Secure Credentials
```json
{
  "name": "API Credential",
  "type": "httpHeaderAuth",
  "data": {
    "name": "Authorization",
    "value": "Bearer {{ $env.API_TOKEN }}"
  }
}
```

### Network Security
```yaml
# Docker network isolation
networks:
  n8n-internal:
    driver: bridge
    internal: true
  n8n-external:
    driver: bridge
```

## Performance Optimization

### Execution Settings
```bash
# Process settings
export EXECUTIONS_PROCESS=main
export EXECUTIONS_TIMEOUT=3600
export EXECUTIONS_TIMEOUT_MAX=86400

# Data retention
export EXECUTIONS_DATA_SAVE_ON_ERROR=all
export EXECUTIONS_DATA_SAVE_ON_SUCCESS=none
export EXECUTIONS_DATA_MAX_AGE=336  # 14 days

# Performance
export N8N_PAYLOAD_SIZE_MAX=16777216  # 16MB
export N8N_METRICS=true
```

### Workflow Optimization Tips
1. **Use Set nodes** to reduce data payload size
2. **Implement proper error handling** with IF nodes
3. **Use Function nodes** for complex data transformations
4. **Minimize HTTP requests** by batching operations
5. **Set appropriate timeouts** for long-running operations

## Backup & Migration

### Backup Script
```bash
#!/bin/bash

# Create backup directory
BACKUP_DIR="/backup/n8n/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Export workflows
n8n export:workflow --all --output="$BACKUP_DIR/workflows"

# Export credentials
n8n export:credential --all --output="$BACKUP_DIR/credentials"

# Backup database (PostgreSQL)
pg_dump -h localhost -U n8n n8n > "$BACKUP_DIR/database.sql"

# Backup configuration
cp -r ~/.n8n "$BACKUP_DIR/config"

echo "Backup completed: $BACKUP_DIR"
```

### Migration Script
```bash
#!/bin/bash

BACKUP_DIR="$1"

if [ -z "$BACKUP_DIR" ]; then
  echo "Usage: $0 <backup_directory>"
  exit 1
fi

# Restore database
psql -h localhost -U n8n n8n < "$BACKUP_DIR/database.sql"

# Import workflows
n8n import:workflow --separate --input="$BACKUP_DIR/workflows"

# Import credentials
n8n import:credential --separate --input="$BACKUP_DIR/credentials"

echo "Migration completed from: $BACKUP_DIR"
```

## Troubleshooting

### Common Issues
```bash
# Check n8n status
systemctl status n8n

# View logs
journalctl -u n8n -f
tail -f /var/log/n8n/n8n.log

# Check disk space
df -h
du -sh ~/.n8n/

# Check memory usage
free -h
ps aux | grep n8n

# Check network connectivity
curl -I http://localhost:5678/healthz
telnet localhost 5678
```

### Database Issues
```bash
# Check database connection
psql -h localhost -U n8n -d n8n -c "SELECT version();"

# Check database size
psql -h localhost -U n8n -d n8n -c "SELECT pg_size_pretty(pg_database_size('n8n'));"

# Cleanup old executions
psql -h localhost -U n8n -d n8n -c "DELETE FROM execution_entity WHERE startedAt < NOW() - INTERVAL '30 days';"
```

### Reset and Recovery
```bash
# Reset n8n (removes all data)
n8n db:reset

# Rebuild database
n8n db:migrate

# Clear cache
rm -rf ~/.n8n/cache/

# Reset user settings
rm ~/.n8n/config
```

## Tips & Best Practices

### Workflow Design
1. **Use descriptive node names** for better readability
2. **Add notes** to complex nodes for documentation
3. **Group related operations** in sub-workflows
4. **Implement proper error handling** at each step
5. **Test with sample data** before activating

### Performance
1. **Limit data passing** between nodes
2. **Use appropriate node types** for each task
3. **Implement rate limiting** for API calls
4. **Monitor execution times** and optimize bottlenecks
5. **Use Set nodes** to transform data efficiently

### Security
1. **Store sensitive data** in credentials
2. **Use environment variables** for configuration
3. **Implement proper authentication** on webhooks
4. **Regularly update** n8n and dependencies
5. **Monitor access logs** for suspicious activity

### Maintenance
1. **Regular backups** of workflows and data
2. **Monitor disk usage** and clean old executions
3. **Update n8n** regularly for security patches
4. **Document workflows** for team collaboration
5. **Set up monitoring** and alerting for critical workflows
```

