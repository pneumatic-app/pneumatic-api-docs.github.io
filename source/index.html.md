# Introduction

Welcome to the Pneumatic API! You can use our API to access Pneumatic public API endpoints, which can get information on templates and run workflows.

All the examples in the dark area are provided in Python code.

# Authentication

```python
import requests

api_key = 'api_key'
headers = { 'Authorization': f'Bearer {api_key}' }
response = requests.get('https://api.pneumatic.app/some-endpoint', headers=headers)
```
Pneumatic expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer api_key`

<aside class="notice">
You must replace <code>api_key</code> with your personal API key from <a href="https://my.pneumatic.app/integrations/">Integrations page</a>.
</aside>

Use bearer token authentication for each API request.

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
    "template_owners": [
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

> If request contains limit/offset parameters then response will have structure like this:

```json
{
  "count": 123,
  "next": "next page link", // If this is the last page then "next" is null
  "previous": "prev page link", // If this is the first page then "previous" is null
  "results": [...] // list of templates
}
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

## Get a Specific Template
<a id="get-template"></a>

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
  "template_owners": [int], // Template owners. Users who can either run or edit it.
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

payload = '<file contents here>'

r = requests.put(
    attachment_info['file_upload_url'],
    data=payload, 
    headers={'Content-Type': content_type},
)
```

> The above command returns JSON structured like this:

```json
{
  "id": int,
  "file_upload_url": str, // URL to upload file
  "thumbnail_upload_url": str?, // URL to upload thumbnail 
}
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

### HTTP Request

`POST https://api.pneumatic.app/workflows/attachments/<ID>/links`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the attachment

# Workflows

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


## Get Workflows fields data
<a id="workflow-fields"></a>

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}',
}

r = requests.get(
    f'https://api.pneumatic.app/workflows/fields', 
    headers=headers,
    params={
      'template_id': 1,
      'status': 'done',
      'fields': 'field-1,field-2,field-3',
      'limit': 15,
      'offset': 0
    }
)
```

> The above command returns JSON structured like this:

```json
{
  "count": int,
  "next": str?, // link to the next page
  "previous": str?, // link to the previous page
  "results":[ 
    {
      "id": str, 
      "name": str, 
      "status": str, 
      "date_created": str, // format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
      "date_completed": str, // format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
      "fields": [
        {
          "id": int, 
          "task_id": int | null,
          "kickoff_id": int | null,
          "type": str, // string|text|radio|checkbox|date|url|dropdown|file|user
          "api_name": str, 
          "name": str,
          "value": str,
          "attachments": [
            {
              "id": int,
              "name": str,
              "url": str
            }
          ],
          "selections": [
            {
              "id": int,
              "api_name": str,
              "value": str,
              "is_selected": bool
            }
          ]
        }
      ]
    }
  ]
}
```

This endpoint returns workflows fields data based on specific template.

### HTTP Request

`GET https://api.pneumatic.app/workflows/fields`

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
template_id | yes |The ID of the template
status | no | Workflows status.<br> Possible values: `running`, `delayed`, `done`
fields | no | List of `api_name` task/kickoff fields separated by `,`.<br> For example: `field-1,field-2`<br> You can take api_names from [Get a Specific Template API](#get-template) 
limit | no | Size of page. Default value is 15
offset | no | Number of page. Default value is 0

# Tasks

## Get a task list
<a id="get-task-list"></a>

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}',
}

r = requests.get(
    f'https://api.pneumatic.app/v3/tasks?assigned_to=<user_id>', 
    headers=headers,
)
```

> The above command returns JSON structured like this:

```json
{
  "count": int,
  "next": str?, // link to the next page
  "previous": str?, // link to the previous page
  "results":[
    {
      "id": str,
      "name": str,
      "workflow_name": str,
      "estimated_end_date": str?, // null if task doesn't have 'due_in'. Format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
      "template_id": int,
      "template_task_id": int,
      "is_urgent": bool
    }
  ]
}
```

This endpoint returns running tasks assigned to specified user.

### HTTP Request

`GET https://api.pneumatic.app/v3/tasks`

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
assigned_to | no | User id assigned to tasks 
limit | no | Size of page
offset | no | Number of page

## Get  task
<a id="get-specific-task"></a>

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}',
}

r = requests.get(
    f'https://api.pneumatic.app/v2/tasks/<id>', 
    headers=headers,
)
```

> The above command returns JSON structured like this:

```json
{
   "id": int,
   "name": str,
   "description": str,
   "date_started": str,
   "date_completed": null | str,
   "estimated_end_date": null | str,
   "is_completed": bool,
   "is_urgent": bool,
   "require_completion_by_all": bool,
   "performers": [int], // users ids
   "delay": null | {
     "id": int,
     "duration": str,
     "start_date": str,
     "end_date": str,
     "estimated_end_date": str
   },
   "output": [
     {
      "id": int, 
      "task_id": int | null,
      "kickoff_id": int | null,
      "type": str, // string|text|radio|checkbox|date|url|dropdown|file|user
      "api_name": str, 
      "name": str,
      "value": str,
      "attachments": [
        {
          "id": int,
          "name": str,
          "url": str
        }
      ],
      "selections": [
        {
          "id": int,
          "api_name": str,
          "value": str,
          "is_selected": bool
        }
      ]
    }
   ]
}
```

This endpoint returns a specified task.

### HTTP Request

`GET https://api.pneumatic.app/v2/tasks/<id>`

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

### HTTP Request

`GET https://api.pneumatic.app/accounts/users`

### URL Parameters

Parameter | Description
--------- | -----------
limit | Use for pagination.
offset | Use for pagination.
order | asc\desc

UserStatusEnum:

- active
- invited

InvitedFromEnum:

- email
- google
- slack

# Webhooks

## Get the Subscription URL

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
end_point = 'https://api.pneumatic.app/webhooks/subscription'
r = requests.get(end_point, headers=headers)
```

> The above command returns a JSON containing your subscription url or null:

```json
{
   "url": str | null
}
```

This endpoint returns the subscription url for all the webhooks in the system

### HTTP Request

`GET https://api.pneumatic.app/webhooks/subscription`

You subscribe to all available events at the same time:

- workflow started
- workflow completed
- task completed
- task returned

## Subscribe

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
end_point = 'https://api.pneumatic.app/webhooks/subscribe'
payload = {
  'url': 'your_subscription_url',
}

r = requests.post(
  end_point, 
  headers=headers,
  data=payload
)
```

> If successful the above command returns an empty json

```json
{}
```

The endpoint subscribes the supplied url to all events.
Any already existing subscription url will be replaced

### HTTP Request

`POST https://api.pneumatic.app/webhooks/subscribe`

### Body Parameters

Parameter | Description
--------- | -----------
url | the url that you want Pneumatic event notifications to be sent to


<aside class="notice">
The commmand subscribes the url to all events. Any existing subscription gets replaced by the new url.
</aside>

## Unsubscribe 
```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
end_point = 'https://api.pneumatic.app/webhooks/unsubscribe'

r = requests.post(end_point)
```

> The above command returns 204 if successful:

```json
{
  "ok": true
}
```

This endpoint deletes an existing webhook subscription.

### HTTP Request

`POST https://api.pneumatic.app/webhooks/unsubscribe`



## Task Completed Webhook Event Schema
<a id="task-completed-webhook"></a>

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

`event: task_completed_v2`

## Task Returned Webhook Event Schema
<a id="task-returned-webhook"></a>

`event: task_returned`

See [Task Completed Webhook Event Schema](#task-completed-webhook)
## Workflow Started Webhook Event Schema
<a id="workflow-started-webhook"></a>

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

`event: workflow_started`

Available workflow statuses:

- 0 - Running
- 1 - Completed
- 2 - Terminated
- 3 - Snoozed

## Workflow Completed Webhook Event Schema
<a id="workflow-completed-webhook"></a>
`event: workflow_completed`

See [Workflow Started Webhook Event Schema](#workflow-started-webhook)
