#	Graph Database

The graph database of COLID is responsible for the data storage of the primary COLID entries. When selecting a suitable database, the primary requirements are that it must comply with the RDF standard and that it can be queried using the query language SPARQL.
It enables the database to store independent data entries that can be linked to each other using individual triples. 
With these connections, the database allows us to traverse over various relationships of the entries.

## Amazon Neptune & Apache Jena Fuseki

While we work with Apache Jena Fuseki in the open source version, Amazon Neptune is used in our productive cloud-first environment. There are a few things to keep in mind when working with the different databases:

*  It is not possible to insert, update or delete data without specifying a graph. A graph must always be specified. Example:
    ``` sparql
    WITH { }
    DELETE { ... }
    INSERT { ... }
    WHERE { ... }
    ```
    
    ``` sparql
    INSERT INTO {
        GRAPH <https://pid.bayer.com/resource/4.0> {
            <...> <..> <...>
        }
    }
    ```
*  Counts in the select statement must be enclosed in parentheses.
    
*  count(distinct ?resource) as ?resources becomes (count(distinct ?resource) as ?resources)
    
*  Aggregations must always be implemented correctly to [W3 Aggregates](https://www.w3.org/TR/sparql11-query/#aggregates)

*  Within a regex, the variable must be explicitly converted to string.
    *  `regex(?s, "foo", "i")` becomes `regex(str(?s), "foo", "i")`
    *  regex: search for `regex\((\?\w+)`, replace with `regex(str($1))`

### Usual Debugging Procedure

- Execute failing operation in COLID Registration Service
- Get query from logs
- Paste query into http://www.sparql.org/query-validator.html
- If the query times out while testing, try replacing filters by explicit references (prefer `?a colid:somePredicate <http://this.is.my.value>` over `?a colid:somePredicate ?b. FILTER(str(?b) = "http://this.is.my.value")`)
- Once query is fixed, test them in the backend.

### Known Issues with some databases

- `DELETE {} INSERT {} WHERE` update queries
    - Neptune must have a `WITH` in front of the queries
    - Places where this is not given
- `INSERT INTO` must have `GRAPH <..>`
    - recommendation: refactor code so that this is added globally
- Dont use a lot of `FILTER` statements, they are slowing down the queries a lot
- NEVER use `SELECT` without `FROM NAMED <...>`
- ALWAYS use `COALESCE` when `CONCAT`ing variables than can be empty
- `GROUP BY` must be used correctly on aggregates (all columns that are not aggregated must be in `GROUP BY` - just like in SQL ;-) )
- `AGGREGATES` must have unique variable names (valid: `SELECT GROUP_CONCAT(?foo, separator = ',') as ?bar`, invalid: `SELECT GROUP_CONCAT(?foo, separator = ',') as ?foo`)

## Loading graph data into the Triple Store

### Apache Jena Fuseki

- Export all graphs so that they are in TTL format.
- Open the directory fuseki-staging/graphs in the setup project
- Copy/move your TTL files to the instance or metadata directory
- If you created a new TTL file to upload into the TripleStore, open the script in fuseki-staging/loader.sh of the setup project and add a line for the new TTL file, e.g.:
```bash
./tdbloader --graph https://pid.bayer.com/my/awesome/graph --loc /fuseki/databases/colid-dataset /staging/graphs/metadata/my_awesome_graph.ttl
```
    - Change the parameter `--graph` in the way the graph should be named and accessible in the database, e.g. https://pid.bayer.com/my/awesome/new/graph
    - Leave the parameter `--loc` as it is. It specifies the path to the database within the docker container of the Apacha Jena Fuseki instance.
    - Change the local path to the newly added TTL file.

### Amazon Neptune

Loading new graphs or updating existing data in Amazon Neptune requires a somewhat complicated procedure.
In the following, the manual steps that have to be performed are described. These can also be automated by an AWS Lambda.

- Export all graphs so that they are in TTL format.
- Log in to the AWS console of the account where the Amazon Neptune is also located.
- Open AWS Service S3
- Upload the TTL files to your bucket, e.g. `neptune-loader-bucket` into the subdirectory `/colid/yyyy-MM-dd/`
    - E.g. s3://neptune-loader-bucket/pid/1970-01-01/my_awesome_new_graph.ttl
- Open Putty with a shell to the Neptune Bastion Hosts
- Run the following command in Shell on bastion host for each TTL file to upload:
```bash
curl -X POST \
    -H 'Content-Type: application/json' \
    http://my-neptune-instance.abcd12345678.eu-central-1.neptune.amazonaws.com:8182/loader -d '
    {
      "source" : "s3://neptune-loader-bucket/colid/[yyyy-MM-dd]/[xyz].ttl",
      "format" : "turtle",
      "iamRoleArn" : "arn:aws:iam::406564331347:role/neptune-loader",
      "region" : "eu-central-1",
      "failOnError" : "FALSE",
      "parallelism" : "MEDIUM",
  "parserConfiguration" : {
    "namedGraphUri" : "[https://pid.bayer.com/xyz]"
     }
    }'
```
- Verify 200 in response.

- Examples
```bash
[user@ip-127-0-0-1 ~]$ curl -X POST \
    -H 'Content-Type: application/json' \
    http://my-neptune-instance.abcd12345678.eu-central-1.neptune.amazonaws.com:8182/loader -d '
    {
      "source" : "s3://neptune-loader-bucket/colid/1970-01-01/my_awesome_new_graph.ttl",
      "format" : "turtle",
      "iamRoleArn" : "arn:aws:iam::406564331347:role/neptune-loader",
      "region" : "eu-central-1",
      "failOnError" : "FALSE",
      "parallelism" : "MEDIUM",
  "parserConfiguration" : {
    "namedGraphUri" : "https://pid.bayer.com/my/awesome/new/graph"
     }
    }'
```

#### Uploading graphs with AWS Lambda

To automate the process of uploading new graphs, we have created an AWS Lambda function. This function is triggered as soon as a TTL file is uploaded to the S3 Bucket. The Lamba function generates a graph name from the file name and executes the curl command against the Amazon Neptune instance.

Graphs need to be uploaded in .ttl format and will be checked against a file name pattern. After the successful pattern match, a Lambda function will first check if the graph exists and sends the command to Amazon Neptune. Double underscores `__` in the file name will be converted to a slash `/`. To keep the overview of the graphs all graphs are versioned with `major.minor.patch` after the semantic versioning.
* Filename: `my__awesome__new__graph__1.3.3.ttl`
* Resulting graphname: `http://pid.bayer.com/my/awesome/new/graph/1.3.3`

---

## AWS Neptune connector with AWS v4 signing via DotNetRDF (Software Design Description)

### Desired state
All COLID environment request, which are sent to AWS Neptune, has to include the AWS signature v4. AWS Neptune is currently accessed via a shared proxy and should be disabled / replaced with IAM authentication.

### Problem
Up to now, communication with sparql endpoints has taken place directly with dotNetRDF. The initial idea of using the 'interceptor' pattern to intervene is not possible. Currently the `SparqlRemoteUpdateEndpoint` is used, which is generically used for all sparql endpoints that can be called with HTTP. This uses `System.Net.WebRequest`, which cannot be intercepted by a middleware or a DelegateHandler, because it's not possbile to inject a custom one into WebRequest.Create.
`HttpWebRequest request = (HttpWebRequest)WebRequest.Create(requestUri.ToString())`;

### Solution idea
Various steps were necessary to convert and integrate AWS Signature v4, which are described below. The implementation of this has also enabled the easy integration and use of other graph databases and services. All dotNetRDF connectors can be used now, like Stardog, Fuseki or other supported ones.

#### 1. AWSNeptuneConnector / AWS Signature v4 & Token aquirement
Analog to the available _Storage Providers_ in dotNetRDF (see https://github.com/dotnetrdf/dotnetrdf/wiki/UserGuide-Storage-Providers), an additional connector for AWS Neptune has been developed. For this purpose, existing classes such as the 'FusekiConector' were used as a reference. A necessary step was the implementation of the following interfaces and classes 'SparqlHttpProtocolConnector, IAsyncUpdateableStorage, IConfigurationSerializable, IUpdateableStorage'.

For AWS Neptune, the connector also required support for AWS Signature v4 for authenticating all requests. A sample implementation that does not require the full AWS SDK can be found here: https://github.com/tsibelman/aws-signer-v4-dot-net/blob/master/Aws4Signer/AWS4RequestSigner.cs. This signing implementation was included into the connector and can be configured separately. A detailed documentation of the signing process can be found wihtin the [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html).

The solution was implemented to avoid direct storage of credentials for a separate AWS user. This also makes management easier, because no new machine user has to be created and maintained in first place. The application runs in a pod within the Kubernetes cluster on AWS and has direct acces to the metadata of the EC2 instance. Thus, authentication can take place immediately without separate storage of access data. In general the authentication with the cluster is handled via [kube2IAM](https://medium.com/bench-engineering/kube2iam-secure-kubernetes-services-on-aws-b3873b8b664a). To enforce this behaviour, there are the following two steps necessary:

- Pull the role from the annotation of the pod (or from a config)
<br>`export AWS_IAM_ROLE=$(curl http://169.254.169.254/latest/meta-data/iam/security-credentials/)`
- Get the Credentials for the Role for authentication (use with C# HttpClient)
<br>`export AWS_CREDENTIALS=$(curl http://169.254.169.254/latest/meta-data/iam/security-credentials/$AWS_IAM_ROLE)`
- Cache Token/Credentials by validity period.
- It may be necessary to fetch the Token with specified expiration time `curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 3600"`. This time has to be considered for caching.


#### 2. COLID Triplestore-Service adjustments
The existing TripleStore repository was extended to include a method for selecting the appropriate connector. The FusekiConnector is used for local execution -- the AWSNeptuneConnector for DEV, QA and production environment. It is also possible to select these separately in the `Startup.cs` file to inject them directory by extension modules.

In addition, all services that implement and use the COLID TripleStore (e.g. registration-service, reporting-service) are changed accordingly, so they implement the required components. 

#### 3. Deactivating AWS Neptune proxy 
After implementation and testing, the proxy endpoint was disabled for the respective AWS Neptune instance. The self configured proxy is running within the Kubernetes cluster and managed with Helm, which also changed.


