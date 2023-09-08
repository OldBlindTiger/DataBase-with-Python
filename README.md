# DataBase-with-Python
Приклади створення **engine** для різних видів підключення, до різних типів БД

## PostgreSQL
* Підключення напряму до хоста з БД
```python
import psycopg2
from sqlalchemy import create_engine

# =====================================================================================
dict_params = {
    "host": "10.160.10.10", 
    "port": "5432", 
    "database": "db_pg_test", 
    "user": "postgres", 
    "password": "1234567"
}

def connector():
    return psycopg2.connect(**dict_params)

engine = create_engine('postgresql+psycopg2://', creator=connector)
```

## MySQL
* Підключення напряму до хоста з БД
```python
# py -m pip install pymysql
import pymysql
from sqlalchemy import create_engine

# =====================================================================================
dict_params = {
	'host': '10.160.10.10',
	'db': 'db_mysql_test',
	'user': 'admin',
	'password': '1234567',
	'charset': 'utf8mb4'
}

def connector():
    return pymysql.connect(**dict_params)

engine = create_engine("mysql+pymysql://", creator=connector)
```

* Підключення через SSH-тунель
```python
import pymysql
from sshtunnel import SSHTunnelForwarder
from sqlalchemy import create_engine, pool

# -------------------------------------------------------------------------------------
# Налаштування SSH тунелю
ssh_host = '10.160.10.10'
ssh_port = 22
ssh_username = 'admin'
ssh_password = '12345678'

# -------------------------------------------------------------------------------------
# Налаштування підключення до бази даних MySQL
mysql_host = 'localhost'  # SSH тунель перенаправляє на цей хост
mysql_port = 3306  # порт MySQL сервера
mysql_username = 'user_local'
mysql_password = '87654321'
mysql_db = 'db_mysl_ssh_test'

# =====================================================================================
# Функція для створення з'єднань з базою даних через SSH тунель
# =====================================================================================
def create_mysql_connection():
    tunnel = SSHTunnelForwarder(
        (ssh_host, ssh_port),
        ssh_username=ssh_username,
        ssh_password=ssh_password,
        remote_bind_address=(mysql_host, mysql_port)
    )
    tunnel.start()
    
    connection = pymysql.connect(
        host=mysql_host,
        port=tunnel.local_bind_port,
        user=mysql_username,
        password=mysql_password,
        db=mysql_db
    )
    
    return connection

# -------------------------------------------------------------------------------------
# Створення SQLAlchemy engine з використанням pool.QueuePool та власного creator
engine = create_engine(
    f"mysql+pymysql://{mysql_username}:{mysql_password}@localhost:{mysql_port}/{mysql_db}",
    poolclass=pool.QueuePool,
    creator=create_mysql_connection
)
```

## Перевірка підключення до БД за допомогою **engine**
```python
from sqlalchemy.exc import OperationalError  # Додана операційна помилка

def run_script():
    path_name = os.path.dirname(__file__)

    # Перевірка конекту
    try:
        connection = engine.connect()
    except OperationalError as e:
        print(f'[ERROR] Помилка при спробі підключитися до бази даних:\n{str(e)}')
        return -1

    connection.close()
    return 1

if __name__ == '__main__':
    run_script()
```

## Вибірка даних у DataFrame за допомогою SQL-запиту
```python
import pandas as pd

# Відбір даних у DataFrame за допомогою SQL-запиту
sSQL = """
SELECT *
FROM public.persons p
LIMIT 5
"""

df_soc_proc = pd.read_sql_query(sSQL, engine)
```

