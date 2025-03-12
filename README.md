# Robust CSharp CRUD

# Integrate Ember.js with your .NET Core API, follow these steps:

Steps to Integrate Ember.js

* Set up Ember.js frontend
* Enable CORS in .NET Core API
* Expose API Endpoints
* Consume API from Ember.js

## 1. Set Up Ember.js Frontend
Run the following command to create an Ember.js project:
```
npx ember-cli new ember-app --lang=en
cd ember-app
npx ember install ember-fetch
```

## 2. Enable CORS in .NET Core API
Modify Startup.cs in your .NET Core API to allow Ember.js to access the API:
```
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options =>
    {
        options.AddPolicy("AllowSpecificOrigin",
            builder => builder.WithOrigins("http://localhost:4200") // Ember.js default port
                              .AllowAnyMethod()
                              .AllowAnyHeader());
    });

    services.AddControllers();
}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseCors("AllowSpecificOrigin");
    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints => { endpoints.MapControllers(); });
}
```

## 3. Expose API Endpoints
Your .NET Core API is already set up with CRUD endpoints for ProductsController, so it's ready to be consumed by Ember.js.

## 4. Consume API from Ember.js

Modify ember-app/app/adapters/application.js to configure Ember Data:
```
import JSONAPIAdapter from '@ember-data/adapter/json-api';

export default class ApplicationAdapter extends JSONAPIAdapter {
  host = 'http://localhost:5000'; // .NET Core API URL
  namespace = 'api'; // Match with the API route
}
```

#### Fetch Products in Ember Model

Modify ember-app/app/models/product.js:

```
import Model, { attr } from '@ember-data/model';

export default class ProductModel extends Model {
  @attr('string') name;
  @attr('number') price;
}
```

#### List Products in Ember Route

Modify ember-app/app/routes/products.js:

```
import Route from '@ember/routing/route';

export default class ProductsRoute extends Route {
  async model() {
    return this.store.findAll('product');
  }
}
```

#### Render Products in Ember Template

Modify ember-app/app/templates/products.hbs:

```
<h1>Product List</h1>
<ul>
  {{#each @model as |product|}}
    <li>{{product.name}} - ${{product.price}}</li>
  {{/each}}
</ul>
```

#### Run Everything

Start .NET Core API
```
dotnet run
```

Start Ember.js
```
cd ember-app
npm start
```

Now, Ember.js can consume the .NET Core API. Let me know if you need enhancements! 🚀

# Deploy Ember.js and the .NET Core API in the same port: 

To serve both Ember.js and .NET Core API from the same port, you can configure IIS, Kestrel, or use an Ember proxy during development.

## Solution 1: Serve Ember from .NET Core API (Recommended for Deployment)
This approach configures .NET Core to serve Ember.js as static files.

### 1. Build Ember.js for Production
In your Ember project, run:
```
cd ember-app
ember build --environment=production
```

This generates the production files in dist/.

### 2. Copy Ember Build to .NET Core wwwroot
Move the dist/ folder to the wwwroot/ folder in your .NET Core project.

### 3. Configure .NET Core to Serve Ember.js
Modify Startup.cs:
```
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseCors("AllowSpecificOrigin");
    app.UseRouting();
    app.UseAuthorization();

    // Serve Ember.js from wwwroot
    app.UseStaticFiles();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });

    // Catch-all route for Ember.js
    app.Use(async (context, next) =>
    {
        if (context.Request.Path.Value.StartsWith("/api"))
        {
            await next();
            return;
        }
        context.Request.Path = "/index.html";
        await next();
    });
}
```

### 4. Run .NET Core API
```
dotnet run
```

Ember.js will be served from the same domain and port as the API.

## Solution 2: Use an Ember Proxy for Development

If you want to keep Ember.js and .NET Core separate during development but still run them on the same port, modify ember-cli-build.js:

```
module.exports = function (defaults) {
  let app = new EmberApp(defaults, {
    proxy: 'http://localhost:5000' // .NET Core API
  });

  return app.toTree();
};
```

Then start both:

* Run .NET Core API: dotnet run
* Run Ember.js: ember serve

This will proxy API requests from Ember.js to your .NET Core API without CORS issues.

## Final Thoughts
* Use Solution 1 for Production (serving Ember from .NET Core).
* Use Solution 2 for Development (proxy for API requests).

## PowerShell Automation script to deploy your .NET Core application to IIS using MSBuild:
```
# Define variables
$solutionPath = "C:\Path\To\YourSolution.sln"
$publishPath = "C:\Path\To\Publish"
$siteName = "YourIISSite"
$serverName = "localhost"
$physicalPath = "C:\inetpub\wwwroot\YourApp"

# Ensure MSBuild is available
$msbuildPath = "C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe"

# Clean and publish the solution
Write-Host "Building and publishing the solution..."
& $msbuildPath $solutionPath /p:DeployOnBuild=true /p:PublishProfile=FolderProfile /p:Configuration=Release /p:OutputPath=$publishPath

# Stop IIS Site
Write-Host "Stopping IIS site: $siteName..."
iisreset /stop

# Remove old files
Write-Host "Cleaning up old files..."
Remove-Item -Recurse -Force $physicalPath\*

# Copy new files
Write-Host "Deploying new files..."
Copy-Item -Recurse -Force "$publishPath\*" $physicalPath

# Start IIS Site
Write-Host "Starting IIS site: $siteName..."
iisreset /start

Write-Host "Deployment completed successfully!"
```

## {YourProject}\Properties\PublishProfiles\FolderProfile.pubxml

```
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <PublishProtocol>FileSystem</PublishProtocol>
    <Configuration>Release</Configuration>
    <Platform>Any CPU</Platform>
    <TargetFramework>net6.0</TargetFramework>
    <PublishDir>C:\Deployments\MyApp\</PublishDir>
    <DeleteExistingFiles>True</DeleteExistingFiles>
    <SelfContained>false</SelfContained>
    <EnableMSDeployBackup>false</EnableMSDeployBackup>
  </PropertyGroup>
</Project>
```

## Add Dapper.FastCRUD
```
    public class ProductService : IProductService
    {
        private readonly string _connectionString;

        public ProductService(IConfiguration configuration)
        {
            _connectionString = configuration.GetConnectionString("DefaultConnection");
        }

        public async Task<IEnumerable<Product>> GetAllAsync()
        {
            using var connection = new SqlConnection(_connectionString);
            return await connection.FindAsync<Product>();
        }

        public async Task<Product> GetByIdAsync(int id)
        {
            using var connection = new SqlConnection(_connectionString);
            return await connection.GetAsync(new Product { Id = id });
        }

        public async Task AddAsync(Product product)
        {
            using var connection = new SqlConnection(_connectionString);
            await connection.InsertAsync(product);
        }

        public async Task UpdateAsync(Product product)
        {
            using var connection = new SqlConnection(_connectionString);
            await connection.UpdateAsync(product);
        }

        public async Task DeleteAsync(int id)
        {
            using var connection = new SqlConnection(_connectionString);
            await connection.DeleteAsync(new Product { Id = id });
        }
    }
```

## Add to use the PROD, STAG, DEVELOP environment
```
public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .UseSerilog()
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            })
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var env = hostingContext.HostingEnvironment.EnvironmentName;
                config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                      .AddJsonFile($"appsettings.{env}.json", optional: true, reloadOnChange: true)
                      .AddEnvironmentVariables();
            });
```

## Add serilog and configure to use the SQL Server
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using Serilog;
using Serilog.Sinks.MSSqlServer;
using Microsoft.Extensions.Configuration;

namespace RobustApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        private readonly IProductService _productService;
        private readonly ILogger<ProductsController> _logger;

        public ProductsController(IProductService productService, ILogger<ProductsController> logger)
        {
            _productService = productService;
            _logger = logger;
        }

        [HttpGet]
        public async Task<ActionResult<IEnumerable<Product>>> GetAll()
        {
            _logger.LogInformation("Fetching all products");
            return Ok(await _productService.GetAllAsync());
        }

        [HttpGet("{id}")]
        public async Task<ActionResult<Product>> GetById(int id)
        {
            var product = await _productService.GetByIdAsync(id);
            if (product == null)
            {
                _logger.LogWarning("Product with ID {Id} not found", id);
                return NotFound();
            }
            return Ok(product);
        }

        [HttpPost]
        public async Task<ActionResult> Create(Product product)
        {
            await _productService.AddAsync(product);
            return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
        }

        [HttpPut("{id}")]
        public async Task<ActionResult> Update(int id, Product product)
        {
            if (id != product.Id)
            {
                return BadRequest();
            }
            await _productService.UpdateAsync(product);
            return NoContent();
        }

        [HttpDelete("{id}")]
        public async Task<ActionResult> Delete(int id)
        {
            await _productService.DeleteAsync(id);
            return NoContent();
        }
    }

    public interface IProductService
    {
        Task<IEnumerable<Product>> GetAllAsync();
        Task<Product> GetByIdAsync(int id);
        Task AddAsync(Product product);
        Task UpdateAsync(Product product);
        Task DeleteAsync(int id);
    }

    public class ProductService : IProductService
    {
        private readonly Queue<Product> _products = new();

        public Task<IEnumerable<Product>> GetAllAsync() => Task.FromResult(_products.AsEnumerable());

        public Task<Product> GetByIdAsync(int id) => Task.FromResult(_products.FirstOrDefault(p => p.Id == id));

        public Task AddAsync(Product product)
        {
            product.Id = _products.Count + 1;
            _products.Enqueue(product);
            return Task.CompletedTask;
        }

        public Task UpdateAsync(Product product)
        {
            var items = _products.ToList();
            var index = items.FindIndex(p => p.Id == product.Id);
            if (index >= 0)
            {
                items[index] = product;
                _products.Clear();
                foreach (var item in items)
                {
                    _products.Enqueue(item);
                }
            }
            return Task.CompletedTask;
        }

        public Task DeleteAsync(int id)
        {
            var items = _products.Where(p => p.Id != id).ToList();
            _products.Clear();
            foreach (var item in items)
            {
                _products.Enqueue(item);
            }
            return Task.CompletedTask;
        }
    }

    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }
}

public class Program
{
    public static void Main(string[] args)
    {
        var configuration = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json")
            .Build();

        Log.Logger = new LoggerConfiguration()
            .WriteTo.MSSqlServer(
                configuration.GetConnectionString("DefaultConnection"),
                sinkOptions: new MSSqlServerSinkOptions { TableName = "Logs" })
            .CreateLogger();

        try
        {
            Log.Information("Starting web application");
            CreateHostBuilder(args).Build().Run();
        }
        catch (Exception ex)
        {
            Log.Fatal(ex, "Application start-up failed");
        }
        finally
        {
            Log.CloseAndFlush();
        }
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .UseSerilog()
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

## Using Queue
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;

namespace RobustApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        private readonly IProductService _productService;
        private readonly ILogger<ProductsController> _logger;

        public ProductsController(IProductService productService, ILogger<ProductsController> logger)
        {
            _productService = productService;
            _logger = logger;
        }

        [HttpGet]
        public async Task<ActionResult<IEnumerable<Product>>> GetAll()
        {
            _logger.LogInformation("Fetching all products");
            return Ok(await _productService.GetAllAsync());
        }

        [HttpGet("{id}")]
        public async Task<ActionResult<Product>> GetById(int id)
        {
            var product = await _productService.GetByIdAsync(id);
            if (product == null)
            {
                _logger.LogWarning("Product with ID {Id} not found", id);
                return NotFound();
            }
            return Ok(product);
        }

        [HttpPost]
        public async Task<ActionResult> Create(Product product)
        {
            await _productService.AddAsync(product);
            return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
        }

        [HttpPut("{id}")]
        public async Task<ActionResult> Update(int id, Product product)
        {
            if (id != product.Id)
            {
                return BadRequest();
            }
            await _productService.UpdateAsync(product);
            return NoContent();
        }

        [HttpDelete("{id}")]
        public async Task<ActionResult> Delete(int id)
        {
            await _productService.DeleteAsync(id);
            return NoContent();
        }
    }

    public interface IProductService
    {
        Task<IEnumerable<Product>> GetAllAsync();
        Task<Product> GetByIdAsync(int id);
        Task AddAsync(Product product);
        Task UpdateAsync(Product product);
        Task DeleteAsync(int id);
    }

    public class ProductService : IProductService
    {
        private readonly Queue<Product> _products = new();

        public Task<IEnumerable<Product>> GetAllAsync() => Task.FromResult(_products.AsEnumerable());

        public Task<Product> GetByIdAsync(int id) => Task.FromResult(_products.FirstOrDefault(p => p.Id == id));

        public Task AddAsync(Product product)
        {
            product.Id = _products.Count + 1;
            _products.Enqueue(product);
            return Task.CompletedTask;
        }

        public Task UpdateAsync(Product product)
        {
            var items = _products.ToList();
            var index = items.FindIndex(p => p.Id == product.Id);
            if (index >= 0)
            {
                items[index] = product;
                _products.Clear();
                foreach (var item in items)
                {
                    _products.Enqueue(item);
                }
            }
            return Task.CompletedTask;
        }

        public Task DeleteAsync(int id)
        {
            var items = _products.Where(p => p.Id != id).ToList();
            _products.Clear();
            foreach (var item in items)
            {
                _products.Enqueue(item);
            }
            return Task.CompletedTask;
        }
    }

    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }
}
```

## Using List
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;

namespace RobustApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        private readonly IProductService _productService;
        private readonly ILogger<ProductsController> _logger;

        public ProductsController(IProductService productService, ILogger<ProductsController> logger)
        {
            _productService = productService;
            _logger = logger;
        }

        [HttpGet]
        public async Task<ActionResult<IEnumerable<Product>>> GetAll()
        {
            _logger.LogInformation("Fetching all products");
            return Ok(await _productService.GetAllAsync());
        }

        [HttpGet("{id}")]
        public async Task<ActionResult<Product>> GetById(int id)
        {
            var product = await _productService.GetByIdAsync(id);
            if (product == null)
            {
                _logger.LogWarning("Product with ID {Id} not found", id);
                return NotFound();
            }
            return Ok(product);
        }

        [HttpPost]
        public async Task<ActionResult> Create(Product product)
        {
            await _productService.AddAsync(product);
            return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
        }

        [HttpPut("{id}")]
        public async Task<ActionResult> Update(int id, Product product)
        {
            if (id != product.Id)
            {
                return BadRequest();
            }
            await _productService.UpdateAsync(product);
            return NoContent();
        }

        [HttpDelete("{id}")]
        public async Task<ActionResult> Delete(int id)
        {
            await _productService.DeleteAsync(id);
            return NoContent();
        }
    }

    public interface IProductService
    {
        Task<IEnumerable<Product>> GetAllAsync();
        Task<Product> GetByIdAsync(int id);
        Task AddAsync(Product product);
        Task UpdateAsync(Product product);
        Task DeleteAsync(int id);
    }

    public class ProductService : IProductService
    {
        private readonly List<Product> _products = new();

        public Task<IEnumerable<Product>> GetAllAsync() => Task.FromResult(_products.AsEnumerable());

        public Task<Product> GetByIdAsync(int id) => Task.FromResult(_products.FirstOrDefault(p => p.Id == id));

        public Task AddAsync(Product product)
        {
            product.Id = _products.Count + 1;
            _products.Add(product);
            return Task.CompletedTask;
        }

        public Task UpdateAsync(Product product)
        {
            var existing = _products.FirstOrDefault(p => p.Id == product.Id);
            if (existing != null)
            {
                existing.Name = product.Name;
                existing.Price = product.Price;
            }
            return Task.CompletedTask;
        }

        public Task DeleteAsync(int id)
        {
            _products.RemoveAll(p => p.Id == id);
            return Task.CompletedTask;
        }
    }

    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }
}
```
