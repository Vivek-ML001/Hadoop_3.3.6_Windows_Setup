# Hadoop 3.3.6 on Windows — Zero-to-Setup Guide

A beginner-friendly guide for setting up **Java 8 + Apache Hadoop 3.3.6 on Windows 10/11**, running Standalone and Pseudo-distributed modes, and practicing HDFS commands.

> Recommended lab setup: Windows 10/11 64-bit + Java 8 + Hadoop 3.3.6.

## 1. Download links

### Java 8
Eclipse Temurin OpenJDK 8:
https://adoptium.net/temurin/releases/?version=8

Choose Windows + x64 + JDK + Java 8.

### Apache Hadoop 3.3.6
Official release:
https://hadoop.apache.org/release/3.3.6.html

Documentation:
https://hadoop.apache.org/docs/r3.3.6/

Download the Hadoop 3.3.6 binary distribution from an Apache mirror.

### Windows native Hadoop binaries
Community repository:
https://github.com/cdarlint/winutils

For Windows Hadoop 3.3.6, use the matching `hadoop-3.3.6` directory if your setup requires Windows native binaries.

**Important:** These Windows binaries are community-provided, not part of the official Apache Hadoop binary distribution. Do not mix binaries from different Hadoop versions.

---

# 2. What are the three Hadoop modes?

## Standalone

One local Java process; no NameNode/DataNode daemons.

```text
Local files -> Hadoop program -> Local output
```

Used mainly for testing and learning.

## Pseudo-distributed

One computer runs Hadoop daemons separately.

```text
One PC
 ├── NameNode
 ├── DataNode
 ├── ResourceManager
 └── NodeManager
```

This is the recommended single-PC student lab setup.

## Fully distributed

Multiple computers/nodes form a real Hadoop cluster.

```text
Master
 ├── NameNode
 └── ResourceManager
        |
        +── Worker 1: DataNode + NodeManager
        +── Worker 2: DataNode + NodeManager
        +── Worker 3: DataNode + NodeManager
```

---

# 3. Install Java 8

Install Java 8 and find its installation folder.

Example:

```text
C:\Program Files\Eclipse Adoptium\jdk-8.0.XXX-hotspot
```

Do not include `\bin` in `JAVA_HOME`.

Verify:

```cmd
java -version
javac -version
```

Both should work.

---

# 4. Set JAVA_HOME

Open:

**Environment Variables -> User variables -> New**

Set:

```text
Variable name: JAVA_HOME
Variable value: C:\Program Files\Eclipse Adoptium\your-jdk-8-folder
```

Edit `Path` and add:

```text
%JAVA_HOME%\bin
```

Open a new CMD and verify:

```cmd
echo %JAVA_HOME%
java -version
javac -version
```

---

# 5. Install Hadoop

Extract Hadoop so that the main directory is:

```text
C:\hadoop
```

It should contain:

```text
C:\hadoop\bin
C:\hadoop\etc
C:\hadoop\sbin
C:\hadoop\share
```

Avoid accidentally creating:

```text
C:\hadoop\hadoop-3.3.6\bin
```

if you want to follow this guide exactly.

---

# 6. Set HADOOP_HOME

Create:

```text
HADOOP_HOME=C:\hadoop
```

Add to `Path`:

```text
%HADOOP_HOME%\bin
%HADOOP_HOME%\sbin
```

Open a new CMD and test:

```cmd
hadoop version
where hadoop
```

You should see Hadoop 3.3.6.

---

# 7. Windows native binaries

If your Windows setup requires them, copy the Hadoop-version-matching files from the community `winutils` repository into:

```text
C:\hadoop\bin
```

Common files include:

```text
winutils.exe
hadoop.dll
hdfs.dll
```

Check:

```cmd
dir C:\hadoop\bin\winutils.exe
```

Do not use a random version of `winutils.exe` with Hadoop 3.3.6.

---

# 8. Create Hadoop data directories

Run:

```cmd
mkdir C:\hadoop\data
mkdir C:\hadoop\data\namenode
mkdir C:\hadoop\data\datanode
```

---

# 9. Configure core-site.xml

File:

```text
C:\hadoop\etc\hadoop\core-site.xml
```

Use:

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

---

# 10. Configure hdfs-site.xml

File:

```text
C:\hadoop\etc\hadoop\hdfs-site.xml
```

Use:

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///C:/hadoop/data/namenode</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///C:/hadoop/data/datanode</value>
    </property>
</configuration>
```

Because this is a one-DataNode lab setup, replication is set to `1`.

---

# 11. Configure mapred-site.xml

If it does not exist, copy:

```text
mapred-site.xml.template
```

to:

```text
mapred-site.xml
```

Use:

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

---

# 12. Configure yarn-site.xml

File:

```text
C:\hadoop\etc\hadoop\yarn-site.xml
```

Use:

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
</configuration>
```

---

# 13. Configure hadoop-env.cmd

File:

```text
C:\hadoop\etc\hadoop\hadoop-env.cmd
```

Set `JAVA_HOME` to your actual Java 8 directory.

Example:

```cmd
set JAVA_HOME=C:\Program Files\Eclipse Adoptium\jdk-8.0.XXX-hotspot
```

Your exact Java folder name may differ.

---

# 14. Verify installation

Open a **new Command Prompt**:

```cmd
java -version
```

```cmd
javac -version
```

```cmd
hadoop version
```

```cmd
echo %JAVA_HOME%
```

```cmd
echo %HADOOP_HOME%
```

Expected:

```text
JAVA_HOME = your Java 8 folder
HADOOP_HOME = C:\hadoop
```

---

# 15. Format NameNode — first setup only

For a new Hadoop data directory:

```cmd
hdfs namenode -format
```

**Do not run this every time you start Hadoop.**

Formatting creates a new NameNode namespace. Re-formatting an existing installation can make previous HDFS metadata inaccessible.

---

# 16. Start HDFS

```cmd
cd /d C:\hadoop
start-dfs.cmd
```

Check:

```cmd
jps
```

Expected:

```text
NameNode
DataNode
Jps
```

---

# 17. Start YARN

If your practical needs YARN/MapReduce:

```cmd
start-yarn.cmd
```

Check:

```cmd
jps
```

Expected:

```text
NameNode
DataNode
ResourceManager
NodeManager
Jps
```

---

# 18. Stop Hadoop

Stop YARN:

```cmd
stop-yarn.cmd
```

Stop HDFS:

```cmd
stop-dfs.cmd
```

Then:

```cmd
jps
```

---

# 19. HDFS file management commands

## Create directory

```cmd
hdfs dfs -mkdir /test
```

## Create nested directory

```cmd
hdfs dfs -mkdir -p /data/input
```

## List root

```cmd
hdfs dfs -ls /
```

## Create local test file

```cmd
echo Hello Hadoop > hello.txt
```

## Upload to HDFS

```cmd
hdfs dfs -put hello.txt /test
```

## Verify

```cmd
hdfs dfs -ls /test
```

## Read file

```cmd
hdfs dfs -cat /test/hello.txt
```

## Download from HDFS

```cmd
hdfs dfs -get /test/hello.txt downloaded.txt
```

## Copy inside HDFS

```cmd
hdfs dfs -cp /test/hello.txt /test/hello-copy.txt
```

## Move/rename

```cmd
hdfs dfs -mv /test/hello-copy.txt /test/renamed.txt
```

## Delete file

```cmd
hdfs dfs -rm /test/renamed.txt
```

## Delete directory recursively

```cmd
hdfs dfs -rm -r /test
```

---

# 20. Check DataNode and HDFS storage

```cmd
hdfs dfsadmin -report
```

This reports:

- Configured capacity
- DFS used
- DFS remaining
- Live DataNodes
- Blocks
- DataNode health

---

# 21. Hadoop Web UI

When NameNode is running, open:

```text
http://localhost:9870
```

Then:

```text
Utilities -> Browse the file system
```

You can inspect HDFS directories and files.

For example:

```text
/
└── test
    └── hello.txt
```

---

# 22. Standalone mode

Standalone mode does not require NameNode/DataNode.

Do NOT start HDFS for the standalone demonstration.

Create:

```cmd
mkdir C:\hadoop\standalone
cd /d C:\hadoop\standalone
```

Create a local file:

```cmd
echo Hello Hadoop Hello World > input.txt
```

Read it:

```cmd
type input.txt
```

The file is stored on the normal Windows filesystem, not HDFS.

Standalone MapReduce programs can run locally.

---

# 23. Pseudo-distributed mode

Start:

```cmd
start-dfs.cmd
```

Check:

```cmd
jps
```

Then:

```cmd
hdfs dfs -mkdir /test
```

Upload:

```cmd
hdfs dfs -put hello.txt /test
```

List:

```cmd
hdfs dfs -ls /test
```

Read:

```cmd
hdfs dfs -cat /test/hello.txt
```

This demonstrates HDFS on one computer using separate Hadoop daemons.

---

# 24. One-command startup script

Create:

```text
C:\hadoop\start-hadoop.bat
```

Put:

```bat
@echo off
echo Starting Hadoop HDFS...
call C:\hadoop\sbin\start-dfs.cmd

echo Starting Hadoop YARN...
call C:\hadoop\sbin\start-yarn.cmd

echo.
echo Hadoop startup commands completed.
echo Run "jps" to verify.
pause
```

Run:

```cmd
C:\hadoop\start-hadoop.bat
```

---

# 25. One-command shutdown script

Create:

```text
C:\hadoop\stop-hadoop.bat
```

Put:

```bat
@echo off
echo Stopping Hadoop YARN...
call C:\hadoop\sbin\stop-yarn.cmd

echo Stopping Hadoop HDFS...
call C:\hadoop\sbin\stop-dfs.cmd

echo.
echo Hadoop shutdown commands completed.
pause
```

Run:

```cmd
C:\hadoop\stop-hadoop.bat
```

---

# 26. Useful command cheat sheet

| Command | Purpose |
|---|---|
| `java -version` | Check Java |
| `javac -version` | Check Java compiler |
| `hadoop version` | Check Hadoop |
| `hadoop` | Hadoop help |
| `jps` | Check Hadoop/Java processes |
| `start-dfs.cmd` | Start HDFS |
| `stop-dfs.cmd` | Stop HDFS |
| `start-yarn.cmd` | Start YARN |
| `stop-yarn.cmd` | Stop YARN |
| `hdfs namenode -format` | Format a new NameNode |
| `hdfs dfs -ls /` | List HDFS root |
| `hdfs dfs -mkdir /dir` | Create HDFS directory |
| `hdfs dfs -put file /dir` | Upload file |
| `hdfs dfs -get /dir/file` | Download file |
| `hdfs dfs -cat /dir/file` | Read file |
| `hdfs dfs -cp` | Copy file |
| `hdfs dfs -mv` | Move/rename |
| `hdfs dfs -rm` | Delete file |
| `hdfs dfs -rm -r` | Delete directory recursively |
| `hdfs dfsadmin -report` | HDFS/DataNode report |

Official Hadoop command manual:

https://hadoop.apache.org/docs/r3.3.6/hadoop-project-dist/hadoop-common/CommandsManual.html

---

# 27. Troubleshooting

## Java is not recognized

Run:

```cmd
java -version
```

Then:

```cmd
echo %JAVA_HOME%
```

Make sure `%JAVA_HOME%\bin` is in `Path`.

Open a new CMD after changing environment variables.

## Hadoop is not recognized

Run:

```cmd
echo %HADOOP_HOME%
```

and:

```cmd
where hadoop
```

Make sure these are in `Path`:

```text
%HADOOP_HOME%\bin
%HADOOP_HOME%\sbin
```

## winutils error

Check:

```cmd
dir C:\hadoop\bin\winutils.exe
```

Use a version matching your Hadoop installation.

## NameNode does not start

Run:

```cmd
jps
```

Check logs:

```text
C:\hadoop\logs
```

Verify:

```text
core-site.xml
hdfs-site.xml
hadoop-env.cmd
```

## DataNode does not start

Check:

```text
C:\hadoop\data\datanode
```

and:

```text
C:\hadoop\logs
```

## HDFS connection error

Check:

```cmd
jps
```

Make sure `NameNode` is running.

Then:

```cmd
hdfs dfsadmin -report
```

---

# 28. Normal daily workflow

You only format the NameNode during initial setup.

For a normal lab session:

```cmd
start-dfs.cmd
```

If needed:

```cmd
start-yarn.cmd
```

Check:

```cmd
jps
```

Use HDFS:

```cmd
hdfs dfs -ls /
```

At the end:

```cmd
stop-yarn.cmd
```

```cmd
stop-dfs.cmd
```

---

# 29. Lab practical roadmap

After installation, work through:

1. Hadoop operating modes: Standalone, Pseudo-distributed, Fully distributed
2. HDFS file management: add, retrieve, delete
3. Word Count MapReduce
4. Weather data analysis using MapReduce
5. Matrix multiplication using MapReduce
6. Apache Pig: sort, group, join, project, filter
7. Apache Hive: databases, tables, views, functions, and syllabus-required operations
8. Real-life Big Data problem
9. Market Basket Analysis using Apriori and Association Rules
10. PCA for House Data

---

# 30. Final verification

Run:

```cmd
java -version
```

```cmd
javac -version
```

```cmd
hadoop version
```

Start:

```cmd
start-dfs.cmd
```

If YARN is needed:

```cmd
start-yarn.cmd
```

Check:

```cmd
jps
```

Check HDFS:

```cmd
hdfs dfsadmin -report
```

Create a test directory:

```cmd
hdfs dfs -mkdir /test
```

Create a local file:

```cmd
echo Hello Hadoop > hello.txt
```

Upload:

```cmd
hdfs dfs -put hello.txt /test
```

Verify:

```cmd
hdfs dfs -ls /test
```

Read:

```cmd
hdfs dfs -cat /test/hello.txt
```

If these work, your basic Hadoop + HDFS setup is working.

---

## Official/reference links

- Apache Hadoop 3.3.6: https://hadoop.apache.org/release/3.3.6.html
- Hadoop 3.3.6 Documentation: https://hadoop.apache.org/docs/r3.3.6/
- Hadoop Commands Manual: https://hadoop.apache.org/docs/r3.3.6/hadoop-project-dist/hadoop-common/CommandsManual.html
- Eclipse Temurin Java 8: https://adoptium.net/temurin/releases/?version=8
- Windows Hadoop native binaries: https://github.com/cdarlint/winutils

## Note

Apache Hadoop is an Apache Software Foundation project. Follow the licenses and terms of the individual software components and third-party repositories you download. For classroom distribution, it is preferable for students to download installers/software from the official project or vendor sources rather than redistributing executable installers.
