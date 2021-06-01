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
        2. enter variable name.
        3. enter variable value.
    2. Appending an item to an existing value.
        1. open the existing item.
        2. append a semicolon  (';') and the value you want to add to the existing text.
2. Click the "OK" button.

### Linux v Windows

A lot of open source applications are written with Linux in mind so it's usefull to know some of the differences you might face when it comes to configuration.

|  | Linux | Windows |
| --- | --- | --- |
| Path Seperators | backslash '/' | forwardslash '\' |
| Quotation Marks | single quotes ' ' | double quotes " " |
| Closing Console Applications | CTRL+D | CTRL+C |

## Preparing the environment

You should keep track of the versions for all the software you install so that you can use it in the paths below.

### Installing Java

1. Download the jre8 installer from <https://download.java.net/openjdk/jdk11/ri/openjdk-11+28_windows-x64_bin.zip>.
2. Extract the contents of the folder to C:\\Program Files\\Java\\.
3. Testing Java.
   1. Open a command prompt instance
   2. Navigate to C:\\Program Files\\Java
   3. Run the following command within the command line.
      - > java -version
   4. You should see the text.
      - > java version and the Java Version you installed

### MySql

#### Installation

1. Download the MySQL windows installer from <https://dev.mysql.com/downloads/installer/>.
2. Run the downloaded installer.
   - Choose Developer Default as your option.
   - Wait for the installation to ask you for a username and password.
   - Set your root user password to "password".
3. Copy the JDBC driver file
   - From
    > C:\Program Files (x86)\MySQL\Connector J 8.0\mysql-connector-java-8.0.25.jar
   - To
    > C:\Elastic\logstash-7.13.0\jdk\connectors
4. Open an instance of MySQL Workbench.
5. Execute the below create table statement within your sqlite session.

    CREATE TABLE sys.`logWatcherTest` (
        `Id` int NOT NULL AUTO_INCREMENT PRIMARY KEY,
        `Item` VARCHAR(255) NOT NULL,
        `Price` DECIMAL(19, 3) NOT NULL
    );

### Logstash for Windows

#### Installation

1. Download the logstash installer from <https://www.elastic.co/downloads/logstash-oss>.
2. To install logstash follow these steps.
   1. Create  the folder "C:\\Elastic".
   2. Unzip the contents of the downloaded file into "C:\\Elastic".
   3. You should see the following folder "C:\\Elastic\\logstash-[version]".
3. Testing your installation.
   1. Open command line.
   2. Navigate to the new folder you created by executing the following command.
      - > cd C:\\Elastic\\logstash-[version]
   3. Execute the following command.
      - > bin\\logstash -e "input { stdin {} } output { stdout {} }"
4. Set the following Environment Variables.
    1. Create/Replace "JAVA_HOME" with "C:\\Elastic\\logstash-[version]\\jdk" (or the latest installation path for Java).
    2. Append "PATH" with "JAVA_HOME\\bin".

#### Configuration

1. Backup the following files (copy-paste and rename the extensions to .bak)
    > C:\\Elastic\\logstash-[version]\\config\\logstash.yml\
    > C:\\Elastic\\logstash-[version]\\config\\pypelines.yml\
    > C:\\Elastic\\logstash-[version]\\Gemfile
2. Create the following files
    > C:\\Elastic\\logstash-[version]\\config\\example.mysql.cfg
3. Replace the contents of the files as below

- C:\\Elastic\\logstash-[version]\\config\\logstash.yml
    > log.level : info
- C:\\Elastic\\logstash-[version]\\config\\pipelines.yml

        \- pipeline.id: example-mysql-pipeline\
          path.config: config/example.mysql.cfg\
          pipeline.workers: 1

- C:\\Elastic\\logstash-[version]\\config\\example.mysql.stdout.cfg

        input {
            jdbc {
                jdbc_driver_library => "./jdk/connectors/mysql-connector-java-8.0.25.jar"
                jdbc_driver_class => "com.mysql.jdbc.Driver"
                jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/sys"
                jdbc_user => "root"
                jdbc_password => "password"
                schedule => "*/3 * * * * *"
                statement => "SELECT Id, Item, Price FROM LogWatcherTest WHERE Id > :sql_last_value"
                tracking_column => "id"
                use_column_value => true
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

### Running the websocket Plugin

1. Add the below line to the end of the "C:\\Elastic\\logstash-[version]\\Gemfile" file
    > gem "logstash-output-websocket", :path => "./logstash-output-websocket"
2. Copy the logstash-output-websocket folder from 
2. Within your logstash folder run the following commands within command prompt
    > bin/logstash-plugin install --no-verify

## Executing the applications

### Logstash

1. Run the following commands in command line.
   1. cd C:\\Elastic\\logstash-[version]
   2. run bin\\logstash
2. Don't close the window until you are finished with playing around.

### HTML Websocket Client

1. Download the WebSocket Application from the github repository.
2. Open the index.html file in a browser and leave it running.

### Running the Demo

1. Open a MySQL workbench instance
2. Run the following sql query.
   - > INSERT INTO LogWatcherTest (Item, Price) VALUES (\"Makita Drill\", \"12,2\");
3. Finally check your logstash output for the row that was just inserted as well as your browser for a new line within the site.

## Alternative Methods

### Sqlite

> Try as I might I was unable to get the Sqlite plugin working, and couldnt find a solution online

### Installing Sqlite

1. "DLL_Files": [
        { "download": "https://sqlite.org/2019/sqlite-dll-win64-x64-3270100.zip" },
        { "unzip_destination": "C:\\Sqlite" }
2. "Tools": [
        { "download": "https://sqlite.org/2019/sqlite-tools-win32-x86-3270100.zip" },
        { "unzip_destination": "C:\\Sqlite" }
3. "environment_variables": [
        { "PATH": "C:\\Sqlite" }
4. "test": [
        "open_command_line",
        "sqlite3.exe"
    ]

### Sqlite JDBC Driver

    "install": [
        { "download": "https://bitbucket.org/xerial/sqlite-jdbc/downloads/sqlite-jdbc-3.23.1.jar" },
        { "unzip_destination": "C:\\Common\\jdbc\\sqlitejdbc_3.23.1" }
    ],
    "environment_variables": [
        { "CLASSPATH": "C:\\Common\\jdbc\\sqlitejdbc_3.23.1\\sqlite-jdbc-3.23.1.jar" }
    ]
}
    { "C:\\Elastic\\logstash-6.6.0\\config\\example.sqlite.stdout.cfg": "" }
    "{
        input {
            jdbc: {
                    path => "C:\\elastic\\test\\test.db"
                    type => logWatcherTest
            }
        },
        output {
            websocket => {
                id => "sqlite_web_socket_output_id",
                host => "0.0.0.0",
                port => "3232",
                codec => "json",
            }
        }
    }"

### Configurating Sqlite

1. Create a new folder new folder "test" within "C:\\elastic".
2. Create a new file called "test.db" within "C:\\elastic\\test".
3. Execute the following command in command prompt.
   - > Sqlite3.exe C:/elastic/test/test.db
4. Execute the below create table statement within your sqlite session
   - > CREATE TABLE logWatcherTest (Id INTEGER PRIMARY KEY AUTOINCREMENT, Item VARCHAR NOT NULL, Price NUMERIC NOT NULL);

### Running the Sqlite Demo

1. Open your current sqlite
2. Execute the following command in command prompt
   - > Sqlite3.exe C:/elastic/test/test.db
3. Run the following in the above session
   - > INSERT INTO LogWatcherTest (Item, Price) VALUES (\"Makita Drill\", \"12,2\");
4. Finally check your logstash output for the row that was just inserted as well as your browser for a new line within the site.

### Sql Server

## Installing MS SQL JDBC Driver

1. Download the JDBC connector from <https://go.microsoft.com/fwlink/?linkid=2155948>.
2. Create a new folder "C:\\SQL JDBC".
3. Unzip the contents of the download into "C:\\SQL JDBC".
4. Copy the following file.
   - From "C:\\SQL JDBC\\sqljdbc_[version]\\enu\\auth\\x86\\sqljdbc_auth.dll"
   - To "%JAVA_HOME%\\bin"
