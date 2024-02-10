# Data Enrichment Service Achitecture
This document aims to demonstrate the proposed design for the Data Enrichment Service and its components. In order to facilitate the reading, let us name the service as DES.

## Solution design
The proposed long-term/ideal architecture diagram for the service described in the Take Home Assesment requirements is defined below:

![alt text](architecture.drawio.png "Architecture")

Now let's dive in more details in each component of the proposed solution, in order to understand what role each one plays on it and how they interact with each other.

### Ingestion layer
In order to start the DES ingestion layer, let us break it in two ways for inputting data into the service: (1) Push-oriented and (2) Pull-oriented methods. This definition takes into account that each company transaction service might have it's own level of maturity/complexity, therefore we need to provide a flexible integration as possible for others take advantage of our services provided.

#### Push-oriented
This approach is triggered by the transactional service that wants to enrich its data. For that, it was defined a interfacing API component on which this service can talk with the DES starting the enrichment request; it needs to send a JSON containing both the payload (contact list and a list of fields to be enriched) and a **enrichment resquest ID (ERID)**. Here is an example:
```
{
    "erid": "3c3f4b80-f728-4d7c-8567-1bc01df5f0a1",
    "contact_list": [
        {
            "first_name": "John",
            "last_name": "Doe",
            "company_name": "sayprimer.com"
        },
        {
            "first_name": "Foo",
            "last_name": "Bar",
            "company_name": "sayprimer.com"
        }
    ],
    "fields_to_be_enriched": [
        "professional_email",
        "phone_number" 
    ]
}
```
The `erid` is an important part of the flow because it defines a single enrichment transaction and as long as the requests made to the DES interfacing API have the same `erid`, they will be grouped at the output, in a single data object that can be referred to later.

It's important to highlight that it's an asynchronous request, therefore no enriched response will be returned at this point. Each request containing thousands of data objects will be forwarded to a Streaming Broker/Messaging System, so they can be processed and written in a near real-time fashion. Please check for more details in [Processing Layer](#processing-layer) and [Processing Layer](#serving-layer).

P.S: Even though the diagram doesn't show the authentication on the API side, we can leverage some functionalities on AWS API Gateway for that purpose keeping our API secure and compliance.

#### Pull-oriented
In this method, the service can't, for any reason, integrate with our Asynchronous Ingest API but still, it needs to enrich data. The proposed architecture contemplates this situation as well, where instead of receiving the data enrichment requests, the DES service will start the process. 

To accomplish this requirement, there are two key components of the architecture:
- A workflow orchestrator, AWS MWAA (Managed Workflow with Apache Airflow), accountable for managing the flows, job schedules, and dependencies for the data-pulling process
- An ETL framework, built on top of AWS Lambda or Spark batch jobs, that can handle multiple data sources using generic interfaces and specialized mechanism source-oriented implemented using object inheritance (circles on the architecture).

Either the service has to query the transactional data directly in their database, or APIs or has to read from an intermediate storage layer (s3 buckets, FTP serves, and others), by using the pull-oriented method, the service will also be able to bring the data to the service and process it in a scalable fashion. After querying the data on the source, it will forward it to the Broker/Messaging system that keeps the boundaries between the ingestion and processing layer.

#### Broker/Messaging system
It will be one of the key components of the architecture that will provide us high scalability, as well as a fault-tolerant service which linked to the processing layer will form the high performatic core of the solution.

The broker will act as a data storage engine for real-time data reading, allowing streaming processes to read from it in a performant way, at the same time ensuring the data consistency with an _end-to-end exactly-once_ semantics in case of failure.

### Processing Layer


### Serving Layer