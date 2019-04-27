## Building for the API Economy

In this chapter, you'll learn:
- How to use ASP.NET Core action filters to capture data change events 
- How to create an Event Hub and popoulate it with messages when data changes
- How to respond to Event Hub messages with an Azure function

### Overview

You've completed our mission of creating secure, reusable REST APIs. But what if you want to share your data with an external audience. You've learned that API Management can help you share your APIs, but should an external audience hav access to the same data as your internal devleopment teams?

Definitely not.

In the last several chapters, we'll be endeavoring to build an external data mart accesible by another REST API hosted via APIM. This effort is part of a multi-university data-sharing program where you have agreed to share aggregate course, department, and student performance. 

A key aspect to this type of data mart is that it's use does not affect the use of your internal APIs. For example:
1. If the external API has a large amount of traffic, it should not affect the internal API availability
2. If there were a breach of the external API, there should be no access to the same data accessible from the internal API (i.e., Student names, PII, etc.)

To accomplish these goals, you'll be designing a system composed of 5 additional Azure resources:
1. Event Hub that receives a message when data is updated in the internal ContosoUniversity3 database
2. An Azure Function that listens for Event Hub messages and rebuilds/aggregates the course, department, and student performance data.
3. A Cosmos DB instance that will house the aggregate data in an efficient JSON data structure.
4. A Web API project with REST API endpoint to retrieve the aggregate demographic data.
5. An API Management API that exposes the new REST API service for other universities to access.

We have a lot to build over the next several chapter, so let's get started.

### Provisioning the Azure Resources

<h4 class="exercise-start">
    <b>Exercise</b>: Create the necessary resources
</h4>

We'll provision all of the Azure resources at once. 

#### Event Hub

Create an Event Hub with the following settings (be sure to use your own unique name):

<img src="images/chapter7/eh-create.png" />

Creating the Event Hub actually creates an Event Hub Namespace. When it's finished deploying, navigate to it and create an Event Hub named *dataupdated*:

<img src="images/chapter7/eh-dataupdated.png" />

Next, navigate to the *Shared access policies* section, click on *RootManageSharedAccessPolicy* and copy the *Primary key* and *Connection string primary key* values:

<img src="images/chapter7/eh-sap.png" class="img-medium" />

Save these values for later.

#### Function App

Create a Function App with the following settings (be sure to use your own unique name for the function app and the storage account):

<img src="images/chapter7/func-create.png" />

#### Cosmos Db

Create a Cosmos Db with the following settings (be sure to use your own unique name):

<img src="images/chapter7/cosmos-create.png" />

#### Web App for External API

Create a Web App with the following settings to host the external Web API project (be sure to use your own unique name):

<img src="images/chapter7/web-app-create.png" />

This concludes the exercise. 

<div class="exercise-end"></div>

### Triggering Data Updated Messages with Action Filters

Now that our Azure resources are created, let's return to our code. The goal is to write a message to the Event Hub each time data is changed in our database, so that an Azure Function App (which is listening for messages on the Event Hub) can aggregate our internal data and write it to a Cosmos Db.

The best time to write a message on to the Event Hub is immediatly after data has been updated in the SQL database, which occurs in our central Web API REST service. But, it feels sloppy to add a bunch of code to every method that updates data. 

Instead, it's better to create a decoupled process that can augment the Web API controller actions. Luckily, this si the exact purpose of Action Filters.

There are several types of ASP.NET Core action filters, but we're really only concerned with one that will run code after controller actions succeed. 

So, if we do this the right way and put the Event Hub message logic in an action filter, we can annotate any controller action that updates SQL data with the filter. And viola! Decoupled design where we have a very low risk of affecting the code within our existing REST API methods. 

<h4 class="exercise-start">
    <b>Exercise</b>: Creating an ASP.NET Core Action Filter
</h4>

Create a new .NET Core Class Library named ContosoUniversityInfra in your solution. It will store shared Event Hub message data types that can be access by the API project and a Function App project (to be provisioned in a latter chapter).

<img src="images/chapter7/infra.png" />

Add the following NuGet packages (being careful to use the exact versions specified):
- Newtonsoft.Json, v11.0.2

Remove the `Class1.cs` file.

Create a *Models* folder and add 2 classes: *CourseAggregateDto* and *DemographicsDto*. These classes will hold the aggregate course, department, and student performance demographic data that is stored in Cosmos Db.

Use the following code snippets to build out each class.

#### CourseAggregateDto

```c#
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Text;

namespace ContosoUniversityInfra.Models
{
    public class CourseAggregateDto
    {
        [JsonProperty(PropertyName = "courseId")]
        public int CourseId { get; set; }

        [JsonProperty(PropertyName = "courseName")]
        public string CourseName { get; set; }

        [JsonProperty(PropertyName = "departmentId")]
        public int DepartmentId { get; set; }

        [JsonProperty(PropertyName = "departmentName")]
        public string DepartmentName { get; set; }

        [JsonProperty(PropertyName = "enrollmentCount")]
        public int EnrollmentCount { get; set; }

        [JsonProperty(PropertyName = "gradeACount")]
        public int GradeACount { get; set; }

        [JsonProperty(PropertyName = "gradeBCount")]
        public int GradeBCount { get; set; }

        [JsonProperty(PropertyName = "gradeCCount")]
        public int GradeCCount { get; set; }

        [JsonProperty(PropertyName = "gradeDCount")]
        public int GradeDCount { get; set; }

        [JsonProperty(PropertyName = "gradeFCount")]
        public int GradeFCount { get; set; }

        [JsonProperty(PropertyName = "gradeOtherCount")]
        public int GradeOtherCount { get; set; }
    }
}
```

#### DemographicsDto.cs

```c#
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Text;

namespace ContosoUniversityInfra.Models
{
    public class DemographicsDto
    {
        [JsonProperty(PropertyName = "id")]
        public string Id { get; set; }

        [JsonProperty(PropertyName = "schoolYear")]
        public int SchoolYear { get; set; }

        [JsonProperty(PropertyName = "courses")]
        public List<CourseAggregateDto> Courses { get; set; }

        public DemographicsDto(int schoolYear)
        {
            this.SchoolYear = schoolYear;
            this.Courses = new List<CourseAggregateDto>();
        }
    }
}
```

#### Event Hub Message

Add a class named *DataUpdatedMessage* to the root of the project. Use the following code to complete the class:

```c#
using System;

namespace ContosoUniversityInfra
{
    public class DataUpdatedMessage
    {
        public MessageType MessageType { get; set; }
    }

    public enum MessageType
    {
        None = 0,
        Rebuild = 1
    }
}
```

#### Create an Action Filter

Add a few staple NuGet packages to the ContosoUniversityApi project (be mindful of the specific versions we're referencing):
- Microsoft.Azure.EventHubs, v3.0.0

Add a reference to the ContosoUniversityInfra project.

In the ContosoUniversityApi project, create a folder named *Filters* and add a class named *PublishDataUpdatedEventActionFilter* to the folder. Populate the class with the following code:

```c#
using ContosoUniversityInfra;
using Microsoft.AspNetCore.Mvc.Filters;
using Microsoft.Azure.EventHubs;
using Microsoft.Extensions.Configuration;
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ContosoUniversityApi.Filters
{
    public class PublishDataUpdatedEventActionFilter : IActionFilter
    {
        private readonly IConfiguration _configuration;

        public PublishDataUpdatedEventActionFilter(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        // runs after a controller action completed
        public void OnActionExecuted(ActionExecutedContext context)
        {
            // construct a message to send on the event hub
            var message = new DataUpdatedMessage() { MessageType = MessageType.Rebuild };

            // send the message
            SendMessageToEventHub(message, _configuration["EventHub:ConnectionString"])
                .GetAwaiter()
                .GetResult();
        }

        public void OnActionExecuting(ActionExecutingContext context)
        {
            return; // do nothing 
        }

        private static async Task SendMessageToEventHub(DataUpdatedMessage message, string eventHubConnectionString)
        {
            // Creates an EventHubsConnectionStringBuilder object from the connection string, and sets the EntityPath.
            // Typically, the connection string should have the entity path in it, but this simple scenario
            // uses the connection string from the namespace.
            var connectionStringBuilder = new EventHubsConnectionStringBuilder(eventHubConnectionString)
            {
                EntityPath = "dataupdated"
            };

            var eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());

            await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(message))));
            await eventHubClient.CloseAsync();
        }
    }
}
```

#### Update the App Configuration

Next, open the API project configuration file and add a value for *EventHub:ConnectionString* that contains the Event Hub connection string you saved earlier in this chapter.

```json
  "EventHub": {
    "ConnectionString": "{your event hub connection string here}"
  }
```

After updating the configuration file, add the same value in the Azure Key Vault.

> **Event Hubs and Development**
>
> Azure Event Hubs do not have a solution for developing locally, so you have to use a real Event Hub for local development. Usually, I would have a separate Event Hub I use for development and production work, but to same time in the workshop, we're using the same Event Hub. So, when you get back to work, do what I say, not what I do ;-)

#### Register the Action Filter

Before you can use the Action Filter you need to register it in the API project's startup process.

Update the *ConfigureServices()* function in the Startup class with the following code:

```c#
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<SchoolContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services.AddScoped<PublishDataUpdatedEventActionFilter>();

    services
        .AddMvc()
        .SetCompatibilityVersion(CompatibilityVersion.Version_2_2)
        .AddJsonOptions(
            options => options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
        );

    // Register the Swagger generator, defining 1 or more Swagger documents
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo { Title = "ContosoUniversity", Version = "v1" });
    });
}
```

You'll notice we added a line to register the *PublishDataUpdatedEventActionFilter* class.

#### Applying the Action Filter

The final step is to apply the action filter to any controller actions that update data. Because our solution is purposefully small, the only qualifying controller action is the Student contoller's *Create()* method. Update it with the followin code:

```c#
// POST: api/Students
[HttpPost]
[ServiceFilter(typeof(PublishDataUpdatedEventActionFilter))]
public async Task<StatusCodeResult> Post([FromBody] Student student)
{
    try
    {
        _context.Add(student);
        await _context.SaveChangesAsync();
    }
    catch (DbUpdateException /* ex */)
    {
        return new StatusCodeResult((int)HttpStatusCode.BadRequest);
    }
    return new StatusCodeResult((int)HttpStatusCode.Created);
}
```

You'll notice the annotation applying the action filter.

#### Testing the changes

Publish the API project to Azure, open the Contoso University web app, and create a new Student.

Wait approx 2-3 minutes, then navigate to the Event Hub Overview page in the Azure portal. Switch the metic type to *Messages* and look for a spike on the chart, indicating that a message was sent to the Event Hub when the student was created.

<img src="images/chapter7/eh-message.png" class="img-override" />

This concludes the exercise. 

<div class="exercise-end"></div>

### Processing the Event Hub Message

Now that the Event Hub has our messages, it's time to create an Azure Function to listen for the message and build the course, department, and student performance demographic data.

<h4 class="exercise-start">
    <b>Exercise</b>: Create an Azure Function
</h4>

Add an Azure Function project named *ContosoUniversityFunction* to the Visual Studio solution:

<img src="images/chapter7/add-func.png" class="img-override" />

When specifying the details for the Azure Function project, select *Azure Functions v2 (.NET Core)*, choose the *Event Hub trigger*, *Storage Emulator* for the storage account, *EventHubConnectionString* for the connection string setting, and *dataupdated* as the Event Hub name:

<img src="images/chapter7/eh-create-2.png" />

Click *Ok*.

This will add the Azure Function app project to the solution. Unfortunately, it add the Function project as a .NET Core 2.1 project. 

Immediately right-click the project, select *Properties*, and set the *Target Framework* to .NET Core 2.2:

<img src="images/chapter7/prop.png" class="img-override" />

Remove the *Function1.cs* file.

Next, add project references to the following projects:
- ContosoUniversityData (for Entity Framework class references)
- ContosoUniversityInfra (for Event Hub message class references)

Add the follow NuGet Packages:
- Microsoft.Azure.DocumentDB.Core, v2.3.0

Open the *local.settings.json* file and add a key under the *Values* area for *EventHubConnectionString*. Set the value to the Event Hub conenction string you saved earlier in this chapter. Your file should look like this:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "EventHubConnectionString": "{your event hub connection string here}"
  }
}
```

Next, create a class named *UpdateDataFunction* and replace it's contents with the following code:

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.EventHubs;
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using ContosoUniversityInfra;
using ContosoUniversityData.Data;
using Microsoft.EntityFrameworkCore;
using Microsoft.Azure.Documents.Client;
using ContosoUniversityInfra.Models;
using Microsoft.Azure.Documents.Linq;

namespace ContosoUniversityFunction
{
    public static class UpdateDataFunction
    {

        [FunctionName("UpdateData")]
        public static async Task Run([EventHubTrigger("dataupdated", Connection = "EventHubConnectionString")] EventData[] events, ILogger log)
        {
            var exceptions = new List<Exception>();

            foreach (EventData eventData in events)
            {
                try
                {
                    string messageBody = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
                    var message = JsonConvert.DeserializeObject<DataUpdatedMessage>(messageBody);

                    if (message.MessageType == MessageType.Rebuild)
                    {
                        log.LogInformation($"C# Event Hub trigger function processed a message: {messageBody}");
                    }

                    await Task.Yield();
                }
                catch (Exception e)
                {
                    // We need to keep processing the rest of the batch - capture this exception and continue.
                    // Also, consider capturing details of the message that failed processing so it can be processed again later.
                    exceptions.Add(e);
                }
            }

            // Once processing of the batch is complete, if any messages in the batch failed processing throw an exception so that there is a record of the failure.
            if (exceptions.Count > 1)
                throw new AggregateException(exceptions);

            if (exceptions.Count == 1)
                throw exceptions.Single();
        }
    }
}
```

This code is a standard Azure Function definition that is bound to the Event Hub, and runs each time a message is sent to the hub.

#### Starting Storage Emulator

Before we test our code, you'll have to start the Azure Storage Emulator locally. Storage Emulator is a locally-running application that creates a storage account for use by the Azure Function. 

In the start menu of your workshop VM, type *Storage Emulator* and it will appear in the list of available apps. CLick it to run. Wait approx. 1 minute while it runs for the first time:

<img src="images/chapter7/storage.png" class="img-override" />

That's it! You should see a message saying it started successfully.

#### Testing your Function

Now that you've written you Function, run it in Visual Studio. Change the startup project to the Function project and debug!

<img src="images/chapter7/func.gif" class="img-override" />

Near the end of the file, you'll see a message indicating that a message was received from the Event Hub and processed. This means everything is wired up correctly.

This concludes the exercise. 

<div class="exercise-end"></div>

Before we continue, let's take a short break and learn a little about Cosmos Db.

### About Cosmos Db

Today’s applications are required to be highly responsive and always online. To achieve low latency and high availability, instances of these applications need to be deployed in datacenters that are close to their users. Applications need to respond in real time to large changes in usage at peak hours, store ever increasing volumes of data, and make this data available to users in milliseconds.

Azure Cosmos DB is Microsoft's globally distributed, multi-model database service. With a click of a button, Cosmos DB enables you to elastically and independently scale throughput and storage across any number of Azure regions worldwide. You can elastically scale throughput and storage, and take advantage of fast, single-digit-millisecond data access using your favorite API including SQL, MongoDB, Cassandra, Tables, or Gremlin. Cosmos DB provides comprehensive service level agreements (SLAs) for throughput, latency, availability, and consistency guarantees, something no other database service offers.

#### Key Benefits

**Turnkey global distribution**

Cosmos DB enables you to build highly responsive and highly available applications worldwide. Cosmos DB transparently replicates your data wherever your users are, so your users can interact with a replica of the data that is closest to them.

Cosmos DB allows you to add or remove any of the Azure regions to your Cosmos account at any time, with a click of a button. Cosmos DB will seamlessly replicate your data to all the regions associated with your Cosmos account while your application continues to be highly available, thanks to the multi-homing capabilities of the service. For more information, see the global distribution article.

**Always On**

By virtue of deep integration with Azure infrastructure and transparent multi-master replication, Cosmos DB provides 99.999% high availability for both reads and writes. Cosmos DB also provides you with the ability to programmatically (or via Portal) invoke the regional failover of your Cosmos account. This capability helps ensure that your application is designed to failover in the case of regional disaster.

**Elastic scalability of throughput and storage, worldwide**

Designed with transparent horizontal partitioning and multi-master replication, Cosmos DB offers unprecedented elastic scalability for your writes and reads, all around the globe. You can elastically scale up from thousands to hundreds of millions of requests/sec around the globe, with a single API call and pay only for the throughput (and storage) you need. This capability helps you to deal with unexpected spikes in your workloads without having to over-provision for the peak. For more information, see partitioning in Cosmos DB, provisioned throughput on containers and databases, and scaling provisioned throughput globally.

**Guaranteed low latency at 99th percentile, worldwide**

Using Cosmos DB, you can build highly responsive, planet scale applications. With its novel multi-master replication protocol and latch-free and write-optimized database engine, Cosmos DB guarantees less than 10-ms latencies for both, reads and (indexed) writes at the 99th percentile, all around the world. This capability enables sustained ingestion of data and blazing-fast queries for highly responsive apps.

**Precisely defined, multiple consistency choices**

When building globally distributed applications in Cosmos DB, you no longer have to make extreme tradeoffs between consistency, availability, latency, and throughput. Cosmos DB’s multi-master replication protocol is carefully designed to offer five well-defined consistency choices - strong, bounded staleness, session, consistent prefix, and eventual — for an intuitive programming model with low latency and high availability for your globally distributed application.

**No schema or index management**

Keeping database schema and indexes in-sync with an application’s schema is especially painful for globally distributed apps. With Cosmos DB, you do not need to deal with schema or index management. The database engine is fully schema-agnostic. Since no schema and index management is required, you also don’t have to worry about application downtime while migrating schemas. Cosmos DB automatically indexes all data and serves queries fast.

**Battle tested database service**

Cosmos DB is a foundational service in Azure. For nearly a decade, Cosmos DB has been used by many of Microsoft’s products for mission critical applications at global scale, including Skype, Xbox, Office 365, Azure, and many others. Today, Cosmos DB is one of the fastest growing services on Azure, used by many external customers and mission-critical applications that require elastic scale, turnkey global distribution, multi-master replication for low latency and high availability of both reads and writes.

**Ubiquitous regional presence**

Cosmos DB is available in all Azure regions worldwide, including 54+ regions in public cloud, Azure China 21Vianet, Azure Germany, Azure Government, and Azure Government for Department of Defense (DoD). See Cosmos DB’s regional presence.

**Secure by default and enterprise ready**

Cosmos DB is certified for a wide array of compliance standards. Additionally, all data in Cosmos DB is encrypted at rest and in motion. Cosmos DB provides row level authorization and adheres to strict security standards.

**Significant TCO savings**

Since Cosmos DB is a fully managed service, you no longer need to manage and operate complex multi datacenter deployments and upgrades of your database software, pay for the support, licensing, or operations or have to provision your database for the peak workload. For more information, see Optimize cost with Cosmos DB.

**Industry leading comprehensive SLAs**

Cosmos DB is the first and only service to offer industry-leading comprehensive SLAs encompassing 99.999% high availability, read and write latency at the 99th percentile, guaranteed throughput, and consistency.

**Globally distributed operational analytics with Spark**

You can run Spark directly on data stored in Cosmos DB. This capability allows you to do low-latency, operational analytics at global scale without impacting transactional workloads operating directly against Cosmos DB. For more information, see Globally distributed operational analytics.

**Develop applications on Cosmos DB using popular NoSQL APIs**

Cosmos DB offers a choice of APIs to work with your data stored in your Cosmos database. By default, you can use SQL (a core API) for querying your Cosmos database. Cosmos DB also implements APIs for Cassandra, MongoDB, Gremlin and Azure Table Storage. You can point client drivers (and tools) for the commonly used NoSQL (e.g., MongoDB, Cassandra, Gremlin) directly to your Cosmos database. By supporting the wire protocols of commonly used NoSQL APIs, Cosmos DB allows you to:

- Easily migrate your application to Cosmos DB while preserving significant portions of your application logic.
- Keep your application portable and continue to remain cloud vendor-agnostic.
- Get a fully-managed cloud service with industry leading, financially backed SLAs for the common NoSQL APIs.
- Elastically scale the provisioned throughput and storage for your databases based on your need and pay only for the throughput and storage you need. This leads to significant cost savings.


### Aggregating Data in an Azure Function

Now that we know a little bit aobut Cosmos Db and also know the Azure Function can process Event Hub messages, let's build out the data aggregation pipeline. First, we'll create a Cosmos Db collection to hold our demographics data.

<h4 class="exercise-start">
    <b>Exercise</b>: Create a Cosmos Db collection.
</h4>

Navigate to the Cosmos Db you created earlier in the workshop. On the *Overview* tab, click *+ Add Collection*:

<img src="images/chapter7/add-collection.png" />

This will prompt you to create a new Database Container. Enter the following values:

<img src="images/chapter7/cosmos-db.png" />

Next, navigate to the *Keys* area of the Cosmos Db and copy the follwoing values:
- URI
- PRIMARY KEY

<img src="images/chapter7/cosmos-keys.png" />

Save them for later.

This concludes the exercise. 

<div class="exercise-end"></div>


In this next exercise, we'll be updating the Azure function to write to Cosmos Db.


<h4 class="exercise-start">
    <b>Exercise</b>: Writing to Cosmos Db
</h4>

Return to Visual Studio and replace the *UpdateDataFunction* class with the following code:

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.EventHubs;
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using ContosoUniversityInfra;
using ContosoUniversityData.Data;
using Microsoft.EntityFrameworkCore;
using Microsoft.Azure.Documents.Client;
using ContosoUniversityInfra.Models;
using Microsoft.Azure.Documents.Linq;

namespace ContosoUniversityFunction
{
    public static class UpdateDataFunction
    {

        [FunctionName("UpdateData")]
        public static async Task Run([EventHubTrigger("dataupdated", Connection = "EventHubConnectionString")] EventData[] events, ILogger log)
        {
            var exceptions = new List<Exception>();
            var options = new DbContextOptionsBuilder<SchoolContext>();
            options.UseSqlServer(Environment.GetEnvironmentVariable("ConnectionString"));
            var context = new SchoolContext(options.Options);

            foreach (EventData eventData in events)
            {
                try
                {
                    string messageBody = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
                    var message = JsonConvert.DeserializeObject<DataUpdatedMessage>(messageBody);

                    if (message.MessageType == MessageType.Rebuild)
                    {
                        log.LogInformation($"C# Event Hub trigger function processed a message: {messageBody}");

                        // recalc CosmosDb for students

                        // get demographics from cosmos db
                        var client = new DocumentClient(new Uri(Environment.GetEnvironmentVariable("CosmosDbUri")), Environment.GetEnvironmentVariable("CosmosDbAuthKey"));

                        var queryOptions = new FeedOptions { MaxItemCount = -1 };
                        var query = client.CreateDocumentQuery<DemographicsDto>(
                                UriFactory.CreateDocumentCollectionUri("demographics", "demographics"), queryOptions)
                                .Where(d => d.SchoolYear == 2019)
                                .AsDocumentQuery();

                        var results = new List<DemographicsDto>();
                        while (query.HasMoreResults)
                            results.AddRange(await query.ExecuteNextAsync<DemographicsDto>());

                        DemographicsDto demographicsDto =
                            results.Count == 0 ? 
                                new DemographicsDto(2019) : 
                                results[0];

                        // clean out existing courses b/c we're going to rebuild it
                        demographicsDto.Courses.Clear();

                        var courses = await context.Courses
                            .Include(i => i.Department)
                            .Include(i => i.Enrollments)
                            .ToListAsync();
                        foreach (var course in courses)
                        {
                            demographicsDto.Courses.Add(new CourseAggregateDto()
                            {
                                CourseId = course.CourseID,
                                CourseName = course.Title,
                                DepartmentId = course.DepartmentID,
                                DepartmentName = course.Department.Name,
                                EnrollmentCount = course.Enrollments.Count,
                                GradeACount = course.Enrollments.Count(e => e.Grade == ContosoUniversityData.Models.Grade.A),
                                GradeBCount = course.Enrollments.Count(e => e.Grade == ContosoUniversityData.Models.Grade.B),
                                GradeCCount = course.Enrollments.Count(e => e.Grade == ContosoUniversityData.Models.Grade.C),
                                GradeDCount = course.Enrollments.Count(e => e.Grade == ContosoUniversityData.Models.Grade.D),
                                GradeFCount = course.Enrollments.Count(e => e.Grade == ContosoUniversityData.Models.Grade.F),
                                GradeOtherCount = course.Enrollments.Count(e => !e.Grade.HasValue)
                            });
                        }

                        // check if document exists, upsert or replace
                        if (demographicsDto.Id == null)
                            await client.UpsertDocumentAsync(UriFactory.CreateDocumentCollectionUri("demographics", "demographics"), demographicsDto);
                        else
                            await client.ReplaceDocumentAsync(UriFactory.CreateDocumentUri("demographics", "demographics", demographicsDto.Id), demographicsDto);
                    }

                    await Task.Yield();
                }
                catch (Exception e)
                {
                    // We need to keep processing the rest of the batch - capture this exception and continue.
                    // Also, consider capturing details of the message that failed processing so it can be processed again later.
                    exceptions.Add(e);
                }
            }

            // Once processing of the batch is complete, if any messages in the batch failed processing throw an exception so that there is a record of the failure.
            if (exceptions.Count > 1)
                throw new AggregateException(exceptions);

            if (exceptions.Count == 1)
                throw exceptions.Single();
        }
    }
}
```

This updated code queries Cosmos Db for the demographic data. It then uses Entity Framework to query enrolment and student performance stats for each course. Finally, it either creates a new entry in Cosmos or updates the existing entry.

Due to time constraints, we won't spend time explaining the details any further; however, you're encouraged to continue exploring on your own.

#### Update the local.settings.json file

Next, add several configuration settings to the *local.settings.json* file in the *Values* section:

```json
"ConnectionString": "{your Azure SQL Server connection string}",
"CosmosDbUri": "{your Cosmos Db URI}",
"CosmosDbAuthKey": "{your Cosmos Db PRIMARY KEY}"
```

#### Testing your code

Run the Azure function, then navigate to the Azure-hosted Contoso University web app and add a Student.

You should see the Function app console window respond almost immediately, indicating it aggregated the data and wrote it to Cosmos Db.

Stop the Function app. Navigate to the Cosmos Db service in the Azure portal. Locate the *Data Explorer* area and find the list of documents stored int he demographics container. You shoudl see the document just created by the Azure function:

<img src="images/chapter7/documentdb.png" class="img-override" />

This concludes the exercise. 

#### Deploying the Function App

Right-click the Function app and select *Publish*. When prompted, *Select an Existing* Function app and locate the app you created earlier in this chapter.

This will compile your Function and deploy it to Azure.

After you've deployed to Azure, navigate to the Function app configuration area:

<img src="images/chapter7/config.png" class="img-override" />

Add several app settings into the Configuration area:
- EventHubConnectionString
- ConnectionString
- CosmosDbUri
- CosmosDbAuthKey

NOTE: These are the same values you had locally stored in the *local.settings.json* file.

Save the settings. Return to the main Function page and restart the Function app.

<div class="exercise-end"></div>

### The Super Challenge

At the beginning of this chapter, we outlined a multi-step process to creating an external REST API. Together, we've completed 75% of that work. We have created the first 3 of these items:
1. Event Hub that receives a message when data is updated in the internal ContosoUniversity3 database
2. An Azure Function that listens for Event Hub messages and rebuilds/aggregates the course, department, and student performance data.
3. A Cosmos DB instance that will house the aggregate data in an efficient JSON data structure.
4. A Web API project with REST API endpoint to retrieve the aggregate demographic data.
5. An API Management API that exposes the new REST API service for other universities to access.

With what you've learned in this workshop, I feel confident you should be able to complete #4 and #5 on your own. The workshop isn't over, and I'll be around to help out when needed or if you get stuck.

Good luck!