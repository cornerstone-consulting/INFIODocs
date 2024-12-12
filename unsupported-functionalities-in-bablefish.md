## **Unsupported functionalities in Babelfish**

In the following table and lists, you can find functionality that isn't currently supported in Bablefish.

| ID             | UNSUPPORTED FEATURE                                                                                       |
|----------------|-----------------------------------------------------------------------------------------------------------|
| CCG-001        | SELECT FOR XML AUTO is not supported by Babelfish                                                         |
| CCG-002        | Numeric(20,0) with IDENTITY column not supported                                                          |
| CCG-003        | WHERE CURRENT OF not supported in cursors                                                                 |
| CCG-004        | Global Temp Tables not supported                                                                          |
| CCG-005        | UNPIVOT in function not supported                                                                         |
| CCG-006        | ALTER view/function/procedure/schema is not supported                                                     |
| CCG-007        | Postgres has 63-character limit vs. SQL which has 128-character limit for identifiers (table, index names, etc.)   |
| CCG-008        | Custom data types are not supported                                                                       |
| CCG-009        | WITH CHECK OPTION is supported only on automatically updatable views                                      |
| CCG-010        | Babelfish only supports migrating BINARY, VARBINARY, and IMAGE data types using the BYTEA data type        |
| CCG-011        | XML VALUE is not currently supported                                                                      |
| CCG-012        | ALTER TABLE [dbo].[FileImportHistory] ADD DEFAULT ((0)) FOR [Hash]                                        |
| CCG-013        | DBCC' is not currently supported as used in the code                                                     |
| CCG-014        | Using SQL Server as source with Computed Columns errors with DMS ERROR: extra data after last expected column   |
| CCG-015        | EXECUTE procedure sp_addtype                                                                              |
| CCG-016        | Dynamically created cursors (with the DECLARE CURSOR statement in EXECUTE() or sp_executesql) are not supported. \n Rewrite DECLARE CURSOR instead. |
| CCG-017        | DELETE with a CTE as the table being modified is not currently supported                                  |
| CCG-018        | UPDATE with a CTE as the table being modified is not currently supported                                  |
| CCG-019        | Merge not supported - used by NServiceBus in BAM+ for message queuing                                     |
| CCG-020        | Bulk Insert is not supported by Babelfish                                                                |
| CCG-021        | Alter table change nullability of a column via the Babelfish endpoint                                     |
| CCG-022        | Synonyms are not supported in Babelfish/Postgres                                                         |
| CCG-023        | Unable to Create/ALTER/DROP tables created in certain cross-database cases                                |
| CCG-024        | Updatable View Not Functioning                                                                           |
| CCG-025        | CURSOR_ROWS returns different result Babelfish versus SQL                                                |
| CCG-026        | TransactionScope not supported                                                                           |
| CCG-027        | PIVOT not supported in CTE/View/Join                                                                     |
| CCG-028        | sp_rename requires explicit object_type argument                                                         |
| CCG-029        | Setting Identity Insert on On/Off Across Databases                                                      |
| CCG-030        | sp_rename does not support INDEX type                                                                    |
| CCG-031        | TOP # PERCENT not yet supported                                                                          |

---
