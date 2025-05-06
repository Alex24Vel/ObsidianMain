Source: https://youtu.be/YuhMGeh_Riw?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk

Date: 26.04.2025 17:50

## After adding/deleting a property:
```Bash
Add-Migration "MigrationName" -OutputDir Database\Migrations
Update-Database
```
## Save changes to a Database:
```Bash
Update-Database
```

## Rollback changes:
```Bash
Update-Database "PREVIOUS MigrationName"
```
*Use `Remove-Migration` if `Update-Database` was not yet executed to rollback mirgation.*


## Migrating new property name (IMPORTANT):
Timecode: [12:50](https://youtu.be/YuhMGeh_Riw?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk&t=774)

Use `migrationBuilder.RenameColumn(columnName, tableName, newColumnName)`;
