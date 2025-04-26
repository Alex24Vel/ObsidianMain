Source: https://youtu.be/6145Q1juVHI?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk

Date: 26.04.2025 19:20

## Do not use Constructors to map Entities to Models
Timecode: [8:53 - 15:00](https://youtu.be/6145Q1juVHI?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk)

```C#
var books = await _context.Books
		.Include(b => b.Author)
		.Select(b => new BookModel(b))
		.ToListAsync();
```
EF takes unnecessary data when using constructor instead of property initializer.

The Author entity has a `DateOfBirth` property which the `BookModel` doesn't use.
To avoid fetching obsolete data -> use property initializers.
Using `.Include(b => b.Author)` becomes optional now -> can be removed.

```C#
var books = await _context.Books
		.Select(b => new BookModel
		{
			Id = b.Id,
			Title = b.Title,
			YearOfPublication = b.YearOfPublication,
			Author = b.Author.Name
		})
		.ToListAsync();
```

**The problem it introduces: Code Duplication** - How to avoid it? Let's take a better look:

1. Define `public static class {ModelClassName}Extensions`
2. Define `public static IQueryable<BookModel> ToModel(this IQueryable<Book> source)`.
3. The defined method should look like this:
```C#
public static class BookModelExtensions
{
	public static IQueryable<BookModel> ToModel(this IQueryable<Book> source)
	{
		return source.Select(b => new BookModel
		{
			Id = b.Id,
			Title = b.Title,
			YearOfPublication = b.YearOfPublication,
			Author = b.Author.Name
		});
	}
}
```

**In the Service, Controller or elsewhere the previous `var books = ...` will look like this:**
```C#
var books = await _context.Books.ToModel().ToListAsync();
```

**Why not use `AutoMapper` ?** - less overhead with configurating `AutoMapper` properly so it does exactly what you expect it to do.
