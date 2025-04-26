Source: https://youtu.be/5OPXNQu2glQ?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk

Date: 26.04.2025 16:23

**Reference the example code in the repo.**

## Summary: Avoid lazy loading.

If a single query gets so big that it would be more performant to issue a couple of smaller ones -> use explicit loading.

```C#
var books = await _context.Books.ToListAsync();
    foreach (var b in books)
        {
            _context.Entry(b).Reference(book => book.Author).Load();
        }
```

`.Entry(object)` - wraps any entity object to make it clear that it's actually connected to a database.
`.Reference()` - an alternative to eager loading way: `.Includ  e()` -> means that another query will be issues, but it does it explicitly.
`.Collection()` - same as `.Reference()` but for collections.

**Explicit Loading is just as inefficient as Lazy Loading** - also issues N+1 queries. 
Why use explicit loading at all? - you know what's going on -> less chance of an unexpected error.


