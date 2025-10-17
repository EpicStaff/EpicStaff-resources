# 📘 Organization Variables Documentation

## **Overview**

Organization Variables allow you to manage persistent state across flow executions for organizations and their users. Each organization and user can maintain their own set of variables that persist between sessions when enabled.

---

## **API Endpoints**

### Organization Endpoints
```bash
GET    /api/organizations/
POST   /api/organizations/
GET    /api/organizations/{id}/
PUT    /api/organizations/{id}/
PATCH  /api/organizations/{id}/
DELETE /api/organizations/{id}/
PATCH  /api/organizations/{id}/change_secret_key/
```

### Organization User Endpoints
```bash
GET    /api/organization-users/
POST   /api/organization-users/
GET    /api/organization-users/{id}/
PUT    /api/organization-users/{id}/
PATCH  /api/organization-users/{id}/
DELETE /api/organization-users/{id}/
PATCH  /api/organization-users/{id}/change_secret_key/
```

---

## **Key Concepts**

### Organization Properties
| Property              | Type    | Description |
|-----------------------|---------|-------------|
| `name`                | string  | Unique organization identifier |
| `secret_key`          | string  | Authentication key for API operations |
| `variables`           | object  | Key-value pairs stored for the organization |
| `persistent_variables`| boolean | If `true`, variables are saved after flow execution |

### Organization User Properties
| Property              | Type    | Description |
|-----------------------|---------|-------------|
| `username`            | string  | Unique user identifier within organization |
| `secret_key`          | string  | Authentication key for API operations |
| `variables`           | object  | Key-value pairs stored for the user |
| `persistent_variables`| boolean | If `true`, variables are saved after flow execution |

---

## **Authentication**

### Secret Key Requirements
- All endpoints that modify data require the `secret_key` to be provided
- To change a secret key, you must provide both the old and new secret keys

### Changing Secret Key

**Request**
```bash
PATCH /api/organizations/{id}/change_secret_key/
PATCH /api/organization-users/{id}/change_secret_key/
```

**Headers**
| Name            | Value              | Required |
|-----------------|-------------------|----------|
| `Content-Type`  | `application/json` | ✅       |

**Body Parameters**
| Field              | Type   | Required | Description |
|-------------------|--------|----------|-------------|
| `old_secret_key`  | string | ✅       | Current secret key |
| `new_secret_key`  | string | ✅       | New secret key to set |

---

## **Using Variables in Flow Execution**

### Run Session Endpoint
```bash
POST /api/run-session/
```

### Headers
| Name            | Value                 | Required | Notes                   |
|-----------------|-----------------------|----------|-------------------------|
| `Content-Type`  | `multipart/form-data` | ✅       | Required for file uploads |

### Body Parameters
| Field              | Type        | Required | Description |
|-------------------|-------------|----------|-------------|
| `graph_id`        | integer     | ✅       | ID of the flow to run |
| `variables`       | JSON object | ❌       | Key/value pairs used as runtime variables |
| `organization_data` | JSON object | ❌     | Organization credentials |
| `organization_user_data`       | JSON object | ❌       | User credentials |

### Organization Data Format
```json
{
  "name": "organization_name",
  "secret_key": "organization_secret_key"
}
```

### User Data Format
```json
{
  "username": "user_username",
  "secret_key": "user_secret_key"
}
```

---

## **Example Requests**

### Using `curl`

**With Organization Variables**
```bash
curl -X POST http://localhost:8000/api/run-session/ \
  -F "graph_id=123" \
  -F "variables={\"var1\": \"my_value\"}" \
  -F "organization_data={\"name\": \"acme_corp\", \"secret_key\": \"org_secret_123\"}"
```

**With User Variables**
```bash
curl -X POST http://localhost:8000/api/run-session/ \
  -F "graph_id=123" \
  -F "variables={\"var1\": \"my_value\"}" \
  -F "organization_user_data={\"username\": \"john_doe\", \"secret_key\": \"user_secret_456\"}"
```

**With Both Organization and User Variables**
```bash
curl -X POST http://localhost:8000/api/run-session/ \
  -F "graph_id=123" \
  -F "variables={\"var1\": \"my_value\"}" \
  -F "organization_data={\"name\": \"acme_corp\", \"secret_key\": \"org_secret_123\"}" \
  -F "organization_user_data={\"username\": \"john_doe\", \"secret_key\": \"user_secret_456\"}" \
  -F "file1=@example.txt"
```

### Using Python

**With Organization Variables**
```python
import requests

url = "http://localhost:8000/api/run-session/"
data = {
    "graph_id": 123,
    "variables": '{"var1": "my_value"}',
    "organization_data": '{"name": "acme_corp", "secret_key": "org_secret_123"}'
}

response = requests.post(url, data=data)
print(response.json())
```

**With User Variables**
```python
import requests

url = "http://localhost:8000/api/run-session/"
data = {
    "graph_id": 123,
    "variables": '{"var1": "my_value"}',
    "organization_user_data": '{"username": "john_doe", "secret_key": "user_secret_456"}'
}

response = requests.post(url, data=data)
print(response.json())
```

---

## **Accessing Variables in Flow**

### Organization Variables
If valid organization credentials are provided, variables will be available at:
```
variables.organization
```

### User Variables
If valid user credentials are provided, variables will be available at:
```
variables.user
```
---

## **Persistent Variables**

### How It Works
When `persistent_variables` is set to `true`:
1. Variables are loaded at the start of flow execution
2. Variables can be modified during flow execution with output_variable_path 
3. **At the end of the flow**, the modified variables are automatically saved back to the database

### Example Workflow
1. Organization starts with: `{"api_calls": 0, "credits": 100}`
2. During flow execution, increment: `variables.organization.api_calls += 1`
3. Return new value of `api_calls` in python node and set `output_variable_path` to `variables.organization.api_calls`
3. If `persistent_variables = true`, the database is updated with: `{"api_calls": 1, "credits": 100}`
4. Any other flow execution will start with the updated values

---

## **Important Notes**

📌 **Content Type**
- `/api/run-session/` endpoint uses `multipart/form-data`
- All other endpoints use `application/json`


📌 **Variable Persistence**
- Only enabled when `persistent_variables = true`
- Variables are saved **after** successful flow completion
- Failed flows do not persist variable changes

📌 **Security**
- Keep secret keys confidential
- Rotate secret keys regularly using the change secret key endpoint
- Each organization and user has independent authentication

---
