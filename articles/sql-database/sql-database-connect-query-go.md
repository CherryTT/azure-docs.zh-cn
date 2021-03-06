---
title: 使用 Go 查询 Azure SQL 数据库 | Microsoft Docs
description: 使用 Go 创建连接到 Azure SQL 数据库的程序并使用 Transact-SQL 语句对其进行查询和修改数据。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: go
ms.topic: quickstart
author: David-Engel
ms.author: v-daveng
ms.reviewer: MightyPen
manager: craigg
ms.date: 11/01/2018
ms.openlocfilehash: c270fef40b732f170add32ef52eeadc790d8cd83
ms.sourcegitcommit: 799a4da85cf0fec54403688e88a934e6ad149001
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/02/2018
ms.locfileid: "50913487"
---
# <a name="quickstart-use-go-to-query-an-azure-sql-database"></a>快速入门：使用 Go 查询 Azure SQL 数据库

本快速入门演示了如何使用 [Go](https://godoc.org/github.com/denisenkom/go-mssqldb) 连接到 Azure SQL 数据库。 此外演示了用于查询和修改数据的 Transact-SQL 语句。

## <a name="prerequisites"></a>先决条件

若要完成本快速入门，请确保符合以下先决条件：

[!INCLUDE [prerequisites-create-db](../../includes/sql-database-connect-query-prerequisites-create-db-includes.md)]

- 针对用于本快速入门的计算机的公共 IP 地址制定[服务器级防火墙规则](sql-database-get-started-portal-firewall.md)。

- 已为操作系统安装 Go 和相关软件。

    - **MacOS**：安装 Homebrew 和 GoLang。 请参阅[步骤 1.2](https://www.microsoft.com/sql-server/developer-get-started/go/mac/)。
    - **Ubuntu**：安装 GoLang。 请参阅[步骤 1.2](https://www.microsoft.com/sql-server/developer-get-started/go/ubuntu/)。
    - **Windows**：安装 GoLang。 请参阅[步骤 1.2](https://www.microsoft.com/sql-server/developer-get-started/go/windows/)。    

## <a name="sql-server-connection-information"></a>SQL Server 连接信息

[!INCLUDE [prerequisites-server-connection-info](../../includes/sql-database-connect-query-prerequisites-server-connection-info-includes.md)]

## <a name="create-go-project-and-dependencies"></a>创建 Go 项目和依赖项

1. 从终端创建一个名为 **SqlServerSample** 的新项目文件夹。 

   ```bash
   mkdir SqlServerSample
   ```

2. 将目录切换到 **SqlServerSample**，获取并安装适用于 Go 的 SQL Server 驱动程序：

   ```bash
   cd SqlServerSample
   go get github.com/denisenkom/go-mssqldb
   go install github.com/denisenkom/go-mssqldb
   ```

## <a name="create-sample-data"></a>创建示例数据

1. 使用偏好的文本编辑器，在 **SqlServerSample** 文件夹中创建名为 **CreateTestData.sql** 的文件。 在该文件中复制并粘贴以下 T-SQL 代码。 此代码创建架构、表并插入少量的行。

   ```sql
   CREATE SCHEMA TestSchema;
   GO

   CREATE TABLE TestSchema.Employees (
     Id       INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
     Name     NVARCHAR(50),
     Location NVARCHAR(50)
   );
   GO

   INSERT INTO TestSchema.Employees (Name, Location) VALUES
     (N'Jared',  N'Australia'),
     (N'Nikita', N'India'),
     (N'Tom',    N'Germany');
   GO

   SELECT * FROM TestSchema.Employees;
   GO
   ```

2. 使用 sqlcmd 连接到数据库，并运行 SQL 脚本，以创建架构、表并插入新一些行。 替换服务器、数据库、用户名和密码的相应值。

   ```bash
   sqlcmd -S your_server.database.windows.net -U your_username -P your_password -d your_database -i ./CreateTestData.sql
   ```

## <a name="insert-code-to-query-sql-database"></a>插入用于查询 SQL 数据库的代码

1. 在 **SqlServerSample** 文件夹中创建名为 **sample.go** 的文件。

2. 打开该文件并将其内容替换为以下代码。 添加服务器、数据库、用户名和密码的相应值。 此示例使用 GoLang 上下文方法来确保与数据库服务器建立有效连接。

   ```go
   package main

   import (
       _ "github.com/denisenkom/go-mssqldb"
       "database/sql"
       "context"
       "log"
       "fmt"
       "errors"
   )

   var db *sql.DB

   var server = "your_server.database.windows.net"
   var port = 1433
   var user = "your_username"
   var password = "your_password"
   var database = "your_database"

   func main() {
       // Build connection string
       connString := fmt.Sprintf("server=%s;user id=%s;password=%s;port=%d;database=%s;",
           server, user, password, port, database)

       var err error

       // Create connection pool
       db, err = sql.Open("sqlserver", connString)
       if err != nil {
           log.Fatal("Error creating connection pool: ", err.Error())
       }
       ctx := context.Background()
       err = db.PingContext(ctx)
       if err != nil {
           log.Fatal(err.Error())
       }
       fmt.Printf("Connected!\n")

       // Create employee
       createID, err := CreateEmployee("Jake", "United States")
       if err != nil {
           log.Fatal("Error creating Employee: ", err.Error())
       }
       fmt.Printf("Inserted ID: %d successfully.\n", createID)

       // Read employees
       count, err := ReadEmployees()
       if err != nil {
           log.Fatal("Error reading Employees: ", err.Error())
       }
       fmt.Printf("Read %d row(s) successfully.\n", count)

       // Update from database
       updatedRows, err := UpdateEmployee("Jake", "Poland")
       if err != nil {
           log.Fatal("Error updating Employee: ", err.Error())
       }
       fmt.Printf("Updated %d row(s) successfully.\n", updatedRows)

       // Delete from database
       deletedRows, err := DeleteEmployee("Jake")
       if err != nil {
           log.Fatal("Error deleting Employee: ", err.Error())
       }
       fmt.Printf("Deleted %d row(s) successfully.\n", deletedRows)
   }

   // CreateEmployee inserts an employee record
   func CreateEmployee(name string, location string) (int64, error) {
       ctx := context.Background()
       var err error

       if db == nil {
           err = errors.New("CreateEmployee: db is null")
           return -1, err
       }

       // Check if database is alive.
       err = db.PingContext(ctx)
       if err != nil {
           return -1, err
       }

       tsql := "INSERT INTO TestSchema.Employees (Name, Location) VALUES (@Name, @Location); select convert(bigint, SCOPE_IDENTITY());"

       stmt, err := db.Prepare(tsql)
       if err != nil {
          return -1, err
       }
       defer stmt.Close()

       row := stmt.QueryRowContext(
           ctx,
           sql.Named("Name", name),
           sql.Named("Location", location))
       var newID int64
       err = row.Scan(&newID)
       if err != nil {
           return -1, err
       }

       return newID, nil
   }

   // ReadEmployees reads all employee records
   func ReadEmployees() (int, error) {
       ctx := context.Background()

       // Check if database is alive.
       err := db.PingContext(ctx)
       if err != nil {
           return -1, err
       }

       tsql := fmt.Sprintf("SELECT Id, Name, Location FROM TestSchema.Employees;")

       // Execute query
       rows, err := db.QueryContext(ctx, tsql)
       if err != nil {
           return -1, err
       }

       defer rows.Close()

       var count int

       // Iterate through the result set.
       for rows.Next() {
           var name, location string
           var id int

           // Get values from row.
           err := rows.Scan(&id, &name, &location)
           if err != nil {
               return -1, err
           }

           fmt.Printf("ID: %d, Name: %s, Location: %s\n", id, name, location)
           count++
       }

       return count, nil
   }

   // UpdateEmployee updates an employee's information
   func UpdateEmployee(name string, location string) (int64, error) {
       ctx := context.Background()

       // Check if database is alive.
       err := db.PingContext(ctx)
       if err != nil {
           return -1, err
       }

       tsql := fmt.Sprintf("UPDATE TestSchema.Employees SET Location = @Location WHERE Name = @Name")

       // Execute non-query with named parameters
       result, err := db.ExecContext(
           ctx,
           tsql,
           sql.Named("Location", location),
           sql.Named("Name", name))
       if err != nil {
           return -1, err
       }

       return result.RowsAffected()
   }

   // DeleteEmployee deletes an employee from the database
   func DeleteEmployee(name string) (int64, error) {
       ctx := context.Background()

       // Check if database is alive.
       err := db.PingContext(ctx)
       if err != nil {
           return -1, err
       }

       tsql := fmt.Sprintf("DELETE FROM TestSchema.Employees WHERE Name = @Name;")

       // Execute non-query with named parameters
       result, err := db.ExecContext(ctx, tsql, sql.Named("Name", name))
       if err != nil {
           return -1, err
       }

       return result.RowsAffected()
   }
   ```

## <a name="run-the-code"></a>运行代码

1. 请在命令提示符处运行以下命令：

   ```bash
   go run sample.go
   ```

2. 验证输出：

   ```text
   Connected!
   Inserted ID: 4 successfully.
   ID: 1, Name: Jared, Location: Australia
   ID: 2, Name: Nikita, Location: India
   ID: 3, Name: Tom, Location: Germany
   ID: 4, Name: Jake, Location: United States
   Read 4 row(s) successfully.
   Updated 1 row(s) successfully.
   Deleted 1 row(s) successfully.
   ```

## <a name="next-steps"></a>后续步骤

- [设计第一个 Azure SQL 数据库](sql-database-design-first-database.md)
- [适用于 Microsoft SQL Server 的 Go 驱动程序](https://github.com/denisenkom/go-mssqldb)
- [报告问题或提出问题](https://github.com/denisenkom/go-mssqldb/issues)

