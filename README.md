# Robust CSharp CRUD

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
