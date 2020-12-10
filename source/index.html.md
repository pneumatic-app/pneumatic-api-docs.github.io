
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


> Make sure to replace `your_api_key` with your API key from [profile page](https://my.pneumatic.app/settings/profile/).

Pneumatic expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer api_key`

<aside class="notice">
You must replace <code>api_key</code> with your personal API key.
</aside>

# Templates

## Get All Templates

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
              {"id": 3, "value": "checkbox value 3"},
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

`GET https://api.pneumatic.app/workflows`

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

```
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
workflow_id = 1

r = requests.get(f'https://api.pneumatic.app/workflows/{workflow_id}', headers=headers)
```

> The above command returns JSON structured like this:

```json
{
    "id": 1,
    "draft": null,
    "tasks_count": 6,
    "name": "Template name",
    "description": "Template description", // nullable
    "is_active": false,
    "finalizable": false,
    "date_updated": "2020-12-04T10:02:50.074891+00:00",
    "updated_by": {
       "id": 2,
       "first_name": "John",
       "last_name": "Doe"
    },
    "performers_count": 1
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
    "tasks": [
        {
            "id": 1,
            "name": "Name of the first task of the workflow",
            "description": "Its description",
            "number": 1,
            "responsible": [],
            "url": null,
            "require_completion_by_all": false,
            "delay": null,
            "is_starter_responsible": true,
            "output": {
               "id": 123,
               "fields": [
                  {
                    "name": "Radio Buttons Field",
                    "type": "radio",
                    "is_required": false,
                    "description": "Radio Field Description",
                    "api_name": "radio-buttons-field-3",
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
                  }
               ]
            }
        },
        {
            "id": 2,
            "name": "Second step of the process",
            "description": null,
            "number": 2,
            "delay": "00 01:00:00", // DD HH:mm:ss, nullable
            "is_starter_responsible": false,
            "responsible": [
                {
                    "id": 1,
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
                        "id": 3,
                        "value": "Option 1"
                    },
                    {
                        "id": 4,
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
                        "id": 5,
                        "value": "Option 1"
                    },
                    {
                        "id": 6,
                        "value": "Option 2"
                    }
                ]
            },
            ...
        ]
    }
}
```

This endpoint retrieves a specific template.

### HTTP Request

`GET https://api.pneumatic.app/workflows/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the template to retrieve

Enum type of kickoff/task field:

- string
- text
- radio
- checkbox
- date
- url
- dropdown
- file

## Launch a Workflow Based on Specific Template

```python
import requests

api_key = 'your_api_key'
headers = {
  'Authorization': f'Bearer {api_key}'
}
template_id = :template_id
process_info = {
  "id": :template_id,
  "name":"Workflow Name",
  "kickoff": { // nullable
    "string-field-1": "Value 1",
    "text-field-2": "Value 2",
    "dropdown-field-3": "selection ID",
    "checkbox-field-4":["selection ID", "one more selection ID"],
    "radio-field-5": "selection ID",
    "date-field-6": "12/11/2020",
    "file-field-7": ["attachment ID"], 
    "url-field-8": "https://grave.com/"
  }
}

r = requests.post(
    f'https://api.pneumatic.app/workflows/{template_id}/run', 
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

This endpoint launches a new workflow based on specific template.

### HTTP Request

`POST https://api.pneumatic.app/workflows/<ID>/run`

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
import requests

api_key = 'your_api_key'
content_type = 'image/png'
headers = {
  'Authorization': f'Bearer {api_key}'
}
file_info = {
   'filename': 'pic.png',
   'thumbnail': true,
   'content_type': content_type,
   'size': 103613
}

attachment_info = requests.post(
    'https://api.pneumatic.app/processes/attachments', 
    headers=headers,
    data=file_info,
).json()
```

### HTTP Request

`POST https://api.pneumatic.app/processes/attachments`

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
  'Authorization': f'Bearer {api_key}'
}

attachment_id = 123

r = requests.post(
    f'https://api.pneumatic.app/processes/attachments/{attachment_id}/links', 
    headers=headers,
)
```

### HTTP Request

`POST https://api.pneumatic.app/processes/attachments/<ID>/links`

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
