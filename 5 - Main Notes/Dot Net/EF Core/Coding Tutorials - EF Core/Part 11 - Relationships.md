Source: https://youtu.be/eHT6G912po0?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk

Date: 26.04.2025 10:13

![[5 - Main Notes/Dot Net/EF Core/Coding Tutorials - EF Core/Part 11 - Relationships ERD.png]]

## Models 

```C#
public class Teacher
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // One-To-One
    public int? ClassroomId { get; set; }
    public Classroom Classroom { get; set; }
    
    // One-To-Many
    [ForeignKey("AuthorId")]
    public ICollection<Course> CoursesWritten { get; set; }
    [ForeignKey("EditorId")]
    public ICollection<Course> CoursesEditted { get; set; }
}
public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }
    
    // Many-To-Many
    public ICollection<Student> Students { get; set; }
    // Many-to-One
    public Teacher Author { get; set; }
    public Teacher Editor { get; set; }
}
public class Classroom
{
    public int Id { get; set; }
    public string Number { get; set; }
    // One-To-Many
    public ICollection<Student> Students { get; set; }
    // One-To-One
    public Teacher Teacher { get; set; }
}
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // [Required] - makes it non-optional Many-to-One
    [Required]
    public Classroom Classroom { get; set; }
    // Many-To-Many
    public ICollection<Course> Courses { get; set; }
}
```

## DbContext

```C#
public class SchoolDbContext : DbContext
{
    public SchoolDbContext() { }

    public SchoolDbContext(DbContextOptions<SchoolDbContext> options)
        : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
    
// Configure the One-to-Many relationship between Teacher (Author) and Course
        modelBuilder
            .Entity<Course>()
            .HasOne(c => c.Author) // A Course has one Author
            .WithMany(t => t.CoursesWritten) // A Teacher can write many Courses
            .HasForeignKey("AuthorId"); // Explicitly define the foreign key
            
// Configure the One-to-Many relationship between Teacher (Editor) and Course
        modelBuilder
            .Entity<Course>()
            .HasOne(c => c.Editor) // A Course has one Editor
            .WithMany(t => t.CoursesEditted) // A Teacher can edit many Courses
            .HasForeignKey("EditorId"); // Explicitly define the foreign key

// Configure the One-to-Many relationship between Classroom and Student
// EF Core's conventions can often handle this, but explicit configuration provides clarity.
        modelBuilder
            .Entity<Student>()
            .HasOne(s => s.Classroom)    // A Student has one Classroom
            .WithMany(c => c.Students)   // A Classroom can have many Students
            .IsRequired(); // Enforces the [Required] attribute on Student.Classroom

// Configure the Many-to-Many relationship between Student and Course
// EF Core's conventions handle this automatically with ICollection on both sides,
// but you could configure a join table explicitly if needed (not shown here as it's typically handled by convention).

// Configure the One-to-One relationship between Classroom and Teacher
        modelBuilder
            .Entity<Classroom>()
            .HasOne(c => c.Teacher)      // A Classroom has one Teacher
            .WithOne(t => t.Classroom)  // A Teacher has one Classroom
            .HasForeignKey<Teacher>(t => t.ClassroomId) // The foreign key is in the Teacher table
            .IsRequired(false); // The ClassroomId in Teacher is nullable
    }

    public DbSet<Classroom> Classrooms { get; set; }
    public DbSet<Course> Courses { get; set; }
    public DbSet<Student> Students { get; set; }
    public DbSet<Teacher> Teachers { get; set; }
}
```

## Program.cs
```C#
class Program
{
    static void CreateData(TrainingContext db)
    {
        Teacher t1 = new Teacher { Name = "Miss Anderson" };
        Teacher t2 = new Teacher { Name = "Miss Bingham" };
        
        Classroom r1 = new Classroom { Number = "R101" };
        Classroom r2 = new Classroom { Number = "R202" };
        Course c1 = new Course { Title = "Introduction to EF Core", Author = t1, Editor = t2 };
        Course c2 = new Course { Title = "Basic Car Maintenance", Author = t2, Editor = t1 };
        
        Student s1 = new Student { Name = "Jenny Jones", Classroom = r1 };
        Student s2 = new Student { Name = "Kenny Kent", Classroom = r1 };
        Student s3 = new Student { Name = "Lucy Locket", Classroom = r1 };
        Student s4 = new Student { Name = "Micky Most", Classroom = r2 };
        Student s5 = new Student { Name = "Nelly Norton", Classroom = r2 };
        Student s6 = new Student { Name = "Ozzy Osborne", Classroom = r2 };

        c1.Students = new Student[] { s1, s2, s3, s4 };
        c2.Students = new Student[] { s3, s4, s5, s6 };

        db.Add(c1);
        db.Add(c2);
        db.SaveChanges();
    }
  
    static void Main()
    {
        IConfiguration config = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json")
            .Build();

        var options = new DbContextOptionsBuilder<TrainingContext>()
            .UseSqlServer(config.GetConnectionString("TrainingDB"))
            .Options;

        using var db = new TrainingContext(options);

        db.Database.EnsureDeleted();
        db.Database.EnsureCreated();
        CreateData(db);
    }
}
```