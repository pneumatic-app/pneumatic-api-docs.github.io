
# Introduction

Welcome to the Pneumatic API! You can use our API to access Pneumatic public API endpoints, which can get information on workflows and run processes.

All the examples in the dark area are provided in Python code.

# Authentication

> Use bearer token authentication for each API request:

```python
import requests

api_key = 'your_api_key'
headers = { 'Authorization': f'Bearer {api_key}' }
response = requests.get('https://api.pneumatic.app/some-endpoint', headers=headers)
```


> Make sure to replace `your_api_key` with your API key from [profile page](https://my.pneumatic.app/settings/profile/).

Pneumatic expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer api_key`

<aside class="notice">
You must replace <code>api_key</code> with your personal API key.
</aside>

# Workflows

## Get All Workflows

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}

r = requests.get('https://api.pneumatic.app/workflows', headers=headers)
```

> The above command returns JSON structured like this:

```json
[
  {
      "id": 1,
      "name": "Workflow name",
      "is_active": true,
      "description": "Description of this workflow",
      "performers_count": 3,
      "tasks_count": 6,
      "run_allowed": [
         {
            "id": 1,
            "email": "john.doe@example.com",
            "first_name": "John",
            "last_name": "Doe",
            "photo": "https://example.com/avatar.jpeg",
            "status": "active"
         }
      ],
      "urls": [
         "trello.com",
         "gmail.com",
         "pandadocs.com",
         "zoom.us",
         "docusign.com"
      ],
      "kickoff": {
         "id": 1,
         "description": "Kick-off form description",
         "fields": [
            {
              "name": "Field name",
              "description": "Field description",
              "type": "string",
              "is_required": false, 
              "api_name": "api-name-1",
              "selections": []
            }
         ]
      }
  },
  ...
]
```

This endpoint retrieves all workflows in your account.

### HTTP Request

`GET https://api.pneumatic.app/workflows`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
ordering | -date | Available values: date, name, -date, -name.

<aside class="success">
Minus before the value will sort in descending order.
</aside>

## Get a Specific Workflow


```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
workflow_id = 1

r = requests.get(f'https://api.pneumatic.app/workflows/{workflow_id}', headers=headers)
```

> The above command returns JSON structured like this:

```json
{
    "id": 1,
    "tasks_count": 6,
    "name": "Workflow name",
    "description": "Workflow description",
    "is_active": false,
    "finalizable": false,
    "run_allowed": [
        {
            "id": 1,
            "email": "john.doe@example.com",
            "first_name": "John",
            "last_name": "Doe",
            "photo": "https://example.com/avatar.jpeg",
            "status": "active"
        }
    ],
    "tasks": [
        {
            "id": 1,
            "name": "Name of the first task of the workflow",
            "description": "Its description",
            "number": 1,
            "responsible": [],
            "url": null,
            "require_completion_by_all": false
        },
        {
            "id": 2,
            "name": "Second step of the process",
            "description": null,
            "number": 2,
            "responsible": [
                {
                    "id": 1,
                    "email": "john.doe@example.com",
                    "first_name": "John",
                    "last_name": "Doe",
                    "photo": "https://example.com/avatar.jpeg",
                    "status": "active"
                }
            ],
            "url": "https://www.example.com/",
            "require_completion_by_all": false
        },
        ...
    ],
    "kickoff": {
        "id": 1,
        "description": "Kick-off description",
        "fields": [
            {
                "name": "Text Field",
                "type": "string",
                "is_required": false,
                "description": "Field description",
                "api_name": "field-1"
            },
            {
                "name": "Radio Buttons Field",
                "type": "radio",
                "is_required": false,
                "description": "Radio Field Description",
                "api_name": "radio-field-2",
                "selections": [
                    {
                        "id": 1,
                        "value": "Option 1"
                    },
                    {
                        "id": 2,
                        "value": "Option 2"
                    }
                ]
            },
            {
                "name": "Checkbox Field",
                "type": "checkbox",
                "is_required": false,
                "description": "",
                "api_name": "checkbox-field-3",
                "selections": [
                    {
                        "id": 1,
                        "value": "Option 1"
                    },
                    {
                        "id": 2,
                        "value": "Option 2"
                    }
                ]
            },
            ...
        ]
    }
}
```

This endpoint retrieves a specific workflow.

### HTTP Request

`GET https://api.pneumatic.app/workflows/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the workflow to retrieve


## Launch a Process Based on Specific Workflow

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
workflow_id = :workflow_id
process_info = {
    'name': 'My Process Name',
    'kickoff': {
        'text-field-api-name': 'Text Field Value',
        'radio-field-api-name': :value_id,
        'checkbox-field-api-name': [ :value_1_id, :value_2_id ]
    },
    'tasks': {
        :task_id: {
            'url': 'https://overwritten-url.com/'
        }    
    }
}

r = requests.post(
    f'https://api.pneumatic.app/workflows/{workflow_id}/run', 
    headers=headers,
    data=process_info
)
```

> The above command returns JSON structured like this:

```json
{
  "process_id": 1
}
```

This endpoint launches a new process based on specific workflow.

### HTTP Request

`POST https://api.pneumatic.app/workflows/<ID>/run`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the workflow to launch


### Body Parameters

Parameter | Required | Description
--------- | --------- | -----------
name | Yes | The name of the process you'd like to launch
tasks | No | Used if needed to overwrite some workflow's tasks properties designed by owner. For now, only `url` supported.
kickoff | Yes if there are required fields in kickoff | Dictionary where the keys are API names of kickoff fields. Checkbox and radio fields take as values IDs of selections. Retrieve a workflow to get these IDs.
