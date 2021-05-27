# Instructions

## Before You Start

> hint: Always backup anything before making a change to it

### How To

#### Open Command Line

1. WINDOWS+R
2. type CMD
3. press enter

#### Run Application/Command

1. "open_command_line"
2. type `<COMMAND TEXT>
3. press enter

#### Environment Variables

> before appending any values it is advised to copy the existing value into a text file and save that.

1. run the 'rundll32 sysdm.cpl,EditEnvironmentVariables' command.
    1. Adding a new value
        1. new
        2. enter variable name
        3. enter variable value
    2. Appending an item to an existing value
        1. open the existing item.
        2. append a semicolon  (';') and the value you want to add to the existing text.
2. Click the "OK" button

### Linux v Windows

|  | Linux | Windows |
| --- | --- | --- |
| Path Seperators | backslash '/' | forwardslash '\' |
| Quotation Marks | single quotes ' ' | double quotes " " |
| Closing Console Applications | CTRL+D | CTRL+C |

## Preparing the environment

You should keep track of the version of the versions for all the software you install.

### Installing Java

1. Download the jre8 installer from <https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html>
   - Recently Oracle change the requirements for Java and they require that you create an account
2. Run the downloaded java installer
3. Set the following Environment Variables
    1. Create/Replace "JAVA_HOME" with "C:\\Program Files\\Java\\jre[version]" (or the latest installation path for Java)
    2. Append "PATH" with "JAVA_HOME\\bin"
4. Testing Java
   1. Run the following command within the command line
      - > java -version
   2. You should see the text
      - > java version and the Java Version you installed

### Installing Logstash for Windows

1. Download the logstash installer from <https://www.elastic.co/downloads/logstash-oss>
2. To install logstash follow these steps
   1. Create  the folder "C:\\Elastic"
   2. Unzip the contents of the downloaded file into "C:\\Elastic"
   3. You should see the following folder "C:\\Elastic\\logstash-[version]"
3. Install the websocket plugin by executing the following command
   - > C:\\Elastic\\logstash-[version]\\bin\\logstash-plugin install logstash-output-websocket
4. Testing your installation
   1. Open command line
   2. Navigate to the new folder you created by executing the following command
      - > cd C:\\Elastic\\logstash-[version]
   3. Execute the following command
      - > bin\\logstash -e "input { stdin {} } output { stdout {} }"

### Installing MySql

1. Download the MySQL windows installer from <https://dev.mysql.com/downloads/installer/>
2. Run the downloaded installer
   - Choose Developer Default as your option

### Installing the JDBC Connector

1. Download the JDBC connector from <https://go.microsoft.com/fwlink/?linkid=2155948>
2. Create a new folder "C:\\SQL JDBC"
3. Unzip the contents of the download into "C:\\SQL JDBC"
4. Copy the following file
   - From "C:\\SQL JDBC\\sqljdbc_[version]\\enu\\auth\\x86\\sqljdbc_auth.dll"
   - To "%JAVA_HOME%\\bin"

#### Configuring Logstash

1. Backup the following files (copy-paste and rename the extensions to .bak)
   - > C:\\Elastic\\logstash-[version]\\config\\logstash.yml
   - > C:\\Elastic\\logstash-[version]\\config\\pypelines.yml
2. Create the following files
   - > C:\\Elastic\\logstash-[version]\\config\\example.mysql.cfg
   - > C:\\Elastic\\logstash-[version]\\config\\example.sqlite.stdout.cfg
3. Replace the contents of the files like this

| File | Content |
| --- | --- |
| C:\\Elastic\\logstash-[version]\\config\\logstash.yml | log.level : info |
| C:\\Elastic\\logstash-[version]\\config\\pipelines.yml | - pipeline.id: example-mysql-pipeline\\npath.config: config/example.mysql.cfg\\npipeline.workers: 1 |
| C:\\Elastic\\logstash-[version]\\config\\example.mysql.stdout.cfg | see below |
    {
        input {
            jdbc {
                jdbc_driver_library => "mssql-jdbc-[jdbc-version].jre8.jar"
                jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
                jdbc_connection_string => "jdbc:sqlserver://localhost:1433;database=master;integratedSecurity=true;"
                jdbc_user => ""
                schedule => "* * * * *"
                statement => "SELECT Id, League, Amount, Odds FROM BetTickerTest WHERE Id > :sql_last_value"
                use_column_value => true
                tracking_column => "date"
                tracking_column_type => "timestamp"
                last_run_metadata_path => "C:/Elastic/logstash/temp/.logstash_jdbc_last_run"
            }
        }
        output {
            websocket {
                id => "sqlite_web_socket_output_id"
                host => "0.0.0.0"
                port => "3232"
                codec => "json"
            }
        }
    }

## Executing the applications

### Logstash

1. Run the following commands in command line.
   1. cd C:\\Elastic\\logstash-[version]
   2. run bin\\logstash
2. Don't close the window until you are finished with playing around.

### "html_websocket_client": {},
    "test": [
        { "open": "current sqlite instance" },
        { "run_command": "INSERT INTO BetTickerTest (Amount, Odds, League) VALUES (\"12,2\", \"36,12\", \"Super Rugby\");" },
        { "Finally": "check your logstash output for the row that you just inserted" }
    ]

## Alternative (Sqlite)

"note": "Try as I might I was unable to get the Sqlite plugin working, and couldnt find a solution online",
    "alternate": {
        "preparations": {
            "Sqlite": {
                "DLL_Files": [
                    { "download": "https://sqlite.org/2019/sqlite-dll-win64-x64-3270100.zip" },
                    { "unzip_destination": "C:\\Sqlite" }
                ],
                "Tools": [
                    { "download": "https://sqlite.org/2019/sqlite-tools-win32-x86-3270100.zip" },
                    { "unzip_destination": "C:\\Sqlite" }
                ],
                "environment_variables": [
                    { "PATH": "C:\\Sqlite" }
                ],
                "test": [
                    "open_command_line",
                    "sqlite3.exe"
                ]
            },
            "sqlite_jdbc_driver": {
                "install": [
                    { "download": "https://bitbucket.org/xerial/sqlite-jdbc/downloads/sqlite-jdbc-3.23.1.jar" },
                    { "unzip_destination": "C:\\Common\\jdbc\\sqlitejdbc_3.23.1" }
                ],
                "environment_variables": [
                    { "CLASSPATH": "C:\\Common\\jdbc\\sqlitejdbc_3.23.1\\sqlite-jdbc-3.23.1.jar" }
                ]
            },
            "mssql_jdbc_driver": {
                "install": [
                    { "download": "https://docs.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-2017" },
                    { "unzip_destination": "C:\\Common\\jdbc\\sqljdbc_7.2\\enu" },
                    { "note": "specified in <https://docs.microsoft.com/en-us/sql/connect/jdbc/using-the-jdbc-driver?view=sql-server-2017>" }
                ],
                "environment_variables": [
                    { "CLASSPATH": "C:\\Common\\jdbc\\sqljdbc_7.2\\enu\\mssql-jdbc-7.2.0.jre8.jar" }
                ]
            }
        },
        "set_content": [
            { "C:\\Elastic\\logstash-6.6.0\\config\\pipelines.yml": "- pipeline.id: example-sqlite-pipeline\\npath.config: config/example.sqlite.cfg\\npipeline.workers: 1" },
            { "C:\\Elastic\\logstash-6.6.0\\config\\example.sqlite.stdout.cfg": "" }
// "{
//     input {
//         jdbc: {
//             jdbc_driver_library => "mssql-jdbc-7.2.0.jre8.jar"
//             jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
//             jdbc_connection_string => "jdbc:sqlserver://localhost:1433;database=master;integratedSecurity=true;"
//             jdbc_user => ""
//             schedule => "* * * * *"
//             statement => "SELECT Id, League, Amount, Odds FROM BetTickerTest WHERE Id > :sql_last_value"
//             use_column_value => true
//             tracking_column => "date"
//             tracking_column_type => "timestamp"
//             last_run_metadata_path => "C:/Elastic/logstash/temp/.logstash_jdbc_last_run"
//         }
//     },
//     output {
//         websocket => {
//             id => "sqlite_web_socket_output_id",
//             host => "0.0.0.0",
//             port => "3232",
//             codec => "json",
//         }
//     }
// }"
        ],
        "sqlite": [
            { "create": "test.db in C:\\elastic\\test" },
            { "run": "Sqlite3.exe C:/elastic/test/test.db" }, 
            { "run_command": "CREATE TABLE BetTickerTest (Id INTEGER PRIMARY KEY AUTOINCREMENT, Amount NUMERIC NOT NULL, Odds NUMERIC NOT NULL, League VARCHAR NOT NULL);" },
            "dont_close_window"
        ],
        "test": [
            { "open": "current sqlite instance" },
            { "run_command": "INSERT INTO BetTickerTest (Amount, Odds, League) VALUES (\"12,2\", \"36,12\", \"Super Rugby\");" },
            { "Finally": "check your logstash output for the row that you just inserted" }
        ]
    }
}