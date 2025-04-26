Source: https://youtu.be/Fo8VEtIjA5I?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk

Date: 26.04.2025 11:13

Relevant: [[MySqlException (0x80004005) Host 'x' is not allowed to connect to this MySQL server]] 

## Models

```C#
public class Author
{
    public int Id { get; set; }
    public string Name { get; set; } = null!;

    // One-to-Many
    public ICollection<Book> Books { get; set; } = null!;

    public override string ToString()
    {
        return Name;
    }
}
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; } = null!;
    public int YearOfPublication { get; set; }
    
    // Many-to-One
    public Author Author { get; set; } = null!;

    public override string ToString()
    {
        return $"{Title} ({YearOfPublication})";
    }
}
```
## DbContext
```C#
public class CodeFirstDbContext : DbContext
{
    public CodeFirstDbContext(DbContextOptions<CodeFirstDbContext> options)
        : base(options)
    {}
    public DbSet<Author> Authors { get; set; }
    public DbSet<Book> Books { get; set; }
}
```
## Tables produced:

**Authors:**
```SQL
CREATE TABLE `authors` (
  `Id` int NOT NULL AUTO_INCREMENT,
  `Name` longtext CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  PRIMARY KEY (`Id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
**Books:**

```SQL
CREATE TABLE `books` (
  `Id` int NOT NULL AUTO_INCREMENT,
  `Title` longtext CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `YearOfPublication` int NOT NULL,
  `AuthorId` int NOT NULL,
  PRIMARY KEY (`Id`),
  KEY `IX_Books_AuthorId` (`AuthorId`),
  CONSTRAINT `FK_Books_Authors_AuthorId` FOREIGN KEY (`AuthorId`) REFERENCES `authors` (`Id`) ON DELETE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
## Program.cs
```C#
public class Program
{
    static List<Author> CreateFakeData()
    {
        var authors = new List<Author>
        {
            new ()
            {
                Name = "Jane Austen", Books =
                [
                    new Book {Title = "Emma", YearOfPublication = 1815},
                    new Book {Title = "Persuasion", YearOfPublication = 1818},
                    new Book {Title = "Mansfield Park", YearOfPublication = 1814}
                ]
            },
            new() {
                Name = "Ian Fleming", Books =
                [
                    new Book {Title = "Dr No", YearOfPublication = 1958},
                    new Book {Title = "Goldfinger", YearOfPublication = 1959},
                    new Book {Title = "From Russia with Love", YearOfPublication = 1957}
                ]
            }
        };

        return authors;
    }

    static void Main()
    {
        string connectionString = "Server=localhost;Database=yt_coding_tutorials_ef_core_p1;Username=root;Password=root;";

        var serverVersion = ServerVersion.AutoDetect(connectionString);

        var options = new DbContextOptionsBuilder<CodeFirstDbContext>()
            .UseMySql(connectionString, serverVersion)
            .Options;

        using var db = new CodeFirstDbContext(options);

        db.Database.EnsureCreated();

        var authors = CreateFakeData();

        db.Authors.AddRange(authors);

        db.SaveChanges();

        var recentBooks = from b in db.Books where b.YearOfPublication > 1900 select b;

//Refer to: [[Part 4 - Eager vs Lazy Loading]]
// .Incude - prevent N+1 Query error. Now if Author had 1000 books it won't make 1001 database calls (1 for author, 1000 for books) -> it will execute one join query instead.
        foreach (var book in recentBooks.Include(b => b.Author))
        {
            Console.WriteLine($"{book} is by {book.Author}");
        }
    }
}
```

Relevant: [[Part 4 - Eager vs Lazy Loading]]