
# End Node

## Description  

The **End Node** is used to structure and collect required variables from a flow.  

### `output_map` (required)  

The `output_map` defines which variables are included in the output and how they are mapped. It must be an object in the following format:  

```json
{
    "context": "variables",
    "any_name": "variables.some_node_output_path",
    "some_digit": "variables.some_node_output_path.important_digit"
}
```

- Keys in `output_map` can be named arbitrarily.  
- Values must always be strings representing a path to a specific variable, and must start with `"variables"`.  
- If the provided path points to a non-existent variable, the value will be set to `"not found"`.
- `"context"` is default field to access all variables from the flow

The output of the End Node will contain the variables under the names defined in `output_map`, enabling clear extraction and renaming of variables for downstream use.  


---

## Example

Let’s create a simple flow step by step.

### 1. Domain (Start Node) 

```python
{
    "start_number": 7
}
```

### 2. Python Node

***Input map:***

```start_number```  ```variables.start_number```


***Output variable path:***

```variables.python_node_output```

***Code:***

```python
def main(start_number: int) -> dict:
    return {
        "result_number": start_number + 100,
        "text": "Hello, World!"
    }
```

### 3. End Node

```
{
    "context": "variables",
    "important_number": "variables.python_node_output.result_number",
    "whole_python_node_output": "variables.python_node_output",
    "non_existing_value": "variables.non_existing_variable_path"
}
```

**Output Description:**

- `context` contains all variables collected during the flow execution.  
- `important_number` shows a specific extracted value.  
- `whole_python_node_output` exposes the full output of the Python Node.  
- `non_existing_value` demonstrates handling of a missing variable path (returns `"not found"`).  

---

## Results

### UI Representation

![End Node Result](images/end-node-image.png)

### API Results

To access the results of the End Node via the API, follow these steps:


#### 1.Connect to SSE Endpoint
Subscribe to the session's event stream:
```
/api/run-session/subscribe/{session_id}/
```

#### 2. Listen for Completion Event
Monitor for messages containing the flow completion signal:
```json
{
    "message_data": {
        "message_type": "graph_end"
    }
}
```

#### 3. Extract End Node Results
The final results are available in the `end_node_result` field:
```json
{
    "message_data": {
        "end_node_result": {
            // Your structured output here
        }
    }
}
```

### Example Message

```json
{
  "session_id": 1,
  "name": "",
  "execution_order": 0,
  "message_data": {
    "message_type": "graph_end",
    "end_node_result": {
      "context": {
        "start_number": 7,
        "files": {},
        "python_node_output": {
          "result_number": 107,
          "text": "Hello, World!"
        }
      },
      "important_number": 107,
      "non_existing_value": "not found",
      "whole_python_node_output": {
        "result_number": 107,
        "text": "Hello, World!"
      }
    }
  },
  "timestamp": "2025-09-25T13:20:41.616Z",
  "uuid": "694992f5-05c0-48a7-abe0-4ecba187d033"
}
```