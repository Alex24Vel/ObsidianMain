Source: https://youtu.be/0ndu7Zhc84Q?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk

Date: 26.04.2025 09:40

![[Part 12 - Inheritance - ERD.png]]

## Models 
```C#
public abstract class Pet

{

    public int Id { get; set; }
    public string Name { get; set; }
    
    public override string ToString()
    {
        return $"My name is {Name}";
    }
}

public class Dog : Pet
{
    public string FavouriteBone { get; set; }

    public override string ToString()
    {
        return $"{base.ToString()} and I like {FavouriteBone} bones.";
    }
}

  

public class Cat : Pet
{
    public int Lives { get; set; }

    public override string ToString()
    {
        return $"{base.ToString()} and I have {Lives} lives left.";
    }
}

public class Owner
{
    public int Id { get; set; }
    public ICollection<Pet> Pets { get; set; }
  
    public override string ToString()
    {
        StringBuilder bob = new StringBuilder();
        foreach (var p in Pets)
            bob.AppendLine(p.ToString());
        return bob.ToString();
    }
}
```

## DbContext Class: TPH
**TPH - Table Per Hierarchy (Single table)** - Generally the most performant way to go.

```C#
public class PetsContext 
{
	//...
	public DbSet<Owner> Owners { get; set; }
	public DbSet<Pet> Pets { get; set; }
	public DbSet<Dog> Dogs { get; set; }
	public DbSet<Cat> Cats { get; set; }
}
```

**Tables created**: **Owners**(Id), **Pets**(Name, Discriminator (Dog/Cat), OwnerId, Lives, FavoriteBone)

## DbContext Class: CTI
**CTI - Class Table Inheritance (Table per Type)** - 

```C#
public class PetsContext 
{
	//...
	protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Dog>().ToTable("Dogs");
	    modelBuilder.Entity<Cat>().ToTable("Cats");
    }

	public DbSet<Owner> Owners { get; set; }
    public DbSet<Pet> Pets { get; set; }
	public DbSet<Dog> Dogs { get; set; }
    public DbSet<Cat> Cats { get; set; }
}
```
**Tables produced**: **Owners**(Id), **Pets**(Id, Name, OwnerId), **Dogs**(Id, FavoriteBone), **Cats** (Id, Lives)

