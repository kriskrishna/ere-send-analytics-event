# spring-data-flow-analytics-event
This repository is a data mart for analytics-event ingested from ERE/RVO3/RWS using Spring Cloud Dataflow

https://dzone.com/articles/data-mart-with-spring-cloud-data-flow

docker-compose up

### Import apps:
Register all built in Spring Cloud Stream apps via Maven (alternatively may use Docker images)

    app import --uri http://bit.ly/stream-applications-kafka-maven
    
Register custom event fact processor

    app register --name car-fact-processor --type processor --uri 

### Example streams:

#### SCDF shell
Create the event fact stream:

    stream create event-fact-stream --definition "http --port=10101 --spring.cloud.stream.bindings.output.contentType='application/json' | event-fact-processor  --spring.cloud.stream.bindings.input.contentType='application/x-java-object;type=com.github.wkennedy.dto.Car' | jdbc --spring.datasource.url=jdbc:mysql://127.0.0.1:3306/galaxy_schema?user=user --spring.datasource.password=pass --spring.datasource.driver-class-name=org.mariadb.jdbc.Driver --jdbc.table-name=car_fact --jdbc.columns=engine,make" --deploy

Create the engine dim stream:

    stream create engine-dim-stream --definition "http --port=10102 --spring.cloud.stream.bindings.output.contentType='application/json' | jdbc --spring.datasource.url=jdbc:mysql://127.0.0.1:3306/galaxy_schema?user=user --spring.datasource.password=pass --spring.datasource.driver-class-name=org.mariadb.jdbc.Driver --jdbc.table-name=engine_dim --jdbc.columns=code,displacement,fuel" --deploy

#### SCDF dashboard
Create the car fact stream:

    http --port=10101 --spring.cloud.stream.bindings.output.contentType='application/json' | car-fact-processor  --spring.cloud.stream.bindings.input.contentType='application/x-java-object;type=com.github.wkennedy.dto.Car' | jdbc --spring.datasource.url=jdbc:mysql://127.0.0.1:3306/galaxy_schema?user=user --spring.datasource.password=pass --spring.datasource.driver-class-name=org.mariadb.jdbc.Driver --jdbc.table-name=car_fact --jdbc.columns=engine,make


#### cURL examples to post data
Create car fact

    curl -X POST -H "Content-Type: application/json" -H "Cache-Control: no-cache" -d '{
    	"engine":"SR20",
    	"make":"Nissan"
    }' "http://localhost:10101"

Create new engine dim

    curl -X POST -H "Content-Type: application/json" -H "Cache-Control: no-cache" -d '{
    	"code":"ISB 6.7",
    	"displacement":"6.7 litres",
    	"fuel":"diesel"
    }' "http://localhost:10102"