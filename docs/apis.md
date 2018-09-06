# APIs

## SystemIndexManagement

### POST /system_index_management/create

Create a new system index, this operation init. a new system index and is required 
to start using starchat.

Output JSON

#### Requirements

* The function requires "admin" credentials

#### Sample call 

```bash
PORT="${1:-8888}"
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/system_index_management/create"
```

Sample response:

```json
{
   "message" : "IndexCreation: user(starchat_system_0.user, true) refresh_decisiontable(starchat_system_0.refresh_decisiontable, true)"
}
```

### GET /system_index_management

Fetch and returns the informations about the system index

Output JSON

#### Requirements

* The function requires "admin" credentials

#### Sample call 

```bash
PORT="${1:-8888}"
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X GET "http://localhost:${PORT}/system_index_management"
```

Sample response:

```json
{
   "message" : "IndexCheck: user(starchat_system_0.user, true) refresh_decisiontable(starchat_system_0.refresh_decisiontable, true)"
}
```

### DELETE /system_index_management

Delete a system index, this operation destroy any user created

Output JSON

#### Requirements

* The function requires "admin" credentials

#### Sample call 

```bash
PORT="${1:-8888}"
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X DELETE "http://localhost:${PORT}/system_index_management"
```

Sample response:

```json
{
   "message" : "IndexDeletion: user(starchat_system_0.user, true) refresh_decisiontable(starchat_system_0.refresh_decisiontable, true)"
}
```

## User

### POST /user

Insert a new user to the system, the user record can be generated 
using the '/user_gen/test_user' endpoint

Output JSON

#### Requirements

* The function requires "admin" credentials

#### Sample call 

```bash
PORT="${1:-8888}"
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/user -d '{
        "id": "test_user",
        "password": "3c98bf19cb962ac4cd0227142b3495ab1be46534061919f792254b80c0f3e566f7819cae73bdc616af0ff555f7460ac96d88d56338d659ebd93e2be858ce1cf9", 
        "salt": "salt",
        "permissions": {
                "index_getjenny_english_0": ["read", "write"]
        }
}'
```

Sample response:

```json
{
   "version" : 1,
   "created" : true,
   "index" : "starchat_system_0.user",
   "dtype" : "user",
   "id" : "test_user"
}
```

### GET /user

Fetch the informations about a user

Output JSON

#### Requirements

* The function requires "admin" credentials

#### Sample call 

```bash
PORT="${1:-8888}"
USERNAME=${2:-"admin"}
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X GET http://localhost:${PORT}/user/${USERNAME}
```

Sample response:

```json
{
   "permissions" : {
      "admin" : [
         "admin"
      ]
   },
   "salt" : "salt2",
   "id" : "admin",
   "password" : "ce822ea3bd2ac45ed908f0fac0c81d95df7e59ad554ebed5e173113f5fb97a6c585803233136dd6b16b02742f50dd8cff6fac97ff827394e694f63198618e02c"
}
```

### PUT /user

Update a user record

Output JSON

#### Requirements

* The function requires "admin" credentials

#### Sample call 

```bash
PORT="${1:-8888}"
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
 -H "Content-Type: application/json" -X PUT http://localhost:${PORT}/user/test_user -d '{
        "permissions": {
                "index_getjenny_english_0": ["read"]
        }
}'
```

Sample response:

```json
{
   "version" : 2,
   "created" : false,
   "dtype" : "user",
   "index" : "starchat_system_0.user",
   "id" : "test_user"
}
```

### DELETE /user/user_id

Delete an existing user

Output JSON

#### Requirements

* The function requires "admin" credentials

#### Sample call 

```bash
PORT="${1:-8888}"
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X DELETE http://localhost:${PORT}/user/test_user 
```

Sample response:

```json
{
   "version" : 10,
   "dtype" : "user",
   "id" : "test_user",
   "index" : "starchat_system_0.user",
   "found" : true
}
```

### POST /user_gen/test_user

Generate the record for a user with hashed password and a randomly generated salt.
The user must be then inserted into the system.

Output JSON

#### Requirements

* The function requires "admin" credentials

#### Sample call 

```bash
PORT=${1:-8888}

curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/user_gen/test_user -d '{
        "password": "plain text password",
        "permissions": {
                "index_getjenny_english_0": ["read"],
                "index_finnish_1": ["read", "write"]
        }
}'
```

Sample response:

```json
{
   "password" : "d4cc0586d5d9116e755f5011d2c03e627821c2596820236ef230ff094176586d05fbfa98a198aaa08935df2849196d2648012f1e40ff2e40ca8c3f627b41e9db",
   "salt" : "qoKJUyUwpvWM53PI",
   "id" : "test_user",
   "permissions" : {
      "index_finnish_1" : [
         "read",
         "write"
      ],
      "index_getjenny_english_0" : [
         "read"
      ]
   }
}
```

## DecisionTable

### POST /get_next_response

Tell StarChat about the user actions (wrote something, clicked a button etc) and receives instruction 
about the next state.

Data to post:

```json
{
  "conversation_id": "1234",
  "user_input": { "text": "the text typed by the user" }, // optional
  "values": {
    "return_value": "the value either in success_value or in failure_value (Optional)", 
    "data": {} // all the variables, e.g. for the STRING TEMPLATEs (Optional)
  },
  "threshold": 0.0, // the minimum match threshold
  "max_results": 4 // the max number of result to return
}

```

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes

##### 200

Similar Json, see examples below

###### Example 1

User input is "how to install starchat":

```bash
QUERY=${1:-"how to install starchat"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
 -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/get_next_response -d "{
        \"conversation_id\": \"1234\",
        \"user_input\": { \"text\": \"${QUERY}\" },
        \"values\": {
                \"return_value\": \"\",
                \"data\": {\"varname1\": \"value1\", \"varname2\": \"value2\"}
        },
        \"threshold\": 0.0,
        \"max_results\": 4
}"
```

returns:

```json
[
   {
      "bubble" : "Just choose one of the two:\n<ul>\n<li>docker install (recommended)</li>\n<li>standalone install</li>\n</ul>",
      "analyzer" : "band(bor(keyword(\"setup\"), keyword(\"install.*\")), bnot(bor(keyword(\"standalone\"), keyword(\"docker\"))))",
      "state_data" : {},
      "data" : {
         "varname1" : "value1",
         "varname2" : "value2"
      },
      "failure_value" : "",
      "state" : "install",
      "traversed_states" : [
         "install"
      ],
      "conversation_id" : "1234",
      "action" : "",
      "action_input" : {},
      "score" : 1,
      "max_state_count" : 0,
      "success_value" : ""
   }
]
```

###### Example 2

User input is: "how can I contribute to starchat?"

```bash
QUERY=${1:-"how can I contribute to starchat?"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
 -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/get_next_response -d "{
        \"conversation_id\": \"1234\",
        \"user_input\": { \"text\": \"${QUERY}\" },
        \"values\": {
                \"return_value\": \"\",
                \"data\": {\"varname1\": \"value1\", \"varname2\": \"value2\"}
        },
        \"threshold\": 0.0,
        \"max_results\": 4
}"
```

and gets:

```json
[
   {
      "analyzer" : "bor(keyword(\"contribute\"))",
      "state_data" : {},
      "data" : {
         "varname2" : "value2",
         "varname1" : "value1"
      },
      "traversed_states" : [
         "contribute"
      ],
      "conversation_id" : "1234",
      "bubble" : "To contribute to <a href=\"http://git.io/*chat\">StarChat</a>, please send us a pull request from your fork of this repository.\n<br>Our concise contribution guideline contains the bare minimum requirements of the code contributions.\n<br>Before contributing (or opening issues), you might want to email us at starchat@getjenny.com.",
      "action" : "",
      "state" : "contribute",
      "max_state_count" : 0,
      "score" : 1,
      "failure_value" : "",
      "action_input" : {},
      "success_value" : ""
   }
]
```

##### 204

No response was found

##### 500 (error)

Internal server error

##### 400 (error)

Bad request: 

* meaning: the input data structure is not valid
* output data: no data returned

##### 422 (error)

* meaning: bad request data, the input data is formally valid but there is some issue with data interpretation
* output data: the output data structure is a json dictionary with two fields: code and message. The following code are supported:
* code: 100
* message: "error evaluating the template strings, bad values"

##### 404 (error)

* meaning: not found
* output data: no data returned

### GET /decisiontable

Get a document by ID

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
# retrieve one or more entries with given ids; ids can be specified multiple times
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
# retrieve one or more entries with given ids; ids can be specified multiple times
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" "http://localhost:${PORT}/${INDEX_NAME}/decisiontable?ids=further_details_access_question"
```

Sample output

```json
{
   "total" : 1,
   "hits" : [
      {
         "document" : {
            "failure_value" : "dont_understand",
            "state" : "further_details_access_question",
            "bubble" : "Can you specify which of the following problems you have? [NB works only if buttons can be shown!]",
            "success_value" : "eval(show_buttons)",
            "action_input" : {
               "I want to call an operator" : "call_operator",
               "Forgot Password" : "forgot_password",
               "Specify your problem" : "specify_problem",
               "None of the above" : "start",
               "Account locked" : "account_locked"
            },
            "max_state_count" : 0,
            "analyzer" : "or(and(or(keyword(\"problem.*\"),keyword(\"issue.*\"),keyword(\"trouble.*\")),keyword(\"account\")))",
            "queries" : [
               "cannot access account",
               "problem access account"
            ],
            "action" : "show_buttons",
            "state_data" : {
               "verification" : "did you mean you can't access to your account?"
            },
            "execution_order" : 1
         },
         "score" : 0
      }
   ],
   "max_score" : 0
}
```

### PUT /decisiontable
 
Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes

##### 201

Sample call

```bash
# update the "further_details_access_question" entry in the DT
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
# update the "further_details_access_question" entry in the DT
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X PUT http://localhost:${PORT}/${INDEX_NAME}/decisiontable/further_details_access_question -d '{
        "queries": ["cannot access account", "problem access account", "unable to access to my account", "completely forgot my password"]
}'
```

Sample output
```json
{
   "dtype" : "state",
   "created" : false,
   "index" : "index_getjenny_english_0.state",
   "version" : 2,
   "id" : "further_details_access_question"
}
```

### POST /decisiontable

Insert a new document.

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

#### Return codes

##### 201

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/decisiontable -d '{
        "state": "further_details_access_question",
        "max_state_count": 0,
        "execution_order": 0,
        "analyzer": "",
        "queries": ["cannot access account", "problem access account"],
        "bubble": "What seems to be the problem exactly?",
        "action": "show_buttons",
        "action_input": {"Forgot Password": "forgot_password", "Account locked": "account_locked", "Payment problem": "payment_problem", "Specify your problem": "specify_problem", "I want to call an operator": "call_operator", "None of the above": "start"},
    "state_data": {},
        "success_value": "eval(show_buttons)",
        "failure_value": "dont_understand"
}'
```

Sample output

```json
{
    "created": true,
    "dtype": "state",
    "id": "further_details_access_question",
    "index": "index_getjenny_english_0.state",
    "version": 1
}
```

### DELETE /decisiontable

Delete a document by ID

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 200

Sample call
```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
 -H "Content-Type: application/json" -X DELETE http://localhost:${PORT}/${INDEX_NAME}/decisiontable/further_details_access_question
```

Sample output

```json
{
  "dtype": "state",
  "version": 7,
  "found": true,
  "id": "further_details_access_question",
  "index":"index_getjenny_english_0.state"
}
```

Sample call: delete all

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
 -H "Content-Type: application/json" -X DELETE http://localhost:${PORT}/${INDEX_NAME}/decisiontable
```

Sample output: delete all

```json
{
   "message" : "delete",
   "deleted" : 18
}
```


### POST /decisiontable_upload_csv

upload load a csv file on decisiontable

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

#### Return codes

##### 201

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-'index_getjenny_english_0'}
FILENAME=${3:-"`readlink -e ../../doc/decision_table_starchat_doc.csv`"}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
 -X POST --form "csv=@${FILENAME}" http://localhost:8888/${INDEX_NAME}/decisiontable_upload_csv
```

Sample output

```json
{
   "data" : [
      {
         "index" : "index_getjenny_english_0.state",
         "id" : "help",
         "created" : true,
         "dtype" : "state",
         "version" : 1
      },
      {
         "dtype" : "state",
         "version" : 1,
         "created" : true,
         "id" : "further_details_access_question",
         "index" : "index_getjenny_english_0.state"
      },
      {
         "id" : "contribute",
         "created" : true,
         "index" : "index_getjenny_english_0.state",
         "dtype" : "state",
         "version" : 1
      },
      {
         "version" : 1,
         "dtype" : "state",
         "index" : "index_getjenny_english_0.state",
         "created" : true,
         "id" : "quickstart"
      },
      {
         "dtype" : "state",
         "version" : 1,
         "index" : "index_getjenny_english_0.state",
         "created" : true,
         "id" : "docker_install"
      },
      {
         "dtype" : "state",
         "version" : 1,
         "index" : "index_getjenny_english_0.state",
         "id" : "create_es_indices",
         "created" : true
      },
      {
         "dtype" : "state",
         "version" : 1,
         "id" : "delete_es_indexes",
         "created" : true,
         "index" : "index_getjenny_english_0.state"
      },
      {
         "version" : 1,
         "dtype" : "state",
         "index" : "index_getjenny_english_0.state",
         "created" : true,
         "id" : "create_es_indexes"
      },
      {
         "created" : true,
         "id" : "index_data",
         "index" : "index_getjenny_english_0.state",
         "dtype" : "state",
         "version" : 1
      },
      {
         "version" : 1,
         "dtype" : "state",
         "id" : "index_analyzer",
         "created" : true,
         "index" : "index_getjenny_english_0.state"
      },
      {
         "index" : "index_getjenny_english_0.state",
         "id" : "load_conf_file",
         "created" : true,
         "dtype" : "state",
         "version" : 1
      },
      {
         "dtype" : "state",
         "version" : 1,
         "created" : true,
         "id" : "install",
         "index" : "index_getjenny_english_0.state"
      },
      {
         "index" : "index_getjenny_english_0.state",
         "id" : "standalone_install",
         "created" : true,
         "dtype" : "state",
         "version" : 1
      },
      {
         "dtype" : "state",
         "version" : 1,
         "index" : "index_getjenny_english_0.state",
         "id" : "code_78",
         "created" : true
      },
      {
         "index" : "index_getjenny_english_0.state",
         "id" : "licence",
         "created" : true,
         "dtype" : "state",
         "version" : 1
      },
      {
         "index" : "index_getjenny_english_0.state",
         "created" : true,
         "id" : "terrible_feedback",
         "version" : 1,
         "dtype" : "state"
      },
      {
         "id" : "call_operator",
         "created" : true,
         "index" : "index_getjenny_english_0.state",
         "dtype" : "state",
         "version" : 1
      },
      {
         "version" : 1,
         "dtype" : "state",
         "id" : "any_further",
         "created" : true,
         "index" : "index_getjenny_english_0.state"
      },
      {
         "index" : "index_getjenny_english_0.state",
         "id" : "dont_understand",
         "created" : true,
         "version" : 1,
         "dtype" : "state"
      }
   ]
}
```

### POST /decisiontable_search

Update a document

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
Q="${1:-'cannot access my account'}"
S="${2:-0.0}"
B="${3:-100.0}"
PORT=${4:-8888}
INDEX_NAME=${5:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/decisiontable_search -d "{
        \"queries\": \"${Q}\",
        \"min_score\": ${S},
        \"boost_exact_match_factor\": ${B},
        \"from\": 0,
        \"size\": 10
}"
```

Sample response 

```json
{
   "max_score" : 140.38037109375,
   "total" : 1,
   "hits" : [
      {
         "score" : 140.38037109375,
         "document" : {
            "action" : "show_buttons",
            "state_data" : {
               "verification" : "did you mean you can't access to your account?"
            },
            "success_value" : "eval(show_buttons)",
            "execution_order" : 1,
            "bubble" : "Can you specify which of the following problems you have? [NB works only if buttons can be shown!]",
            "action_input" : {
               "None of the above" : "start",
               "Forgot Password" : "forgot_password",
               "I want to call an operator" : "call_operator",
               "Specify your problem" : "specify_problem",
               "Account locked" : "account_locked"
            },
            "queries" : [
               "cannot access account",
               "problem access account"
            ],
            "failure_value" : "dont_understand",
            "max_state_count" : 0,
            "analyzer" : "or(and(or(keyword(\"problem.*\"),keyword(\"issue.*\"),keyword(\"trouble.*\")),keyword(\"account\")))",
            "state" : "further_details_access_question"
         }
      }
   ]
}
```

### GET /decisiontable_analyzer

Get and return the map of analyzer for each state

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X GET "http://localhost:${PORT}/${INDEX_NAME}/decisiontable_analyzer"
```

Sample response

```json
{
   "analyzer_map" : {
      "help" : {
         "execution_order" : 1,
         "build" : true,
         "analyzer" : "band(keyword(\"help\"))"
      },
      "contribute" : {
         "build" : true,
         "analyzer" : "bor(keyword(\"contribute\"))",
         "execution_order" : 1
      },
      "index_analyzer" : {
         "analyzer" : "band(bor(keyword(\"index\"),keyword(\"load\")), keyword(\"analyzer\"))",
         "build" : true,
         "execution_order" : 1
      },
      "further_details_access_question" : {
         "analyzer" : "or(and(or(keyword(\"problem.*\"),keyword(\"issue.*\"),keyword(\"trouble.*\")),keyword(\"account\")))",
         "build" : true,
         "execution_order" : 1
      },
      "create_es_indices" : {
         "analyzer" : "band(keyword(\"create\"), keyword(\"elastic.*\"),  bor(keyword(\"index\"),  keyword(\"indices\"),  keyword(\"indeces\"),  keyword(\"indexes\")))",
         "build" : true,
         "execution_order" : 1
      },
      "create_es_indexes" : {
         "build" : true,
         "analyzer" : "band(keyword(\"create\"), bor(keyword(\"index.*\"), keyword(\"indic.*\")))",
         "execution_order" : 1
      },
      "call_operator" : {
         "analyzer" : "band(bor(keyword(\"call\"),keyword(\"talk\"),keyword(\"speak\")),keyword(\"operator\"))",
         "build" : true,
         "execution_order" : 1
      },
      "index_data" : {
         "execution_order" : 1,
         "analyzer" : "band(keyword(\"index\"), keyword(\"data\"))",
         "build" : true
      },
      "install" : {
         "execution_order" : 1,
         "build" : true,
         "analyzer" : "band(bor(keyword(\"setup\"), keyword(\"install.*\")), bnot(bor(keyword(\"standalone\"), keyword(\"docker\"))))"
      },
      "load_conf_file" : {
         "execution_order" : 1,
         "analyzer" : "band(keyword(\"load.*\"), bor(keyword(\"config.*\"), band(keyword(\"decision\"), keyword(\"table\"))), keyword(\"file.*\"))",
         "build" : true
      },
      "docker_install" : {
         "execution_order" : 1,
         "analyzer" : "band(keyword(\"docker\"), keyword(\"install.*\"))",
         "build" : true
      },
      "delete_es_indexes" : {
         "execution_order" : 1,
         "build" : true,
         "analyzer" : "band(keyword(\"delete\"), bor(keyword(\"index.*\"), keyword(\"indic.*\")))"
      },
      "code_78" : {
         "analyzer" : "band(keyword(\"code\"),keyword(\"78\"))",
         "build" : true,
         "execution_order" : 1
      },
      "terrible_feedback" : {
         "execution_order" : 1,
         "build" : true,
         "analyzer" : "booleanor(keyword(\"idiot\"), keyword(\"fuck.*\"), keyword(\"screw\"), keyword(\"damn.*\"), keyword(\"asshole\"))"
      },
      "standalone_install" : {
         "analyzer" : "band(keyword(\"standal.*\"), keyword(\"install\"))",
         "build" : true,
         "execution_order" : 1
      },
      "quickstart" : {
         "execution_order" : 1,
         "build" : true,
         "analyzer" : "band(bor(keyword(\"start\"), keyword(\"quickstart\")), keyword(\"starchat\"))"
      },
      "licence" : {
         "build" : true,
         "analyzer" : "bor(band(keyword(\"open\"), keyword(\"source\")), keyword(\"opensource\"), keyword(\"licence\"))",
         "execution_order" : 1
      }
   }
}
```

### POST /decisiontable_analyzer

Load/reload the map of analyzer from ES

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/decisiontable_analyzer"
```

Sample response

```json
{
   "num_of_entries" : 17
}
```

## QuestionAnswer (KnowledgeBase, PriorData, ConversationLogs)

The following REST endpoints have the same syntax and semantic
for KnowledgeBase, PriorData, ConversationLogs.

The only difference is the route path component:
* KnowledgeBase: "knowledgebase"
* Priodata: "prior_data"
* ConversationLogs: "conversation_logs"

In the API description we will refer to this path component as <QAPath>

### GET /<QAPath>

Return a document by ID

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
ID=${1:-0}
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
# retrieve one or more entries with given ids; ids can be specified multiple times
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" "http://localhost:${PORT}/${INDEX_NAME}/knowledgebase?ids=${ID}"
```

Sample response

```json
{
   "max_score" : 0,
   "total" : 1,
   "hits" : [
      {
         "score" : 0,
         "document" : {
            "conversation" : "id:1000",
            "id" : "0",
            "status" : 0,
            "question_scored_terms" : [
               [
                  "thank",
		  1.09
               ]
            ],
            "verified" : true,
            "answer" : "you are welcome!",
            "topics" : "t1 t2",
            "doctype" : "normal",
            "index_in_conversation" : 1,
            "question" : "thank you",
            "question_negative" : [
              "thank you anyway"
            ],
            "state" : ""
         }
      }
   ]
}
```

### POST /<QAPath>

Insert a new document

#### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 201

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/knowledgebase -d '{
	"id": "0",
	"conversation": "id:1000",
	"index_in_conversation": 1,
	"question": "thank you",
        "question_negative": ["ok, I will not talk with you anymore", "thank you anyway"],
	"answer": "you are welcome!",
	"question_scored_terms": [
		[
			"currently",
			1.0901874131103333
		],
		[
			"installing",
			2.11472759638322
		],
		[
			"mac",
			9.000484252244254
		],
		[
			"reset",
			4.34483238516225
		],
		[
			"app",
			1.2219061535961406
		],
		[
			"device",
			2.1679468390743414E-213
		],
		[
			"devices",
			4.1987625801077624E-268
		]
	],
	"verified": true,
	"topics": "t1 t2",
	"doctype": "normal",
	"state": "",
	"status": 0
}'
```

Sample response

```json
{
   "created" : true,
   "dtype" : "question",
   "version" : 1,
   "index" : "index_getjenny_english_0.question",
   "id" : "0"
}
```

### DELETE /<QAPath>

Delete a document by ID

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X DELETE http://localhost:${PORT}/${INDEX_NAME}/knowledgebase/0
```

Sample output

```bash
{
    "dtype" : "question",
    "version" : 5,
    "found" : false,
    "id" : "0",
    "index" : "index_getjenny_english_0.question"
}
```

Sample call: delete all

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
 -H "Content-Type: application/json" -X DELETE http://localhost:${PORT}/${INDEX_NAME}/knowledgebase
```

Sample output: delete all

```json
{
   "message" : "delete",
   "deleted" : 2
}
```

### PUT /<QAPath>

Update an existing document

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X PUT http://localhost:${PORT}/${INDEX_NAME}/knowledgebase/0 -d '{
        "conversation": "id:1001",
        "question": "thank you",
        "answer": "you are welcome!",
        "verified": true,
        "topics": "t1 t2",
        "doctype": "normal",
        "state": "",
        "status": 0
}' 
```

Sample response

```json
{
   "dtype" : "question",
   "index" : "index_getjenny_english_0.question",
   "id" : "0",
   "version" : 4,
   "created" : false
}
```

### POST /<QAPath>_search

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
QUERY=${1:-"how are you?"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/knowledgebase_search -d "{
        \"question\": \"${QUERY}\",
        \"doctype\": \"normal\",
        \"min_score\": 0.0
}"
```

Sample output

```json
{
   "hits" : [
      {
         "document" : {
            "question_scored_terms" : [
               [
                  "currently",
                  1.09018741311033
               ],
               [
                  "installing",
                  2.11472759638322
               ],
               [
                  "mac",
                  9.00048425224425
               ],
               [
                  "reset",
                  4.34483238516225
               ],
               [
                  "app",
                  1.22190615359614
               ],
               [
                  "device",
                  2.16794683907434e-213
               ],
               [
                  "devices",
                  4.19876258010776e-268
               ]
            ],
            "question" : "thank you",
            "id" : "0",
            "state" : "",
            "conversation" : "id:1001",
            "answer" : "you are welcome!",
            "topics" : "t1 t2",
            "index_in_conversation" : 1,
            "doctype" : "normal",
            "status" : 0,
            "verified" : true,
            "question_negative" : [
               "ok, I will not talk with you anymore",
               "thank you anyway"
            ]
         },
         "score" : 0.287682086229324
      }
   ],
   "total" : 1,
   "max_score" : 0.287682086229324
}
```

### GET /term_count/<QAPath>

Count the occurrence of a term.

The endpoint supports the following query arguments:
* field: can be "answer" or "question" (optional, default question)
* stale: represent the max stale time in millis ; 0 means no cache (Optional)
* term: the term to count (mandatory)

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
TERM=${1:-"hello"}
ROUTE=${2:-knowledgebase}
INDEX_NAME=${3:-index_getjenny_english_0}
FIELD=${4:-"question"}
PORT=${5:-8888}
curl -v -H "Authorization: Basic $(echo -n 'test_user:p4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X GET "http://localhost:${PORT}/${INDEX_NAME}/term_count/${ROUTE}?field=${FIELD}&term=${TERM}"
```

Sample response

```json
{
   "numDocs" : 994,
   "count" : 1037
}
```

### GET /dict_size/<QAPath>

Measure the dictionary size: number of unique terms on question and answer field

The endpoint supports the following query arguments:
* stale: represent the max stale time in millis ; 0 means no cache (Optional)

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
PORT=${1:-8888}
DATATYPE=${2:-knowledgebase}
INDEX_NAME=${3:-index_getjenny_english_0}
curl -v -H "Authorization: Basic $(echo -n 'test_user:p4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X GET "http://localhost:${PORT}/${INDEX_NAME}/dict_size/${DATATYPE}"
```

Sample response

```json
{
   "numDocs" : 135865,
   "answer" : 9171,
   "question" : 25200,
   "total" : 29258
}
```

### GET /total_terms/<QAPath>

calculate the total number of terms

The endpoint supports the following query arguments:
* stale: represent the max stale time in millis ; 0 means no cache (Optional)

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
DATATYPE=${3:-knowledgebase}
curl -v -H "Authorization: Basic $(echo -n 'test_user:p4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X GET "http://localhost:${PORT}/${INDEX_NAME}/total_terms/${DATATYPE}"
```

Sample response

```json
{
   "question" : 821982,
   "numDocs" : 135865,
   "answer" : 1098490
}
```

### GET /cache/<QAPath>

return the cache counter parameters and the number of elements in cache

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
INDEX_NAME=${1:-index_getjenny_english_0}
PORT=${2:-8888}
ROUTE=${3:-knowledgebase}
curl -v -H "Authorization: Basic $(echo -n 'admin:adminp4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X GET http://localhost:${PORT}/${INDEX_NAME}/cache/${ROUTE} 
```

Sample response

```json
[
   {
      "totalTermsCacheMaxSize" : 1000,
      "countTermCacheMaxSize" : 100000,
      "dictSizeCacheMaxSize" : 1000,
      "cacheStealTimeMillis" : 43200000
   },
   {
      "countTermCacheSize" : 0,
      "dictSizeCacheSize" : 0,
      "totalTermsCacheSize" : 0
   }
]
```

### POST /cache/<QAPath>

set the cache counter parameters

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
INDEX_NAME=${1:-index_getjenny_english_0}
PORT=${2:-8888}
ROUTE=${3:-knowledgebase}
curl -v -H "Authorization: Basic $(echo -n 'admin:adminp4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/cache/${ROUTE} -d'{ 
    "dictSizeCacheMaxSize": 1000,
    "totalTermsCacheMaxSize": 1000,
    "countTermCacheMaxSize": 100000,
    "cacheStealTimeMillis": 43200000
}'
```

Sample response

```json
{
   "totalTermsCacheMaxSize" : 1000,
   "dictSizeCacheMaxSize" : 1000,
   "countTermCacheMaxSize" : 100000,
   "cacheStealTimeMillis" : 43200000
}
```

### DELETE /cache/<QAPath>

reset the cache for counters

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
INDEX_NAME=${1:-index_getjenny_english_0}
PORT=${2:-8888}
ROUTE=${3:-knowledgebase}
curl -v -H "Authorization: Basic $(echo -n 'admin:adminp4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X DELETE http://localhost:${PORT}/${INDEX_NAME}/cache/${ROUTE}
```

Sample response

```json
[
   {
      "countTermCacheMaxSize" : 100000,
      "totalTermsCacheMaxSize" : 1000,
      "dictSizeCacheMaxSize" : 1000,
      "cacheStealTimeMillis" : 43200000
   },
   {
      "totalTermsCacheSize" : 0,
      "dictSizeCacheSize" : 0,
      "countTermCacheSize" : 0
   }
]
```

### PUT /updateTerms/<QAPath>

update the manaus terms for a QuestionAnswer document

Output JSON

#### Requirements

* Index must exists
* The document to update must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
DOCID=${1:-0}
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
ROUTE=${4:-knowledgebase}
curl -v -H "Authorization: Basic $(echo -n 'test_user:p4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X PUT http://localhost:${PORT}/${INDEX_NAME}/updateTerms/${ROUTE} -d"{
        \"id\": \"${DOCID}\"
}"
```

Sample response

```json
[
   {
      "id" : "0",
      "created" : false,
      "dtype" : "question_answer",
      "version" : 2,
      "index" : "index_getjenny_english_0.question_answer"
   }
]

```

### GET /updateTerms/<QAPath>

update the manaus terms for all the QuestionAnswer documents

Output JSON

#### Requirements

* Index must exists
* The user has stream permissions on the index
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call
```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
ROUTE=${3:-knowledgebase}
curl -v -H "Authorization: Basic $(echo -n 'test_user:p4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X GET http://localhost:${PORT}/${INDEX_NAME}/updateTerms/${ROUTE} -d'{
        "id": ""
}'
```

Sample response

```json
{"dtype":"question_answer","version":2,"id":"3bcafc79400f201bcbb2e9290d60231fb096845d97cd3a83b032eab90a9b6f8f","index":"index_getjenny_english_0.question_answer","created":false}
{"dtype":"question_answer","version":2,"id":"f5239013416fc98c558560659b00ec96e694df243aace574e9dd32166e982ccd","index":"index_getjenny_english_0.question_answer","created":false}
{"dtype":"question_answer","version":2,"id":"23cb4dc14284b278867b743d6e7387a3ba20e96afb4a34ddf58a8397b173a24f","index":"index_getjenny_english_0.question_answer","created":false}
{"dtype":"question_answer","version":2,"id":"5f948728fbca7617907ab4ad031020f9da214926f2fbe272295c799d568a3491","index":"index_getjenny_english_0.question_answer","created":false}
{"dtype":"question_answer","version":2,"id":"cfb544023bcc1378a59695bc6135fb7b71bfcdaba6fb2429e151aba73b2774b8","index":"index_getjenny_english_0.question_answer","created":false}
{"dtype":"question_answer","version":2,"id":"cce15f6984f473fe7534f9e0f699e5e11a61e39cbe82989818a023d626ea5e1f","index":"index_getjenny_english_0.question_answer","created":false}
{"dtype":"question_answer","version":2,"id":"b37db2bef879f22d3792840efa1a41b1d46cdbee4af5ccb1b935b3c944732544","index":"index_getjenny_english_0.question_answer","created":false}
```

## TermsExtraction

### POST /extraction/frequencies

Extract term frequencies from a sentence
The fields of the data structure have the following meaning:

* commonOrSpecificSearchPrior: specify if the prior_data table is from the common index or is index specific. Values are COMMON or IDXSPECIFIC
* commonOrSpecificSearchObserved: specify if the knowledgebase table is from the common index or is index specific. Values are COMMON or IDXSPECIFIC
* observedDataSource: specify which table should be considered for observed data, values are KNOWLEDGEBASE or CONV_LOGS
* fieldsPrior: specify the fields which must be considered, values are question, answer, all
* fieldsObserved: specify the fields which must be considered, values are question, answer, all

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
QUERY=${1:-"good morning, may I ask you a question?"}
PORT=${2:-8888}
INDEX_NAME=${4:-index_getjenny_english_0}
curl -v -H "Authorization: Basic $(echo -n 'test_user:p4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/extraction/frequencies" -d"
{
        \"text\": \"${QUERY}\",
        \"commonOrSpecificSearchPrior\": \"COMMON\",
        \"commonOrSpecificSearchObserved\": \"IDXSPECIFIC\",
        \"observedDataSource\": \"KNOWLEDGEBASE\",
        \"fieldsPrior\": \"all\",
        \"fieldsObserved\": \"all\",
        \"tokenizer\": \"space_punctuation\"
}"
```

Sample output

```json
{
   "observedTotalTerms" : 1920472,
   "priorTotalTerms" : 2311033,
   "tokensFreq" : [
      {
         "observedFrequency" : 1187,
         "token" : "good",
         "priorFrequency" : 7024
      },
      {
         "priorFrequency" : 1228,
         "token" : "morning",
         "observedFrequency" : 246
      },
      {
         "priorFrequency" : 1343,
         "observedFrequency" : 4185,
         "token" : "may"
      },
      {
         "observedFrequency" : 68554,
         "token" : "i",
         "priorFrequency" : 79384
      },
      {
         "priorFrequency" : 1377,
         "observedFrequency" : 471,
         "token" : "ask"
      },
      {
         "priorFrequency" : 93571,
         "observedFrequency" : 79470,
         "token" : "you"
      },
      {
         "priorFrequency" : 47701,
         "token" : "a",
         "observedFrequency" : 25688
      },
      {
         "observedFrequency" : 539,
         "token" : "question",
         "priorFrequency" : 525
      }
   ]
}
```

### POST /extraction/frequencies

Extract manaus terms from a sentence
The fields of the data structure have the following meaning:

* commonOrSpecificSearchPrior: specify if the prior_data table is from the common index or is index specific. Values are COMMON or IDXSPECIFIC
* commonOrSpecificSearchObserved: specify if the knowledgebase table is from the common index or is index specific. Values are COMMON or IDXSPECIFIC
* observedDataSource: specify which table should be considered for observed data, values are KNOWLEDGEBASE or CONV_LOGS
* fieldsPrior: specify the fields which must be considered, values are question, answer, all
* fieldsObserved: specify the fields which must be considered, values are question, answer, all

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
QUERY=${1:-"good morning, may I ask you a question?"}
PORT=${2:-8888}
INDEX_NAME=${4:-index_getjenny_english_0}
curl -v -H "Authorization: Basic $(echo -n 'test_user:p4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/extraction/keywords" -d "
{
        \"text\": \"${QUERY}\",
        \"commonOrSpecificSearchPrior\": \"COMMON\",
        \"commonOrSpecificSearchObserved\": \"IDXSPECIFIC\",
        \"observedDataSource\": \"KNOWLEDGEBASE\",
        \"fieldsPrior\": \"all\",
        \"fieldsObserved\": \"all\",
        \"minWordsPerSentence\": 5,
        \"pruneTermsThreshold\": 100000,
        \"misspellMaxOccurrence\": 10,
        \"activePotentialDecay\": 1,
        \"activePotential\": true,
        \"totalInfo\": true
}
"
```

Sample output

```json
{
   "question" : 0.229660917208135
}
```

### POST /extraction/synonyms

Tokenize a sentence, extract terms and decorate each term with manaus informations and the possible synonyms using a vector model.
The fields of the data structure have the following meaning:

* text: the text to analyze
* tokenizer: the tokenizer, default is "base"
* sentencesThreshold: discard sentences if the sentence with the synonym is below the threshold, default us 0.0
* synonymsThreshold: discard the synonyms if the distance is below the threshold, default is 0.0
* distanceFunction: a distance function values are SUMCOSINE (sum of word vectors) or EMDCOSINE (Earth movers distance)
* commonOrSpecificSearchTerms: specify if the terms table is from the common index or is index specific. Values are COMMON or IDXSPECIFIC
* commonOrSpecificSearchPrior: specify if the prior_data table is from the common index or is index specific. Values are COMMON or IDXSPECIFIC
* commonOrSpecificSearchObserved: specify if the knowledgebase table is from the common index or is index specific. Values are COMMON or IDXSPECIFIC
* observedDataSource: specify which table should be considered for observed data, values are KNOWLEDGEBASE or CONV_LOGS
* fieldsPrior: specify the fields which must be considered, values are question, answer, all
* fieldsObserved: specify the fields which must be considered, values are question, answer, all
* minWordsPerSentence: manaus parameter, ignore sentences with less than N terms, default is 10
* pruneTermsThreshold: manaus parameter, a threshold on the number of terms for trigger pruning, default value is 100000
* misspellMaxOccurrence: manaus parameter, discard terms with a frequency below the threshold, default is 5
* activePotentialDecay: manaus parameter, a decay value for the active potential
* activePotential: manaus parameter, tell wether to calculate the active potential or not, default is true
* minSentenceInfoBit: min number of information bits for the sentence
* minKeywordInfo: min number of information bits for a word
* totalInfo: manaus parameter, tell wether to consider the total info or not, default is true

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
QUERY=${1:-"good morning, may I ask you a question?"}
PORT=${2:-8888}
INDEX_NAME=${4:-index_getjenny_english_0}

curl -v -H "Authorization: Basic $(echo -n 'test_user:p4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/extraction/synonyms" -d "
{
        \"text\": \"${QUERY}\",
        \"tokenizer\": \"base\",
        \"sentencesThreshold\": 0.9,
        \"synonymsThreshold\": 0.3,
        \"distanceFunction\": \"EMDCOSINE\",
        \"commonOrSpecificSearchPrior\": \"COMMON\",
        \"commonOrSpecificSearchObserved\": \"IDXSPECIFIC\",
        \"observedDataSource\": \"KNOWLEDGEBASE\",
        \"fieldsPrior\": \"all\",
        \"fieldsObserved\": \"all\",
        \"minWordsPerSentence\": 5,
        \"pruneTermsThreshold\": 100000,
        \"misspellMaxOccurrence\": 10,
        \"activePotentialDecay\": 1,
        \"activePotential\": true,
        \"totalInfo\": true
}
"
```

Sample output

```json
[
   {
      "synonymItem" : [
         {
            "termSimilarityScore" : 0.303461623209445,
            "synonym" : "serious",
            "synonymScore" : 0.988391942416316,
            "textDistanceWithSynonym" : 0.988391942416316
         },
         {
            "textDistanceWithSynonym" : 0.98686397528116,
            "synonymScore" : 0.98686397528116,
            "synonym" : "safe",
            "termSimilarityScore" : 0.310671998517923
         },
         {
            "textDistanceWithSynonym" : 0.986168126185455,
            "synonymScore" : 0.986168126185455,
            "synonym" : "effective",
            "termSimilarityScore" : 0.337205437720856
         },
         {
            "textDistanceWithSynonym" : 0.985984970241292,
            "synonymScore" : 0.985984970241292,
            "termSimilarityScore" : 0.30013467985373,
            "synonym" : "salutary"
         },
         {
            "textDistanceWithSynonym" : 0.985895039899644,
            "synonymScore" : 0.985895039899644,
            "termSimilarityScore" : 0.301306430755499,
            "synonym" : "dear"
         },
         {
            "termSimilarityScore" : 0.319592899798586,
            "synonym" : "estimable",
            "synonymScore" : 0.985759252837724,
            "textDistanceWithSynonym" : 0.985759252837724
         },
         {
            "textDistanceWithSynonym" : 0.985249380845675,
            "synonymScore" : 0.985249380845675,
            "synonym" : "thoroughly",
            "termSimilarityScore" : 0.337964488042003
         },
         {
            "termSimilarityScore" : 0.360057098563907,
            "synonym" : "right",
            "synonymScore" : 0.984778611750602,
            "textDistanceWithSynonym" : 0.984778611750602
         },
         {
            "textDistanceWithSynonym" : 0.984227558357218,
            "synonymScore" : 0.984227558357218,
            "synonym" : "adept",
            "termSimilarityScore" : 0.330252861229703
         },
         {
            "termSimilarityScore" : 0.360657543519664,
            "synonym" : "expert",
            "synonymScore" : 0.984204967194361,
            "textDistanceWithSynonym" : 0.984204967194361
         },
         {
            "synonymScore" : 0.9839801846329,
            "textDistanceWithSynonym" : 0.9839801846329,
            "termSimilarityScore" : 0.387968844459879,
            "synonym" : "full"
         },
         {
            "synonymScore" : 0.983341764529566,
            "textDistanceWithSynonym" : 0.983341764529566,
            "termSimilarityScore" : 0.307056458999165,
            "synonym" : "skilful"
         },
         {
            "synonym" : "secure",
            "termSimilarityScore" : 0.386750372411043,
            "synonymScore" : 0.9828697525105,
            "textDistanceWithSynonym" : 0.9828697525105
         },
         {
            "termSimilarityScore" : 0.436633496912848,
            "synonym" : "near",
            "textDistanceWithSynonym" : 0.9823153867721,
            "synonymScore" : 0.9823153867721
         },
         {
            "textDistanceWithSynonym" : 0.98229857645758,
            "synonymScore" : 0.98229857645758,
            "termSimilarityScore" : 0.387852275460498,
            "synonym" : "soundly"
         },
         {
            "textDistanceWithSynonym" : 0.981854513089778,
            "synonymScore" : 0.981854513089778,
            "termSimilarityScore" : 0.375971735071977,
            "synonym" : "sound"
         },
         {
            "textDistanceWithSynonym" : 0.981012225642823,
            "synonymScore" : 0.981012225642823,
            "termSimilarityScore" : 0.349548530861366,
            "synonym" : "unspoiled"
         },
         {
            "textDistanceWithSynonym" : 0.980134947418645,
            "synonymScore" : 0.980134947418645,
            "synonym" : "honorable",
            "termSimilarityScore" : 0.36656103347195
         },
         {
            "synonym" : "ripe",
            "termSimilarityScore" : 0.361927513363036,
            "synonymScore" : 0.979828029318668,
            "textDistanceWithSynonym" : 0.979828029318668
         },
         {
            "termSimilarityScore" : 0.354594107764147,
            "synonym" : "proficient",
            "synonymScore" : 0.97897497858666,
            "textDistanceWithSynonym" : 0.97897497858666
         },
         {
            "synonym" : "commodity",
            "termSimilarityScore" : 0.400576579081682,
            "synonymScore" : 0.978353047238428,
            "textDistanceWithSynonym" : 0.978353047238428
         },
         {
            "textDistanceWithSynonym" : 0.978097513490299,
            "synonymScore" : 0.978097513490299,
            "synonym" : "practiced",
            "termSimilarityScore" : 0.417713639606198
         },
         {
            "synonym" : "unspoilt",
            "termSimilarityScore" : 0.344935826089365,
            "textDistanceWithSynonym" : 0.977965860781802,
            "synonymScore" : 0.977965860781802
         },
         {
            "textDistanceWithSynonym" : 0.977574200871494,
            "synonymScore" : 0.977574200871494,
            "termSimilarityScore" : 0.368691045104583,
            "synonym" : "upright"
         }
      ],
      "isKeywordToken" : false,
      "keywordExtractionScore" : 0,
      "token" : {
         "position" : 0,
         "token" : "good",
         "token_type" : "<ALPHANUM>",
         "start_offset" : 0,
         "end_offset" : 4
      }
   },
   {
      "keywordExtractionScore" : 0,
      "token" : {
         "position" : 1,
         "end_offset" : 12,
         "token" : "morning",
         "token_type" : "<ALPHANUM>",
         "start_offset" : 5
      },
      "synonymItem" : [
         {
            "synonymScore" : 0.982435251017214,
            "textDistanceWithSynonym" : 0.982435251017214,
            "termSimilarityScore" : 0.328796456566772,
            "synonym" : "dawning"
         },
         {
            "termSimilarityScore" : 0.320363553861273,
            "synonym" : "sunup",
            "textDistanceWithSynonym" : 0.981187446721127,
            "synonymScore" : 0.981187446721127
         },
         {
            "synonymScore" : 0.980827133578433,
            "textDistanceWithSynonym" : 0.980827133578433,
            "termSimilarityScore" : 0.351556981730201,
            "synonym" : "cockcrow"
         },
         {
            "synonym" : "dayspring",
            "termSimilarityScore" : 0.348148650582253,
            "textDistanceWithSynonym" : 0.98000434808506,
            "synonymScore" : 0.98000434808506
         },
         {
            "synonym" : "morn",
            "termSimilarityScore" : 0.310762418953851,
            "textDistanceWithSynonym" : 0.979694497080588,
            "synonymScore" : 0.979694497080588
         },
         {
            "synonymScore" : 0.975365086887233,
            "textDistanceWithSynonym" : 0.975365086887233,
            "synonym" : "aurora",
            "termSimilarityScore" : 0.403555992661807
         },
         {
            "synonym" : "first_light",
            "termSimilarityScore" : 0.389117497640442,
            "synonymScore" : 0.973279032352032,
            "textDistanceWithSynonym" : 0.973279032352032
         }
      ],
      "isKeywordToken" : false
   },
   {
      "keywordExtractionScore" : 0,
      "token" : {
         "position" : 2,
         "end_offset" : 17,
         "token" : "may",
         "token_type" : "<ALPHANUM>",
         "start_offset" : 14
      },
      "synonymItem" : [
         {
            "textDistanceWithSynonym" : 0.977735408571704,
            "synonymScore" : 0.977735408571704,
            "termSimilarityScore" : 0.430110506850729,
            "synonym" : "whitethorn"
         }
      ],
      "isKeywordToken" : false
   },
   {
      "keywordExtractionScore" : 0,
      "token" : {
         "position" : 3,
         "end_offset" : 19,
         "token_type" : "<ALPHANUM>",
         "start_offset" : 18,
         "token" : "i"
      },
      "synonymItem" : [
         {
            "synonymScore" : 0.990079660169983,
            "textDistanceWithSynonym" : 0.990079660169983,
            "termSimilarityScore" : 0.33685060765497,
            "synonym" : "one"
         },
         {
            "termSimilarityScore" : 0.402164830285073,
            "synonym" : "single",
            "textDistanceWithSynonym" : 0.982443662914587,
            "synonymScore" : 0.982443662914587
         },
         {
            "synonymScore" : 0.978915403087889,
            "textDistanceWithSynonym" : 0.978915403087889,
            "termSimilarityScore" : 0.421938029658631,
            "synonym" : "unity"
         },
         {
            "synonymScore" : 0.976796274715053,
            "textDistanceWithSynonym" : 0.976796274715053,
            "termSimilarityScore" : 0.411104820586555,
            "synonym" : "ace"
         },
         {
            "termSimilarityScore" : 0.409793052227205,
            "synonym" : "iodine",
            "synonymScore" : 0.971809310282386,
            "textDistanceWithSynonym" : 0.971809310282386
         },
         {
            "synonymScore" : 0.970492090788282,
            "textDistanceWithSynonym" : 0.970492090788282,
            "synonym" : "ane",
            "termSimilarityScore" : 0.426139287692068
         }
      ],
      "isKeywordToken" : false
   },
   {
      "isKeywordToken" : false,
      "synonymItem" : [
         {
            "termSimilarityScore" : 0.344875478792656,
            "synonym" : "involve",
            "synonymScore" : 0.985084402822893,
            "textDistanceWithSynonym" : 0.985084402822893
         },
         {
            "synonym" : "require",
            "termSimilarityScore" : 0.313982723187962,
            "synonymScore" : 0.985048997600635,
            "textDistanceWithSynonym" : 0.985048997600635
         },
         {
            "synonym" : "demand",
            "termSimilarityScore" : 0.340885896054422,
            "textDistanceWithSynonym" : 0.984973652020142,
            "synonymScore" : 0.984973652020142
         },
         {
            "textDistanceWithSynonym" : 0.984179229975704,
            "synonymScore" : 0.984179229975704,
            "termSimilarityScore" : 0.348080045767017,
            "synonym" : "necessitate"
         },
         {
            "termSimilarityScore" : 0.363270794438576,
            "synonym" : "postulate",
            "synonymScore" : 0.980113837090212,
            "textDistanceWithSynonym" : 0.980113837090212
         }
      ],
      "keywordExtractionScore" : 0,
      "token" : {
         "position" : 4,
         "end_offset" : 23,
         "token" : "ask",
         "start_offset" : 20,
         "token_type" : "<ALPHANUM>"
      }
   },
   {
      "token" : {
         "position" : 5,
         "end_offset" : 27,
         "token" : "you",
         "token_type" : "<ALPHANUM>",
         "start_offset" : 24
      },
      "keywordExtractionScore" : 0,
      "synonymItem" : [],
      "isKeywordToken" : false
   },
   {
      "isKeywordToken" : false,
      "synonymItem" : [
         {
            "synonym" : "angstrom",
            "termSimilarityScore" : 0.389515130731534,
            "synonymScore" : 0.981827876880409,
            "textDistanceWithSynonym" : 0.981827876880409
         },
         {
            "synonym" : "amp",
            "termSimilarityScore" : 0.410951525810052,
            "synonymScore" : 0.978805416352854,
            "textDistanceWithSynonym" : 0.978805416352854
         },
         {
            "termSimilarityScore" : 0.399132821680172,
            "synonym" : "ampere",
            "synonymScore" : 0.978184525582464,
            "textDistanceWithSynonym" : 0.978184525582464
         },
         {
            "textDistanceWithSynonym" : 0.970833723014969,
            "synonymScore" : 0.970833723014969,
            "synonym" : "adenine",
            "termSimilarityScore" : 0.417303913993053
         }
      ],
      "keywordExtractionScore" : 0,
      "token" : {
         "end_offset" : 29,
         "start_offset" : 28,
         "token_type" : "<ALPHANUM>",
         "token" : "a",
         "position" : 6
      }
   },
   {
      "token" : {
         "token_type" : "<ALPHANUM>",
         "start_offset" : 30,
         "token" : "question",
         "end_offset" : 38,
         "position" : 7
      },
      "keywordExtractionScore" : 0,
      "isKeywordToken" : false,
      "synonymItem" : [
         {
            "termSimilarityScore" : 0.302928140147459,
            "synonym" : "doubtfulness",
            "textDistanceWithSynonym" : 0.986332041308894,
            "synonymScore" : 0.986332041308894
         },
         {
            "textDistanceWithSynonym" : 0.984752050785226,
            "synonymScore" : 0.984752050785226,
            "synonym" : "interview",
            "termSimilarityScore" : 0.337223120697986
         },
         {
            "synonym" : "enquiry",
            "termSimilarityScore" : 0.30268890030876,
            "textDistanceWithSynonym" : 0.983960314458422,
            "synonymScore" : 0.983960314458422
         },
         {
            "synonym" : "interrogate",
            "termSimilarityScore" : 0.347190376576428,
            "synonymScore" : 0.982717754194259,
            "textDistanceWithSynonym" : 0.982717754194259
         },
         {
            "termSimilarityScore" : 0.402127027019405,
            "synonym" : "head",
            "textDistanceWithSynonym" : 0.981584262241235,
            "synonymScore" : 0.981584262241235
         },
         {
            "termSimilarityScore" : 0.353450470156032,
            "synonym" : "interrogation",
            "textDistanceWithSynonym" : 0.980529096104673,
            "synonymScore" : 0.980529096104673
         },
         {
            "termSimilarityScore" : 0.386000200471808,
            "synonym" : "motion",
            "textDistanceWithSynonym" : 0.978495629389701,
            "synonymScore" : 0.978495629389701
         },
         {
            "synonym" : "inquiry",
            "termSimilarityScore" : 0.314067668506634,
            "textDistanceWithSynonym" : 0.977996268442931,
            "synonymScore" : 0.977996268442931
         }
      ]
   }
]
```

## LanguageGuesser

### POST /language_guesser

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
QUERY=${1:-"good morning, may I ask you a question?"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/language_guesser" -d "
{
        \"input_text\": \"${QUERY}\"
}"
```

Sample output

```json
{
   "enhough_text" : false,
   "language" : "en",
   "confidence" : "MEDIUM",
   "score" : 0.571426689624786
}
```

### GET /language_guesser

Check if a language is recognizable by the guesser

Output JSON

#### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
LANG=${1:-en} 
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X GET "http://localhost:${PORT}/${INDEX_NAME}/language_guesser/${LANG}"
```

Sample output

```json
{
   "supported_languages" : {
      "languages" : {
         "en" : true
      }
   }
}
```

## IndexManagement

### POST /index_management/create

Output JSON

#### Requirements

* Index name must start with the "index_" prefix followed by a language, and a sequence of alphanumeric characters separated by underscore e.g. index_<LANG>_<SUFFIX>
* The function requires "admin" credentials

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/index_management/create"
```

Sample output

```json
{
   "message" : "IndexCreation: state(index_getjenny_english_0.state, true) question(index_getjenny_english_0.question, true) term(index_getjenny_english_0.term, true)"
}
```

### POST /index_management/refresh

Output JSON

#### Requirements

* The index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/index_management/refresh"
```

Sample output

```json
{
   "results" : [
      {
         "failed_shards" : [],
         "successful_shards_n" : 5,
         "failed_shards_n" : 0,
         "index_name" : "index_getjenny_english_0.state",
         "total_shards_n" : 10
      },
      {
         "failed_shards" : [],
         "total_shards_n" : 10,
         "index_name" : "index_getjenny_english_0.question",
         "failed_shards_n" : 0,
         "successful_shards_n" : 5
      },
      {
         "failed_shards" : [],
         "successful_shards_n" : 5,
         "index_name" : "index_getjenny_english_0.term",
         "failed_shards_n" : 0,
         "total_shards_n" : 10
      }
   ]
}
```

### GET /index_management

Output JSON

#### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X GET "http://localhost:${PORT}/${INDEX_NAME}/index_management"
```

Sample output

```json
{
   "message" : "IndexCheck: state(index_getjenny_english_0.state, true) question(index_getjenny_english_0.question, true) term(index_getjenny_english_0.term, true)"
}
```

### PUT /index_management

Output JSON

#### Requirements

* Index must exists
* The function requires "admin" credentials

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
LANGUAGE=${3:-english}
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
 -H "Content-Type: application/json" -X PUT "http://localhost:${PORT}/${INDEX_NAME}/${LANGUAGE}/index_management"
```

Sample output

```json
{
   "message" : "IndexCheck: state(index_getjenny_english_0.state, true) question(index_getjenny_english_0.question, true) term(index_getjenny_english_0.term, true)"
}
```

### DELETE /index_management

Output JSON

#### Requirements

* The index must exists
* The function requires admin credentials

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X DELETE "http://localhost:${PORT}/${INDEX_NAME}/index_management"
```

Sample output

```json
{
   "message" : "IndexDeletion: state(index_getjenny_english_0.state, true) question(index_getjenny_english_0.question, true) term(index_getjenny_english_0.term, true)"
}
```

## Term

### POST /term/index

Index the term as indicated in the JSON. 

#### Requirements

* The index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/term/index -d '{
	"terms": [
	    {
            "term": "मराठी",
            "frequency_base": 1.0,
            "frequency_stem": 1.0,
            "vector": [1.0, 2.0, 3.0],
            "synonyms":
            {
                "bla1": 0.1,
                "bla2": 0.2
            },
            "antonyms":
            {
                "bla3": 0.1,
                "bla4": 0.2
            },
            "tags": "tag1 tag2",
            "features":
            {
                "NUM": "S",
                "GEN": "M"
            }
	    },
	    {
            "term": "term2",
            "frequency_base": 1.0,
            "frequency_stem": 1.0,
            "vector": [1.0, 2.0, 3.0],
            "synonyms":
            {
                "bla1": 0.1,
                "bla2": 0.2
            },
            "antonyms":
            {
                "bla3": 0.1,
                "bla4": 0.2
            },
            "tags": "tag1 tag2",
            "features":
            {
                "NUM": "P",
                "GEN": "F"
            }
	    }
   ]
}'
```

Sample output

```json
{
   "data" : [
      {
         "version" : 1,
         "created" : true,
         "dtype" : "term",
         "index" : "jenny-en-0",
         "id" : "मराठी"
      },
      {
         "dtype" : "term",
         "created" : true,
         "version" : 1,
         "id" : "term2",
         "index" : "jenny-en-0"
      }
   ]
}
```

### POST /term/index_default_synonyms

Index a set of default synonyms provided with starchat.

#### Requirements

* The index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
GROUP_SIZE=${3:-1000}
curl --max-time 600 -v -H "Authorization: Basic $(echo -n 'admin:adminp4ssw0rd' | base64)" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/term/index_default_synonyms?groupsize=${GROUP_SIZE}"
```

Sample output

```json
{
   "message" : "Indexed synonyms, blocks of 1000 items => success(124) failures(0)",
   "code" : 100
}
```
### POST /term/get

Get one or more terms entry.

#### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
QUERY=${1:-"\"term\""}
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/term/get -d "{
        \"ids\": [${QUERY}]
}"
```

Sample output

```json
{
   "terms" : [
      {
         "vector" : [
            1,
            2,
            3
         ],
        "frequency_base": 1.0,
        "frequency_stem": 1.0,
         "term" : "मराठी",
         "antonyms" : {
            "bla4" : 0.2,
            "bla3" : 0.1
         },
         "features" : {
            "NUM" : "S",
            "GEN" : "M"
         },
         "synonyms" : {
            "bla2" : 0.2,
            "bla1" : 0.1
         },
         "tags" : "tag1 tag2"
      },
      {
         "antonyms" : {
            "bla3" : 0.1,
            "bla4" : 0.2
         },
         "features" : {
            "NUM" : "P",
            "GEN" : "F"
         },
         "term" : "term2",
         "frequency_base": 1.0,
         "frequency_stem": 1.0,
         "vector" : [
            1,
            2,
            3
         ],
         "synonyms" : {
            "bla1" : 0.1,
            "bla2" : 0.2
         },
         "tags" : "tag1 tag2"
      }
   ]
}

```

### DELETE /term

Delete the term.

#### Requirements

* The index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X DELETE http://localhost:${PORT}/${INDEX_NAME}/term -d '{
        "ids": ["मराठी", "term2"]
}'
```

Sample output

```json
{
   "data" : [
      {
         "dtype" : "term",
         "version" : 2,
         "id" : "मराठी",
         "index" : "jenny-en-0",
         "found" : true
      },
      {
         "dtype" : "term",
         "id" : "term2",
         "version" : 2,
         "found" : true,
         "index" : "jenny-en-0"
      }
   ]
}

```

Sample call: delete all

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X DELETE http://localhost:${PORT}/${INDEX_NAME}/term -d'{
        "ids": []
}'
```

Sample call: delete all
```json
{
  "message":"delete",
  "deleted":2
}
```

### PUT /term

Update the entry.

#### Requirements

* The index must exists
* The function requires user credentials with write permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X PUT http://localhost:${PORT}/${INDEX_NAME}/term -d '{
	"terms": [
	    {
            "term": "मराठी",
            "frequency_base": 1.0,
            "frequency_stem": 1.0,
            "vector": [1.2, 2.3, 3.4, 4.5],
            "synonyms":
            {
                "bla1": 0.1,
                "bla2": 0.2
            },
            "antonyms":
            {
                "term2": 0.1,
                "bla4": 0.2
            },
            "tags": "tag1 tag2",
            "features":
            {
                "FEATURE_NEW1": "V",
                "GEN": "M"
            }
	    },
	    {
            "term": "term2",
            "frequency_base": 1.0,
            "frequency_stem": 1.0,
            "vector": [1.6, 2.7, 3.8, 5.9],
            "synonyms":
            {
                "bla1": 0.1,
                "bla2": 0.2
            },
            "antonyms":
            {
                "bla3": 0.1,
                "bla4": 0.2
            },
            "tags": "tag1 tag2",
            "features":
            {
                "FEATURE_NEW1": "N",
                "GEN": "F"
            }
	    }
   ]
}'
```

Sample output

```json
{
   "data" : [
      {
         "version" : 2,
         "id" : "मराठी",
         "index" : "jenny-en-0",
         "created" : false,
         "dtype" : "term"
      },
      {
         "index" : "jenny-en-0",
         "id" : "term2",
         "version" : 2,
         "dtype" : "term",
         "created" : false
      }
   ]
}

```

### GET /term/term

Search for term (using Elasticsearch).

#### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
QUERY=${1:-"term2"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X GET http://localhost:${PORT}/${INDEX_NAME}/term/term -d "{
    \"term\": \"${QUERY}\"
}"
```

Sample output

```json
{
   "hits" : {
      "terms" : [
         {
            "vector" : [
               1.2,
               2.3,
               3.4,
               4.5
            ],
            "antonyms" : {
               "bla4" : 0.2,
               "term2" : 0.1
            },
            "frequency_base": 1.0,
            "frequency_stem": 1.0,
            "features" : {
               "FEATURE_NEW1" : "V",
               "GEN" : "M"
            },
            "score" : 0.6931471824646,
            "tags" : "tag1 tag2",
            "term" : "मराठी",
            "synonyms" : {
               "bla2" : 0.2,
               "bla1" : 0.1
            }
         }
      ]
   },
   "total" : 1,
   "max_score" : 0.6931471824646
}
```

### GET /term/text

Search for all the terms in the text and return the entries.

#### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X GET http://localhost:${PORT}/${INDEX_NAME}/term/text -d 'term2 मराठी'
```

Sample output

```json
{
   "max_score" : 0.6931471824646,
   "hits" : {
      "terms" : [
         {
            "term" : "मराठी",
            "score" : 0.6931471824646,
            "tags" : "tag1 tag2",
            "vector" : [
               1.2,
               2.3,
               3.4,
               4.5
            ],
            "features" : {
               "GEN" : "M",
               "FEATURE_NEW1" : "V"
            },
            "antonyms" : {
               "bla4" : 0.2,
               "term2" : 0.1
            },
            "synonyms" : {
               "bla2" : 0.2,
               "bla1" : 0.1
            },
            "frequency_base": 1.0,
            "frequency_stem": 1.0
         },
         {
            "term" : "term2",
            "tags" : "tag1 tag2",
            "score" : 0.6931471824646,
            "features" : {
               "FEATURE_NEW1" : "N",
               "GEN" : "F"
            },
            "vector" : [
               1.6,
               2.7,
               3.8,
               5.9
            ],
            "antonyms" : {
               "bla3" : 0.1,
               "bla4" : 0.2
            },
            "frequency_base": 1.0,
            "frequency_stem": 1.0,
            "synonyms" : {
               "bla1" : 0.1,
               "bla2" : 0.2
            }
         }
      ]
   },
   "total" : 2
}
```

## Tokenizers

### GET /tokenizers

Show a list of supported methods for tokenization and stemming

#### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X GET "http://localhost:${PORT}/${INDEX_NAME}/tokenizers"
```

Sample output

```json
{
   "shingles2" : "2 words shingles",
   "shingles3" : "3 words shingles",
   "shingles2_10" : "from 2 to 10 shingles",
   "base_stem" : "lowercase + stemming",
   "base" : "lowercase",
   "stop" : "lowercase + stopwords elimination",
   "shingles4" : "4 words shingles",
   "stop_stem" : "lowercase + stopwords elimination + stemming"
}
```

### POST /tokenizers

get a list of token using the selected analyzer

#### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call

```bash
ANALYZER=${1:-"stop"}
QUERY=${2:-"good morning, may I ask you a question?"}
PORT=${3:-8888}
INDEX_NAME=${4:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/tokenizers" -d "
{
        \"text\": \"${QUERY}\",
        \"tokenizer\": \"${ANALYZER}\"
}"
```

Sample output

```json
{
   "tokens" : [
      {
         "start_offset" : 0,
         "end_offset" : 4,
         "token_type" : "word",
         "token" : "good",
         "position" : 0
      },
      {
         "token" : "morning",
         "position" : 1,
         "token_type" : "word",
         "end_offset" : 12,
         "start_offset" : 5
      },
      {
         "start_offset" : 14,
         "end_offset" : 17,
         "token_type" : "word",
         "token" : "may",
         "position" : 2
      },
      {
         "token_type" : "word",
         "token" : "i",
         "position" : 3,
         "start_offset" : 18,
         "end_offset" : 19
      },
      {
         "end_offset" : 23,
         "start_offset" : 20,
         "position" : 4,
         "token" : "ask",
         "token_type" : "word"
      },
      {
         "end_offset" : 27,
         "start_offset" : 24,
         "position" : 5,
         "token" : "you",
         "token_type" : "word"
      },
      {
         "end_offset" : 38,
         "start_offset" : 30,
         "token" : "question",
         "position" : 7,
         "token_type" : "word"
      }
   ]
}
```

## AnalyzersPlayground

### POST /analyzers_playground

used to test analyzers on the fly

#### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

#### Return codes 

##### 200

Sample call keyword

```bash
ANALYZER=${1:-"keyword(\\\"test\\\")"}
QUERY=${2:-"this is a test"}
DATA=${3:-"{\"item_list\": [], \"extracted_variables\":{}}"}
PORT=${4:-8888}
INDEX_NAME=${5:-index_getjenny_english_0}
curl -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/analyzers_playground" -d "
{
        \"analyzer\": \"${ANALYZER}\",
        \"query\": \"${QUERY}\",
        \"data\": ${DATA}
}"
```

Sample output keyword

```json
{
   "build_message" : "success",
   "build" : true,
   "value" : 0.25
}
```

Sample states analyzers

```bash
curl -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H 'Content-Type: application/json' -X POST http://localhost:${PORT}/${INDEX_NAME}/analyzers_playground -d '
{
        "analyzer": "hasTravState(\"one\")",
        "query": "query",
        "data": {"item_list": ["one", "two"], "extracted_variables":{}}
}
'
```

Sample output states analyzers

```json
{
   "build_message" : "success",
   "build" : true,
   "value" : 1
}
```

Sample of pattern extraction through analyzers
 
```bash
curl -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H 'Content-Type: application/json' -X POST http://localhost:${PORT}/${INDEX_NAME}/analyzers_playground -d '
{
        "analyzer": "band(keyword(\"on\"), matchPatternRegex(\"[day,month,year](?:(0[1-9]|[12][0-9]|3[01])(?:[- \\\/\\.])(0[1-9]|1[012])(?:[- \\\/\\.])((?:19|20)\\d\\d))\"))",
        "query": "on 31-11-1900"
}'
```

Sample output

```json
{
   "build_message" : "success",
   "data" : {
      "item_list" : [],
      "extracted_variables" : {
         "month.0" : "11",
         "year.0" : "1900",
         "day.0" : "31"
      }
   },
   "value" : 1,
   "build" : true
}
```

## SpellCheck

### POST /spellcheck/terms

terms spellchecker based on knowledgebase text 

#### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index
* the knowledge base must contain data

#### Return codes

##### 200

Sample call

```bash
QUERY=${1:-"this is a tes for splellchecker"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_getjenny_english_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST http://localhost:${PORT}/${INDEX_NAME}/spellcheck/terms -d "{
  \"text\": \"${QUERY}\",
  \"prefix_length\": 3,
  \"min_doc_freq\": 1
}"
```

```json
{
   "tokens" : [
      {
         "offset" : 0,
         "options" : [
            {
               "freq" : 1284,
               "score" : 0.800000011920929,
               "text" : "hello"
            },
            {
               "text" : "hella",
               "score" : 0.800000011920929,
               "freq" : 2
            },
            {
               "freq" : 2,
               "score" : 0.800000011920929,
               "text" : "helle"
            },
            {
               "text" : "help",
               "score" : 0.75,
               "freq" : 35395
            },
            {
               "score" : 0.75,
               "freq" : 5,
               "text" : "hell"
            }
         ],
         "length" : 5,
         "text" : "hellp"
      },
      {
         "length" : 4,
         "options" : [],
         "offset" : 7,
         "text" : "this"
      },
      {
         "length" : 2,
         "options" : [],
         "offset" : 12,
         "text" : "is"
      },
      {
         "length" : 1,
         "offset" : 15,
         "options" : [],
         "text" : "a"
      },
      {
         "length" : 4,
         "offset" : 17,
         "options" : [
            {
               "text" : "test",
               "score" : 0.75,
               "freq" : 191
            },
            {
               "freq" : 10,
               "score" : 0.5,
               "text" : "tessa"
            },
            {
               "text" : "tesco",
               "score" : 0.5,
               "freq" : 9
            },
            {
               "text" : "tesia",
               "score" : 0.5,
               "freq" : 2
            },
            {
               "freq" : 2,
               "score" : 0.5,
               "text" : "tester"
            }
         ],
         "text" : "tesr"
      }
   ]
}
```
