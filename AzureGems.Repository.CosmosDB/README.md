# AzureGems Repository - Cosmos DB

`CosmosDbContainerRepository<T>` provides a Cosmos DB specific implementation of the AzureGems.Repository `IRepository<T>` interface.

The extension method `AddCosmosDbContext<DbContext>()` allows you to define a DbContext (just like EFCore) and have it added to the `ServiceCollection` and injected into your services.

## Getting Started

Defining your `DbContext` is straight forward.

1. Create a new class that inherits from `DbContext`.
2. Declare a public `IRepository<T>` property for each model type (aka Domain Entity) with both `get` and `set` accessors.

```csharp
public class LittleNorthwindDbContext : DbContext
{
    public IRepository<Customer> Customers { get; set; }
    public IRepository<Order> Orders { get; set; }
    public IRepository<Invoice> Invoices { get; set; }
}
```

> NOTE: Each model type must inherit from `BaseType` to satisfy the generic constraints for `IRepository<T>`

3. Register your `DbContext` with the `ServiceCollection` which will take care of instantiating it and initializing all the repositories to be backed by the correct Cosmos DB containers.

> In `Startup.cs` you should already have added Cosmos DB and specified your Container Configurations. See [Adding AzureGems.CosmosDb](https://github.com/william-liebenberg/AzureGems/tree/master/AzureGems.CosmosDB#adding-azuregemscosmosdb)

4. Register your DbContext using the `AddCosmosDbContext()` extension method:

```csharp
services.AddCosmosDbContext<LittleNorthwindDbContext>();
```

Now your DbContext is ready to be injected and used by your controllers/services.

## Example Usage

```csharp
public class LittleNorthwindService
{
	private readonly LittleNorthwindDbContext _dbContext;

	public LittleNorthwindService(LittleNorthwindDbContext dbContext)
	{
		_dbContext = dbContext;
	}

	public async Task<Customer> AddCustomer(string firstName, string lastName, string emailAddress)
	{
		return await _dbContext.Customers.Add(
			new Customer()
			{
				FirstName = firstName,
				LastName = lastName,
				Email = emailAddress,
			});
	}

	public async Task<Customer> FindCustomer(string firstname, string lastname)
	{
		return _dbContext.Customers.Query(q => q
			.Where(c => c.FirstName == firstname && c.LastName == lastname)
			.OrderBy(c => c.FirstName))
			.FirstOrDefault();
	}

	public async Task<IEnumerable<Customer>> GetValuableCustomers(decimal dollarThreshold)
	{
		return _dbContext.Customers.Query(q => q
			.Where(c => c.TotalSpend > dollarThreshold)
			.OrderBy(c => c.TotalSpend));
	}

	// ... you get the idea ... //
	
}
```
