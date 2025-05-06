
## 1. What the console might tell us:

```Bash
Unhandled exception. System.InvalidOperationException: 

The dependent side could not be determined for the one-to-one relationship between 'Classroom.Teacher' and 'Teacher.Classroom'. 

To identify the dependent side of the relationship, configure the foreign key property. 

If these navigations should not be part of the same relationship, configure them independently via separate method chains in 'OnModelCreating'. 

See https://go.microsoft.com/fwlink/?LinkId=724062 for more details.
```

## 2. What does it mean on a simple example:

Either `Teacher` or `Student` has to have `{EntityName}Id` property to configure the foreign key.

The entity that contains the foreign key property is typically considered the **dependent entity**, and the entity that the foreign key refers to is the **principal entity**.

In this case: a `Teacher` depends on `Classroom` entity.

```C#
public class Classroom
{
    public int Id { get; set; }
    public string Number { get; set; } = string.Empty;
    
    public ICollection<Student> Students { get; set; } = [];

    [Required]
    public Teacher Teacher { get; set; } = default!;
}

public class Teacher
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;

    // Missing field:
    //public int? ClassroomId { get; set; }
    
    [Required]
    public Classroom Classroom { get; set; } = default!;
}
```