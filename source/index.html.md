# Introduction

Welcome to the Pneumatic API! You can use it to get data and perform actions in Pneumatic programmatically.

All the examples in this guide are in python.

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

> The above request returns JSON structured like this:

```json
{
  "count": 10,
  "next": "str",
  "previous": "str",
  "results": [
    {
      "id": 1,
      "name": "Sample Template",
      "tasks_count": "int",
      "performers_count": "str",
      "template_owners": ["int"],
      "is_active": "bool",
      "is_public": "bool",
      "is_embedded": "bool",
      "description": "str",
      "kickoff": {
        "id": "int",
        "description": "str",
        "fields": [
          {
            "name": "str",
            "type": "str",
            "order": "int",
            "is_required": "bool",
            "description": "str",
            "api_name": "str",
            "selections": [
              {
                "id": "int",
                "value": "str"
              }
            ]
          }
        ]
      }
    }
  ]
}
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

| Parameter | Default | Description                                 |
| --------- | ------- | ------------------------------------------- |
| ordering  | -date   | Available values: date, name, -date, -name. |
| limit     | 20      | Use for pagination                          |
| offset    | 0       | Use for pagination                          |

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

> The above request returns JSON structured like this:

```json
{
  "id": "int",
  "name": "str",
  "description": "str",
  "is_active": "bool",
  "is_public": "bool",
  "public_url": "str | null",
  "public_url": "str | null",
  "public_success_url": "str | null",
  "is_embedded": "?bool,     // default: false"
  "embed_url": "str | null",
  "finalizable": "bool",
  "date_updated": "str", //" ISO 8601 format: YYYY-MM-DDThh:mm:ss[.SSS]"
  "updated_by": "int",
  "template_owners": ["int"],
  "tasks_count": "int",
  "performers_count": "int",
  "kickoff": {
    "id": "int",
    "description": "str",
    "fields": [
      {
        "id": "int",
        "order": "int",
        "name": "str",
        "type": "str",
        "is_required": "bool",
        "description": "str",
        "default": "str",
        "api_name": "str",
        "selections": [ // these are only passed in for radio button checkbox and dropdown fields
          {
            "id": "int",
            "value": "str"
          }
        ]
      }
    ]
  },
  "tasks": [
    {
      "id": "int",
      "number": "int",
      "name": "str",
      "description": "str",
       "delay": "str",       // null if the formal is not aset as: '[DD] [[hh:]mm:]ss'
      "require_completion_by_all": "bool",
      "raw_due_date": {
          "api_name": "str",
          "duration": "str",
          "duration_months": "int, default 0",
          "rule": "str",
          "source_id": "?str | null"
      },
      "raw_performers": [
        {
          "id": "int",
          "type": "user|field|workflow_starter",
          "source_id": "?str", // id the id of a user, group or the api name of the field or null
          "label": "str"       // this is a read only parameter: the user name, group name or field name
        },
      ],
      "checklists": ?[
        {
          "api_name": "str",
          "selecions": [
            {
              "api_name": "str",
              "value": "str"
            }
          ]
        }
      ]
      "fields": ?[
    	{
          "id": "int",
    	    "order": "int",
    	    "name": "str",
    	    "type": "str",
    	    "is_required": "bool",
    	    "description": "str",
          "default": "str",
    	    "api_name": "str",
            "selections": [ // these are only passed in for radio button checkbox and dropdown fields
              {
                "id": "int",
                "value": "str"
              }
            ]
        }
      ],
      "conditions": ?[
        {
          "id": "int",
          "action": "str", // end_process, skip_task, start_task
          "order": "int",
          "api_name": "str",
          "rules": [
            {
              "id": "int",
              "api_name": "str",
              "predicates": [
                {
                  "id": "int",
                  "field_type": "str",
                  "value": "Optional[str]",
                  "api_name": "str",
                  "field": "str", // the api_name of the field being used
                  "operator": "str" // equals, not_equals, 	exists, not_exists, contains, not_contains, more_than, less_than
                }
              ]
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

| Parameter | Description                        |
| --------- | ---------------------------------- |
| ID        | The ID of the template to retrieve |

Empty field values:

- null for strings and numbers
- [] for lists
- false for booleans

## Create a new template

<a id="create-template"></a>

```python
import json
import requests

api_key = 'your_api_key'

headers = {
  'Authorization': f'Bearer {api_key}',
  'Content-Type': 'application/json'
}

template_info = {
  "name": "template name",
  "description": "template descrition optiona",
  "template_owners": "[int]",
  "is_active": "bool optional, default false",
  "is_public": "bool optional, default false",
  "public_url": "str | null",
  "public_success_url": "string optional, defualt null",
  "is_embedded": "bool optional default false",
  "embed_url": "str | null",
  "finalizable":"bool optional, default false",
  "kickoff": {
    "description": "string optional",
    "fields": [
      {
        "api_name": "string optional",
        "order": "int",
        "name": "int",
        "type": "int",
        "is_required": "bool optional default false",
        "description": "string optional",
        "default:": "string optional"
        "selections": [      #only passed in if there are radio button/checkbox fields
          {
            "value": "string"
          }
        ]
      }
    ]
  },
  "tasks": [
    {
      "number": "int",
      "name": "string",
      "description": "string optional",
      "require_completion_by_all": "bool",
      "delay": "string or null: [DD] [[hh:]mm:]ss",
      "raw_due_date": {
          "api_name": "str",
          "duration": "str",
          "duration_months": "int, default 0",
          "rule": "str",
          "source_id": "?str | null"
      },
      "raw_performers": [
        {
          "id": "int",
          "type": "user|field|workflow_starter",
          "source_id": "?str - user or group id or the api_name or null"
          "label": "str -  user/group/field name, read only"
        },
      ],
      "checklists": [
        {
          "api_name": "string optional",
          "selecions": [
            {
              "api_name": "string optional",
              "value": "string"
            }
          ]
        }
      ],
      "fields": [#fields are optional
    	{
    	  "api_name": "string optional",
    	  "order": "int",
          "name": "string",
          "type": "string,
          "is_required": "bool optional, default - false",
          "description": "string optiona",
          "default": "string optional",
          "selections": [     #only passed in for radio buttons, checkboxes and dropdown lists
            {
              "value": "string"
            }
          ]
    	}
      ],
      "conditions": [#an optional parameter
        {
          "action": "string end_process, skip_task, start_task",
          "order": "int",
          "api_name": "string",
          "rules": [
            {
              "api_name": "string",
              "predicates": [
                {
                  "field_type": "string",
                  "value": "string optional",
                  "api_name": "string optional",
                  "field": "string - api name",
                  "operator": "string equals, not_equals, exists, not_exists, contains, not_contains, more_than, less_than"
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}

json_data = json.dumps(template_info)
r = requests.post(
    f'https://api.pneumatic.app/templates',
    headers=headers,
    data=json_data,
)

```

The endpoint creates a new template as per the parametrs passed in the json.

### HTTP Request

`POST https://api.pneumatic.app/templates`

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

> The above request returns JSON structured like this:

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

| Parameter    | Required | Description                                                                  |
| ------------ | -------- | ---------------------------------------------------------------------------- |
| filename     | Yes      |
| content_type | Yes      | Header is used to indicate the media type of the resource (e.g. `image/png`) |
| size         | Yes      | Size of file (less than 104857600 bytes)                                     |
| thumbnail    | No       | Boolean. Set `true` if you'd like to upload thumbnail image                  |

## Make a file public

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}',
}

attachment_id = 123

r = requests.post(
    f'https://api.pneumatic.app/workflows/attachments/{attachment_id}/publish',
    headers=headers,
)
```

> The above request returns JSON structured like this:

```json
{
  "url": "str",
  "thumbnail_url": "str"
}
```

You have to make your file public. Otherwise, nobody can see it.

### HTTP Request

`POST https://api.pneumatic.app/workflows/attachments/<ID>/publish`

### URL Parameters

| Parameter | Description              |
| --------- | ------------------------ |
| ID        | The ID of the attachment |

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
  "name":"workflow_name - optional",
  "due_date_tsp":"timestamp due date(optional)",
  "ancestor_task_id":"int|nill - the id of the parent task if launching as an embedded workflow",
  "is_urgent":"bool - set to true if the new workflow is to be marked as urgent",
  "kickoff": { // nullable
    "string-field-1": "Value 1",
    "text-field-2": "Value 2",
    "dropdown-field-3": "selection api_name",
    "checkbox-field-4":["selection api_name", ...],
    "radio-field-5": "selection api_name",
    "date-field-6": "a timestamp as a float",
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

> The above request the entire workflow, the same as the GET  /workflows/:id :

```json
    {
      "id": int,
      "name": str,
      "owners": [int], // id users
      "description": str,
      "date_created": str,
      "date_created_tsp": float,    // timestamp in seconds
      "date_completed": str | null, // format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
      "date_completed_tsp": float,  // timestamp in seconds
      "due_date": str | null,       // the task expiration date in ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
      "due_date_tsp": float,        // timestamp in seconds
      "is_external": bool,
      "is_urgent": bool,
      "tasks_count": int,
      "finalizable": bool,
      "is_legacy_template": bool,
      "legacy_template_name": str,
      "status": int,
      "workflow_starter": int | null // null if is_external
      "ancestor_task_id": int | null // for subprocesses
      "template": {
        "id": int,
        "name": str,
        "is_active": bool,
        "wf_name_template": str | null,
      },
      "current_task": {
        "id": int,
        "number": int,
        "name": str,
        "date_started": str,
        "date_started_tsp": str,          // timestamp in seconds
        "due_date": str | null,           // format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
        "due_date_tsp": float,            // timestamp in seconds
        "status": str                     // pending, active, completed, snoozed, skipped
        "performers": [
          {
            "type": "user"|"group", 
            "source_id": int,   // the ide of the user or the group assigned as performer
          }
        ],
        "checklists_total": int,
        "checklists_marked": int,
        "delay": null|{
          "duration": str,         // format: D hh:mm:ss[.SSS] 
          "start_date": str,       // format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
          "start_date_tsp": float, // timestamp in seconds
          "end_date": null|str,    // format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
          "end_date_tsp": float,   // timestamp in seconds
          "estimated_end_date": null|str // format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS] 
          "estimated_end_date_tsp": float, // timestamp in seconds
        }
      },
      "started_by": {...}, // deprecated
      "kickoff": {
        "output": [
          {
            "id": int, 
            "type": str, // string|text|radio|checkbox|date|url|dropdown|file|user
            "api_name": str, 
            "name": str,
            "value": str,   // a string reprsentation of the value
            "user_id": int, // for user type fields
            "attachments": [
              {
                "id": int,
                "name": str,
                "url": str,
                "size": int
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
      },
      "passed_tasks": [
        {
          "id": int,
          "name": str,
          "number": int
        }
      ]
    }
```

This endpoint launches a new workflow based on specific template.
The **first_task_performers** response parameter contains the ids of the performers assigned to the first task.

### HTTP Request

`POST https://api.pneumatic.app/templates/<ID>/run`

### URL Parameters

| Parameter | Description                                        |
| --------- | -------------------------------------------------- |
| ID        | The ID of the template to launch the workflow from |

### Body Parameters

| Parameter        | Required                                    | Description                                                                                                                                                  |
| ---------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| name             | No                                          | The name of the workflow you'd like to launch. If no name is supplied the default name 'current date - template name' will be used                           |
| is_urgent        | No                                          | The default value is false, the parameter determines whether the new workflow will be marked as urgent.                                                      |
| ancestor_task_id | No                                          | Supply this parameter if you want the new workflow to be embedded in an existing task.                                                                       |
| due_date_tsp     | No                                          | Supply this parameter if you want to set the due date for the workflow as a timestamp.                                                                       |
| kickoff          | Yes if there are required fields in kickoff | Dictionary where the keys are API names of kickoff fields. Checkbox and radio fields take as values IDs of selections. Retrieve a template to get these IDs. |

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

> The above request returns JSON structured like this:

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

This endpoint returns workflows fields data based on a specific template.

### HTTP Request

`GET https://api.pneumatic.app/workflows/fields`

### URL Parameters

| Parameter   | Required | Description                                                                                                                                                              |
| ----------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| template_id | yes      | The ID of the template                                                                                                                                                   |
| status      | no       | Workflows status.<br> Possible values: `running`, `delayed`, `done`                                                                                                      |
| fields      | no       | List of `api_name` task/kickoff fields separated by `,`.<br> For example: `field-1,field-2`<br> You can take api_names from [Get a Specific Template API](#get-template) |
| limit       | no       | Size of page. Default value is 15                                                                                                                                        |
| offset      | no       | Number of page. Default value is 0                                                                                                                                       |

# Tasks

## Get a list of tasks

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

> The above request returns JSON structured like this:

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
      "due_date": null|str?, // null if task doesn't have a due date. Format ISO 8601: YYYY-MM-DDThh:mm:ss[.SSS]
      "template_id": int,
      "template_task_id": int,
      "is_urgent": bool
    }
  ]
}
```

This endpoint returns running tasks assigned to a specified user.

### HTTP Request

`GET https://api.pneumatic.app/v3/tasks`

### URL Parameters

| Parameter   | Required | Description               |
| ----------- | -------- | ------------------------- |
| assigned_to | no       | User id assigned to tasks |
| limit       | no       | Size of page              |
| offset      | no       | Number of page            |

## Get a single task

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

> The above request returns JSON structured like this:

```json
{
   "id": int,
   "name": str,
   "description": str,
   "date_started": str,
   "date_completed": null | str,
   "due_date": null | str,
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

This endpoint returns a specific task.

### HTTP Request

`GET https://api.pneumatic.app/v2/tasks/<id>`

## Complete a task

<a id="complete-task"></a>

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}',
}

workflow_id ='the id of the workflow you want to complete the curren task in'
workflow = requests.get(f"https://api.pneumatic.app/workflows/{workflow_id}", headers=headers).json()

task_id =workflow['current_task']['id']
user_id='the id of the user who will be completing the task(optiona)'

headers = {
  'Authorization': f'Bearer {api_key}',
  'Content-Type':'application/json'
}

data_and_outputs ={
  'task_id':task_id,
  'user_id':user_id,
  'output': {
    'output-field-1':'value',
    'output-field-2:['selection-id-1', 'selection-id-2' ...],
    'output-field-3: 'selection-id',
  }
}
json_data = json.dumps(data_and_outputs)
r = requests.post(
  f'https://api.pneumatic.app/workflows/{workflow_id}/task-complete',
  headers=headers,
  data = json_data
)
```

> if successful the above request returns a success code
> otherwise a json describing the nature of the error will be returned

The endpoint completes the current task in a workflow

### HTTP Request

`POST https://api.pneumatic.app/<workflow-id>/task-complete`

Only the currently active task in a workflow can be completed.
To do so you need to pass in the task-id along with data for the output fields you want filled out before the task is completed.

These are passed in as a json of the form shown in the code example on the right where it is assigned to the data_and_outputs variable.

Thus, to complete a task, you need to know:

- the id of the workflow the task belongs to – you can look this up in Pneumatic
- the id of the task to complete – you can query the workflow for the id of the current task
- the API names of the output fields you want to fill out, along with the selection ids of the checkbox and radio button fields you'll be filling out – you can look these up on the API tab in the integrations section of your template

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

> The above request returns JSON structured as follows:

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

| Parameter | Description            |
| --------- | ---------------------- |
| ordering  | the field to order by. |
| type      | filter by user type.   |
| status    | user status.           |
| groups    | filter by group.       |
| limit     | Use for pagination.    |
| offset    | Use for pagination.    |

**Odering** specifies the field the results are ordered by, several values can be passed in separated by a comma. Use a '-' to invert the order.
Possible fields users can be ordered by:

- last_name(default),
- first_name,
- status

The **type** parameter lets you filter users by user type. Several comma separated values can be passed in to get several types of users in the response. The available values are:

- guest
- user

The **status** parameter filters users by status. Several comma separated values can be passed in to get users with severeal different statuses. The available values are:

- active(default)
- inactive (this refers to deleted users)
- invited (users that have been sent invites but haven't accepted them yet)

The **groups** parameter lets you to only select users that belong to a specific group identified by its **id**. Several group ids can be passed in as a comma separated list.

The **limit** parameter limits the number of user records returned per page.
The **offset** parameter tells the API which page to start with.

**Response options**

**200** means the reqeuest has been processed successfully return a response in the format show above.

**400** means there was a validation error

# Webhooks

## Events

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
end_point = 'https://api.pneumatic.app/webhooks/events'
r = requests.get(end_point, headers=headers)
```

> The request returns a list of events with the current state of the subscription for each one of them:

```json
[
  {
     "event": 'workflow_completed',
     "url": str | null
  },
  {
     "event": 'workflow_started',
     "url": str | null
  },
    {
     "event": 'task_completed_v2',
     "url": str | null
  },
  {
     "event": 'task_returned',
     "url": str | null
  }
]
```

The endpoint returns a list of events with their subscriptions

### HTTP Request

`GET https://api.pneumatic.app/webhooks/events`

For each event in the returned list you get:

- the name of the event
- the subscription url or null

Pneumatic has four basic events you can subscribe to via webhooks:

- workflow started
- workflow completed
- task completed
- task returned

## Get a Specific Event

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
end_point = 'https://api.pneumatic.app/webhooks/events/event_name'

r = requests.get(
  end_point,
  headers=headers,
)
```

> If successful the above request returns a jason with the url subscribed to the event or null

```json
{
   "url": str | null
}
```

The endpoint returns the url subscribed to a specific event or nill

### HTTP Request

`GET https://api.pneumatic.app/webhooks/events/event_name`

### Available event names

| Event name         | Description                     |
| ------------------ | ------------------------------- |
| workflow_completed | an entire workflow is completed |
| workflow_started   | a new workflow is started       |
| task_completed_v2  | a task is completed             |
| task_returned      | a task is returned              |

<aside class="notice">
The events are template-agnostic, i.e. you will get notified about completed/returned tasks and completed/started workflows regadless of the template 
</aside>

## Subscribe to an event

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
end_point = 'https://api.pneumatic.app/webhooks/events/event_name/subscribe'
payload = {
  'url': subscription_url
}

r = requests.post(end_point, headers=headers, data = payload)
```

> if successful, the request will subscribe subscription_url to event_name:
> If executed successfully, the request returns an ampty body

```json
{}
```

This endpoint subscribes you to the event_name event.

### HTTP Request

`POST https://api.pneumatic.app/webhooks/events/event_name/subscribe`

### Available event names

| Event name         | Description                     |
| ------------------ | ------------------------------- |
| workflow_completed | an entire workflow is completed |
| workflow_started   | a new workflow is started       |
| task_completed_v2  | a task is completed             |
| task_returned      | a task is returned              |

<aside class="notice">
Note: any existing subscription will be overwritten
</aside>

## Unsubscribe from an event

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
end_point = 'https://api.pneumatic.app/webhooks/events/event_name/unsubscribe'

r = requests.post(end_point, headers=headers)
```

> if successful, the request will ubsubscibe you from the event_name event:
> If successfuly request returns 204 and an empty body

```json
{}
```

This endpoint unsubscribes you from the event_name event.

### HTTP Request

`POST https://api.pneumatic.app/webhooks/events/event_name/unsubscribe`

### Available event names

| Event name         | Description                     |
| ------------------ | ------------------------------- |
| workflow_completed | an entire workflow is completed |
| workflow_started   | a new workflow is started       |
| task_completed_v2  | a task is completed             |
| task_returned      | a task is returned              |

There is no need to supply a specific url, the webhook url for the specified event will simply be deleted.

## Subscribe to all events at once

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
end_point = 'https://api.pneumatic.app/webhooks/subscribe'

r = requests.post(end_point, headers=headers)
```

> if successful, the request will subscribe you to all events at once:
> The request returns the subscription url that will be set for all events:

```json
{
  'url': str
}
```

This endpoint subscribes you to all events at once.

### HTTP Request

`POST https://api.pneumatic.app/webhooks/subscribe`

## Unsubscribe from all events at once

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
end_point = 'https://api.pneumatic.app/webhooks/unsubscribe'

r = requests.post(end_point, headers=headers)
```

> if successful, the request will cancel all your subscriptions at once.
> All the urls for all the events will have been wiped.
> The request returns an empty body

```json
{}
```

This endpoint cancels all your webhook subscriptions at once.

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
