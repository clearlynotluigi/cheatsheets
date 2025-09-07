# RabbitMQ Power Cheatsheet 🐇

> A practical, beautiful guide to managing RabbitMQ from the command line, covering `rabbitmqadmin`, `rabbitmqctl`, and direct API calls with `curl`.

This cheatsheet provides generic, copy-paste-friendly examples for common administrative and development tasks.

---
### 🤵 `rabbitmqadmin` (HTTP API CLI)
---
*A Python-based command-line tool that interacts with the RabbitMQ Management API. It's great for high-level tasks and scripting.*

#### `rabbitmqadmin publish`
*Publishes a message to an exchange.*
> The `payload` is a JSON string. Be careful with shell quoting to ensure the JSON is passed correctly.

```shell
rabbitmqadmin publish exchange=my_exchange routing_key=my.routing.key \
payload='{
    "message_id": "a-unique-id-123",
    "task": "process_data",
    "payload": {
        "user_id": 42,
        "data_source": "/path/to/data"
    }
}'
```

#### `rabbitmqadmin export`
*Exports all broker definitions (queues, exchanges, bindings, etc.) to a JSON file.*
> This is essential for backing up your RabbitMQ configuration or migrating it to another server.

```shell
# Make sure to provide your username and password
rabbitmqadmin --username=myuser --password=mypassword export definitions.json
```
> **Pro Tip:** For automated scripts, use environment variables to avoid hardcoding credentials.
```shell
rabbitmqadmin --username=$RABBITMQ_USER --password=$RABBITMQ_PASS export defs.json
```

---
### 🔌 `curl` (Direct Management API)
---
*Interact directly with the RabbitMQ Management HTTP API. This gives you the most control and is great for integration from any language.*

> **Authentication:** The API uses Basic HTTP Authentication. You need to provide a Base64-encoded string of `username:password`.
> To generate it: `echo -n 'guest:guest' | base64` which produces `Z3Vlc3Q6Z3Vlc3Q=`.

#### Get All Broker Definitions
*Retrieves a complete JSON object of all exchanges, queues, bindings, and users.*

```shell
curl -X GET -u 'guest:guest' 'http://localhost:15672/api/definitions'

# Or using the pre-generated Authorization header
curl -X GET -H 'Authorization: Basic Z3Vlc3Q6Z3Vlc3Q=' 'http://localhost:15672/api/definitions'
```

#### Publish a Message
*Publishes a message to a specific exchange in a given virtual host.*
> Note that the virtual host (`%2F` for the default `/`) must be URL-encoded in the path.

```shell
curl -X POST -u 'guest:guest' \
--header 'Content-Type: application/json' \
'http://localhost:15672/api/exchanges/%2F/my_exchange/publish' \
--data '{
    "properties": {},
    "routing_key": "my.routing.key",
    "payload": "{\"task\":\"process_data\",\"user_id\":42}",
    "payload_encoding": "string"
}'
```

#### Get Queues with Messages
*Lists all queues and filters the list to show only those with one or more messages.*
> This example requires `jq`, a powerful command-line JSON processor. It's an essential tool for working with APIs.

```shell
# The jq command selects queues where the 'messages' field is greater than 0,
# then prints the queue name and message count.
curl -s -u guest:guest http://localhost:15672/api/queues | \
  jq -r '.[] | select(.messages > 0) | "\(.name) \(.messages)"'
```

---
### 🔧 `rabbitmqctl` (Direct Broker Control)
---
*A lower-level tool that communicates directly with the RabbitMQ broker. It's often used for cluster management and bulk operations.*

#### Show Queues with at Least One Message
*Lists all queues and their message count, filtering out empty ones.*
> This uses a simple `awk` script to parse the output. `NR > 1` skips the header line, and `$2 > 0` checks if the second column (messages) is greater than zero.

```shell
rabbitmqctl list_queues name messages | awk 'NR > 1 && $2 > 0 {print $1, $2}'
```

#### Delete All Queues in a Virtual Host
*A powerful one-liner to clear all queues, for example, in a development environment.*
> **☢️ Destructive Action:** This will permanently delete queues. Be careful! `xargs` takes each queue name from the previous command and passes it to `rabbitmqadmin delete`.

```shell
# Delete all queues in the default vhost ('/')
rabbitmqctl list_queues | awk 'NR>1 {print $1}' | xargs -I qn rabbitmqadmin delete queue name=qn

# Delete all EMPTY queues in a specific vhost
rabbitmqctl list_queues -p my_vhost name messages | awk 'NR > 1 && $2 == 0 {print $1}' | \
  xargs -I qn rabbitmqadmin -u user -p pass -V my_vhost delete queue name=qn
```

#### Delete All Exchanges in a Virtual Host
*Similar to deleting queues, this command clears all exchanges except for the default, internal ones.*
> **☢️ Destructive Action:** This will permanently delete exchanges and break any applications that rely on them.

```shell
# Delete all non-default exchanges in a specific vhost
rabbitmqctl list_exchanges -p my_vhost | awk 'NR>1 {print $1}' | \
  grep -vE "^(amq\.)" | \
  xargs -I en rabbitmqadmin -u user -p pass -V my_vhost delete exchange name=en
```
