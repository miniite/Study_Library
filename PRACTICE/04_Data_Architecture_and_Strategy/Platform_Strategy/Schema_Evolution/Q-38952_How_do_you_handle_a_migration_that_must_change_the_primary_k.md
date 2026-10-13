---
type: "practice"
difficulty: "Expert"
---

# How do you handle a migration that must change the primary key type (e.g. int to bigint, or bigint to UUID) on a table with dozens of inbound foreign keys and terabytes of data?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrations with Read Your Writes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When facing a migration that necessitates changing the primary key type from int to bigint, or vice versa, in a database with numerous foreign keys and substantial data volume, it's crucial to plan meticulously and implement a robust strategy. The core idea is to minimize downtime and ensure data integrity throughout the process.

#### Triaging the Question

The question asks how to migrate primary key types on a table with many inbound foreign keys and large amounts of data. This implies that the migration involves complex interdependencies, high risk due to potential data loss or corruption, and significant performance considerations given the massive dataset size. The interviewer is probing your ability to handle multi-step migrations, manage dependencies, ensure data consistency, and minimize disruption.

#### Assumptions
The database system supports transactions.
Foreign keys are defined in a way that they refer to the primary key of another table.
There’s sufficient storage and computational resources for the migration process.
The database is normalized with well-defined foreign key relationships.

#### First-Principles Solution
Step 1: Data Backup

The first step is to create a full backup of the current database. This ensures that you can revert any changes if something goes wrong during the migration, minimizing risk and downtime.

SQL
-- Example SQL for backing up a PostgreSQL database
pg_dump -U username -F c -b -v --data-only dbname > db_backup.sql
Step 2: Schema Design Consideration

You need to design a new schema that accommodates both the old and new primary key types. This can be achieved by introducing a new column with the desired type as an additional key, then gradually transferring data.

SQL
-- Example SQL for adding a new column in PostgreSQL
ALTER TABLE table_name ADD COLUMN new_column_name new_key_type;
Step 3: Data Migration

Using a batch or bulk update approach to transfer the primary keys from int/bigint to the new type. This step can be performed using a script that iterates over the data and updates it incrementally.

SQL
-- Example SQL for updating rows in PostgreSQL
UPDATE table_name SET new_column_name = generate_new_key(old_column_name);
Step 4: Foreign Key Adjustment

Once the new keys are populated, update all foreign key references to point to the new column. Ensure that any existing constraints and triggers are adjusted accordingly.

SQL
-- Example SQL for updating foreign key relationships in PostgreSQL
ALTER TABLE referencing_table DROP CONSTRAINT fk_constraint;
ALTER TABLE referencing_table ADD CONSTRAINT fk_constraint FOREIGN KEY (new_column_name) REFERENCES table_name(new_column_name);
Step 5: Clean-Up and Verification

After the migration is complete, remove the old primary key column or constraints if they are no longer needed. Perform a thorough verification to ensure that all data integrity checks pass.

SQL
-- Example SQL for cleaning up in PostgreSQL
ALTER TABLE table_name DROP CONSTRAINT pk_constraint;
Step 6: Testing and Monitoring

After completing these steps, perform extensive testing and monitoring under non-production conditions to identify any issues or performance bottlenecks before rolling it out to production.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key to a successful migration is thorough planning, careful execution, and rigorous testing. Common mistakes include neglecting data backups, failing to handle foreign keys properly, and underestimating the complexity of the task. Follow-ups might explore how to optimize performance during large-scale migrations or discuss alternative strategies like using distributed transactions in multi-node databases.

**Code:**

```

-- Example SQL for adding a new column in PostgreSQL ALTER TABLE table_name ADD COLUMN new_column_name new_key_type;

CODE
Copy
```sql
-- Example SQL for updating rows in PostgreSQL
UPDATE table_name SET new_column_name = generate_new_key(old_column_name);
SQL
Copy
-- Example SQL for updating foreign key relationships in PostgreSQL
ALTER TABLE referencing_table DROP CONSTRAINT fk_constraint;
ALTER TABLE referencing_table ADD CONSTRAINT fk_constraint FOREIGN KEY (new_column_name) REFERENCES table_name(new_column_name);
SQL
Copy
-- Example SQL for cleaning up in PostgreSQL
ALTER TABLE table_name DROP CONSTRAINT pk_constraint;
```

