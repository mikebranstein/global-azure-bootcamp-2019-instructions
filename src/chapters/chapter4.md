## Decoupling Web Apps from Data Access with a REST API Service Layer

In this chapter, you'll learn how to:
- Create a Web API REST service that can be shared across applications
- Remove direct data access from MVC controllers  
- Use Swagger to create an easy-to-use REST API explorer

### Overview

If you take a moment to investigate the ContosoUniversity MVC app, you'll notice that the MVC model and Entity Framework DbContext classes are split out in a shared library (ContosoUniversityData); however, much of the data access is being done in the controllers. This isn't "wrong", and it's a valid approach for small, simple applications. 

Althgouh not "wrong" it can reduce code reuse and encourge developers to re-write the same type of code over and over again. For example, let's assume Contoso's accounting department needs access to student data in their application - all of a sudden it becomes easier for the developers working on that project to tie directly into the database, or write their own DbContext. And, before you know it, there are 4 different applications all attached to this database, extracting and manipulating data separately. 

This is wrong, and it's a code smell.

Now, imaging that one of these teams needs a change made to how Students are tracked - how do you manage that across all the disparate applications? This is how an innocent seemingly-innocuous change can affect an entire organization downstream. 

So, what's a devleoper to do? 

There are a number of approaches and architectural patterns that exist to combat this scenario. In today's workshop, we're going to take a very niave, first-step approach to decoupling the web UI layer from the data access layer by creating a Web API project that abstracts the data access into a single, re-usable place. When that's finished, you'll have a single (and versioned) interface to access Contoso University data.

Let's get started.

### Creating the Web API Project

Our first step is to create a Web API project.

<h4 class="exercise-start">
    <b>Exercise</b>: Creating a Web API Project
</h4>

In Visual Studio, add a new .NET Core 2.2 Web API Project to the existing solution.

Choose the *ASP.NET Core Web Application* project type from the *Add New Project* window:

<img src="images/chapter4/select-project.png" class="img-override" />

Name the project *ContosoUniversityAPI*:

<img src="images/chapter4/name-project.png" class="img-medium" />

For the project type, ensure you've selected *ASP.NET Core 2.2* and *API*. Click *Create*:

<img src="images/chapter4/project-type.png" class="img-override" />

#### Adding the Key Vault components to the API

Next, you'll have to back-track a bit and complete the same steps from the previous chapter related to the Key Vault components, but doing so for the Web API project. 

I'm not going to detail these steps for you here, because you can look. But I will give you the high-levle beats of what you'll be doing. Feel free to use this list as a TODO checklist:
1. Copy the contents of the app configuration file into the API's config file 
2. Add Key Vault related NuGet Packages 
3. Add a reference to the ContosoUniversityData project
4. Add the *PrefixKeyVaultSecretManager* class (be sure to change the namespace if you're copy/pasting the actual file)
5. Update *Program.cs* to add Key Vault configuration to the API project

#### Configuring Entity Framework

In addition to these steps, you'll also need to configure the Entity Framework School DbContext class for the API project.

Copy the contents of the *Program.cs Main()* function, and move it into the API project's *Program.cs* file.

Finally, replace the *Configuration()* function in *Startup.cs* with the following:

```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<SchoolContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services
        .AddMvc()
        .SetCompatibilityVersion(CompatibilityVersion.Version_2_2)
        .AddJsonOptions(
            options => options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
        );
}
```

These code changes register the School DbContext class and ensure JSON data returned by REST API is complete.

#### Testing your changes

Run the API project to test it. The Values Controller should return static values in a browser window:

<img src="images/chapter4/values.png" />

This concludes the exercise. 

<div class="exercise-end"></div>  

### Moving Web Controller Data Access

Next, we'll be moving select data access logic from the Student and Courses controllers into Student and Courses controllers in the API project. Let's get started.

<h4 class="exercise-start">
    <b>Exercise</b>: REST API Controller Creation and Moving Data Access Logic
</h4>

> **You may have noticed...**
>
> ...there's a lot of data access logic in the Web controllers. I'm not walking you through moving every controller action. Instead, we'll be moving 2 controller actions, and I'll leave the others up to you. Think of it as a challenge (and practice for doing this IRL).

#### API Project - Values Controller

Now that we know the API project runs, delete the Values controller - it's not needed.

#### API Project - Courses Controller

Let's start with the Courses controller. Create a Courses controller in the API project, then add the code below.

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using ContosoUniversityData.Data;
using ContosoUniversityData.Models;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace ContosoUniversityAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class CoursesController : ControllerBase
    {
        private readonly SchoolContext _context;

        public CoursesController(SchoolContext context)
        {
            _context = context;
        }

        // GET: api/Courses
        [HttpGet]
        public IEnumerable<Course> Get()
        {
            var courses = _context.Courses
                .Include(c => c.Department)
                .AsNoTracking();
            return courses.ToList();
        }
    }
}
```

#### API Project - Students Controller

Now, add a Students controller and the following code:

```c#
using System.Net;
using System.Threading.Tasks;
using ContosoUniversityData.Data;
using ContosoUniversityData.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace ContosoUniversityApi.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class StudentsController : ControllerBase
    {
        private readonly SchoolContext _context;

        public StudentsController(SchoolContext context)
        {
            _context = context;
        }

        // POST: api/Students
        [HttpPost]
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

    }
}
```

#### Testing your changes

Finally, open the *launchSettings.json* file and modify the *launchUrl* value to be *api/Courses* in al places. Note - you will need to change it in 2 places.

<img src="images/chapter4/launch-settings.png" class="img-small" />

As you can see, we added the HTTP GET for courses and HTTP POST for students. Run the API project to test your changes. You shoud see a list of Courses, returned as JSON:

<img src="images/chapter4/courses.png" class="img-override" />

This concludes the exercise. 

<div class="exercise-end"></div>  

Now that you've created the API project, it's time to refactor the Web project controllers to call the API project's REST endpoints instead of using Entity Framework.

Let's get to work.


<h4 class="exercise-start">
    <b>Exercise</b>: Refactoring the Web project to call a REST API
</h4>

Start with the Web Courses controller.

#### Web Project - Courses Controller

First, replace the CoursesController constructor with the following:

```c#
private readonly IConfiguration _configuration;

public CoursesController(SchoolContext context, IConfiguration configuration)
{
    _context = context;
    _configuration = configuration;
}
```

This will use ASP.NET Core's built-in dependency injection to obtain a reference to the app's configuration objects. 

Next, replace the HTTP GET Index() method with the following:

```c#
// GET: Courses
public async Task<IActionResult> Index()
{
    using (var httpClient = new HttpClient())
    {
        try
        {
            httpClient.BaseAddress = new Uri(_configuration["Api:BaseAddress"]);
            var response = await httpClient.GetAsync($"{_configuration["Api:ApiPath"]}/api/Courses");
            response.EnsureSuccessStatusCode();

            var stringResult = await response.Content.ReadAsStringAsync();

            var courses = JsonConvert.DeserializeObject<IEnumerable<Course>>(stringResult);
            return View(courses);
        }
        catch (HttpRequestException httpRequestException)
        {
            return BadRequest($"Error getting request: {httpRequestException.Message}");
        }
    }
}
```

As you'll notice, this code creates an HTTP client, configures it to access a Uri (configured from the Api:BaseAddress configuration setting), then gets the resource at {Api:ApiPath}/api/Courses.

This means that we'll need to create a few configuration variables. Add values for Api:BaseAddress and Api:ApiPath to the configuration file:

```json
"Api": {
    "BaseAddress": "https://localhost:XXXXX",
    "ApiPath":  ""
}
```

> **BaseAdress needs customization**
>
> You'll notice that the code snippet above has http://localhost:XXXXX for the BaseAddress value. You'll need to replace the *XXXXX* with the port number your local API project is running on. Check the API project's launchSettings.json file for the *sslPort* value.

You'll also notice that the Api:ApiPath value is empty - that's OK. Sometimes, IIS hosts APIs in a virtual directory - and in fact, we'll be using it in a later chapter.

#### Web Project - Students Controller

Next, update the Students controller constructor with this code:

```c#
private readonly IConfiguration _configuration;

public StudentsController(SchoolContext context, IConfiguration configuration)
{
    _context = context;
    _configuration = configuration;
}
```

Then, update the HTTP POST Create() method:

```c#
// POST: Students/Create
// To protect from overposting attacks, please enable the specific properties you want to bind to, for 
// more details see http://go.microsoft.com/fwlink/?LinkId=317598.
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create(
    [Bind("EnrollmentDate,FirstMidName,LastName")] Student student)
{
    if (ModelState.IsValid)
    {
        using (var httpClient = new HttpClient())
        {
            try
            {
                httpClient.BaseAddress = new Uri(_configuration["Api:BaseAddress"]);
                var response = await httpClient.PostAsJsonAsync<Student>($"{_configuration["Api:ApiPath"]}/api/Students", student);
                response.EnsureSuccessStatusCode();
                return RedirectToAction(nameof(Index));
            }
            catch (HttpRequestException httpRequestException)
            {
                //Log the error (uncomment ex variable name and write a log.
                ModelState.AddModelError("", "Unable to save changes. " +
                    "Try again, and if the problem persists " +
                    "see your system administrator.");
            }
        }
    }
    return View(student);
}
```

Just like the Course controller, the HttpClient class is used to make an HTTP request to the API project (using the same configuration variables).

#### Testing your changes

To run both projects at once, right-click the solution, navigate to the *Properties* page, and under *Startup Project*, select *Multiple startup projects* and *Start* for the Web and API projects.

<img src="images/chapter4/multiple-startup.png" class="img-medium" />

Run the projects, then:
- Navigate to the Courses page
- Try adding a Student

This concludes the exercise. 

<div class="exercise-end"></div>  


### Using Swagger to Build an API Definition 

Having a REST API deployed somewhere is just fine, but you need other developers know how they can interact with the API. That's where Swagger Specifications (a.k.a. OpenAPI Specifications) come in.

#### What is Swagger (OpenAPI) Specification?

The OpenAPI Specification (OAS), formerly known as the Swagger Specification, is the world’s standard for defining RESTful interfaces. The OAS enables developers to design a technology-agnostic API interface that forms the basis of their API development and consumption.

You can learn more at [Swagger's website](https://swagger.io/resources/open-api/) if you're interested, but in summary, Swagger provides tools that can scan your API project and create a common-language specification document (in JSON) that describes all available REST commands for your API.

Swagger is easy to use, so let's get started.
 
<h4 class="exercise-start">
    <b>Exercise</b>: Adding Swagger to your API project
</h4>

To add Swagger to your API project, add the following NuGet packages (and specific versions) to your API project:
- Swashbuckle.AspNetCore, v5.0.0-rc2
- Swashbuckle.AspNetCore.Swagger, v5.0.0-rc2

These packages add code generation capabilities to auto-generate a swagger.json file to describe your API and a middleware layer that hosts a Swagger UI to visualize and quickly test your API.

Next, update the *ConfigurationServices()* method of the Startup class:

```c#
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<SchoolContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

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

Then update the *Configure()* method:

```c#
// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
        app.UseHsts();
    }

    // Enable middleware to serve generated Swagger as a JSON endpoint.
    app.UseSwagger();

    // Enable middleware to serve swagger-ui (HTML, JS, CSS, etc.), 
    // specifying the Swagger JSON endpoint.
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
    });

    app.UseHttpsRedirection();
    app.UseMvc();
}
```

#### Testing our your changes

Launch the API project and navigate to https://localhost:XXXXX/swagger/v1/swagger.json. You should see a JSON file the browser:

<img src="images/chapter4/swagger-json.png" class="img-medium" />

Next, navigate to https://localhost:XXXXX/swagger to see the interactive UI:

<img src="images/chapter4/swagger-ui.png" class="img-medium" />

Nice work! Take a few minutes to experiment with the Swagger UI - you'l notice that it has scanned your .NET code and determined there are 2 API endpoints: api/courses and api/students. You can even try out calls to it dynamically via the UI.

Exciting!

This concludes the exercise. 

<div class="exercise-end"></div> 

### Deploying the updated Projects to Azure

Now that we've tested our code locally, let's get it into Azure. 

<h4 class="exercise-start">
    <b>Exercise</b>: Deploying and testing in Azure
</h4>

To deploy an test in Azure we have 3 steps:
1. Publish the API project to a new Web App
2. Add the new secrets to our Key Vault (for Api:BaseAddress and Api:ApiPath)
3. Publish the Web project to the existing Web App

Let's get started.

#### Publishing the API project

You've done this before with the Web project, so follow the same steps to create a new Web App. A few things to remember as you go through this process:
- When creating the new Web App, take notes of it's URL
- Use the same resource group you created earlier
- You can reuse the Free App Plan

After publishing the API project, you'll notice that the site won't load:

<img src="images/chapter4/failure.png" class="img-override" />

That's because it cannot access your Key Vault! If you recall in the previous chapter, we added the Web project MSI to the access policies of the Key Vault. Now, we need to do the same steps for the API project. 

I won't walk you through the details, but will outline the high-level steps and let you try it on your own:
1. Add an MSI account to the API project you just created
2. Add this MSI account to the *Secrets Management* access policy on the Key Vault
3. Restart the API web app to have it attempt to access the Key Vault 

#### Updating the Key Vault

Navigate to the Secrets area of your Key Vault and add the following key/value pairs:
- Api--BaseAddress: the URL of the new API web app, mine was https://contoso-web-api-meb.azurewebsites.net
- Api--ApiPath: {empty string, for now}

Note: now that you've added these values, you may need to restart the API web app again to have it read the updated values.

#### Publishing the Web project

You've done this before, so just republish. If you need help, check back at an earlier chapter.

#### Testing your changes

After you've published the Web Site, navigate to it and try to:
- View the list of courses
- Add a new student


This concludes the exercise. 

<div class="exercise-end"></div> 


In this chapter, you learned:
- How to create a Web API project to centralize data access
- How to advertise REST APIs using Swagger
- How to call REST APIs from a ASP.NET Core MVC app 

