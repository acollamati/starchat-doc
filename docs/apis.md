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
### Return codes

####200

Similar Json, see examples below

##### Example 1

User input is "I forgot my password":

```bash
curl  -H "Content-Type: application/json" -X POST http://localhost:8888/get_next_response -d '{   
    "conversation_id": "1234",
    "user_input": { "text": "I forgot my password" },
    "values": {
        "return_value": "",
        "data": {}
    },
    "threshold": 0.0,
    "max_results": 4
}'
```

returns:

```json
[
   {
      "analyzer" : "and(or(keyword(\"reset\"),keyword(\"forgot\")),keyword(\"password\"))",
      "state" : "forgot_password",
      "score" : 0.25,
      "action" : "input_form",
      "action_input" : {
         "email" : "email"
      },
      "traversed_states" : [
         "forgot_password"
      ],
      "success_value" : "send_password_generation_link",
      "data" : {},
      "bubble" : "We can reset your password by sending you a message to your registered e-mail address. Please type your email address:",
      "state_data" : {
         "verification" : "did you mean you forgot the password?"
      },
      "max_state_count" : 0,
      "failure_value" : "dont_understand",
      "conversation_id" : "1234"
   }
]
```

##### Example 2

User inserts their email after having been in `forgot_password`. 
The client sends:

```bash
curl  -H "Content-Type: application/json" -X POST http://localhost:8888/get_next_response -d '
{
    "conversation_id": "1234",
    "user_input": { "text": "" },
    "values": {
        "return_value": "send_password_generation_link",
        "data": { "email": "john@example.com" }
    }
}'
```
and gets:

```json
[
   {
      "traversed_states" : [],
      "failure_value" : "call_operator",
      "success_value" : "any_further",
      "action" : "send_password_generation_link",
      "state" : "send_password_generation_link",
      "max_state_count" : 0,
      "state_data" : {},
      "conversation_id" : "1234",
      "data" : {
         "email" : "john@example.com"
      },
      "score" : 1,
      "analyzer" : "",
      "action_input" : {
         "email" : "john@example.com",
         "subject" : "New password",
         "template" : "Hi,\nSomeone requested a new password for your account. You can set a new password here: %link%\nIf you did not request this, just ignore this message."
      },
      "bubble" : "Thank you. An e-mail will be sent to this address: john@example.com with your account details and the necessary steps for you to reset your password."
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

### Return codes 

#### 200

Sample call

```bash
# retrieve one or more entries with given ids; ids can be specified multiple times
curl -v -H "Content-Type: application/json" "http://localhost:8888/decisiontable?ids=further_details_access_question"
```

Sample output

```json
{
   "hits" : [
      {
         "score" : 0,
         "document" : {
            "execution_order" : 1,
            "bubble" : "Hello and welcome to our customer service chat. Please note that while I am not a human operator, I will do my very best to assist You today. How may I help you?",
            "state" : "further_details_access_question",
            "max_state_count" : 0,
            "queries" : [
               "cannot access account",
               "problem access account"
            ],
            "state_data" : {
               "verification" : "did you mean you can't access to your account?"
            },
            "action_input" : {
               "None of the above" : "start",
               "Account locked" : "account_locked",
               "Forgot Password" : "forgot_password",
               "Specify your problem" : "specify_problem",
               "I want to call an operator" : "call_operator"
            },
            "analyzer" : "or(and(or(keyword(\"problem.*\"),keyword(\"issue.*\"),keyword(\"trouble.*\")),keyword(\"account\")),search(\"further_details_access_question\"))",
            "success_value" : "eval(show_buttons)",
            "action" : "show_buttons",
            "failure_value" : "dont_understand"
         }
      }
   ],
   "total" : 1,
   "max_score" : 0
}
```

## `PUT /decisiontable`
 
Output JSON

### Return codes

#### 201

Sample call

```bash
# update the "further_details_access_question" entry in the DT
curl -v -H "Content-Type: application/json" -X PUT http://localhost:8888/decisiontable/further_details_access_question -d '{
  "queries": ["cannot access account", "problem access account", "unable to access to my account"]
}'
```

Sample output
```json
{
    "created": false,
    "dtype": "state",
    "id": "further_details_access_question",
    "index": "jenny-en-0",
    "version": 2
}
```

## `POST /decisiontable`

Insert a new document.

Output JSON

### Return codes

#### 201

Sample call

```bash
curl -v -H "Content-Type: application/json" -X POST http://localhost:8888/decisiontable -d '{
  "state": "further_details_access_question",
  "execution_order": 1,
  "max_state_count": 0,
  "analyzer": "",
  "queries": ["cannot access account", "problem access account"],
  "bubble": "What seems to be the problem exactly?",
  "action": "show_buttons",
  "action_input": {"Forgot Password": "forgot_password", "Account locked": "account_locked", "Payment problem": "payment_problem", "Specify your problem": "specify_problem", "I want to call an operator": "call_operator", "None of the above": "start"},
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
    "index": "jenny-en-0",
    "version": 1
}
```

## `DELETE /decisiontable`

Delete a document by ID

Output JSON

### Return codes 

#### 200

Sample call
```bash
curl -v -H "Content-Type: application/json" -X DELETE http://localhost:8888/decisiontable/further_details_access_question
```

Sample output

```json
{
    "dtype": "state",
    "found": true,
    "id": "further_details_access_question",
    "index": "jenny-en-0",
    "version": 3
}
```

## `POST /decisiontable_search`

Update a document

Output JSON

### Return codes 

#### 200

Sample call
```bash
curl -v -H "Content-Type: application/json" -X POST http://localhost:8888/decisiontable_search -d '{
  "queries": "cannot access my account",
  "min_score": 0.1,
  "boost_exact_match_factor": 2.0
}'
```

Sample response 

```json
{
   "max_score" : 0.930855453014374,
   "hits" : [
      {
         "document" : {
            "action_input" : {
               "I want to call an operator" : "call_operator",
               "Forgot Password" : "forgot_password",
               "None of the above" : "start",
               "Account locked" : "account_locked",
               "Specify your problem" : "specify_problem"
            },
            "bubble" : "Hello and welcome to our customer service chat. Please note that while I am not a human operator, I will do my very best to assist You today. How may I help you?",
            "success_value" : "eval(show_buttons)",
            "action" : "show_buttons",
            "queries" : [
               "cannot access account",
               "problem access account"
            ],
            "execution_order" : 1,
            "max_state_count" : 0,
            "failure_value" : "dont_understand",
            "state_data" : {
               "verification" : "did you mean you can't access to your account?"
            },
            "analyzer" : "or(and(or(keyword(\"problem.*\"),keyword(\"issue.*\"),keyword(\"trouble.*\")),keyword(\"account\")),search(\"further_details_access_question\"))",
            "state" : "further_details_access_question"
         },
         "score" : 0.930855453014374
      }
   ],
   "total" : 1
}
```

## `GET /decisiontable_analyzer` 

(WORK IN PROGRESS, PARTIALLY IMPLEMENTED)

Get and return the map of analyzer for each state

Output JSON

### Return codes 

#### 200

Sample call
```bash
curl -v -H "Content-Type: application/json" -X GET "http://localhost:8888/decisiontable_analyzer"
```

Sample response

```json
{
   "analyzer_map" : {
      "account_locked" : {
         "analyzer" : "booleanand(keyword(\"locked\"), keyword(\"account\"), )",
         "execution_order" : 1,
         "build" : true
      },
      "call_operator" : {
         "analyzer" : "and(or(keyword(\"call\"),keyword(\"talk\"),keyword(\"speak\")),keyword(\"operator\"))",
         "execution_order" : 1,
         "build" : true
      },
      "forgot_password" : {
         "execution_order" : 1,
         "build" : true,
         "analyzer" : "and(or(keyword(\"reset\"),keyword(\"forgot\")),keyword(\"password\"))"
      },
      "terrible_feedback" : {
         "build" : true,
         "execution_order" : 1,
         "analyzer" : "booleanor(keyword(\"idiot\"), keyword(\"fuck.*\"), keyword(\"screw\"), keyword(\"damn.*\"), keyword(\"asshole\"))"
      },
      "test_state" : {
         "analyzer" : "booleanAnd(booleanNot(booleanOr(keyword(\"dont\"),keyword(\"don't\"))), keyword(\"test\"), booleanOr(keyword(\"send\"), keyword(\"get\")))",
         "execution_order" : 1,
         "build" : true
      },
      "further_details_access_question" : {
         "execution_order" : 1,
         "build" : true,
         "analyzer" : "or(and(or(keyword(\"problem.*\"),keyword(\"issue.*\"),keyword(\"trouble.*\")),keyword(\"account\")),search(\"further_details_access_question\"))"
      }
   }
}
```

## `POST /decisiontable_analyzer`

Load/reload the map of analyzer from ES

Output JSON

### Return codes 

#### 200

Sample call
```bash
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/decisiontable_analyzer"
```

Sample response

```json
{"num_of_entries":1}
```

## `GET /knowledgebase`

Return a document by ID

Output JSON

### Return codes 

#### 200

Sample call
```bash
# retrieve one or more entries with given ids; ids can be specified multiple times
curl -v -H "Content-Type: application/json" "http://localhost:8888/knowledgebase?ids=0"
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
            "verified" : true,
            "answer" : "you are welcome!",
            "topics" : "t1 t2",
            "doctype" : "normal",
            "index_in_conversation" : 1,
            "question" : "thank you",
            "state" : ""
         }
      }
   ]
}
```

## `POST /knowledgebase`

Insert a new document

### Return codes 

#### 201

```bash
curl -v -H "Content-Type: application/json" -X POST http://localhost:8888/knowledgebase -d '{
	"id": "0",
	"conversation": "id:1000",
	"index_in_conversation": 1,
	"question": "thank you",
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
{   "dtype": "question",
    "version": 1,
    "id": "1",
    "index": "jenny-en-0",
    "created":true
}
```

## `DELETE /knowledgebase`

Delete a document by ID

Output JSON

### Return codes 

#### 200

Sample call

curl -v -H "Content-Type: application/json" -X DELETE http://localhost:8888/knowledgebase/0

Sample output

```bash
{
   "id" : "0",
   "version" : 5,
   "index" : "jenny-en-0",
   "dtype" : "question",
   "found" : true
}
```

## `PUT /knowledgebase`

Update an existing document

Output JSON

### Return codes 

#### 200

Sample call

```bash
{
   "max_score" : 0,
   "total" : 1,
   "hits" : [
      {
         "score" : 0,
         "document" : {
            "conversation" : "id:1001",
            "verified" : true,
            "id" : "0",
            "doctype" : "normal",
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
            "index_in_conversation" : 1,
            "state" : "",
            "topics" : "t1 t2",
            "answer" : "you are welcome!",
            "status" : 0
         }
      }
   ]
}'
```

Sample response

```json
{
   "index" : "jenny-en-0",
   "dtype" : "question",
   "id" : "0",
   "version" : 3,
   "created" : false
}
```

## `POST /knowledgebase_search`

Output JSON

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X POST http://localhost:8888/knowledgebase_search -d '{
  "question": "thank you",
  "verified": true,
  "doctype": "normal"
}'
```

Sample output

```json
{
   "max_score" : 1.15013706684113,
   "total" : 2,
   "hits" : [
      {
         "document" : {
            "id" : "1",
            "doctype" : "normal",
            "question_scored_terms" : [
               [
                  "validation",
                  0.0343148699683119
               ],
               [
                  "imac",
                  1.12982760046835
               ],
               [
                  "aware",
                  3.15048958129597
               ],
               [
                  "ios",
                  6.14545226791214
               ],
               [
                  "activation",
                  4.92133804309887
               ]
            ],
            "answer" : "fine thanks",
            "conversation" : "id:1000",
            "state" : "",
            "question" : "how are you?",
            "status" : 0,
            "index_in_conversation" : 1,
            "topics" : "t1 t2",
            "verified" : true
         },
         "score" : 1.15013706684113
      }
   ]
}
```

## `POST /language_guesser`

Output JSON

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/language_guesser" -d "
{
	\"input_text\": \"good morning, may I ask you a question?\"
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

Output JSON

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X GET "http://localhost:8888/language_guesser/en"
```

Sample output

```json
{"message":"updated index: jenny-en-0 dt_type_ack(true) kb_type_ack(true) kb_type_ack(true)"}

```

## `POST /index_management/create`

Output JSON

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/index_management"
```

Sample output

```json
{"message":"create index: jenny-en-0 create_index_ack(true)"}
```

## `POST /index_management/refresh`

Output JSON

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/index_management/refresh"
```

Sample output

```json
{
   "failed_shards_n" : 0,
   "total_shards_n" : 10,
   "failed_shards" : [],
   "successful_shards_n" : 5
}
```

## `GET /index_management`

Output JSON

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X GET "http://localhost:8888/index_management"
```

Sample output

```json
{"message":"settings index: jenny-en-0 dt_type_check(state:true) kb_type_check(question:true) term_type_name(term:true)"}
```

## `PUT /index_management`

Output JSON

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X PUT "http://localhost:8888/index_management"
```

Sample output

```json
{"message":"updated index: jenny-en-0 dt_type_ack(true) kb_type_ack(true) kb_type_ack(true)"}
```

## `DELETE /index_management`

Output JSON

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X DELETE "http://localhost:8888/language_guesser/en"
```

Sample output

```json
{"message":"removed index: jenny-en-0 index_ack(true)"}
```

## `POST /term/index`

Index the term as indicated in the JSON. 

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X POST http://localhost:8888/term/index -d '{
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

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X POST http://localhost:8888/term/get -d '{
     "ids": ["मराठी", "term2"]
}'
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

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X DELETE http://localhost:8888/term -d '{
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

## `PUT /term`

Update the entry.

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X PUT http://localhost:8888/term -d '{
     "terms": [
         {
            "term": "मराठी",
            "frequency_base": 1.0,
            "frequency_stem": 1.0,
            "vector": [1.0, 2.0, 3.0, 4.0],
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
            "vector": [1.0, 2.0, 3.0, 5.0],
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

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X GET http://localhost:8888/term/term -d '{
    "term": "मराठी"
}'
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

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X GET http://localhost:8888/term/text -d 'term2 मराठी'
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

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X GET "http://localhost:8888/tokenizers"
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

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/tokenizers" -d "
{
	\"text\": \"good morning, may I ask you a question?\",
		  \"tokenizer\": \"stop\"
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

### Return codes 

#### 200

Sample call

```bash
ANALYZER="keyword(\\\"test\\\")"
QUERY="this is a test"
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/analyzers_playground" -d "
{
        \"analyzer\": \"${ANALYZER}\",
	        \"query\": \"${QUERY}\"
		}"
```

Sample output

```json
{
   "value" : 0.25,
   "build_message" : "success",
   "build" : true
}
```

Sample of pattern extraction through analyzers
 
```json
curl -v -H 'Content-Type: application/json' -X POST http://localhost:8888/analyzers_playground -d' 
{
        "analyzer": "band(keyword(\"on\"), matchPatternRegex(\"[day,month,year](?:(0[1-9]|[12][0-9]|3[01])(?:[- \\\/\\.])(0[1-9]|1[012])(?:[- \\\/\\.])((?:19|20)\\d\\d))\"))",
        "query": "on 31-11-1900"
}'```

Sample output

```json
{
   "build_message" : "success",
   "variables" : {
      "month.0" : "11",
      "day.0" : "31",
      "year.0" : "1900"
   },
   "build" : true,
   "value" : 1
}
```


## `POST /spellcheck/terms`

terms spellchecker based on knowledgebase text 

### Return codes

#### 200

Sample call

```bash
QUERY=${1:-"this is a tes for splellchecker"}
curl -v -H "Content-Type: application/json" -X POST http://localhost:8888/spellcheck/terms -d "{
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
