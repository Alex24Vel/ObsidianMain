Source: https://youtu.be/b09PnRWzWds?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk

Date: 26.04.2025 16:59

## Configuring DbContext

### Option 1: Standard DI in Program.cs

**Program.cs**
```C#
 builder.Services.AddDbContextFactory<EagerVsLazyLoadingDbContext>(options =>
 {
     var connectionString = builder.Configuration.GetConnectionString("EagerVsLazyLoadingDb");

     if (string.IsNullOrEmpty(connectionString))
     {
         throw new InvalidOperationException("Connection string 'EagerVsLazyLoadingDb' not found.");
     }

     var serverVersion = ServerVersion.AutoDetect(connectionString);

     options.UseMySql(connectionString, serverVersion)
            .EnableSensitiveDataLogging(builder.Environment.IsDevelopment())
            .EnableDetailedErrors();
 });

// --- Ensure the Database gets created automatically ---

// EnsureCreated() should ideally not be used for production scenarios with migrations.
using (var scope = app.Services.CreateScope())
{
	var dbContextFactory = scope
		.ServiceProvider
		.GetRequiredService<IDbContextFactory<EagerVsLazyLoadingDbContext>>();
	
	using var dbContext = dbContextFactory.CreateDbContext();
	dbContext.Database.EnsureCreated();
}
// --- END Ensure the Database gets created automatically ---
```

### Option 2: Overriding OnConfiguring in DbContext class

**Not recommended** 

**Program.cs**
```C#
builder.Services.AddDbContextFactory<EagerVsLazyLoadingDbContext>(); 
```

**AppDbContext.cs**
```C#
public class EagerVsLazyLoadingDbContext : DbContext
{
	public EagerVsLazyLoadingDbContext
		(DbContextOptions<EagerVsLazyLoadingDbContext> options) : base(options) { }
	
	protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
	    if (!optionsBuilder.IsConfigured)
        {
	        IConfiguration config = new ConfigurationBuilder()
#if DEBUG
	            .AddJsonFile("appsettings.Development.json")
#else
                .AddJsonFile("appsettings.json")
#endif
                .Build();

            string connectionString = config
	            .GetConnectionString("EagerVsLazyLoadingDb")!;

			var serverVersion = ServerVersion.AutoDetect(connectionString);
			
	        optionsBuilder.UseMySql(connectionString, serverVersion)
                .EnableSensitiveDataLogging()
			    .EnableDetailedErrors();
            }
        }
	}
	
	public DbSet<Author> Authors { get; set; }
    public DbSet<Book> Books { get; set; }
}
```

## Setting up MSTest Environment

**See the related Github Repository**