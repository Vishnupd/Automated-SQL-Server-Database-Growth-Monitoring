# Automated SQL Server Database Growth Monitoring

## Objective

The SQL Server Database Growth Monitoring project aimed to automate database space monitoring and proactively identify when a database data file approaches its configured storage threshold. The goal was to monitor database file utilization, compare used space against a defined percentage limit, and automatically send email alerts when the threshold was exceeded. This hands-on project provided practical experience in SQL Server administration, T-SQL automation, database monitoring, and Database Mail configuration.

### Skills Learned

* Development and modification of T-SQL stored procedures.
* Monitoring SQL Server database file size and space utilization.
* Working with dynamic SQL and SQL Server system tables.
* Using `FILEPROPERTY()` to determine database file space usage.
* Configuring and troubleshooting SQL Server Database Mail.
* Implementing automated email notifications for database monitoring.
* Performing proactive database capacity monitoring and administration.

### Tools Used

* **Microsoft SQL Server 2016** for database administration and monitoring.
* **SQL Server Management Studio (SSMS)** for T-SQL development and execution.
* **T-SQL** for stored procedure development and database monitoring.
* **SQL Server Database Mail** for automated email notifications.
* **Gmail SMTP** for sending database monitoring alerts.
* **msdb system tables** for verifying Database Mail activity.

## Steps

Below are the key steps taken in the database growth monitoring process:

### 1. Enable SQL Server Database Mail

Database Mail XPs were enabled at the SQL Server instance level using T-SQL. This allowed the server to use the `sp_send_dbmail` stored procedure for sending automated email notifications.

```sql
USE master;
GO

EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
GO

EXEC sp_configure 'Database Mail XPs', 1;
RECONFIGURE;
GO
```

*Ref 1: Database Mail XPs Configuration*
This screenshot shows the SQL Server configuration commands used to enable Database Mail functionality.

![Database Mail XPs Configuration](https://github.com/Vishnupd/Automated-SQL-Server-Database-Growth-Monitoring/blob/main/Mail_Configuration.png)

### 2. Configure the Database Mail Account

Since Database Mail could not be configured through the SSMS graphical interface, the mail account was created using T-SQL. Gmail SMTP was configured with SSL/TLS authentication.

```sql
USE msdb;
GO

EXEC dbo.sysmail_add_account_sp
    @account_name = 'SQL Server Mail Account',
    @description = 'Database Mail account',
    @email_address = 'vishnuprasad19931994@gmail.com',
    @display_name = 'SQL Server',
    @mailserver_name = 'smtp.gmail.com',
    @port = 587,
    @enable_ssl = 1,
    @username = 'vishnuprasad19931994@gmail.com',
    @password = '#######';
```

*Ref 2: Database Mail Account Configuration*
This screenshot shows the T-SQL configuration used to create the SQL Server Database Mail account.

![Database Mail Account](https://github.com/Vishnupd/Automated-SQL-Server-Database-Growth-Monitoring/blob/main/Mail_Account.png)

### 3. Create and Associate the Database Mail Profile

A Database Mail profile was created and associated with the configured mail account. The profile provides a reusable interface for SQL Server procedures and jobs to send email notifications.

```sql
USE msdb;
GO

EXEC dbo.sysmail_add_profile_sp
    @profile_name = 'SQL Server Mail Profile',
    @description = 'SQL Server Database Mail Profile';
GO

EXEC dbo.sysmail_add_profileaccount_sp
    @profile_name = 'SQL Server Mail Profile',
    @account_name = 'SQL Server Mail Account',
    @sequence_number = 1;
GO
```

*Ref 3: Database Mail Profile Configuration*
This screenshot shows the creation of the Database Mail profile and its association with the mail account.

![Database Mail Profile](https://github.com/Vishnupd/Automated-SQL-Server-Database-Growth-Monitoring/blob/main/Mail_Profile.png)

### 4. Test and Verify Database Mail

Before integrating Database Mail with the monitoring procedure, an email was sent using `sp_send_dbmail` to verify that the configuration was working correctly.

```sql
EXEC msdb.dbo.sp_send_dbmail
    @profile_name = 'SQL Server Mail Profile',
    @recipients = 'vishnuprasad19931994@gmail.com',
    @subject = 'SQL Server Database Mail Test',
    @body = 'Database Mail is working successfully.';
```

The Database Mail system table was then queried to verify the email request.

```sql
SELECT *
FROM msdb.dbo.sysmail_allitems
ORDER BY send_request_date DESC;
```

*Ref 4: Database Mail Test and Verification*
This screenshot shows the successful Database Mail test and the corresponding mail item recorded in the SQL Server `msdb` database.

![Database Mail Test](https://github.com/Vishnupd/Automated-SQL-Server-Database-Growth-Monitoring/blob/main/Mail_Test.png)
![Database Mail Test](https://github.com/Vishnupd/Automated-SQL-Server-Database-Growth-Monitoring/blob/main/Test_Mail.png)

### 5. Database File Space Monitoring

The `db_growth_monitoring` stored procedure was developed to accept the database name and usage threshold as parameters. Dynamic SQL was used to retrieve the database file size and current used space.

```sql
EXEC dbo.db_growth_monitoring
    @limit_percent = 80,
    @db_name = 'TechHealthDb';
```

The procedure calculates the file size and used space using SQL Server system information and `FILEPROPERTY()`.

*Ref 5: Database Growth Monitoring Procedure*
This screenshot shows the stored procedure used to monitor database file utilization and compare it against the configured threshold.

![Database Growth Monitoring Procedure](https://github.com/Vishnupd/Automated-SQL-Server-Database-Growth-Monitoring/blob/main/Stored%20Procedure.png)

### 6. Threshold Comparison and Alert Generation

The procedure compares the current used space against the configured percentage threshold.

```sql
IF @usd_size_value > (@fl_size_value * @limit_percent) / 100
```

If the used space exceeds the specified limit, an email alert is generated using `sp_send_dbmail`.

The email contains information about the current used space and total file size.

*Ref 6: Database Growth Monitoring Execution*
This screenshot shows the execution of the monitoring procedure and the database space utilization check.

![Database Growth Monitoring Execution](https://github.com/Vishnupd/Automated-SQL-Server-Database-Growth-Monitoring/blob/main/DB_Growth_SP_Execution.png)

### 7. Database Growth Alert

When the configured threshold is exceeded, an automated email notification is sent using the Database Mail profile.

The notification includes the database name, configured threshold, current used space, and total file size.

*Ref 7: Database Growth Alert Email*
This screenshot shows the automated email notification generated when the database file utilization exceeded the configured threshold.

![Database Growth Alert](https://github.com/Vishnupd/Automated-SQL-Server-Database-Growth-Monitoring/blob/main/DB_Growth_Mail.png)

## Results

The project successfully demonstrated automated SQL Server database space monitoring and email-based alerting. The solution reduced the need for manual monitoring and provided an early warning when database file utilization reached a defined threshold.

This project demonstrates practical experience in SQL Server Database Administration, T-SQL development, database monitoring, Database Mail configuration, dynamic SQL, and proactive database capacity management.
