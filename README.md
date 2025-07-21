# Tài liệu tích hợp AMIS Support & CHATBOT
## Lịch sử cập nhật
- 21/7/2025: bổ sung tenant_id, khi trao đổi qua syncservice, bên support sẽ gửi thêm tenant_id, bot_id khi có trigger sang chatbot, chatbot phản hồi sẽ gửi lại tenant_id này
- 21/7/2025: bổ sung header x-bot-id khi gọi qua api
## 1. Luồng tích hợp của chatbot với livechat sẽ được chuyển đổi như sau:

### Sử dụng Sync Service thay thế các luồng sau:
- webhook:
  - /incoming_chat
  - /incoming_event
  - /chat_transferred
  - /chat_deactivated
- API:
  - /send_typing_indicator
  - /send_event
  - /tag_thread
  - /transfer_chat
  - /deactivate_chat
### Sử dụng API cho các luồng sau:
- API:
  - /list_routing_statuses
  - /get_chat


### Thông tin Sync Service
Mỗi bot 1 topic, cùng pub/sub trên topic đó, bên sub xử lý hoặc bỏ qua message theo action trong payload
#### Publisher:

AppCode: AMSUP

Topic: support_chatbot_[tên bot]

#### Subscribe:
AppCode:

Topic: support_chatbot_[tên bot]

### Thông tin xác thực API:
- CLientID:
- ClientSecret:
### Thông tin payload truyền qua syncservice
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | object       |  payload  |
|  3  | Guid       |  tenant_id  |

## 2. Chi tiết các luồng

### 2.1. /incoming_chat
Publish vào topic **support_chatbot**:
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | IncomingChatDto       |  payload  |
|  3  | Guid       |  tenant_id  |

```csharp
{
  "action": "incoming_chat",
  "tenant_id": "550e8400-e29b-41d4-a716-446655440000",
  "payload": {
    "chat": {
      "id": "1123e4567-e89b-12d3-a456-426614174000",
      "thread": {
        "id": "789e4567-e89b-12d3-a456-426614174001",
        "created_at": "2025-01-21T11:30:08Z",
        "user_ids": [
          "111e4567-e89b-12d3-a456-426614174002",
          "113e4567-e89b-12d3-a456-426614174003"
        ],
        "access": {
          "group_ids": [
            "333e4567-e89b-12d3-a456-426614174004",
            "444e4567-e89b-12d3-a456-426614174005"
          ]
        },
        "active": true,
        "previous_thread_id": "555e4567-e89b-12d3-a456-426614174006",
        "next_thread_id": null,
        "events": [
          {
            "author_id": "111e4567-e89b-12d3-a456-426614174002"
          }
        ]
      },
      "users": [
        {
          "id": "111e4567-e89b-12d3-a456-426614174002",
          "type": "customer",
          "present": true,
          "name": "Nguyễn Văn A",
          "email": "nguyenvana@example.com",
          "last_seen_time_stamp": 1642781400000,
          "created_at": "2025-01-20T10:00:00Z"
        }
      ],
      "access": {
        "group_ids": [
          "333e4567-e89b-12d3-a456-426614174008"
        ]
      }
    }
  }
}

```

### 2.2. /incoming_event
Publish vào topic **support_chatbot**:
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | IncomingEventDto       |  payload  |
|  3  | Guid       |  tenant_id  |

```csharp
{
  "action": "incoming_event",
  "payload": {
    "thread_id": "123e4567-e89b-12d3-a456-426614174000",
    "chat_id": "123e4567-e89b-12d3-a456-426614174001",
    "event": {
      "type": "message",
      ...
    }
  }
}

```

### 2.3. /chat_transferred
Publish vào topic **support_chatbot**:
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | TransferredEventDto       |  payload  |
|  3  | Guid       |  tenant_id  |
```csharp
{
  "action": "chat_transferred",
  "payload": {
    "chat_id": "123e4567-e89b-12d3-a456-426614174000",
    "thread_id": "123e4567-e89b-12d3-a456-426614174001",
    "reason": "agent_unavailable",
    "transferred_to": {
      "group_ids": [
        "123e4567-e89b-12d3-a456-426614174002"
      ],
      "agent_ids": [
        "123e4567-e89b-12d3-a456-426614174003"
      ]
    },
    "queue": {
      "position": 2,
      "wait_time": 30,
      "queued_at_us": "2025-07-17T10:33:00Z"
    }
  }
}

```

### 2.4. /chat_deactivated
Publish vào topic **support_chatbot**:
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | ChatDeactivatedEventDto       |  payload  |
|  3  | Guid       |  tenant_id  |
```json
{
  "action": "chat_deactivated",
  "payload": {
    "chat_id": "123e4567-e89b-12d3-a456-426614174000",
    "thread_id": "123e4567-e89b-12d3-a456-426614174001",
    "user_id": "123e4567-e89b-12d3-a456-426614174002"
  }
}

```
### 2.5. /send_typing_indicator
Publish vào topic **support_chatbot**:
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | TypingIndicator       |  payload  |
|  3  | Guid       |  tenant_id  |
```json
{
  "action": "send_typing_indicator",
  "payload": {
    "chat_id": "123e4567-e89b-12d3-a456-426614174000",
    "is_typing": true
  }
}
```
### 2.6. /send_event
Publish vào topic **support_chatbot**:
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | SendEvent       |  payload  |
|  3  | Guid       |  tenant_id  |
```json
{
  "action": "send_event",
  "payload": {
    "chat_id": "123e4567-e89b-12d3-a456-426614174001",
    "event": {
      "type": "message",
      ...
    }
  }
}
```

### 2.7. /tag_thread
Publish vào topic **support_chatbot**:
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | TagThread       |  payload  |
|  3  | Guid       |  tenant_id  |

```json
{
  "action": "tag_thread",
  "payload": {
    "chat_id": "123e4567-e89b-12d3-a456-426614174000",
    "thread_id": "123e4567-e89b-12d3-a456-426614174001",
    "tag": "important"
  }
}
```
### 2.8. /transfer_chat
Publish vào topic **support_chatbot**:
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | TransferChat       |  payload  |

```json
{
  "action": "transfer_chat",
  "payload": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "ignore_agents_availability": false,
    "ignore_requester_presence": false,
    "target": {
    "type": "group",
    "ids": [
      "123e4567-e89b-12d3-a456-426614174001",
      "123e4567-e89b-12d3-a456-426614174002"
     ]
    }
  }
}
```

### 2.9. /deactivate_chat
Publish vào topic **support_chatbot**:
|  | Kiểu dữ liệu      | Tên trường |
|-----|----------|------|
|  1  | string     |  action  |
|  2  | ChatDeactivated       |  payload  |
|  3  | Guid       |  tenant_id  |
```json
{
  "action": "deactivate_chat",
  "payload": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "ignore_requester_presence": false
  }
}
```

### 2.10. /list_routing_statuses

**Request: RoutingStatusReq (nullable)** 

**Xác thực:** Signature là mã hóa HMACSHA256 của body request, key là clientSecret

Nếu body null thì coi như chuỗi "" để mã hóa.
```
curl -X POST 'https://domain/test/list_routing_statuses' \
  -H 'Content-Type: application/json' \
  -H 'x-client-id: <CLIENT_ID>' \
  -H 'x-bot-id: <bot_id>' \
  -H 'x-signature: <SIGNATURE>' \
  -d '{
    "filter": {
      "group_ids": []
    }
  }'
```

**Response: List\<RoutingStatusResponse>**
```json
[
    {
        "agent_id": "123e4567-e89b-12d3-a456-426614174000",
        "status": "accepting_chats"
    }
]
```

### 2.11. /get_chat
**Request: GetChatRequest** 

**Xác thực:** Signature là mã hóa HMACSHA256 của body request, key là clientSecret

Nếu body null thì coi như chuỗi "" để mã hóa.
```
curl -X POST 'https://domain/test/get_chat' \
  -H 'Content-Type: application/json' \
  -H 'x-client-id: <CLIENT_ID>' \
  -H 'x-bot-id: <bot_id>' \
  -H 'x-signature: <SIGNATURE>' \
  -d '{
    "chat_id": "",
    "thread_id": ""
  }'
```

**Response: GetChatResponse**
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "thread": {
    "id": "123e4567-e89b-12d3-a456-426614174001",
    "created_at": "2025-07-17T10:00:00Z",
    "user_ids": [
      "123e4567-e89b-12d3-a456-426614174002",
      "123e4567-e89b-12d3-a456-426614174003"
    ]
  },
  "access": {
    "group_ids": [
      "123e4567-e89b-12d3-a456-426614174004"
    ]
  },
  "active": true,
  "previous_thread_id": null,
  "next_thread_id": null,
  "users": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174002",
      "type": "customer",
      "present": true,
      "name": "Nguyễn Văn A",
      "email": "nguyenvana@example.com",
      "last_seen_time_stamp": 1624424520000,
      "created_at": "2025-07-17T09:50:00Z"
    },
    {
      "id": "123e4567-e89b-12d3-a456-426614174003",
      "type": "agent",
      "present": true,
      "name": "Agent Support",
      "email": "agent@company.com",
      "last_seen_time_stamp": 1624424525000,
      "created_at": "2025-07-17T09:45:00Z",
      "routing_status": "available"
    }
  ],
  "access": {
    "group_ids": [
      "123e4567-e89b-12d3-a456-426614174004"
    ]
  }
}

```

## 3. Model
### Access
| Kiểu dữ liệu   | Tên trường   |
|---------------|--------------|
| List\<Guid>    | group_ids     |

### User
| Kiểu dữ liệu   | Tên trường                 |
|---------------|----------------------------|
| Guid          | id                         |
| string        | type ("customer", "agent")                       |
| bool          | present                    |
| string        | name                       |
| string        | email                      |
| long          | last_seen_time_stamp       |
| DateTime      | created_at                 |
| string        | routing_status             |
| object        | session_fields              |

### Thread
| Kiểu dữ liệu     | Tên trường         |
|------------------|--------------------|
| Guid             | id                 |
| DateTime         | created_at         |
| List\<Guid>       | user_ids           |
| Access           | access             |
| bool             | active             |
| Guid             | previous_thread_id |
| Guid             | next_thread_id     |


### Chat

| Kiểu dữ liệu       | Tên trường  |
|--------------------|-------------|
| Guid               | id          |
| Thread             | threads     |
| List\<User>         | users       |
| Access             | access      |

### FileEventRequest : Event
| Kiểu dữ liệu   | Tên trường     |
|---------------|---------------|
| Guid?         | custom_id                        |
| string        | type           |
| string        | url           |

### FileEventResponse : Event
| Kiểu dữ liệu   | Tên trường     |
|---------------|---------------|
| string        | type           |
| string        | url           |
| Guid          | id            |
| Guid          | author_id     |
| DateTime      | created_at    |
| string        | name          |
| string        | content_type  |

### MessageEventRequest : Event
| Kiểu dữ liệu   | Tên trường     |
|---------------|---------------|
| Guid?         | custom_id                        |
| string        | type           |
| string        | text          |

### MessageEventResponse : Event
| Kiểu dữ liệu   | Tên trường     |
|---------------|---------------|
| Guid          | id            |
| string        | type           |
| string        | text          |
| Guid          | author_id     |
| DateTime      | created_at    |

### Image

| Kiểu dữ liệu | Tên trường   |
|--------------|-------------|
| string       | url         |
| string       | name        |
| string       | content_type|

### Button
| Kiểu dữ liệu | Tên trường   |
|--------------|-------------|
| string       | text        |
| string       | type        |
| string       | value       |
| string       | postback_id |

### RichMessageElement
| Kiểu dữ liệu    | Tên trường |
|-----------------|------------|
| string          | title      |
| Image           | image      |
| List\<Button>    | buttons    |

### RichMessageEventRequest : Event
| Kiểu dữ liệu               | Tên trường   |
|---------------------------|--------------|
| Guid?         | custom_id                        |
| string                    | template_id  |
| string        | type           |
| List\<RichMessageElement>  | elements     |

### RichMessageEventResponse : Event
| Kiểu dữ liệu               | Tên trường   |
|---------------------------|--------------|
| Guid         | id                        |
| string                    | template_id  |
| string        | type           |
| List\<RichMessageElement>  | elements     |

### ChatDeactivated
| Kiểu dữ liệu | Tên trường                 |
|--------------|---------------------------|
| Guid         | id                        |
| bool         | ignore_requester_presence |

### SendEvent

| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Guid         | chat_id    |
| Event        | event      |

### TagThread
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Guid         | chat_id    |
| Guid         | thread_id  |
| string       | tag        |

### Target
| Kiểu dữ liệu   | Tên trường |
|---------------|------------|
| string        | type       |
| List\<Guid>    | ids        |

### TransferChat
| Kiểu dữ liệu | Tên trường                 |
|--------------|---------------------------|
| Guid         | id                        |
| bool         | ignore_agents_availability|
| bool         | ignore_requester_presence |
| Target       | target                    |

### TypingIndicator
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Guid         | chat_id    |
| bool         | is_typing  |


### GetChatRequest
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Guid         | chat_id     |
| Guid         | thread_id   |
### GetChatResponse
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Guid         | id         |
| Thread       | thread     |
| List<User>   | users      |
| Access       | access     |
### Filter
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| List\<Guid>   | group_ids  |
### RoutingStatusReq
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Filter       | filter     |

### RoutingStatusResponse
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Guid         | agent_id   |
| string       | status     |

### ChatDeactivatedEventPayload
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Guid         | chat_id    |
| Guid         | thread_id  |
| Guid         | user_id    |
### ChatDeactivatedEventDto
| Kiểu dữ liệu                  | Tên trường |
|-------------------------------|------------|
| string                        | action     |
| ChatDeactivatedEventPayload   | payload    |


### IncomingChatPayload
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Chat         | chat       |
### IncomingChatDto
| Kiểu dữ liệu         | Tên trường |
|---------------------|------------|
| string              | action     |
| IncomingEventPayload| payload    |

### IncomingEventPayload
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| Guid         | thread_id  |
| Guid         | chat_id    |
| Event        | event      |
### IncomingEventDto
| Kiểu dữ liệu           | Tên trường |
|-----------------------|------------|
| string                | action     |
| IncomingEventPayload  | payload    |

### TransferTarget
| Kiểu dữ liệu | Tên trường |
|--------------|------------|
| List<Guid>   | group_ids  |
| List<Guid>   | agent_ids  |
### QueueInfo
| Kiểu dữ liệu | Tên trường    |
|--------------|--------------|
| int          | position     |
| int          | wait_time    |
| DateTime     | queued_at_us |

### TransferredEventPayload
| Kiểu dữ liệu       | Tên trường      |
|-------------------|-----------------|
| Guid              | chat_id         |
| Guid              | thread_id       |
| string            | reason          |
| TransferTarget    | transferred_to  |
| QueueInfo         | queue           |
### TransferredEventDto
| Kiểu dữ liệu             | Tên trường |
|-------------------------|------------|
| string                  | action     |
| TransferredEventPayload | payload    |



























