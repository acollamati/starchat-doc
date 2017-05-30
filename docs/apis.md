# APIs

## `POST /get_next_response` 

Tell StarChat about the user actions (wrote something, clicked a button etc) and receives instruction 
about the next state.

Data to post:

```json
{
    "conversation_id": "1234",
    "user_input": "(Optional)",
    "text" : "the text typed by the user (Optional)",
    "img": "(e.g.) image attached by the user (Optional)",
l
    "return_value": "the value either in success_value or in failure_value (Optional)",
    "data": "all the variables, e.g. for the STRING TEMPLATEs (Optional)"
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
    }
}'
```

returns:

```json
{
    "action": "input_form",
    "action_input": {
        "email": "email"
    },
    "bubble": "We can reset your password by sending you a message to your registered e-mail address. Please tell me your address so I may send you the new password generation link.",
    "conversation_id": "1234",
    "data": {},
    "failure_value": "\"dont_understand\"",
    "max_state_count": 0,
    "analyzer": "",
    "state": "forgot_password",
    "state_data": {
        "verification": "did you mean you forgot the password?"
    },
    "success_value": "\"send_password_generation_link\""
}
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
{
    "action": "send_password_generation_link",
    "action_input": {
        "email": "john@example.com",
        "template": "somebody requested to reset your password, if you requested the password reset follow the link: %link%"
    },
    "bubble": "Thank you. An e-mail will be sent to this address: a@b.com with your account details and the necessary steps for you to reset your password.",
    "conversation_id": "1234",
    "data": {
        "email": "john@example.com"
    },
    "failure_value": "call_operator",
    "max_state_count": 0,
    "analyzer": "",
    "state": "send_password_generation_link",
    "state_data": {},
    "success_value": "\"any_further\""
}

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
  "total": 1,
  "max_score": 0,
  "hits": [
    {
      "score": 0,
      "document": {
        "analyzer": "((forgot).*(password))",
        "queries": [
          "cannot access account",
          "problem access account"
        ],
        "state": "further_details_access_question",
        "state_data": {
          "verification": "did you mean you can't access to your account?"
        },
        "success_value": "eval(show_buttons)",
        "failure_value": "\"dont_understand\"",
        "bubble": "Hello and welcome to our customer service chat. Please note that while I am not a human operator, I will do my very best to assist You today. How may I help you?",
        "action_input": {
          "Specify your problem": "specify_problem",
          "I want to call an operator": "call_operator",
          "None of the above": "start",
          "Forgot Password": "forgot_password",
          "Account locked": "account_locked"
        },
        "max_state_count": 0,
        "action": "show_buttons"
      }
    }
  ]
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
  "analyzer_map": {
    "further_details_access_question": "((forgot).*(password))"
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
    "hits": [
        {
            "document": {
                "answer": "you are welcome!",
                "conversation": "832",
                "doctype": "normal",
                "id": "0",
                "index_in_conversation": 11,
                "question": "thank you",
                "state": "",
                "status": 0,
                "topics": "",
                "verified": false
            },
            "score": 0.0
        }
    ],
    "max_score": 0.0,
    "total": 1
}
```

## `POST /knowledgebase`

Insert a new document

### Return codes 

#### 201

```bash
curl -v -H "Content-Type: application/json" -X POST http://localhost:8888/starchat-en/knowledgebase -d '{
    "answer": "you are welcome!",
    "conversation": "832",
    "doctype": "normal",
    "id": "0",
    "index_in_conversation": 11,
    "question": "thank you",
    "state": "",
    "status": 0,
    "topics": "",
    "verified": true
}'
```

Sample response

```json
{
    "hits": [
        {
            "document": {
                "answer": "you are welcome!",
                "conversation": "832",
                "doctype": "normal",
                "id": "0",
                "index_in_conversation": 11,
                "question": "thank you",
                "state": "",
                "status": 0,
                "topics": "",
                "verified": true
            },
            "score": 0.0
        }
    ],
    "max_score": 0.0,
    "total": 1
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
    "dtype": "question",
    "found": false,
    "id": "0",
    "index": "jenny-en-0",
    "version": 5
}
```

## `PUT /knowledgebase`

Update an existing document

Output JSON

### Return codes 

#### 200

Sample call

```bash
curl -v -H "Content-Type: application/json" -X PUT http://localhost:8888/starchat-en/knowledgebase/                                                   e9d7c04d0c539415620884f8c885fef93e9fd0b49bbea23a7f2d08426e4d185119068365a0c1c4a506c5c43079e1e8da4ef7558a7f74756a8d850cb2d14e5297 -d '{
    "answer": "you are welcome!",
    "conversation": "832",
    "doctype": "normal",
    "index_in_conversation": 11,
    "question": "thank yoy",
    "state": "",
    "status": 0,
    "topics": "",
    "verified": false
}'
```

Sample response

```json
{
    "created": false,
    "dtype": "question",
    "id": "e9d7c04d0c539415620884f8c885fef93e9fd0b49bbea23a7f2d08426e4d185119068365a0c1c4a506c5c43079e1e8da4ef7558a7f74756a8d850cb2d14e5297",
    "index": "jenny-en-0",
    "version": 3
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
    "hits": [
        {
            "document": {
                "answer": "you are welcome",
                "conversation": "4346",
                "doctype": "normal",
                "id": "10",
                "index_in_conversation": 6,
                "question": "thank you",
                "state": "",
                "status": 0,
                "topics": "",
                "verified": true
            },
            "score": 3.5618982315063477
        }
    ],
    "max_score": 3.5618982315063477,
    "total": 1
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
	}
	"
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
