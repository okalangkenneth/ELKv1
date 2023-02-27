# Logging in an ASP.NET Application using Docker, Elasticsearch, and Kibana.

## Introduction:


Logging is an essential part of any application, as it provides valuable insights into its performance and usage. In this demonstration, we'll walk through the process of setting up a logging solution for an ASP.NET application using Docker, Elasticsearch, and Kibana. This stack provides a scalable, centralized, and cost-effective way to store, analyze, and visualize logs generated by your application.

For example, a company running an e-commerce website could use this logging stack to monitor their website's performance and track any errors that may occur. The ASP.NET application running on a Docker container generates logs that are sent to Elasticsearch using the Serilog library. Elasticsearch indexes the logs, and Kibana provides a web interface for searching, filtering, and visualizing the logs. This allows the company to quickly identify issues, track performance over time, and scale the logging solution as their business grows.

In this demonstration, we'll cover the following steps:

- Setting up Docker and running Elasticsearch and Kibana containers.
- Configuring a .NET 6 web API in visual studio code to log to Elasticsearch using Serilog.
- Configuring Serilog to send logs to Elasticsearch.
- Running the web API project in a Docker container and generating logs.
- Using Kibana to search, filter, and visualize the logs.

Throughout the demonstration, we'll include snapshots of the code and Docker containers running, so you can see exactly how everything works together to create a robust logging solution.

## Requirements
Before getting started, make sure you have the following installed on your machine:

Docker

.NET 6 SDK

Visual Studio code.

## Setting up Docker and running Elasticsearch and Kibana containers.

### Step 1: Install Docker.

First, we need to install Docker. Docker provides a platform for developers and sysadmins to develop, ship, and run applications. You can download Docker from the official website here.

### Step 2: Run Elasticsearch and Kibana containers.

Next, we'll run Elasticsearch and Kibana containers using Docker Compose. Docker Compose is a tool for defining and running multi-container Docker applications.

Create a new directory named ELKv1 in your project directory and create a new file named docker-compose.yml inside it in visual studio code. Then, copy the following code into docker-compose.yml:


````C#
version: '3.1'

services:
  elasticsearch:
    container_name: els
    image: docker.elastic.co/elasticsearch/elasticsearch:7.16.1
    ports:
      - 9200:9200
    volumes:

      - elasticsearch-data:/usr/share/elasticsearch/data
    environment:
      - xpack.monitoring.enabled=true
      - xpack.watcher.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.type=single-node
    networks:
      - elastcinetwork

  kibana:
    container_name: kibana
    image: docker.elastic.co/kibana/kibana:7.16.1
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch
    environment:
      - ELASTICSEARCH_URL=http://localhost:9200
    networks:
      - elastcinetwork

networks:
  elastcinetwork:
    driver: bridge

volumes:
  elasticsearch-data:
  ````
  
This code defines two services: elasticsearch and kibana. The elasticsearch service runs the Elasticsearch container and exposes port 9200 for Elasticsearch's REST API. The kibana service runs the Kibana container and exposes port 5601 for Kibana's web interface. The two services are connected to a network named elastcinetwork.

To run the Elasticsearch and Kibana containers, open a terminal and navigate to the ELKv1 directory. Then, run the following command:

````bash
docker-compose up 
````
This command will download the Elasticsearch and Kibana images and start the containers in the background. You should see output similar to the following:

````javascript
Creating network "elastcinetwork" with the default driver
Creating elasticsearch ... done
Creating kibana         ... done
````

### Step 3: Verify Elasticsearch and Kibana are running

Finally, we'll verify that Elasticsearch and Kibana are running. Open a web browser and navigate to http://localhost:9200/. You should see a JSON response that starts with something like:

````json
{
  "name" : "2af12e338b11",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "so1JpXtORfKG6db3xTx25Q",
  "version" : {
    "number" : "7.16.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "5b38441b16b1ebb16a27c107a4c3865776e20c53",
    "build_date" : "2021-12-11T00:29:38.865893768Z",
    "build_snapshot" : false,
    "lucene_version" : "8.10.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
````

This response confirms that Elasticsearch is running.

To verify that Kibana is running, open a web browser and navigate to http://localhost:5601/. You should see the Kibana home page, which looks something like this:

![image](https://user-images.githubusercontent.com/68539411/221278491-2b7e0d8c-d7d5-434d-9c12-179514aa91aa.png)

You should also be able to see the containers running in docker as the shown below.

![image](https://user-images.githubusercontent.com/68539411/221275300-768b055b-6851-464d-9adb-5f1c026400f9.png)


### Step 4: Stop and remove the containers.

To stop and remove the Elasticsearch and Kibana containers, open a terminal and navigate to the ELKv1 directory. Then, run the following command:

````bash
docker-compose down
````

This command will stop and remove the containers. You should see output similar to the following:


````bash
Stopping kibana         ... done
Stopping elasticsearch ... done
Removing kibana         ... done
Removing elasticsearch ... done
Removing network elastcinetwork
````

That's it for the first demonstration! You've now set up Docker and ran Elasticsearch and Kibana containers. In the next demonstration, we'll create a new .NET 6 web API project and configure it to log to Elasticsearch using Serilog.

## Configuring a .NET 6 web API in visual studio code to log to Elasticsearch using Serilog..

In this demonstration, we'll create a new .NET 6 web API project and configure it to log to Elasticsearch using Serilog. Serilog is a popular logging library for .NET that supports structured logging and can send logs to various sinks, including Elasticsearch.

### Step 1: Create a new .NET 6 web API project

Open a terminal and navigate to the directory where you want to create the project. Then, run the following command to create a new .NET 6 web API project:

````bash
dotnet new webapi --no-https -n DotnetELK
````
This command will create a new .NET 6 web API project with the name DotnetELK.

### Step 2: Add Serilog and the Serilog.Sinks.Elasticsearch package

Next, we need to add the Serilog and Serilog.Sinks.Elasticsearch packages to the project. Run the following commands to do so:

````bash
dotnet add package Serilog
dotnet add package Serilog.Sinks.Elasticsearch
dotnet add Serilog.Enrichers.Environment 
dotnet add Serilog.Exceptions
dotnet add Serilog.Sinks.Debug
````
These commands will add the Serilog and Serilog.Sinks.Elasticsearch packages to the project. We will have to update the appsettings.json file with serilog and with the target output ElasticSearch, Uri http://localhost:9200" like below.

````C#
{
  "Serilog": {
    "Minimumlevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Information",
        "System": "Warning"
      }
    }
  },
  "ELKConfiguration": {
    "Uri": "http://localhost:9200"
  },
  "AllowedHosts": "*"
}
````

## Configure Serilog to log to Elasticsearch

Next, we need to configure Serilog to log to Elasticsearch. 

### Step 1: Modifying Program.cs file.

Open the Program.cs file we firts define the function:

````C#
ConfigureLogs();
````

We will create a helper region so we can easily identify the configuration code.

````C#
#region helper
void ConfigureLogs()

{
   // Get the environment variable for the current environment (e.g. Development, Production)
    var env = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");

   // Build the Configuration object, which is used to read settings from the appsettings.json file
    var configuration = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
            .Build();
    // Configure Serilog by setting various options and sinks (destinations)
    Log.Logger = new LoggerConfiguration()
       .Enrich.FromLogContext()
       .Enrich.WithExceptionDetails() // Adds exceptions details
       .WriteTo.Debug()
       .WriteTo.Console()
       .WriteTo.Elasticsearch(ConfigureELS(configuration, env)) // This line adds the Elasticsearch sink
       .CreateLogger();

}
ElasticsearchSinkOptions ConfigureELS(IConfigurationRoot configuration, string env)
{
    return new ElasticsearchSinkOptions(new Uri(configuration["ELKConfiguration:Uri"]))
    {
        AutoRegisterTemplate = true,
        IndexFormat = $"{Assembly.GetExecutingAssembly().GetName().Name.ToLower()}-{env.ToLower().Replace(".", "-")}-{DateTime.UtcNow:yyyy-MM}"
    };
}
#endregion 
 ````
 The code is defining a method called "ConfigureLogs()" which is used to configure Serilog, a logging library, to write log messages to various destinations such as Elasticsearch.
  
 ### Running the web API project in a Docker container and generating logs.
 
 ### Step 1: Add logging to the API
 
 Next, we'll add logging to the API. Open the Controllers/WeatherForecastController.cs file and add the following code to the Get method:
 
 ````C# 
  _logger.LogInformation("WeatherForecastController Get - This message tests the logs by Kenneth Okalang", DateTime.UtcNow);
  ````
  This code logs an informational message when the Get method is called.
 
 ## Running the web API project in a Docker container and generating logs.
 
 ### Step 2: Test the API and view logs in Kibana
 
 Now, we can test the API and view the logs in Kibana. Open a terminal and navigate to the DotnetELK directory. Then, run the following command to start the API:
 
````bash
dotnet run
````
This command will start the API. Now, open a web browser and navigate to http://localhost:5216/weatherforecast. You should see a JSON response with some weather forecast data.

````json
[{"date":"2023-02-28T01:53:10.9049782+01:00","temperatureC":42,"temperatureF":107,"summary":"Freezing"},{"date":"2023-03-01T01:53:10.9049902+01:00","temperatureC":28,"temperatureF":82,"summary":"Mild"},{"date":"2023-03-02T01:53:10.9049906+01:00","temperatureC":33,"temperatureF":91,"summary":"Cool"},{"date":"2023-03-03T01:53:10.9049908+01:00","temperatureC":22,"temperatureF":71,"summary":"Freezing"},{"date":"2023-03-04T01:53:10.904991+01:00","temperatureC":44,"temperatureF":111,"summary":"Bracing"}]
````

Finally, open Kibana by navigating to http://localhost:5601/ in a web browser. Then, go to the "Discover" page and create index pattern and you should see logs similar to the following:

![image](https://user-images.githubusercontent.com/68539411/221548019-811de131-8c21-4d77-89e2-525eea9beb9a.png)

As we refresh the browser with records from our application, the more the records we will get. Expanding one of the logs will give more details. For example in the snapshot below we can see the message that I wrote in the controller, the request path can also be seen, the time stamp and much more..

![image](https://user-images.githubusercontent.com/68539411/221548551-c3d5d326-fbb0-43b3-a63b-a0f8b3d0ae95.png)


## Using Kibana to search, filter, and visualize the logs


Now, we can view the dashboard by going to the "Dashboard" section in the left-hand navigation menu and clicking on the "dotnetelk Dashboard" dashboard. This should bring up a dashboard that looks like the following:

![image](https://user-images.githubusercontent.com/68539411/221553842-637d3dc5-6fde-4609-822a-c3f1b71bb768.png)

This dashboard shows various visualizations of the logs that we've been sending to Elasticsearch. For example, the "HTTP requests over time" visualization shows the number of HTTP requests that have been made to the API over time and the 200 response.

## Demonstration : Summary and cleanup

In this final demonstration, we'll summarize what we've learned and clean up the resources that we've created.

### Step 1: Summary
We've gone through the following steps:

1. Set up a .NET 6 web API project
2. Added Serilog to the project to log HTTP requests and responses
3. Configured Serilog to send logs to Elasticsearch
4. Set up Kibana dashboards to visualize the logs
By following these steps, we've created a logging pipeline that enables us to collect and visualize logs from our .NET 6 web API.

### Step 2: Clean up
To clean up the resources that we've created, we'll stop the Elasticsearch and Kibana Docker containers and delete the log files that we've created.

To stop the Docker containers, open a terminal and navigate to the directory where the docker-compose.yml file is located. Then, run the following command:

````bash
docker-compose down
````

### Step 3: Conclusion
Congratulations! You've successfully set up a logging pipeline using Elasticsearch, Kibana, and Serilog in a .NET 6 web API project. By doing so, you've created a powerful tool for collecting and visualizing logs, which can be very helpful in troubleshooting issues and gaining insights into the behavior of your application.




  











