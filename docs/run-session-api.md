
# 📘 Run Session API

## **Endpoint**

```bash
POST /api/run-session/
```

## **Description**
Starts a new session for a given flow.  

- Accepts **flow variables** and **files**.  
- Returns a unique `session_id` for tracking.  
- Enforces a **15 MB** total file size limit.
- **Flow variables** have higher priority over flow domain variables and will override those that has same key.

---

## **Request**

### Headers
| Name            | Value                  | Required | Notes                              |
|-----------------|------------------------|----------|------------------------------------|
| `Content-Type`  | `multipart/form-data`  | ✅       | Needed for file uploads            |

### Body Parameters
| Field       | Type         | Required | Description |
|-------------|-------------|----------|-------------|
| `graph_id`  | integer     | ✅        | ID of the flow to run |
| `variables` | JSON object | ❌        | Key/value pairs used as runtime variables |
| `files`     | file(s)     | ❌        | One or more files (e.g., `.txt`, `.csv`) |

📌 You can upload multiple files using unique names (`file1`, `file2`, …).

---

## **Example Requests**

### Using `curl`
```bash
curl -X POST http://localhost:8000/api/run-session/ \
  -F "graph_id=123" \
  -F "variables={\"var1\": \"my_value\"}" \
  -F "file1=@example.txt" \
  -F "file2=@example.csv"
```

### Using Python
```py
import requests

url = "http://localhost:8000/api/run-session/"

data = {"graph_id": 123, "variables": '{"var1": "my_value"}'}
files = {
    "file1": ("example.txt", open("example.txt", "rb"), "text/plain"),
    "file2": ("example.csv", open("example.csv", "rb"), "text/csv"),
}

response = requests.post(url, data=data, files=files)
```

## **Usage in Flow**

- Files' data will be avaliable by using `variables.files`, which is `DotDict` object that consists of file objects.
- To access each file individualy you can use `variables.files.{key}` where `{key}` is form key for this file.
- Each file object has 3 variables:
    - **data** - base64 encoded file data
    - **name** - file name
    - **content_type** - a MIME type of the file content
