# Copias de seguridad
## Tipos de copias de seguridad

* Frío-Caliente:
* Física o Binaria:
* Lógica o Textual:
* Completa o Incremental:
* Manual o automática:
## ¿Qué almacena cada archivo?

*.frm: contiene la estructura de las tablas. Hay tantos archivos .frm como tablas. Estos archivos se usan para las tablas de tipo InnoDB y MyISAM.
*.ibd: almacena los datos y los índices de las tablas de tipo InnoDb. Hay tantos archivos .ibd como tablas tenga nuestra base de datos.
*.myd: almacena los datos de las tablas de tipo MyISAM. Hay tantos archivos como tablas tenga la base de datos.
*.myi: almacena los índices de las tablas de tipo MyISAM. Hay tantos archivos como tablas en nuestra base de datos.

## Copias de seguridad físicas o binarias
### ¿Como realizar y restaurar una copia binaria MyISAM?
Consultar la sección: 32.3.1 Making Binary MyISAM Backups del documento MySQL 5.0 Certification Study Guide.
### ¿Como realizar y restaurar una copia binaria InnoDB?
Consultar la sección: 32.3.2 Making Binary InnoDB Backups del documento MySQL 5.0 Certification Study Guide.
```sql
-- Paso 1
USE tienda;

-- Paso 2
SHOW TABLES;

-- Paso 3
SELECT *
FROM producto;

-- This forces the server to close the table and provides your connection with a read lock on the table.
FLUSH TABLES producto FOR EXPORT;

-- Paso 4
-- Hacemos una copia del archivo producto.ibd

-- Then, once you've copied the files, you can release the lock with UNLOCK TABLES:
UNLOCK TABLES;

-- Paso 5
DELETE
FROM producto;

-- Paso 6
SELECT *
FROM producto;

-- Paso 7
ALTER TABLE tienda.producto DISCARD TABLESPACE;

-- Paso 8
-- Restauramos el archivo producto.idb
-- Recuerda que en Linux el propietario y el grupo del archivo tiene que ser mysql.

-- Paso 9
ALTER TABLE tienda.producto IMPORT TABLESPACE;


-- Paso 10
SELECT *
FROM producto;
```
## Copias de seguridad lógicas o textuales
### SELECT ... INTO OUTFILE
La sentencia SELECT ... INTO OUTFILE nos permite realizar copias de seguridad lógicas o textuales, exportando los datos en diferentes formatos de texto.
```sql
-- Paso 1. Seleccionamos una base de datos
USE tienda;

-- Paso 2. Mostramos las tablas de la base de datos
SHOW TABLES;

-- Paso 3. Intentamos exportar todos los datos de la tabla fabricante
SELECT *
INTO OUTFILE 'fabricante.txt'
FROM fabricante;

-- Paso 4. Obtendremos el siguiente mensaje de error
-- Error Code: 1290. The MySQL server is running with the --secure-file-priv option so it cannot execute this statement

-- Paso 5. Consultamos la variable secure_file_priv para obtener la ruta donde se guardarán los archivos que generamos con SELECT .. INT OUTFILE
SHOW VARIABLES LIKE 'secure_file_priv';

-- Paso 6. Añadimos la ruta al nombre del archivo
SELECT *
INTO OUTFILE '/var/lib/mysql-files/fabricante.txt'
FROM fabricante;

-- Paso 7. Añadimos algunos modificadores para mejorar la salida
SELECT *
INTO OUTFILE '/var/lib/mysql-files/fabricante.txt'
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n'
FROM fabricante;

-- Paso 8.
-- ¿Qué error obtenemos en el paso anterior? 
-- ¿Es posible reescribir los archivos con SELECT .. INTO OUTFILE? 
-- ¿Cómo podemos resolverlo?
```
### LOAD DATA
La sentencia LOAD DATA nos permite restaurar copias de seguridad lógicas o textuales, importando los datos desde un archivo de texto a nuestra base de datos.
LOAD DATA
    [LOW_PRIORITY | CONCURRENT] [LOCAL]
    INFILE 'file_name'
    [REPLACE | IGNORE]
    INTO TABLE tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [CHARACTER SET charset_name]
    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string']
        [[OPTIONALLY] ENCLOSED BY 'char']
        [ESCAPED BY 'char']
    ]
    [LINES
        [STARTING BY 'string']
        [TERMINATED BY 'string']
    ]
    [IGNORE number {LINES | ROWS}]
    [(col_name_or_user_var
        [, col_name_or_user_var] ...)]
    [SET col_name={expr | DEFAULT}
        [, col_name={expr | DEFAULT}] ...]
```sql
-- Paso 1. Seleccionamos la base de datos tienda
USE tienda;

-- Paso 2. Creamos una tabla nueva llamada fabricante_backup
CREATE TABLE fabricante_backup (
  codigo INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(100) NOT NULL
);

-- Paso 3. Importamos los datos del archivo /var/lib/mysql-files/fabricante.txt en la nueva tabla
LOAD DATA INFILE '/var/lib/mysql-files/fabricante.txt'
INTO TABLE fabricante_backup
FIELDS TERMINATED BY ',';

-- Paso 4. Comprobamos que los datos se han insertado correctamente
SELECT *
FROM fabricante_backup;
```
### mysqldump
La utilidad mysqldump permite realizar copias de seguridad lógicas o textuales de una base de datos MySQL.
Existen tres formas de usar mysqldump:

Exportar una o varias tablas de una base de datos,
mysqldump [options] db_name [tbl_name ...]
Exportar una o varias bases de datos completas,
mysqldump [options] --databases db_name ...
Exportar todas las bases de datos completas.
mysqldump [options] --all-databases
### Exportar una o varias tablas de una base de datos
mysqldump -u [username] -p [database_name] [tbl_name ...] > [backup_name].sql
Ejemplo: mysqldump -u root -p wordpress > backup.sql
En este ejemplo estamos exportando todas las tablas de la base de datos wordpress y estamos guardando la salida con las sentencias SQL en un archivo llamado backup.sql.
*Nota importante:* En este caso no se incluye la sentencia CREATE DATABASE en el archivo de backup. Sólo se generan sentencias de tipo CREATE TABLE y INSERT.

