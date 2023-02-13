---
title: "Learning Journey: Azure Functions"
date: 2023-02-13T10:50:40+08:00
description: "Tying together Azure Functions, .Net Core Web API, and Entity Framework"
featured_image: "af.svg"
tags: ["Azure", "Web Development", "Serverless Computing", "Database", ".Net Framework"]
---
# Background
After dipping my toes into the water of using .NET to write RESTful APIs, soon enough I find myself amazed by the number of tools, platforms, and options available in the .NET ecosystem. Recently I had the idea of writing a monthly expense tracking app, which I thought would be the perfect opportunity to experiment with different things in .Net and Azure that I haven't touched before.

# The Goals
My main goal is to have a basic CRUD API available for my front end to add, retrieve, and update transactions. I would also like all of my stuff to be on the cloud as I didn't feel like wrestling with hosting stuff myself. With that in mind, I have a couple of decisions to make:

# The Tools

## Cloud Provider - Azure
As the title suggests, I went with Azure. I probably could have gone with a cheaper option but I thought learning it might be a nice addition to my skill set. It's also convenient that since I would be working with .Net stuff, they would integrate nicely.

## Database - Azure SQL
Since I have worked with Azure SQL before, I might as well stick with it. It's easy to set up and once I did I can forget about it. Initially, I also had the itch to try out a document database like CosmosDB and MongoDB but I soon decided it was a rabbit hole destined for another project.

## .NET Core Web API, or is there another option?
My last CRUD app was written as a .Net web API, hosted on Azure as a Web App Service. It was easy to deploy and suited my need, with only one problem -- **Cost**. Azure Web App Service incurs charges as long as it is running, no matter how few requests it receives. Since this is a hobby project I probably will only have a single digit amount of requests each day, I did not like the fact that I would be paying for all the time the web app is running idle.

## Azure Functions
I then learned that I can write my API as Azure Functions, a serverless SaaS solution that's very similar to a .Net Core web app, with the added benefit of being more modular as every endpoint is represented by its own function. The best part is, it will charge me **per request**! Definitely sounds like the perfect candidate for my needs! (And I will also have another thing to brag about on my resume)

# The Confusion Begins...

## "What is .csx?"
After some setting up, it seems that AF can be written as a traditional `.cs` file, which I need to use an IDE for. Or, I could write it as a C# script `.csx` file, which could be edited directly on the Azure Portal, how cool is that! (spoiler: it's not).

## "What is an SQL binding??"
Since I am writing a CRUD API, obviously I need some way to query the database within my web app. In the previous .Net Core project, I used Entity Framework Core, so *"maybe I could use that here?"* -- the naive me thought. A quick Google search yielded -- nothing. Welp, time to look for another solution! I then came across [this official doc page][def] that seems to be what I need -- I just need to write an HTTP trigger function with SQL input binding for each get endpoint, and SQL output binding for each post endpoint, which seems easy enough. (spoiler: it's not)

# Going back to the old recipe
After digging through the non-existent documentation and Stackoverflow posts on what I was trying to do, I finally gave up on writing the `.csx` files and went back to writing `.cs` with VS Code. Suddenly, all the documentation started making sense and I had finally made a working endpoint that retrieves all the records in my database table.

```csharp
[FunctionName("GetAllTransactions")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = null)]
    HttpRequest req,
    [Sql("select * from dbo.Transactions",
        CommandType = System.Data.CommandType.Text,
        ConnectionStringSetting = "SqlConnectionString")]
    IEnumerable<Transaction> transactions)
{
    return new OkObjectResult(transactions);
}
```
Apparently, I overlooked the fact that the documentation was intended for regular old `.cs` functions and not `.csx` functions. Things finally started making sense.

HTTP Trigger + SQL Binding worked pretty nicely so far until I decided to allow query parameters on my endpoints, which looked something like this:  
`http://myapiurl/gettransaction?month=2&type=food`  
I soon realized the problem -- The SQL statement `[Sql("some SQL query")]`was predetermined, and there was no way I could generate statements based on parameters in the request's query string! It seems that going back to Entity Framework Core was inevitable...

## "Getting Entity Framework Core to work with Azure Functions"
Setting up EF Core in a .Net Core project was not easy, but at least doable thanks to the documentation available online. It was, however, not a good experience trying to add EF Core into my Azure Functions. After many hours of digging through documentation and pulling my hair out, I **FINALLY** got it to work.

The first step is to install the packages required for EF Core, which I did by adding the following two lines to the `.csproj` file.
```xml
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="6.*" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="6.*" />
```
Note that I had to tell the app to specifically use the `6.x` version, which is the latest .Net version that Azure Function supports. If I leave the version blank it will use the `7.x` versions which will cause the build to fail.

I then need to create a `Startup.cs` file to configure the builder services like this:
```csharp
[assembly: FunctionsStartup(typeof(MoneyTrackAPI.Startup))]

namespace MoneyTrackAPI
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddDbContext<DbContext>(
              options => SqlServerDbContextOptionsExtensions.UseSqlServer(
                options, Environment.GetEnvironmentVariable(
                    "SQLAZURECONNSTR_ConnectionString")));
        }
    }
}
``` 

Now, I can finally do the usual `dotnet ef dbcontext scaffold` command to generate the `DbContext.cs` and `Transaction.cs` file which allows me to query the database using LINQ, which means I can successfully call the get endpoint with query parameters. The actual function looks like this:

```csharp
public class GetTransactions
{
    private readonly DbContext _dbContext;

    public GetTransactions(DbContext dbContext)
    {
        _dbContext = dbContext;
    }

    [FunctionName("GetTransactions")]
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "transaction")] 
        HttpRequest req,
        ILogger log)
    {
        var query = req.Query;

        var transactions = _dbContext.Transactions.AsNoTracking();

        try
        {
            if (query.ContainsKey("month"))
            {
                transactions = transactions.Where(
                    t => t.TransactionDate.Month == int.Parse(query["month"]));
            }
        }
        catch
        {
            return new BadRequestResult();
        }

        return new OkObjectResult(await transactions.ToListAsync());
    }
}
```

Voila! After jumping through so many hoops I finally have a GET endpoint that functions like how I wanted. Notice when I want to match more parameters in the query string I can simply add another `if (query.ContainsKey()) {}` block, and EF Core will generate the SQL query behind the scene. What's left for me to do is adding the POST and UPDATE endpoints, which should be trivial.

[def]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-azure-sql?tabs=in-process%2Cextensionv4&pivots=programming-language-csharp