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

There's not much more to do with the acoustic model, so let's do the same with our language data set and train a language model

### Language Models

Training language models is just like training acoustic models, so let's dive in.

<h4 class="exercise-start">
    <b>Exercise</b>: Training a language model
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, and navigate to *Language Models*. 

<img src="images/chapter4/lang-model1.png" class="img-override" />

This page shows the various language models you've trained for the CSS.

Click the *Create New* button and complete the following fields:
- Locale: en-US
- Name: Pokemon - Language Model
- Description: *blank*
- Base Language Model: Microsoft Search and Diction Model
- Language Data: Pokemon - Language Data - Training
- Pronunciation Data: Pokemon - Pronunciation Data - Training
- Subscription: *your subscription*
- Accuracy Testing: *unchecked*, b/c we've already run a test

<img src="images/chapter4/lang-model2.png" class="img-override" />

Click *Create* to train the model.

When the model is saved, you'll navigate back to the *Language Models* page:

<img src="images/chapter4/lang-model3.png" class="img-override" />

Note the *Status* of the test run is *NotStarted*. In a few moments, it will change to *Running*, then *Succeeded*.

The training process may take some time to execute (up to 10 minutes). So, it's a good time to take yet another short break. Check back in another 5.

<div style="padding-left: 20px;"> <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/8_lfxPI5ObM?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> </div>

Welcome back, again. This video was for me. I *love* Tesla. Hopefully, I'll get one someday. Someday soon.

So, let's check back in on the model training:

<img src="images/chapter4/lang-model4.png" class="img-override" />

Excellent, it's finished.

This concludes the exercise. 

<div class="exercise-end"></div>  

### Testing the Trained Models

Now that you've built an acoustic model and language model that customizes the base models, let's test them! The original WER was 45%, so I think we can do better. 

<h4 class="exercise-start">
    <b>Exercise</b>: Performing an accuracy test on your trained models
</h4>

Start by navigating to the CSS web portal at <a href="https://cris.ai" target="_blank">https://cris.ai</a>, and navigate to *Accuracy Tests*. 

Click the *Create New* button to begin a test against an acoustic and language model.

Complete the following fields:
- Locale: en-US
- Subscription: *the one you created earlier*
- Base Model: Microsoft Search and Diction Model, then select the Pokemon models below
- Acoustic Data: Pokemon - Acoustic Data - Testing

<img src="images/chapter4/test6.png" class="img-override" />

Click *Create* to begin the test run.

When the test run is saved, you'll navigate back to the *Accuracy Test Results* page:

<img src="images/chapter4/test7.png" class="img-override" />

Note the *Status* of the test run is *NotStarted*. In a few moments, it will change to *Running*, then *Succeeded*.

The test run may take some time to execute (up to 10 minutes). So, it's a good time to take a short break. Check back in 2.

<div style="padding-left: 20px;"> <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/A0FZIwabctw?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> </div>

This one was for everyone. And it's amazing.

So, let's check back in on the accuracy test.

<img src="images/chapter4/test8.png" class="img-override" />

Sweet! Look at that - 6% WER. I'm ok with that (for now). Feel free to explore the details of the accuracy test to learn more.

> **Challenge #5**
>
> Try to get the accuracy test WER down to 0%. Enough said.

This concludes the exercise. 

<div class="exercise-end"></div> 

In this chapter, you learned:
- why it's important to test Microsoft's base model to establish a baseline accuracy
- how to create acoustic and language models
- how to improve CSS accuracy by building customized models 

