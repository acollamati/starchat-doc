# APIs

## `POST /get_next_response` 

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

### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

### Return codes

####200

Similar Json, see examples below

##### Example 1

User input is "how to install starchat":

```bash
QUERY=${1:-"how to install starchat"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_0}
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

##### Example 2

User input is: "how can I contribute to starchat?"

```bash
QUERY=${1:-"how can I contribute to starchat?"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_0}
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

#### 204

No response was found

#### 500 (error)

Internal server error

#### 400 (error)

Bad request: 

* meaning: the input data structure is not valid
* output data: no data returned

#### 422 (error)

* meaning: bad request data, the input data is formally valid but there is some issue with data interpretation
* output data: the output data structure is a json dictionary with two fields: code and message. The following code are supported:
* code: 100
* message: "error evaluating the template strings, bad values"

#### 404 (error)

* meaning: not found
* output data: no data returned

## `GET /decisiontable` 

Get a document by ID

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
# retrieve one or more entries with given ids; ids can be specified multiple times
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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

## `PUT /decisiontable`
 
Output JSON

### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

### Return codes

#### 201

Sample call

```bash
# update the "further_details_access_question" entry in the DT
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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
   "index" : "index_0.state",
   "version" : 2,
   "id" : "further_details_access_question"
}
```

## `POST /decisiontable`

Insert a new document.

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

### Return codes

#### 201

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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
    "index": "index_0.state",
    "version": 1
}
```

## `DELETE /decisiontable`

Delete a document by ID

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

### Return codes 

#### 200

Sample call
```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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
  "index":"index_0.state"
}
```

Sample call: delete all

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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


## `POST /decisiontable_upload_csv`

upload load a csv file on decisiontable

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

### Return codes

#### 201

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-'index_0'}
FILENAME=${3:-"`readlink -e ../../doc/decision_table_starchat_doc.csv`"}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
 -X POST --form "csv=@${FILENAME}" http://localhost:8888/${INDEX_NAME}/decisiontable_upload_csv
```

Sample output

```json
{
   "data" : [
      {
         "index" : "index_0.state",
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
         "index" : "index_0.state"
      },
      {
         "id" : "contribute",
         "created" : true,
         "index" : "index_0.state",
         "dtype" : "state",
         "version" : 1
      },
      {
         "version" : 1,
         "dtype" : "state",
         "index" : "index_0.state",
         "created" : true,
         "id" : "quickstart"
      },
      {
         "dtype" : "state",
         "version" : 1,
         "index" : "index_0.state",
         "created" : true,
         "id" : "docker_install"
      },
      {
         "dtype" : "state",
         "version" : 1,
         "index" : "index_0.state",
         "id" : "create_es_indices",
         "created" : true
      },
      {
         "dtype" : "state",
         "version" : 1,
         "id" : "delete_es_indexes",
         "created" : true,
         "index" : "index_0.state"
      },
      {
         "version" : 1,
         "dtype" : "state",
         "index" : "index_0.state",
         "created" : true,
         "id" : "create_es_indexes"
      },
      {
         "created" : true,
         "id" : "index_data",
         "index" : "index_0.state",
         "dtype" : "state",
         "version" : 1
      },
      {
         "version" : 1,
         "dtype" : "state",
         "id" : "index_analyzer",
         "created" : true,
         "index" : "index_0.state"
      },
      {
         "index" : "index_0.state",
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
         "index" : "index_0.state"
      },
      {
         "index" : "index_0.state",
         "id" : "standalone_install",
         "created" : true,
         "dtype" : "state",
         "version" : 1
      },
      {
         "dtype" : "state",
         "version" : 1,
         "index" : "index_0.state",
         "id" : "code_78",
         "created" : true
      },
      {
         "index" : "index_0.state",
         "id" : "licence",
         "created" : true,
         "dtype" : "state",
         "version" : 1
      },
      {
         "index" : "index_0.state",
         "created" : true,
         "id" : "terrible_feedback",
         "version" : 1,
         "dtype" : "state"
      },
      {
         "id" : "call_operator",
         "created" : true,
         "index" : "index_0.state",
         "dtype" : "state",
         "version" : 1
      },
      {
         "version" : 1,
         "dtype" : "state",
         "id" : "any_further",
         "created" : true,
         "index" : "index_0.state"
      },
      {
         "index" : "index_0.state",
         "id" : "dont_understand",
         "created" : true,
         "version" : 1,
         "dtype" : "state"
      }
   ]
}
```

## `POST /decisiontable_search`

Update a document

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call
```bash
Q="${1:-'cannot access my account'}"
S="${2:-0.0}"
B="${3:-100.0}"
PORT=${4:-8888}
INDEX_NAME=${5:-index_0}
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

## `GET /decisiontable_analyzer` 

Get and return the map of analyzer for each state

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call
```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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

## `POST /decisiontable_analyzer`

Load/reload the map of analyzer from ES

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/decisiontable_analyzer"
```

Sample response

```json
{
   "num_of_entries" : 17
}
```

## `GET /knowledgebase`

Return a document by ID

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call
```bash
ID=${1:-0}
PORT=${2:-8888}
INDEX_NAME=${3:-index_0}
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

## `POST /knowledgebase`

Insert a new document

### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

### Return codes 

#### 201

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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
   "index" : "index_0.question",
   "id" : "0"
}
```

## `DELETE /knowledgebase`

Delete a document by ID

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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
    "index" : "index_0.question"
}
```

Sample call: delete all

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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

## `PUT /knowledgebase`

Update an existing document

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with write permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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
   "index" : "index_0.question",
   "id" : "0",
   "version" : 4,
   "created" : false
}
```

## `POST /knowledgebase_search`

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
QUERY=${1:-"how are you?"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_0}
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

## `POST /language_guesser`

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
QUERY=${1:-"good morning, may I ask you a question?"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_0}
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

## `GET /language_guesser`

Check if a language is recognizable by the guesser

Output JSON

### Requirements

* Index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
LANG=${1:-en} 
PORT=${2:-8888}
INDEX_NAME=${3:-index_0}
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

## `POST /index_management/create`

Output JSON

### Requirements

* Index name must start with the "index_" prefix followed by a sequence of alphanumeric characters plus underscore
* The function requires "admin" credentials

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
LANGUAGE=${3:-english}
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X POST "http://localhost:${PORT}/${INDEX_NAME}/${LANGUAGE}/index_management/create"
```

Sample output

```json
{
   "message" : "IndexCreation: state(index_0.state, true) question(index_0.question, true) term(index_0.term, true)"
}
```

## `POST /index_management/refresh`

Output JSON

### Requirements

* The index must exists
* The function requires user credentials with write permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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
         "index_name" : "index_0.state",
         "total_shards_n" : 10
      },
      {
         "failed_shards" : [],
         "total_shards_n" : 10,
         "index_name" : "index_0.question",
         "failed_shards_n" : 0,
         "successful_shards_n" : 5
      },
      {
         "failed_shards" : [],
         "successful_shards_n" : 5,
         "index_name" : "index_0.term",
         "failed_shards_n" : 0,
         "total_shards_n" : 10
      }
   ]
}
```

## `GET /index_management`

Output JSON

### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
curl -v -H "Authorization: Basic `echo -n 'test_user:p4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X GET "http://localhost:${PORT}/${INDEX_NAME}/index_management"
```

Sample output

```json
{
   "message" : "IndexCheck: state(index_0.state, true) question(index_0.question, true) term(index_0.term, true)"
}
```

## `PUT /index_management`

Output JSON

### Requirements

* Index must exists
* The function requires "admin" credentials

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
LANGUAGE=${3:-english}
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
 -H "Content-Type: application/json" -X PUT "http://localhost:${PORT}/${INDEX_NAME}/${LANGUAGE}/index_management"
```

Sample output

```json
{
   "message" : "IndexCheck: state(index_0.state, true) question(index_0.question, true) term(index_0.term, true)"
}
```

## `DELETE /index_management`

Output JSON

### Requirements

* The index must exists
* The function requires admin credentials

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
curl -v -H "Authorization: Basic `echo -n 'admin:adminp4ssw0rd' | base64`" \
  -H "Content-Type: application/json" -X DELETE "http://localhost:${PORT}/${INDEX_NAME}/index_management"
```

Sample output

```json
{
   "message" : "IndexDeletion: state(index_0.state, true) question(index_0.question, true) term(index_0.term, true)"
}
```

## `POST /term/index`

Index the term as indicated in the JSON. 

### Requirements

* The index must exists
* The function requires user credentials with write permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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

## `POST /term/get`

Get one or more terms entry.

### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
QUERY=${1:-"\"term\""}
PORT=${2:-8888}
INDEX_NAME=${3:-index_0}
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

## `DELETE /term`

Delete the term.

### Requirements

* The index must exists
* The function requires user credentials with write permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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
INDEX_NAME=${2:-index_0}
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

## `PUT /term`

Update the entry.

### Requirements

* The index must exists
* The function requires user credentials with write permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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

## `GET /term/term`

Search for term (using Elasticsearch).

### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
QUERY=${1:-"term2"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_0}
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

## `GET /term/text`

Search for all the terms in the text and return the entries.

### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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


## `GET /tokenizers`

Show a list of supported methods for tokenization and stemming

### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
PORT=${1:-8888}
INDEX_NAME=${2:-index_0}
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

## `POST /tokenizers`

get a list of token using the selected analyzer

### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call

```bash
ANALYZER=${1:-"stop"}
QUERY=${2:-"good morning, may I ask you a question?"}
PORT=${3:-8888}
INDEX_NAME=${4:-index_0}
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

## `POST /analyzers_playground`

used to test analyzers on the fly

### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index

### Return codes 

#### 200

Sample call keyword

```bash
ANALYZER=${1:-"keyword(\\\"test\\\")"}
QUERY=${2:-"this is a test"}
DATA=${3:-"{\"item_list\": [], \"extracted_variables\":{}}"}
PORT=${4:-8888}
INDEX_NAME=${5:-index_0}
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

## `POST /spellcheck/terms`

terms spellchecker based on knowledgebase text 

### Requirements

* The index must exists
* The function requires user credentials with read permissions on the index
* the knowledge base must contain data

### Return codes

#### 200

Sample call

```bash
QUERY=${1:-"this is a tes for splellchecker"}
PORT=${2:-8888}
INDEX_NAME=${3:-index_0}
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
