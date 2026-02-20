
Open Visual Studio Code Editor
Create a folder
Verify Python3 version - It should be getter than 3.6

```
manish@MacBook-Pro-va-FOX airflow_tutorail % pwd
/Users/manish/Documents/airflow_tutorail

manish@MacBook-Pro-va-FOX airflow_tutorail % python3 --version
Python 3.12.

```

Create virtual environment - name py_env
```
manish@MacBook-Pro-va-FOX airflow_tutorail % python3 -m venv py_env

```

It creates bin, include, lib folders.

```
manish@MacBook-Pro-va-FOX airflow_tutorail % cd py_env
manish@MacBook-Pro-va-FOX py_env % ls -lart
total 8
drwxr-xr-x@  3 manish  staff   96 Dec 16 21:26 ..
drwxr-xr-x@  3 manish  staff   96 Dec 16 21:26 include
drwxr-xr-x@  3 manish  staff   96 Dec 16 21:26 lib
drwxr-xr-x@  6 manish  staff  192 Dec 16 21:26 .
-rw-r--r--@  1 manish  staff  284 Dec 16 21:26 pyvenv.cfg
drwxr-xr-x@ 12 manish  staff  384 Dec 16 21:26 bin
```

Activate Python environment py_env.

```
manish@MacBook-Pro-va-FOX airflow_tutorail % pwd
/Users/manish/Documents/airflow_tutorail

manish@MacBook-Pro-va-FOX airflow_tutorail % source ./py_env/bin/activate
(py_env) manish@MacBook-Pro-va-FOX airflow_tutorail % 
```

Install airflow: Adjust the constraints-3.x.txt based on python3 versions. I have python3.12 > constraints-3.12.txt is used. (https://github.com/apache/airflow?tab=readme-ov-file#installing-from-pypi)

```
pip install 'apache-airflow==3.1.5' \
 --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-3.1.5/constraints-3.12.txt"
```

Initialize DB for Airflow
Export AIRFLOW_HOME varibale in airflow_tutoral current directory 

```
(py_env) manish@MacBook-Pro-va-FOX airflow_tutorail % export AIRFLOW_HOME=/Users/manish/Documents/airflow_tutorail
(py_env) manish@MacBook-Pro-va-FOX airflow_tutorail % echo $AIRFLOW_HOME                                          
/Users/manish/Documents/airflow_tutorail

(py_env) manish@MacBook-Pro-va-FOX airflow_tutorail % airflow db migrate

```

Start airflow api-server

```
(py_env) manish@MacBook-Pro-va-FOX airflow_tutorail % airflow api-server
```
To login on the airflow api server - need user and password

Password will be present on AIRFLOW_HOME

login to airflow api-server with user admin and password.


Create the folder dags in AIRFLOW_HOME

copy the code to ~/airflow/dags/hello_dag.py
```
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

def print_hello():
    print("Hello from Airflow! Current time:", datetime.now())

def print_world():
    print("World from Airflow! Current time:", datetime.now())

with DAG(
    dag_id='hello_world',
    start_date=datetime(2025, 12, 16),
    schedule=timedelta(minutes=5),
    catchup=False
) as dag:

    hello_task = PythonOperator(
        task_id='say_hello',
        python_callable=print_hello
    )
    
    world_task = PythonOperator(
        task_id='say_world',
        python_callable=print_world
    )
    
    # Dependency: hello >> world
    hello_task >> world_task

```


Start airflow scheduler - this will run the DAG.
```
airflow scheduler
```
```
airflow dags list-import-errors
airflow dags reserialize # Flush the cache
airflow dags list | grep hello_world  # To list DAG
airflow dags list-runs hello_world #to see the last runs of the dag with status
airflow dags trigger hello_world. # to trigger the DAG


```

