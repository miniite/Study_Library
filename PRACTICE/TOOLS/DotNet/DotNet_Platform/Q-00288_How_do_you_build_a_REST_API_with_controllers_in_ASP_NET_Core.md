---
type: "practice"
difficulty: "Basic"
---

# How do you build a REST API with controllers in ASP.NET Core and map routes to actions?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Web APIs and Auth](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Web_APIs_and_Auth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a REST API using ASP.NET Core, we start by defining our application's structure and then mapping HTTP requests to specific controller actions. The key steps involve creating controllers, defining action methods with appropriate HTTP verbs (GET, POST, PUT, DELETE), and configuring routes in the Startup class. Let’s break this down from first principles.

First, you need a basic understanding that ASP.NET Core is a framework for building web applications using C#. It provides a robust foundation to build both client-server and microservices architectures. For a REST API, we focus on defining controllers which are responsible for handling HTTP requests and returning responses.

#### Triage

The question asks us to explain how to set up a REST API with controllers in ASP.NET Core, specifically detailing the process of mapping routes to actions. The interviewer is probing your knowledge of the basic structure of an ASP.NET Core application, understanding of HTTP verbs, and familiarity with routing mechanisms.

Step-by-Step Solution
Project Setup: Start by creating a new ASP.NET Core Web API project using Visual Studio or the .NET CLI.
Controllers: Define controllers to handle different operations. Each controller can have multiple action methods corresponding to different HTTP requests (GET, POST, PUT, DELETE).
Action Methods: Write action methods within these controllers that perform the required logic and return appropriate responses.
Routing Configuration: Configure routes in the Startup class or use attribute routing directly on your actions.

#### Example

Let’s walk through an example where we build a simple REST API to manage a list of products:

Create Controllers

CSHARP
// ProductsController.cs
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<Product>>> GetProducts()
    {
        return await _productService.GetProductsAsync();
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetProduct(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        if (product == null)
            return NotFound();

        return product;
    }

    [HttpPost]
    public async Task<ActionResult<Product>> CreateProduct([FromBody] Product product)
    {
        await _productService.CreateProductAsync(product);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateProduct(int id, [FromBody] Product product)
    {
        if (id != product.Id) return BadRequest();

        await _productService.UpdateProductAsync(product);
        return NoContent();
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        await _productService.DeleteProductAsync(id);
        return NoContent();
    }
}

Configure Services in Startup

CSHARP
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddScoped<IProductService, ProductService>();
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

Define Product Service

CSHARP
// ProductService.cs
public class ProductService : IProductService
{
    private readonly List<Product> _products = new List<Product>
    {
        new Product { Id = 1, Name = "Laptop", Price = 999.00M },
        new Product { Id = 2, Name = "Monitor", Price = 249.50M }
    };

    public async Task<IEnumerable<Product>> GetProductsAsync()
    {
        return await Task.FromResult(_products);
    }

    public async Task<Product> GetProductByIdAsync(int id)
    {
        return _products.FirstOrDefault(p => p.Id == id);
    }

    public async Task CreateProductAsync(Product product)
    {
        var existing = _products.FirstOrDefault(p => p.Id == product.Id);
        if (existing != null) throw new InvalidOperationException("Product already exists");

        _products.Add(product);
        await Task.CompletedTask;
    }

    public async Task UpdateProductAsync(Product product)
    {
        var existing = _products.FirstOrDefault(p => p.Id == product.Id);
        if (existing == null) throw new InvalidOperationException("Product does not exist");

        existing.Name = product.Name;
        existing.Price = product.Price;
        await Task.CompletedTask;
    }

    public async Task DeleteProductAsync(int id)
    {
        var index = _products.FindIndex(p => p.Id == id);
        if (index == -1) throw new InvalidOperationException("Product does not exist");

        _products.RemoveAt(index);
        await Task.CompletedTask;
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, building a REST API with controllers in ASP.NET Core involves creating controllers to handle HTTP requests, defining action methods for various operations, and configuring routes using either attribute routing or Startup configuration. Common mistakes include not handling exceptions properly, failing to correctly implement CRUD operations, and misconfiguring routes.

For follow-ups, you might be asked about security considerations (e.g., authentication and authorization), performance optimizations, or integration with other services like databases. Always ensure you cover these aspects in your answers.

**Comparison:**

| Action Method | HTTP Verb | Route Template          | Description                           |
|---------------|-----------|------------------------|---------------------------------------|
| GetProducts   | GET       | api/products           | Returns a list of all products        |
| GetProduct    | GET       | api/products/{id}      | Returns a specific product by ID      |
| CreateProduct | POST      | api/products           | Creates a new product                  |
| UpdateProduct | PUT       | api/products/{id}      | Updates an existing product           |
| DeleteProduct | DELETE    | api/products/{id}      | Deletes a product                      |

**Code:**

```

[ApiController] [Route("api/[controller]")] public class ProductsController : ControllerBase { private readonly IProductService _productService;

CODE
Copy
public ProductsController(IProductService productService)
{
    _productService = productService;
}

[HttpGet]
public async Task<ActionResult<IEnumerable<Product>>> GetProducts()
{
    return await _productService.GetProductsAsync();
}

[HttpGet("{id}")]
public async Task<ActionResult<Product>> GetProduct(int id)
{
    var product = await _productService.GetProductByIdAsync(id);
    if (product == null)
        return NotFound();

    return product;
}

[HttpPost]
public async Task<ActionResult<Product>> CreateProduct([FromBody] Product product)
{
    await _productService.CreateProductAsync(product);
    return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
}

[HttpPut("{id}")]
public async Task<IActionResult> UpdateProduct(int id, [FromBody] Product product)
{
    if (id != product.Id) return BadRequest();

    await _productService.UpdateProductAsync(product);
    return NoContent();
}

[HttpDelete("{id}")]
public async Task<IActionResult> DeleteProduct(int id)
{
    await _productService.DeleteProductAsync(id);
    return NoContent();
}

}

CODE
Copy

```csharp
public class ProductService : IProductService
{
    private readonly List<Product> _products = new List<Product>
    {
        new Product { Id = 1, Name = "Laptop", Price = 999.00M },
        new Product { Id = 2, Name = "Monitor", Price = 249.50M }
    };

    public async Task<IEnumerable<Product>> GetProductsAsync()
    {
        return await Task.FromResult(_products);
    }

    public async Task<Product> GetProductByIdAsync(int id)
    {
        return _products.FirstOrDefault(p => p.Id == id);
    }

    public async Task CreateProductAsync(Product product)
    {
        var existing = _products.FirstOrDefault(p => p.Id == product.Id);
        if (existing != null) throw new InvalidOperationException("Product already exists");

        _products.Add(product);
        await Task.CompletedTask;
    }

    public async Task UpdateProductAsync(Product product)
    {
        var existing = _products.FirstOrDefault(p => p.Id == product.Id);
        if (existing == null) throw new InvalidOperationException("Product does not exist");

        existing.Name = product.Name;
        existing.Price = product.Price;
        await Task.CompletedTask;
    }

    public async Task DeleteProductAsync(int id)
    {
        var index = _products.FindIndex(p => p.Id == id);
        if (index == -1) throw new InvalidOperationException("Product does not exist");

        _products.RemoveAt(index);
        await Task.CompletedTask;
    }
}
```

**Example input / output:**

```

GET /api/products: Returns a list of all products.

HTTP
Copy
GET http://localhost:5000/api/products

Response:

JSON
Copy
[
  { "id": 1, "name": "Laptop", "price": 999.00 },
  { "id": 2, "name": "Monitor", "price": 249.50 }
]

GET /api/products/1: Returns a specific product by ID.

HTTP
Copy
GET http://localhost:5000/api/products/1

Response:

JSON
Copy
{ "id": 1, "name": "Laptop", "price": 999.00 }

POST /api/products: Creates a new product.

HTTP
Copy
POST http://localhost:5000/api/products
Content-Type: application/json

{
  "id": 3,
  "name": "Keyboard",
  "price": 99.99
}

Response:

JSON
Copy
HTTP/1.1 201 Created
Location: http://localhost:5000/api/products/3

PUT /api/products/1: Updates an existing product.

HTTP
Copy
PUT http://localhost:5000/api/products/1
Content-Type: application/json

{
  "id": 1,
  "name": "Laptop",
  "price": 999.50
}

Response:

JSON
Copy
HTTP/1.1 204 No Content

DELETE /api/products/1: Deletes a product.

HTTP
Copy
DELETE http://localhost:5000/api/products/1

Response:

JSON
Copy
HTTP/1.1 204 No Content
```

**Data:**

```json
{
  "id": 1,
  "name": "Laptop",
  "price": 999.00
}
```

**Flow:**

1. Valid
2. Invalid
3. Yes
4. No
5. Request
6. Is the request valid?
7. Map to appropriate actionmethod
8. Handle invalid requests (e.g.,400 Bad Request)
9. Execute business logic
10. Are there any errors?
11. Log error, return 500 InternalServer Error
12. Return appropriate response

