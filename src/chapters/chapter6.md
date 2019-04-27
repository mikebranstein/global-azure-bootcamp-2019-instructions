## Consuming a Secure REST API

In this chapter, you'll learn:
- How to modify your Web project to communicate to a backend API via APIM

### Overview

In the last chapter, you deployed a secure API using API Management, then restricted access to your REST API so only API Management could communicate with it. 

This is fantastic! But...now the Contoso University web app you deployed cannot talk to the backend API directly.

In this chapte,r we'll refactor the REST API access code slightly to communicate to the backend API via the APIM gateway.

### Refactoring REST API Access

To communicate through the APIM gateway, you'll need to update the code in the Contoso University web app. Let's get started!

<h4 class="exercise-start">
    <b>Exercise</b>: Update the web app
</h4>

There are a number of steps you'll need to accomplish to ensure the deployed Contoso University web app can speak through the APIM gateway:
1. Update Key Vault with new Api:BaseAddress, Api:SubscriptionKey, and Api:ApiPath values
2. Add a blank Api:SubscriptionKey key/value pair to the app configuration file
3. Refactor the httpClient code to pass the *Ocp-Apim-Subscription-Key* header with each request

#### Updating Key Vault secrets

Navigate to the Key Vault you deployed earlier, and add/update the following keys with their new values:
- Api--BaseAddress: change this to the URL of your API Management gateway, mine was https://contoso-university-meb.azure-api.net
- Api--ApiPath: /contoso/v1
- Api--SubscriptionKey: the subscription key you copied from the Developer portal in the previous chapter

#### Add a black Api:SubscriptionKey key/value pair

Open the web project's configuration file and add a "SubscriptionKey": "" key/value pair to the "Api" section. It should now look like:

```json
"Api": {
    "BaseAddress": "https://localhost:XXXXX",
    "SubscriptionKey": "",
    "ApiPath": ""
}
```

#### Refactor controllers to include the Ocp-Apim-Subscription-Key HTTP header

Open the Students contoller and update the Create() function to:

```c#
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
                httpClient.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", _configuration["Api:SubscriptionKey"]);
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

Then, open the Courses controller and update it's Index() method to:

```c#
// GET: Courses
public async Task<IActionResult> Index()
{
    using (var httpClient = new HttpClient())
    {
        try
        {
            httpClient.BaseAddress = new Uri(_configuration["Api:BaseAddress"]);
            httpClient.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", _configuration["Api:SubscriptionKey"]);
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

#### Testing your changes

Redeploy the ContosoUniversity web app to Azure and visit the site to make sure it still works.

This concludes the exercise. 

<div class="exercise-end"></div>

Congratulations! You've successfully secured your API and accessed it from another application. 

