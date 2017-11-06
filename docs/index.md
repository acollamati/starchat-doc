# Welcome!
This is the official repository for [StarChat](https://github.com/GetJenny/starchat), a scalable conversational engine for B2B applications.

# How to contribute

To contribute to StarChat, please send us a [pull request](https://help.github.com/articles/using-pull-requests/#fork--pull)  from your fork of this repository.

Our concise [contribution guideline](https://github.com/GetJenny/starchat/blob/master/CONTRIBUTING.md) contains the bare
minumum requirements of the code contributions.

Before contributing (or opening issues), you might want send us an email at starchat@getjenny.com.

# Quick Start

## Requirements

The easiest way is to install StarChat using two docker images. You only need:

* [sbt](http://www.scala-sbt.org)
* [docker](https://docs.docker.com/engine/installation/)
* [docker compose](https://docs.docker.com/compose/install/)

In this way, you will put all the indices in the Elasticsearch (version 5.4) image, and StarChat itself in the Java (8) image.

_If you do not use docker_ you therefore need on your machine:

1. [Scala 12.2](http://scala-lang.org)
2. [Elasticsearch 5.4](http://elastic.co)

## Setup with Docker (recommended)

### 1. Launch docker-compose

Generate a packet distribution:
```bash
sbt dist
```

Enter the directory docker-starchat:
```bash
cd  docker-starchat
```
You will get a message like `Your package is ready in ...../target/universal/starchat-4ee.... .zip`.  Extract the packet into the docker-starchat folder:
```bash
unzip ../target/universal/starchat-4eee.....zip
ln -s starchat-4ee..../  starchat
```

The zip packet contains:

* a set of scripts to test the endpoints and as a complement for the documentation: ```starchat/scripts/api_test/```
* a set of command line programs ```starchat/bin``` to run starchat and other tools.
* delete-decision-table: application to delete items from the decision table
* index-corpus-on-knowledge-base: application to index a corpus on knowledge base as hidden (to improve the language model)
* index-decision-table: application to index data on the decision table from a csv
* index-knowledge-base: application to index data into the knowledge base
* index-terms: application to index terms vectors
* starchat: start starchat

Review the configuration files `starchat/config/application.conf` and configure the language if needed (by default you have `index_language = "english"`)

(If you are re-installing StarChat, and want to start from scratch see [start from scratch](#docker-start-from-scratch).)

Start both startchat and elasticsearch:
```bash
docker-compose up -d
```

(Problems like `elastisearch exited with code 78`? have a look at [troubleshooting](#troubleshooting)!)

### 2. Create Elasticsearch indices

Run from a terminal:

```bash
# create the indices in Elasticsearch
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/index_management/create"
```

### 3. Load the configuration file

Now you have to load the configuration file for the actual chat, aka [decision table](#services). We have provided an example csv in English, therefore:

```bash
cd $STARCHAT_DIR  # or cd .. 
sbt "run-main com.getjenny.command.IndexDecisionTable --inputfile doc/decision_table_starchat_doc.csv --skiplines 1"
```

and then (you need to index the analyzer):

```bash
./docker-starchat/starchat/bin/index-decision-table --inputfile doc/decision_table_starchat_doc.csv 
```

This command deletes all the states it finds on the first column in the inputfile:

```bash
sbt "run-main com.getjenny.command.DeleteDecisionTable --inputfile doc/decision_table_starchat_doc.csv"
```

NB This means that if you create a state in the CSV file, index it, then delete it in the CSV and run `DeleteDecisionTable`, it won't be deleted!

### 4. Load external corpus (optional)

To have a good words' statistics, and consequent improved matching, you might want to index a corpus which is hidden from results. For instance, you can index various sentences as hidden using the [POST /knowledgebase](#post-knowledgebase) endpoint with `doctype: "hidden"`.

### 5. Index the FAQs (optional)

TODO: You might want to activate the [knowledge base](#configuration-of-the-answer-recommender-knowledge-base) for simple Question and Anwer. 

## Install without Docker

Note: we do not support this installation.
* Clone the repository and enter the starchat directory.
* Initialize the Elasticsearch instance (see above for Docker)
* Run the service: `sbt compile run`

The service binds on the port 8888 by default.

## Test the installation

Is the service working?

`curl -X GET localhost:8888 | python -mjson.tool`

Get the `test_state`

```bash
curl  -H "Content-Type: application/json" -X POST http://localhost:8888/get_next_response -d '{
 "conversation_id": "1234",
  "user_input": { "text": "install starchat" },
  "values": {
      "return_value": "",
      "data": {}
       }
  }'
```

You should get:

```json
[
   {
      "conversation_id" : "1234",
      "max_state_count" : 0,
      "analyzer" : "band(bor(keyword(\"setup\"), keyword(\"install.*\")), bnot(bor(keyword(\"standalone\"), keyword(\"docker\"))))",
      "success_value" : "",
      "data" : {},
      "bubble" : "Just choose one of the two:\n<ul>\n<li>docker install (recommended)</li>\n<li>standalone install</li>\n</ul>",
      "state" : "install",
      "action" : "",
      "failure_value" : "",
      "state_data" : {},
      "traversed_states" : [
         "install"
      ],
      "action_input" : {},
      "score" : 1
   }
]
```

If you look at the `"analyzer"` field, you'll see that this state is triggered when
the user types the *test* and either *get* or *send*. Try with `"text": "Please dont send me the test state"`
 and StarChat will send an empty message.

## Configuration of the chatbot (Decision Table)

With StarChat's Decision Table you can easily implement workflow-based chatbots. After the installation (see above)
you only have to configure a conversation flow and eventually a front-end client.

## NLP processing

NLP processing is of course the core of any chatbot. As you have noted in the  [CSV provided in the doc/ directory](https://github.com/GetJenny/starchat/blob/master/doc/sample_state_machine_specification.csv) there are two fields defining when StarChat should trigger a state -`analyzer` and `queries`.

### Queries

If the `analyzer` field is empty, StarChat will query Elasticsearch for the state containing the most similar sentence in the field `queries`. We have carefully configured Elasticsearch in order to provide good answers (e.g. boosting results where the same words appear etc), and results are... acceptable. But you are encouraged to use the `analyzer` field, documented below.

### Analyzer

Through the `analyzer`s, you can easily leverage on various NLP algorithms included in StarChat, together with NLP capabilities of Elasticsearch. You can also combine the result of those algorithms. The best way is to look at the simple example included in the  [CSV provided in the doc/ directory](https://github.com/GetJenny/starchat/blob/master/doc/sample_state_machine_specification.csv) for the state `forgot_password`:

`and(or(keyword("reset"),keyword("forgot")),keyword("password"))`

The *expression* `and` and `or` are called the *operators*, while `keyword` is an *atom* 

#### Expressions: Atoms

Presently, the `keyword("reset")` in the example provides a very simple score: occurrence of the word *reset* in the user's query divided by the total number of words. If evaluated agains the sentence "Want to reset my password", `keyword("reset")` will currently return 0.2.  _NB_ This is just a temporary score used while our NLP library [manaus](https://github.com/GetJenny/manaus) is not integrated into StarChat.

These are currently the expression you can use to evaluate the goodness of a query (see [DefaultFactoryAtomic](https://github.com/GetJenny/starchat/blob/master/src/main/scala/com/getjenny/analyzer/atoms/DefaultFactoryAtomic.scala) and [StarchatFactoryAtomic](https://github.com/GetJenny/starchat/blob/master/src/main/scala/com/getjenny/starchat/analyzer/atoms/StarchatFactoryAtomic.scala):

* _keyword("word")_: as explained above, normalized
* _regex_: evaluate a regular expression, not normalized
*  _search(state_name)_: takes a state name as argument, queries elastic search and returns the score of the most similar query in the field  `queries` of the argument's state. In other words, it does what it would do without any analyzer, only with a normalized score -e.g. `search("lost_password_state")` 
* _synonym("word")_: gives a normalized cosine distance between the argument and the closest word in the user's sentence. We use word2vec, to have an idea of two words distance you can use this [word2vec demo](http://bionlp-www.utu.fi/wv_demo/) by [Turku University](http://bionlp.utu.fi/)
* _similar("a whole sentence")_:  gives a normalized cosine distance between the argument and the closest word in the user's sentence (word2vec)
* _similarState(state_name)_:  same as above, but for the sentences in the field "queries" of the state in the argument.
* _similarEucEmd("a whole sentence")_: gives a non-normalized euclidean distance (calculated using the earth movers distance algorithm) between the argument and the closest sentence in the user's sentence (word2vec)
* _similarEucEmdState(state_name)_: same as above, but for the sentences in the field "queries" of the state in the argument.
* _similarCosEmd("a whole sentence")_: gives a normalized cosine distance (calculated using the earth movers distance algorithm) between the argument and the closest sentence in the user's sentence (word2vec)
* _similarCosEmdState(state_name)_: same as above, but for the sentences in the field "queries" of the state in the argument.
* _matchPatternRegex(regex)_: A generic pattern extraction analyzer, it extract named patterns matching a given regex e.g. the following will match tree numbers separated by semicolumn: [first,second,third](?:([0-9]+:[0-9]:[0-9]+) if the regex matches it will create the entries into the state variables dictionary e.g.: 10:11:12 will result in Map("first.0" -> "10", "second.0" -> "11", "third.0" -> "12") the number at the end of the name is an index incremented for multiple occurrences of the pattern in the query
* _matchDateDDMMYYYY(prefix)_: parse a date in DDMMYYYY format is built using the matchPatternRegex with the following regex: "(?:(?:[^0-9]+|\A)(0[1-9]|[12][0-9]|3[01])(?:[- \/\.])(0[1-9]|1[012])(?:[- \/\.])((?:19|20)\d\d)(?:[^0-9]+|$))"
* _existsVariable(variable_name)_: check whether a variable exists or not 
* _hasTravState_(state_name): check if a state_name is present into the history of traversed states
* _lastTravStateIs_(state_name): check if the last traversed state is state_name
* _prevTravStateIs(state_name)_: check if the last but one traversed state is state_name
* _distance("keyword1", ..., "keyword N")_: calculate the cosine distance between the query and the list of keywords, this analyzer can be called using `cosDistanceKeywords`

#### Expressions: Operators

Operators evaluate the output of one or more expression and return a value. Currently, the following operators are implemented (the the [source code](https://github.com/GetJenny/starchat/blob/master/src/main/scala/com/getjenny/analyzer/operators/DefaultFactoryOperator.scala)):

* _boolean or_:   calles matches of all the exprassions it contains and returns true or false. It can be called using `bor`
* _boolean and_:  as above, it's called with `band`
* _boolean not_:  ditto, `bnot`
* _conjunction_:  if the evaluation of the expressions it contains is normalized, and they can be seen as probabilities of them being true, this is the probability that all the expressions are all true (`P(A)*P(B)`)
* _disjunction_:  as above, the probability that at least one is true (`1-(1-P(A))*(1-P(B))`)
* _max_: takes the max score of returned by the expression arguments

#### Technical corner: `expressions`

[Expressions](https://github.com/GetJenny/starchat/blob/master/src/main/scala/com/getjenny/analyzer/expressions/Expression.scala), like [keywords](https://github.com/GetJenny/starchat/blob/master/src/main/scala/com/getjenny/analyzer/atoms/KeywordAtomic.scala#L18) in the example, are called [atoms](https://github.com/GetJenny/starchat/blob/master/src/main/scala/com/getjenny/analyzer/atoms/AbstractAtomic.scala), and have the following methods/members:

1. `def evaluate(query: String): Double`: produce a score. It might be normalized to 1 or not (set `val isEvaluateNormalized: Boolean` accordingly)
2. ` val match_threshold` This is the threshold above which the expression is considered true when `matches` is called. NB The default value is 0.0, which is normally not ideal.
3. `def matches(query: String): Boolean`: calles evaluate and check agains the threshold...
4. `val rx`: the name of the atom, as it should be used in the `analyzer` field.

## Configuration of the answer recommender (Knowledge Base)

Through the [/knowledgebase](https://getjenny.github.io/starchat-doc/apis/#get-knowledgebase) endpoint
you can add, edit and remove pairs of question and answers used by StarChat to recommend possible answers when a question arrives.

Documents containing Q&A must be structured like that:

```json
{
    "id": "0",  // id of the pair
    "conversation": "id:1000",   // id of the conversation. This can be useful to external services
    "index_in_conversation": 1,  // when the pair appears inside the conversation, as above
    "question": "thank you",  // The question to be matched
    "answer": "you are welcome!",  // The answer to be recommended 
    "question_scored_terms": [  // A list of keyword and score. You can use your own keyword extractor or our Manaus (see later)
        [
            "thank", 
            1.9
        ]
    ],
    "verified": true,  // A variable used in some call centers
    "topics": "t1 t2",  // Eventual topics to be associated
    "dclass": "", // Optional field as a searchable class for answer
    "doctype": "normal",
    "state": "",
    "status": 0
}
```

See [POST /knowledgebase](https://getjenny.github.io/starchat-doc/apis/#post-knowledgebase) for an example with `curl`. Other calls (`GET, DELETE, PUT`) are used to get the state, delete it or update it. 

### Test the knowledge base

Just run the example in [POST /knowledgebase_search](https://getjenny.github.io/starchat-doc/apis/#knowledgebase_search).
  
## Manaus

In the Q&A pair above you saw the field
`question_scored_terms`. Having good keywords improves enormously the
quality of the answer. You can of course put them by hand, or run a
software which extracts the keywords from the question. If you prefer
the latter, but don't have any, we provide
[Manaus](https://github.com/GetJenny/manaus).

Manaus is still under development, but it's already included in the
Docker's installation of StarChat. When you launch `docker-compose up
-d`, you also launch a container with Manaus which analyzes all the
Q&A in the Knowledge Base, produces keywords and updates the field
question_scored_terms for all documents. The process is repeated evert
4 hour.

### Manaus configuration

Have a look at the file `docker-starchat/docker-compose.yml`. For
Manaus to have good performance, you need to provide decent language
statistics. Update the file `/manaus/statistics_data/english/word_frequency.tsv` with a
word-frequency file with the following format:

```
1       you     1222421
2       I       1052546
3       to      823661
....
```

We have frequency file for more than 50 languages, but consider that
the choice of good "prior distribution" of word frequency is crucial
for any NLP task.

# Technology

StarChat was design with the following goals in mind:

1. easy deployment
2. horizontally scalability without any service interruption.
3. modularity
4. statelessness

## How does StarChat work?

### Workflow

![alt tag](https://www.websequencediagrams.com/cgi-bin/cdraw?lz=dGl0bGUgc2ltcGxpZmllZCBSZXN0QVBJQ2FsbGluZ01lY2hhbmlzbSBpbiAqQ2hhdAoKVXNlciAtPiBTdGFyY2hhdFJlc291cmNlOiByZXN0IGFwaSBjYWxsIChpbiBqc29uKQoAGhAAKBZqc29uIGRlc2VyaWFsaXphdGlvbiBpbnRvIGVudGl0eQAqHVNlcnZpY2U6AHYFaW5nIGZ1bmMAPQUoaW4AOgcAgH8KACcHACwVADAJZXhlY3V0aW9uABscAIFzCgBoCXJlc3VsdCAob3V0AGgRAIFkHgCBbQ5vZgCBcgcAgX8GamVzAIEACwCCPwxVc2VyAIJyC3Jlc3BvbnNlAHwGAIJ8BgoK&s=napkin)

### Components

StarChat uses Elasticsearch as NoSQL database and, as said above, NLP preprocessor, for
indexing, sentence cleansing, and tokenization.

### Services

StarChat consists of two different services: the "KnowledBase" and the "DecisionTable"

#### KnowledgeBase

For quick setup based on real Q&A logs. It stores question and answers pairs. Given a text as input
 it proposes the pair with the closest match on the question field.
  At the moment the KnowledBase supports only Analyzers implemented on Elasticsearch.

#### DecisionTable

The conversational engine itself. For the usage, see below.

## Configuration of the DecisionTable

You configure the DecisionTable through CSV file. Please have a look at the [CSV provided in the doc/ directory](https://github.com/GetJenny/starchat/blob/master/doc/sample_state_machine_specification.csv).

Fields in the configuration file are of three types:

* **(R)**: Return value: the field is returned by the API
* **(T)**: Triggers to the state: when should we enter this state? 
* **(I)**: Internal: a field not exposed to the API

And the fields are:

* **state**: a unique name of the state (e.g. `forgot_password`)
* **execution_order**: specify an order of evaluation for analyzers the lower is the number earlier is the evaluation of the state
* **max_state_count**: defines how many times StarChat can repropose the state during a conversation.
* **analyzer (T,I)**: specify an analyzer expression which triggers the state
* **query (T,I)**: list of sentences whose meaning identify the state
* **bubble (R)**: content, if any, to be shown to the user. It may contain variables like %email% or %link%.
* **action (R)**: a function to be called on the client side. StarChat developer must provide types of input and output (like an abstract method), and the GUI developer is responsible for the actual implementation (e.g. `show_button`)
* **action_input (R)**: input passed to **action**'s function (e.g., for `show_buttons` can be a list of pairs `("text to be shown on button", state_to_go_when_clicked)` 
* **state_data (R)**: a dictionary of strings with arbitrary data to pass along
* **success_value (R)**: output to return in case of success
* **failure_value (R)**: output to return in case of failure

## Client functions

In StarChat configuration, the developer can specify which function the front-end should
execute when a certain state is triggered, together with input parameters.
**Any function implemented on the front-end can be called.**

### Example show button

- Action: ```show_buttons```
- Action input: ```{"buttons": [("Forgot Password", "forgot_password"), ("Account locked", "account_locked")]}```
- The frontend will call function: ```show_buttons(buttons={"Forgot Password": "forgot_password","Account locked": "account_locked")```

Example "buttons": the front-end implements the function show_buttons and uses "action input" to call it. It will show two buttons, where the first returns forgot_password and the second account_locked.

### Example send email

- Action: ```send_password_link```
- Action input: ```{ "template": "Reset your password here: example.com","email": "%email%","subject": "New password" }```
- The frontend will call function: ```send_password_link(template="Reset your password here: example.com.",email= "john@foo.com", subject="New password")```

Example "send email": the front-end implements the function send_password_link and uses "action input" to call it.
The variable %email% is automatically substituted by the variable email if available in the JSON passed to the
StarchatResource.


### functions for the sample csv

For the CSV in the example above, the client will have to implement the following set of functions:

* show_buttons: tell the client to render a multiple choice button
* input: a key/value pair with the key indicating the text to be shown in the button, and the value indicating the state to follow e.g.: {"Forgot Password": "forgot_password", "Account locked": "account_locked", "Specify your problem": "specify_problem", "I want to call an operator": "call_operator", "None of the above": "start"}
* output: the choice related to the button clicked by the user e.g.: "account_locked"
* input_form: render an input form or collect the input following a specific format
* input: a dictionary with the list of fields and the type of fields, at least "email" must be supported: e.g.: { "email": "email" } where the key is the name and the value is the type
* output: a dictionary with the input values e.g.: { "email": "foo@example.com" }
* send_password_generation_link: send an email with instructions to regenerate the password
* input: a valid email address e.g.: "foo@example.com"
* output: a dictionary with the response fields e.g.: { "user_id": "123", "current_state": "forgot_password", "status": "true" }

Ref: [sample_state_machine_specification.csv](https://github.com/GetJenny/starchat/blob/master/doc/sample_state_machine_specification.csv).

## Mechanics

* The client implements the functions which appear in the action field of the spreadsheet. 
We will provide interfaces.
* The client call the rest API "decisiontable" endpoint communicating a state if any, 
the user input data and other state variables
* The client receive a response with guidance on what to return to the user and what 
are the possible next steps
* The client render the message to the user and eventually collect the input, then 
call again the system to get instructions on what to do next
* When the "decisiontable" functions does not return any result the user can call the "knowledgebase" endpoint which contains all the conversations. 

## Scalability

StarChat consists of two different services: StarChat itself and an Elasticsearch cluster. 
     
### Scaling StarChat instances
     
StarChat can scale horizontally by simple replication. Because StarChat is stateless, instances looking 
at the same Elasticsearch index will behave identically. New instances can then be added together
with a load balancing service.

In the diagram below, a load balancer forward requests coming from the front-end to StarChat instances 
1, 2 or 3. These instances, as said, behave identically because they all refer to `Index 0` in the 
Elasticsearch cluster.

![Image](img/scalability_diagram_starchat.png?raw=true)

### Scaling Elasticsearch

Similarly, Elasticsearch can easily scale horizontally adding new nodes to the cluster, as explained
 in [Elasticsearch Documentation](https://www.elastic.co/guide/en/elasticsearch/guide/master/_scale_horizontally.html).
 
## Security

StarChat is a backend service and *should never* be exposed to the internet,
it should be placed behind a firewall.
One of the most effective and flexible method to add an access control layer is to use 
[Kong](https://getkong.org) in front of StarChat as a gateway, in this way
StarChat can be shield by unwanted accesses.

In addition StarChat support TLS connections, the configuration file allow to
choose if the service should expose an https connection or an http connection
or both.
In order to use the https connection the user must do the following things:

* obtain a pkcs12 server certificate from a certification authority or [create a self signed certificate](http://typesafehub.github.io/ssl-config/CertificateGeneration.html)
* save the certificate inside the folder ```config/tls/certs/``` e.g. ```config/tls/certs/server.p12```
* set the password for the certificate inside the configuration file
* enable the https connection setting to true the https.enable property of the configuration file
* optionally disable the http connection setting to false the http.enable property of the configuration file

Follows the block of the configuration file which is to be modified as described above in
 order to use https:

```yaml
https {
  host = "0.0.0.0"
  host = ${?HOST}
  port = 8443
  port = ${?PORT}
  certificate = "server.p12"
  password = "uma7KnKwvh"
  enable = false
}

http {
  host = "0.0.0.0"
  host = ${?HOST}
  port = 8888
  port = ${?PORT}
  enable = true
}
```

StarChat come with a default self-signed certificate for testing,
using it for production or sensitive environment is highly discouraged
as well as useless from a security point of view.

# Indexing terms on term table

The following program index term vectors on the vector table:

```bash
sbt "run-main com.getjenny.command.IndexTerms --inputfile terms.txt --vecsize 300"
```

The format for each row of an input file with 5 dimension vectors is:
```hello 1.0 2.0 3.0 4.0 0.0```

You can use your ad-hoc trained vector model (as we do) otherwise you can use the google word2vec models
trained on google news. You can find a copy of the [elasticsearch index with a pre-loaded google news terms](https://raw.githubusercontent.com/GetJenny/jenny-public-data/master/elasticsearch.GoogleNewsVectors.tar.bz2).

# Test

## Unit tests

A set of unit test is available using docker-compose to set up a backend, the command to run tests is:

```bash
sbt dockerComposeUp ; sbt test ; sbt dockerComposeStop
```

## test scripts with sample API calls

* A set of test script is present inside scripts/api_test


# Troubleshooting

## Docker: start from scratch

You might want to start from scratch, and delete all docker images. 

If you do so (`docker images` and then `docker rmi -f <java/elasticsearch ids>`) remember that all data for the 
Elasticsearch docker are local, and mounted only when the container is up. Therefore you need to:

```bash
cd docker-starchat
rm -rf elasticsearch/data/nodes/
```

## docker-compose: Analyzers are not loaded

StarChat is started immediately after elasticsearch and it is possible that elasticsearch is
 not ready to respond to REST calls from StarChat (i.e. an index not found error could be
 raised in this case).

Sample error on the logs:
```
2017-06-15 10:37:22,993 >10:37:22.992UTC ERROR c.g.s.s.AnalyzerService(akka://starchat-service) com.getjenny.starchat.services.AnalyzerService(akka://starchat-service) - can't load analyzers: [jenny-en-0]
 IndexNotFoundException[no such index]
```

In order to avoid this problem you can call the services one by one:
```bash
docker-compose up elasticsearch # here wait elasticsearch is up and running
docker-compose up starchat # starchat will retrieve the Analyzers from elasticsearch
```
In alternative is possible to call the command to load/refresh the Analyzers after the docker-compose command: 
```bash
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/decisiontable_analyzer"
```

## Docker: Size of virtual memory

If elasticsearch complain about the size of the virtual memory:

```
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
elastisearch exited with code 78
```

run:

```bash
sysctl -w vm.max_map_count=262144
```
