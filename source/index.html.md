
# Introduction

Welcome to the Pneumatic API! You can use our API to access Pneumatic public API endpoints, which can get information on templates and run workflows.

All the examples in the dark area are provided in Python code.

# Authentication

> Use bearer token authentication for each API request:

```python
import requests

api_key = 'your_api_key'
headers = { 'Authorization': f'Bearer {api_key}' }
response = requests.get('https://api.pneumatic.app/some-endpoint', headers=headers)
```


> Make sure to replace `your_api_key` with your API key from [Integrations page](https://my.pneumatic.app/integrations/).

Pneumatic expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer api_key`

<aside class="notice">
You must replace <code>api_key</code> with your personal API key.
</aside>

# Templates

## Get All Templates
<a id="template-list"></a> 

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}

r = requests.get('https://api.pneumatic.app/templates', headers=headers)
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Template name",
    "is_active": true,
    "description": "Description of this template",
    "performers_count": 3,
    "tasks_count": 6,
    "run_allowed": [
       {
          "id": 2,
          "email": "john.doe@example.com",
          "first_name": "John",
          "last_name": "Doe",
          "photo": "https://example.com/avatar.jpeg",
          "status": "active",
          "invite": { // nullable
             "id": "some UUID",
             "invited_by": 1,
             "date_created": "2020-04-15T06:50:12.682106Z",
             "invited_from": "email", // slack, google
             "by_username": "Pneumatic Owner"
          }
       }
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
            "api_name": "field-name-1",
            "selections": [],
            "order": 1
          },
          {
            "name": "Checkbox Field",
            "description": "Field description",
            "type": "checkbox",
            "is_required": false, 
            "api_name": "checkbox-field-2",
            "selections": [
              {"id": 1, "value": "checkbox value 1"},
              {"id": 2, "value": "checkbox value 2"},
              {"id": 3, "value": "checkbox value 3"}
            ],
            "order": 0
          }
       ]
    },
    "first_task_responsible": [1]
  },
  ...
]
```

This endpoint retrieves all templates in your account.

### HTTP Request

`GET https://api.pneumatic.app/templates`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
ordering | -date | Available values: date, name, -date, -name.
limit | 20  | Use for pagination
offset | 0  | Use for pagination

<aside class="success">
Minus before the value will sort in descending order.
</aside>

If request contains limit/offset parameters then response will have structure like this:

```json
{
  "count": 123,
  "next": "next page link", // If this is the last page then "next" is null
  "previous": "prev page link", // If this is the first page then "previous" is null
  "results": [...] // list of templates
}
```

## Get a Specific Template


```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
template_id = 1

r = requests.get(f'https://api.pneumatic.app/templates/{template_id}', headers=headers)
```

> The above command returns JSON structured like this:

```json
{
  "id": int,
  "name": str,
  "description": str,
  "is_active": bool,
  "finalizable": bool, // This workflow can be finished at any stage.
  "date_updated": str, // format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
  "updated_by": int, // User who made the last update
  "run_allowed": [int], // Template owners. Users who can either run or edit it.
  "tasks_count": int,
  "performers_count": int,
  "kickoff": {
    "id": int, 
    "description": str, 
    "fields": [
      {
        "id": int,
        "order": int,
        "name": str,
        "type": FieldTypeEnum,
        "is_required": bool,
        "description": str,
        "api_name": str,
        "selections": [ // Only for radio, checkbox, dropdown
          {
            "id": int,
            "value": str
          }
        ]
      }
    ]
  },
  "tasks": [
    {
      "id": int,
      "number": int,
      "name": str,
      "description": str,
      "delay": str,       // null for not specified, format: '[DD] [[hh:]mm:]ss'
      "require_completion_by_all": bool,
      "is_starter_responsible": bool, // The workflow runner will be a performer for the task.
      "responsible": [int], // Assigned performers.
      "output_responsible": [ // The dynamic performers who will be assigned on the task from the previous tasks' and kickoff's output fields.
        {
          "api_name": str,
          "name": str
        }
      ],
      "fields": [
    	  {
          "id": int,
    	    "order": int,
    	    "name": str,
    	    "type": FieldTypeEnum,
    	    "is_required": bool,
    	    "description": str,
          "api_name": str,
          "selections": [ // Only for radio, checkbox, dropdown
            {
              "id": int,
              "value": str
            }
          ]
        }
      ]
    }
  ]
}
```

This endpoint retrieves a specific template.

### HTTP Request

`GET https://api.pneumatic.app/templates/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the template to retrieve

FieldTypeEnum:

- string
- text
- radio
- checkbox
- date
- url
- dropdown
- file
- user

## Launch a Workflow Based on Specific Template
<a id="workflow-run"></a> 

```python
import json
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}',
  'Content-Type': 'application/json'
}
template_id = 1
workflow_info = {
  "id": template_id,
  "name": "Workflow Name",
  "kickoff": { // nullable
    "string-field-1": "Value 1",
    "text-field-2": "Value 2",
    "dropdown-field-3": "selection ID",
    "checkbox-field-4":["selection ID", ...],
    "radio-field-5": "selection ID",
    "date-field-6": "date in ISO format",
    "file-field-7": ["attachment ID", ...], 
    "url-field-8": "https://grave.com/",
    "user-field-9": 1 // user id
  }
}

json_data = json.dumps(workflow_info)
r = requests.post(
    f'https://api.pneumatic.app/templates/{template_id}/run', 
    headers=headers,
    data=json_data,
)
```

> The above command returns JSON structured like this:

```json
{
  "workflow_id": 1
}
```

This endpoint launches a new workflow based on specific template.

### HTTP Request

`POST https://api.pneumatic.app/templates/<ID>/run`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the template to launch


### Body Parameters

Parameter | Required | Description
--------- | --------- | -----------
name | Yes | The name of the workflow you'd like to launch
tasks | No | Used if needed to overwrite some template's tasks properties designed by owner. For now, only `url` supported.
kickoff | Yes if there are required fields in kickoff | Dictionary where the keys are API names of kickoff fields. Checkbox and radio fields take as values IDs of selections. Retrieve a template to get these IDs.


## Upload file for attachment

```python
import json
import requests

api_key = 'your_api_key'
content_type = 'image/png'
headers = {
  'Authorization': f'Bearer {api_key}',
  'Content-Type': 'application/json',
}
file_info = json.dumps({
   'filename': 'pic.png',
   'thumbnail': True,
   'content_type': content_type,
   'size': 103613
})

attachment_info = requests.post(
    'https://api.pneumatic.app/workflows/attachments', 
    headers=headers,
    data=file_info,
).json()
```

### HTTP Request

`POST https://api.pneumatic.app/workflows/attachments`

### Body Parameters

Parameter | Required | Description
--------- | --------- | -----------
filename  | Yes |
content_type  | Yes | Header is used to indicate the media type of the resource (e.g. `image/png`)
size  | Yes | Size of file (less than 104857600 bytes)
thumbnail | No  | Boolean. Set `true` if you'd like to upload thumbnail image

> The above command returns JSON structured like this:

```json
{
  "id": 1,  // Use it where you need to attach your file
  "file_upload_url": "url",
  "thumbnail_upload_url": "thumb_url" // nullable
}
```

`PUT` your file to `file_upload_url` with `content-type` like in previous request

```python
headers = {
  'Content-type': content_type,
}

r = requests.put(
    attachment_info['file_upload_url'],
    data=open('file', 'rb'), 
    headers=headers,
)
```

## Make file public

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}',
}

attachment_id = 123

r = requests.post(
    f'https://api.pneumatic.app/workflows/attachments/{attachment_id}/links', 
    headers=headers,
)
```

### HTTP Request

`POST https://api.pneumatic.app/workflows/attachments/<ID>/links`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the attachment

> The above command returns JSON structured like this:

```json
{
  "id": 1,
  "name": "file_name",
  "url": "url",
  "thumbnail_url": "thumb_url",
  "size": 103613
}
```

You have to make your file public. Otherwise, nobody can watch it.


# Users

## Get user list

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}

r = requests.get(
  'https://api.pneumatic.app/accounts/users?offset=0&limit=20', 
  headers=headers,
)
```

> The above command returns JSON structured like this:

```json
{
  "count": int,
  "next": str, // nullable
  "previous": str, // nullable
  "results": [
    {
      "id": int,
      "email": str,
      "first_name": str,
      "last_name": str,
      "phone": str, // nullable
      "photo": str, // nullable
      "status": UserStatusEnum,
      "is_staff": bool,
      "is_admin": bool,
      "is_account_owner": bool,
      "invite": {
        "id": str,
        "by_username": str,
        "date_created": str, // format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS]
        "invited_by": int,
        "invited_from": InvitedFromEnum
      }
    }
  ]
}
```

UserStatusEnum:

- active
- invited

InvitedFromEnum:

- email
- google
- slack

### HTTP Request

`GET https://api.pneumatic.app/accounts/users`

### URL Parameters

Parameter | Description
--------- | -----------
limit | Use for pagination.
offset | Use for pagination.
order | asc\desc


# Webhooks

## Get All Webhooks

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}

r = requests.get('https://api.pneumatic.app/webhooks', headers=headers)
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "created": "datetime",
    "updated": "datetime",
    "event": "task_completed_v2",
    "target": "https://webhook.com/listener-url"
  },
  {
    "id": 2,
    "created": "datetime",
    "updated": "datetime",
    "event": "workflow_started",
    "target": "https://webhook.com/listener-url-2"
  },
  ...
]
```

This endpoint retrieves all active webhooks in your account.

### HTTP Request

`GET https://api.pneumatic.app/webhooks`

Available events:

- task_completed_v2
- task_returned
- workflow_started
- workflow_completed

## Subscribe

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
payload = {
  'target': 'your_webhook_listener_url',
  'event': 'event_name'
}

r = requests.post(
  'https://api.pneumatic.app/webhooks', 
  headers=headers,
  data=payload
)
```

> The above command returns JSON structured like this:

```json
{
  "id": 1,
  "created": "datetime",
  "updated": "datetime",
  "event": "event_name",
  "target": "your_webhook_listener_url"
}
```

This endpoint creates a new webhook subscription.

### HTTP Request

`POST https://api.pneumatic.app/webhooks`

### Body Parameters

Parameter | Description
--------- | -----------
target | URL to create a new webhook subscription
event | Available values: task_completed (Deprecated), process_started (Deprecated), process_completed (Deprecated), task_completed_v2, task_returned, workflow_started, workflow_completed

<aside class="notice">
All parameters are mandatory.
</aside>

## Unsubscribe by Webhook ID

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}

r = requests.delete(
  'https://api.pneumatic.app/webhooks/:id', headers=headers
)
```

> The above command returns JSON structured like this:

```json
{
  "ok": true
}
```

This endpoint deletes an existing webhook subscription.

### HTTP Request

`DELETE https://api.pneumatic.app/webhooks/:id`

## Unsubscribe by Target URL

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
payload = {
  'target': 'your_webhook_listener_url',
}

r = requests.delete(
  'https://api.pneumatic.app/webhooks/target', 
  headers=headers,
  data=payload
)
```

> The above command returns JSON structured like this:

```json
{
  "ok": true
}
```

This endpoint deletes an existing webhook subscription.

### HTTP Request

`DELETE https://api.pneumatic.app/webhooks/target`

### Body Parameters

Parameter | Description
--------- | -----------
target | URL to be removed from webhook listeners.

## Task Completed Webhook Event Schema
<a id="task-completed-webhook"></a>

`event: task_completed_v2`

```json
{
  "hook": {
    "id": 1,
    "event": "task_completed_v2",
    "target": "your_webhook_listener_url"
  },
  "task": {
    "id": 1,
    "name": "Task name",
    "description": "Task description",
    "number": 1,
    "date_started": "2020-10-12T16:25:00.062Z",
    "date_completed": "2020-10-12T16:25:58.675Z",
    "output": [
      {
        "id": 1,
        "name": "Performer",
        "type": "user",
        "api_name": "field-shdgkk",
        "value": "1",
        "is_required": true,
        "selections": [],
        "attachments": []
      },
      {
        "id": 2,
        "name": "Label",
        "type": "radiobutton",
        "api_name": "field-asdsfd",
        "value": null,
        "is_required": true,
        "selections": [
          {
            "id": 1,
            "value": "First Selection",
            "is_selected": false
          },
          {
            "id": 2,
            "value": "Second Selection",
            "is_selected": true
          }
        ],
        "attachments": []
      },
      {
        "id": 3,
        "name": "File",
        "type": "file",
        "api_name": "field-cvkjbk",
        "value": null,
        "is_required": true,
        "selections": [],
        "attachments": [
          {
            "id": 1,
            "name": "filename.png",
            "url": "https://fake-url.com"
          }
        ]
      }
    ],
    "performers": [
      {
        "id": 1,
        "first_name": "John",
        "last_name": "Doe"
      }
    ],
    "workflow": {
      "id": 1,
      "name": "Workflow Name",
      "status": 1,
      "date_created": "2020-10-12T16:12:10.099Z",
      "template": {
        "id": 1,
        "name": "Template name"
      },
      "kickoff": {
        "id": 1,
        "description": "Kickoff description",
        "output": []
      },
      "current_task": {
        "id": 2,
        "name": "Second task",
        "description": "Task description",
        "number": 2,
        "date_started": "2020-10-12T16:25:00.062Z",
        "date_completed": "2020-10-12T16:25:58.675Z",
        "performers": [
          {
            "id": 2,
            "first_name": "John Jr.",
            "last_name": "Doe"
          }
        ]
      }
    }
  }
}
```

## Task Returned Webhook Event Schema
<a id="task-returned-webhook"></a>
`event: task_returned`

See [Task Completed Webhook Event Schema](#task-completed-webhook)
## Workflow Started Webhook Event Schema
<a id="workflow-started-webhook"></a>
`event: workflow_started`

```json
{
  "hook": {
    "id": 1,
    "event": "workflow_started",
    "target": "your_webhook_listener_url"
  },
  "workflow": {
    "id": 1,
    "name": "Workflow Name",
    "status": 1,
    "date_created": "2020-10-12T16:12:10.099Z",
    "template": {
      "id": 1,
      "name": "Template name"
    },
    "kickoff": {
      "id": 1,
      "description": "Kickoff description",
      "output": [
        {
          "id": 1,
          "name": "Performer",
          "type": "user",
          "api_name": "field-shdgkk",
          "value": "1",
          "is_required": true,
          "selections": [],
          "attachments": []
        },
        {
          "id": 2,
          "name": "Label",
          "type": "radiobutton",
          "api_name": "field-asdsfd",
          "value": null,
          "is_required": true,
          "selections": [
            {
              "id": 1,
              "value": "First Selection",
              "is_selected": false
            },
            {
              "id": 2,
              "value": "Second Selection",
              "is_selected": true
            }
          ],
          "attachments": []
        },
        {
          "id": 3,
          "name": "File",
          "type": "file",
          "api_name": "field-cvkjbk",
          "value": null,
          "is_required": true,
          "selections": [],
          "attachments": [
            {
              "id": 1,
              "name": "filename.png",
              "url": "https://fake-url.com"
            }
          ]
        }
      ]
    },
    "current_task": {
      "id": 2,
      "name": "Second task",
      "description": "Task description",
      "number": 2,
      "date_started": "2020-10-12T16:25:00.062Z",
      "date_completed": "2020-10-12T16:25:58.675Z",
      "performers": [
        {
          "id": 2,
          "first_name": "John Jr.",
          "last_name": "Doe"
        }
      ]
    }
  }
}
```

Available workflow statuses:

- 0 - Running
- 1 - Completed
- 2 - Terminated
- 3 - Snoozed

## Workflow Completed Webhook Event Schema
<a id="workflow-completed-webhook"></a>
`event: workflow_completed`

See [Workflow Started Webhook Event Schema](#workflow-started-webhook)
