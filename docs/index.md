# Welcome!
This is the official repository for StarChat, a scalable conversational engine for B2B applications.

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

In this way, you will put all the indices in the Elasticsearch (version 5.3) image, and StarChat itself in the Java (8) image.

_If you do not use docker_ you therefore need on your machine:

1. [Scala 12.2](http://scala-lang.org)
2. [Elasticsearch 5.3](http://elastic.co)

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
Extract the packet into the docker-starchat folder:
```bash
unzip ../target/universal/data/WORK/GetJenny/starchat/target/universal/starchat-581e3255e5141185bb66fa8ffce0257f055596c2-SNAPSHOT.zip
ln -s starchat-581e3255e5141185bb66fa8ffce0257f055596c2-SNAPSHOT starchat
```

The zip packet contains:

* a set of scripts to test the endpoints and as a complement for the documentation: ```starchat-adf23af60508d50b0db61172a101d78339043fc4-SNAPSHOT/scripts/api_test/```
* a set of command line programs ```starchat-adf23af60508d50b0db61172a101d78339043fc4-SNAPSHOT/bin``` to run starchat and other tools.
* delete-decision-table: delete items from the decision table
* index-corpus-on-knowledge-base: index a corpus on knowledge base as hidden (to improve the language model)
* index-decision-table: index data on the decision table from a csv
* index-knowledge-base: index data into the knowledge base
* index-terms: index terms vectors
* starchat: start starchat

Review the configuration files `starchat-581e3255e5141185bb66fa8ffce0257f055596c2-SNAPSHOT/config/application.conf` and configure the language if needed (by default you have `index_language = "english"`)

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
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/index_management"
```

### 3. Load the configuration file

Now you have to load the configuration file for the actual chat, aka [decision table](#services). We have provided an example csv in English, therefore:

```bash
sbt "run-main com.getjenny.command.IndexDecisionTable --inputfile doc/sample_state_machine_specification.csv --skiplines 1"
```

Every time you load the configuration file you need to index the analyzer:

```bash
curl -v -H "Content-Type: application/json" -X POST "http://localhost:8888/decisiontable_analyzer"
```

Items on decision table can be removed using the following command:

```bash
sbt "run-main com.getjenny.command.DeleteDecisionTable --inputfile doc/sample_state_machine_specification.csv"
```

### 4. Load external corpus (optional)

To have a good words' statistics, and consequent improved matching, you might want to index a corpus which is hidden from results. For instance, you can index various sentences as hidden using the [POST /knowledgebase](#post-knowledgebase) endpoint with `doctype: "hidden"`.

### 5. Index the FAQs (optional)

You might want to activate the [knowledge base](#services) for simple Question and Anwer.

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
  "user_input": { "text": "Please send me the test state" },
  "values": {
      "return_value": "",
      "data": {}
       }
  }'
```

You should get:

```json
{
    "action": "",
    "action_input": {},
    "analyzer": "and(keyword(\"test\"), or(keyword(\"send\"), keyword(\"get\")))",
    "bubble": "This is the test state",
    "conversation_id": "1234",
    "data": {},
    "failure_value": "",
    "max_state_count": 0,
    "score": 1.0,
    "state": "test_state",
    "state_data": {},
    "success_value": ""
}
```

If you look at the `"analyzer"` field, you'll see that this state is triggered when
the user types the *test* and either *get* or *send*. Try with `"text": "Please dont send me the test state"`
 and StarChat will send an empty message.

## StarChat configuration

With StarChat you can easily implement workflow-based chatbots. After the installation (see above)
you only have to configure a conversation flow and eventually a front-end client.

In practice, StarChat:

* analyze user's query and identifies a test where such user should be sent to
* creation of dynamic content using variables inferred from the conversation (e.g. "Please write your email so that I can send you a message")

### Simple NLP processing

*Work in progress*

* Elasticsearch and the "queries" field
* The analyzer: atomic expressions and operators

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

You configure the DecisionTable through CSV file. Please have a look at the one provided in `doc/`:

|state|max_state_count|analyzer|queries |bubble|action|action_input|state_data|success_value |failure_value|
|-----|---------------|-----|--------|------|------|------------|----------|--------------|-------------|
|start|0              |     |      |"How may I help you?"||||||
|further_details_access_question|0||"[""cannot access account"", ""problem access account""]"||show_buttons|"{""Forgot Password"": ""forgot_password"", ""Account locked"": ""account_locked"", ""None of the above"": ""start""}"||eval(show_buttons)|"""dont_understand"""|
|forgot_password|0||"[""Forgot password""]"|"I will send you a new password generation link, enter your email."|input_form|"{""email"": ""email""}"||"""send_password_generation_link"""|"""dont_understand"""|
|send_password_generation_link|0|||"Sending message to %email% with instructions."|send_password_generation_link|"{ ""template"": "If you requested a password reset, follow this link: %link%"", ""email"": ""%email%"" }"||"""any_further"""|call_operator|


Fields in the configuration file are of three types:

* **(R)**: Return value: the field is returned by the API
* **(T)**: Triggers to the state: when should we enter this state? 
* **(I)**: Internal: a field not exposed to the API

And the fields are:

* **state**: a unique name of the state (e.g. `forgot_password`)
* **max_state_count**: defines how many times StarChat can repropose the state during a conversation.
* **analyzer**: specify an analyzer expression which triggers the state
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

![Image](doc/readme_images/scalability_diagram_starchat.png?raw=true)

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

# Test

* Unit tests are available with `sbt test` command
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
