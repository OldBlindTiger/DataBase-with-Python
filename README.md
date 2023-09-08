# DataBase-with-Python
Приклади створення engine для різних видів підключення, до різних типів БД

## PostgreSQL
* Підключення напряму до хоста з БД
```python
import pandas as pd
import psycopg2
from sqlalchemy import create_engine
from sqlalchemy.exc import OperationalError  # Додана операційна помилка

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

...
# Перевірка конекту
try:
	connection = soc_proc.engine.connect()
except OperationalError as e:
	print(f'[ERROR] Помилка при спробі підключитися до бази даних:\n{str(e)}')
	return -1
connection.close()

...
# Відбір даних у DataFrame за допомогою SQL-запиту
sSQL = """
SELECT *
FROM public.persons p
LIMIT 5
"""

df_soc_proc = pd.read_sql_query(sSQL, engine)
```

## MySQL
* Підключення напряму до хоста з БД
* Підключення через SSH-тунель
