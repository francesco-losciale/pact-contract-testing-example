# Contract-testing example on JVM 

In the world of distributed systems and functional teams it is considered best practice to implement some 
kind of contract-testing for API consumers and producers to guarantee smooth integration 
and spot breaking changes in the very early stages of the development. 

Generally some iterations are needed to define the final shape of an API. All the teams involved
need to collaborate and choose between a producer-driven or a consumer-driven approach. The latter considers the 
consumers of an API (such as front-end devs) people that know better what kind of data is needed. 
A consumer-driven approach is commonly considered best practice when building an API since in theory 
it should help to avoid useless reworks and improve the process.
Producers (back-end devs) will adapt their system under the consumer guidelines. 
For this reason a continuous communication among people of different teams is crucial. 

If teams don't have any CI pipeline, they need at some point (before the big release) to verify that 
the contract works fine when integrating clients and servers. 
Tools such as [dredd](https://github.com/apiaryio/dredd) or [prism](https://github.com/stoplightio/prism) 
can help to do this manually.

If CI is the case, the ideal would be to build some automated test as part of the pipeline so that any breaking 
changes would be caught immediately. This is the main intent behind [PACT](https://docs.pact.io/) project.
  
Briefly, this is the usual workflow:

1. The Consumer Team creates some unit tests using the PACT library which will automatically    
    build the PACT file (the contract)
2. The contract is rebuild every time the spec in the unit tests of the Consumer Team repository 
change for example in terms of requests or expected response
3. The Consumer Team publishes the contract to the PACT broker 
4. The Producer Team will then implement the API using the contract published in the broker as a guide
5. The Producer Team will continuously verify against the running instance of their API service that the 
implementation is not breaking the contract 

Once the API contract is defined, versioned and published, no more changes on it are expected. 
A new version would be created in that case. 
The PACT verification in the pipeline then can be useful as regression testing. If accidentally someone 
in the Provider Team breaks the contract, their local build would fail and this will prevent them from 
pushing the code in the pipeline.

#### Pipeline execution

In this project we have the following components: 

- PACT broker (running in docker locally)
- a consumer repository with code to produce the pact json file in the `../pacts` directory (see the unit test)
- a producer repository with code of a rest service which will satisfy the contract 
(see commands below to know how verify works)

The following commands below represent the workflow and could give the idea of what a CI pipeline 
should do to make contract-testig working.

- Run the PACT broker locally (or use a cloud managed one) and check out the webpage at http://localhost/. 

```shell script
cd pact-contract-testing-example;
docker-compose up -d;
``` 

- Change direcotry to `pact-example-consumer` and run the build. The unit test will generate a pact json file.
After that you can publish the result to the broker and to make it available to the provider.
```shell script
./gradlew clean build;
./gradlew pactPublish;
```
Check out the webpage at http://localhost/ again to see that the pact has been published 
by the consumer but not verified by the producer. 

- Change directory to `pact-example-provider`  now to know if the provider is breaking the contract or not. 
Using the system property `pact.verifier.publishResults=true` the outcome will be uploaded to the broker 
(which is the right behaviour for CI pipelines but you don't want this when running the pactverify locally...)

```shell script
./gradlew clean build && ./gradlew pactverify -Ppact.verifier.publishResults=true
```

Check out the webpage at http://localhost/ again to see that everything is green. 

### Additional resources:
- https://github.com/Mikuu/Pact-JVM-Example
- https://github.com/DiUS/pact-jvm/tree/master/provider/pact-jvm-provider-gradle
- https://medium.com/@liran.tal/a-comprehensive-guide-to-contract-testing-apis-in-a-service-oriented-architecture-5695ccf9ac5a
