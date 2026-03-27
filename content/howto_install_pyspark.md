## Overview
to install pyspark in a virtual env
java is needed and version to be compatible with spark version

## Steps

### 1. install java

```
apt install openjdk-11-jdk
update-alternatives --list java
java -version
readlink -f $(which java)
echo $JAVA_HOME

if needed:
vi ~/.bashrc 
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
source ~/.bashrc 
```

### 2. install pyspark in venv

```
python3 -m venv venv
source venv/bin/activate
pip install pyspark==3.5.0
pyspark --version
```

### 3. test in IDE

```
create validate_spark.py in projects/spark_projects

from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("TestApp") \
    .master("local[*]") \
    .getOrCreate()

# Simple job
df = spark.range(10)
print(df.count())

create new project from existing source in IDE
define venv python bin as JDK (intellij)
```