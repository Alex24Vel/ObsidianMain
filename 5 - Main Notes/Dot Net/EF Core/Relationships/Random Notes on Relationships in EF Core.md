
Date: 27.04.2025 14:41

## DeleteBehavior

- DeleteBehavior.SetNull: This configures the database constraint. When the principal entity (Theatre) is deleted, the database itself finds dependent rows (Appointments) and sets their foreign key (TheatreId) to NULL.

- Requirement: The foreign key column in the database (Appointments.TheatreId) must be nullable.

- How: EF Core creates a migration that adds the ON DELETE SET NULL constraint to the foreign key definition in the database schema.

- DeleteBehavior.ClientSetNull: This is primarily an EF Core client-side behavior and is the default for optional relationships if no specific database action is configured. 
- When you delete a principal entity (Theatre), EF Core looks at tracked dependent entities (Appointments) in its context, sets their foreign key property (Appointment.TheatreId) to null, and then sends the DELETE command for the Gymnasium to the database.

- Limitation: It only affects entities currently tracked by the DbContext instance. It doesn't configure the database to automatically set nulls for all related rows.
- Requirement: The foreign key property in the entity (Appointment.TheatreId) must be nullable (int?) for EF Core to set it to null.
