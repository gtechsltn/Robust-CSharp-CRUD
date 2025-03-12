# Robust CSharp CRUD
* Windows Server 2022
* Web Server (IIS) 10.0
* .NET Core Web API 9.0
* Ember.js
* SQL Server 2022
* EF Core 9.0
* Dapper
* Dapper.FastCRUD
* Serilog
* Development, Testing, Staging, and Production Environments:
    * Development Environment (Dev)
    * Test Environment (Test)
    * Staging Environment (Staging)
    * Production Environment (Production)
* Development, Testing, Staging, and Production Environments
    * https://learn.microsoft.com/en-us/biztalk/technical-guides/planning-the-development-testing-staging-and-production-environments
    * https://www.linkedin.com/pulse/guide-dev-test-staging-production-environments-george-lebbos/

# Integrate Ember.js with your .NET Core API

## Steps to Integrate Ember.js

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

## Add to support large file upload and download in your API, you need to handle streaming efficiently to avoid memory overload.

Here's how you can modify your project:

### 1. Add File Upload Endpoint

Modify your ProductsController to support file uploads:

```
[HttpPost("upload")]
public async Task<IActionResult> UploadFile(IFormFile file)
{
    if (file == null || file.Length == 0)
        return BadRequest("No file uploaded.");

    var filePath = Path.Combine("Uploads", file.FileName);

    using (var stream = new FileStream(filePath, FileMode.Create))
    {
        await file.CopyToAsync(stream);
    }

    return Ok(new { FileName = file.FileName, FilePath = filePath });
}
```

**Key Points:**

* Uses IFormFile to receive uploaded files.
* Saves them to an Uploads folder.

### 2. Add Large File Download Endpoint

Modify your ProductsController to support file downloads:

```
[HttpGet("download/{fileName}")]
public IActionResult DownloadFile(string fileName)
{
    var filePath = Path.Combine("Uploads", fileName);

    if (!System.IO.File.Exists(filePath))
        return NotFound();

    var stream = new FileStream(filePath, FileMode.Open, FileAccess.Read);
    return File(stream, "application/octet-stream", fileName);
}
```

**Key Points:**

* Uses a FileStream to stream files instead of loading them fully into memory.
* Returns the file with application/octet-stream for generic downloads.

### 3. Configure IIS & Kestrel for Large File Uploads

By default, ASP.NET Core limits file uploads to 30MB. You can increase this in appsettings.json:
```
{
  "Kestrel": {
    "Limits": {
      "MaxRequestBodySize": 1073741824  // 1GB
    }
  }
}
```

**For IIS, modify web.config:**
```
<configuration>
  <system.webServer>
    <security>
      <requestFiltering>
        <requestLimits maxAllowedContentLength="2147483648" /> <!-- 2GB -->
      </requestFiltering>
    </security>
  </system.webServer>
</configuration>
```

### 4. Enable Form Handling in Startup.cs

Ensure that large file handling is enabled:

```
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<FormOptions>(options =>
    {
        options.MultipartBodyLengthLimit = 1073741824; // 1GB
    });

    services.AddControllers();
}
```

**Now your API supports large file uploads and downloads! 🚀 Let me know if you need further refinements!**

## To configure machine.config to support 4GB file uploads, follow these steps:

### 1. Modify machine.config (For IIS Hosting)

The machine.config file is typically located in:

* For 32-bit systems: C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\
* For 64-bit systems: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\

Inside machine.config, modify or add the <system.web> section:
```
<configuration>
  <system.web>
    <httpRuntime maxRequestLength="4194304" executionTimeout="3600" />
  </system.web>
</configuration>
```

* maxRequestLength="4194304" → Allows 4GB file uploads.
* executionTimeout="3600" → Increases timeout to 1 hour for large files.

### 2. Modify web.config (For Application-Level Settings)

If you're using IIS with your .NET Core app, you should also modify web.config:
```
<configuration>
  <system.webServer>
    <security>
      <requestFiltering>
        <requestLimits maxAllowedContentLength="4294967295" /> <!-- 4GB -->
      </requestFiltering>
    </security>
  </system.webServer>
</configuration>
```

* maxAllowedContentLength="4294967295" → Allows up to 4GB file uploads.

### 3. Modify appsettings.json (For Kestrel)

If your API is running directly with Kestrel, update appsettings.json:
```
{
  "Kestrel": {
    "Limits": {
      "MaxRequestBodySize": 4294967295
    }
  }
}
```

### 4. Modify Startup.cs

Ensure the application is configured to allow large file uploads:
```
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<FormOptions>(options =>
    {
        options.MultipartBodyLengthLimit = 4294967295; // 4GB
    });

    services.AddControllers();
}
```

### Final Notes
* ✅ If hosting on IIS, machine.config + web.config settings must be updated.
* ✅ If using Kestrel, modify appsettings.json and Startup.cs.
* ✅ For Azure or Cloud Deployments, verify file size limits in storage configurations.

**Let me know if you need any further modifications! 🚀**

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

## Add to use the environment variables
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

## Apply migrations at runtime
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
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;

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
        private readonly ApplicationDbContext _context;

        public ProductService(ApplicationDbContext context)
        {
            _context = context;
        }

        public async Task<IEnumerable<Product>> GetAllAsync()
        {
            return await _context.Products.ToListAsync();
        }

        public async Task<Product> GetByIdAsync(int id)
        {
            return await _context.Products.FindAsync(id);
        }

        public async Task AddAsync(Product product)
        {
            _context.Products.Add(product);
            await _context.SaveChangesAsync();
        }

        public async Task UpdateAsync(Product product)
        {
            _context.Products.Update(product);
            await _context.SaveChangesAsync();
        }

        public async Task DeleteAsync(int id)
        {
            var product = await _context.Products.FindAsync(id);
            if (product != null)
            {
                _context.Products.Remove(product);
                await _context.SaveChangesAsync();
            }
        }
    }

    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }

    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) { }

        public DbSet<Product> Products { get; set; }
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

        var host = CreateHostBuilder(args).Build();

        using (var scope = host.Services.CreateScope())
        {
            var services = scope.ServiceProvider;
            var context = services.GetRequiredService<ApplicationDbContext>();
            context.Database.Migrate(); // Apply migrations at runtime
        }

        try
        {
            Log.Information("Starting web application");
            host.Run();
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
            })
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var env = hostingContext.HostingEnvironment.EnvironmentName;
                config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                      .AddJsonFile($"appsettings.{env}.json", optional: true, reloadOnChange: true)
                      .AddEnvironmentVariables();
            });
}
```

## Add the EF Core 9 (EF9) or Microsoft.EntityFrameworkCore 9.0 into this solution

https://codewithmukesh.com/blog/aspnet-core-webapi-crud-with-entity-framework-core-full-course/

```
public class Startup
{
    public IConfiguration Configuration { get; }

    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

        services.AddScoped<IProductService, ProductService>();

        services.AddControllers();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}
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
