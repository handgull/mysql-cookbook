## MySql cookbook by hangull
**SQL** = Structured Query Language
Table of Contents
---
---
- [Clients](#clients)
- [mysql CLI commands](#mysql-cli-commands)
  - [Engine orriented commands](#engine-orriented-commands)
  - [Modes](#modes)
  - [Ofted used even by the server that communicate with the DB](#ofted-used-even-by-the-server-that-communicate-with-the-db)
- [Null and Not Null](#null-and-not-null)
- [Primary Keys](#primary-keys)
  - [Auto Increment](#auto-increment)
- [Select statement (focus)](#select-statement-focus)
- [General](#general)
  - [Importing and exporting data](#importing-and-exporting-data)
    - [CLI](#cli)
- [Operators](#operators)
  - [Logical operators](#logical-operators)
# Clients
- mysql CLI
- PhpMyAdmin
- MySqlWorkbench
# mysql CLI commands
Comandi usabili anche dalle soluzioni grafiche (che forniscono anche una console)
```sql
show databases; # Mostra i database sul server mysql
create database name # Crea un nuovo db
use name; # Usa il database 'name' per i successivi comandi
create table name (column typeofcolumn, ecc...) # Crea una tabella nel db
drop table name # Elimina la tabella nel db
# Esempi di tipi di dati:
# text, int, ...
show tables; # Elenco delle tabelle nel db
desc tablename; # Dettagli sulla struttura della tabella
quit # Esce dalla CLI
```
## Engine orriented commands
Volendo le impostazioni, relative agli engine e non, possono anche essere impostate a mano dal file **my.cnf**
```sql
show engines; # mostra gli engine, ovvero i moduli che costruiscono le tabelle e decidono cosa posso e non posso fare
# nota: MyISAM è molto outdated, non supporta transctions e foreign keys per esempio
show table status # Dati tecnici sull tabelle del database
create table name (column typeofcolumn) engine="MYISAM" # Crea una tabella nel db specificando un engine
set default_storage_engine="MYISAM" # Cambia l'engine di default
```
## Modes
Sono delle impostazioni, possono avere valori globali o locali<br>
Se le si setta da CLI o comunque senza modificare il file di config il cambiamento non è salvato per le prossime sessioni
```sql
# La seguente sintassi è molto specifica di mysql, per altre modes vedere documentazione
select @@GLOBAL.SQL_MODE, @@SESSION.SQL_MODE
select @@SESSION.SQL_SAFE_UPDATES # boolean che blocca operazioni pericolose se settato ad 1
set sql_mode = "STRICT_ALL_TABLES" # Setta assieme sia global che session values a "STRICT_ALL_TABLES"
```
## Ofted used even by the server that communicate with the DB
```sql
insert into table (column) values ("text", ecc...) # Inserisce valori nella tabella
# nota: se non specifico tutte le colonne le altre sono messe a null o se sono valori non nullabili la gestione del caso varia a seconda del mode impostato (strict_all_tables fa fallire l'inserimento)
select * from table # Seleziona tutti i valori della tabella
delete from table # Cancella tutta la tabella (fallisce se si è in SQL_SAFE_UPDATES = 1)
delete from table where id=1 # funziona in ogni caso essendo che ne cancella solo 1 alla volta
```
# Null and Not Null
Per evitare che un valore assuma il valore nullo lo si deve specificare nella creazione
```sql
create table name (column typeofcolumn not null, ecc...) # Crea una tabella nel db
```
# Primary Keys
Campo univoco e su cui ci si basa per l'indexabilità
```sql
create table name (id int primary key, ecc...) # Crea una tabella con PK nel db
```
## Auto Increment
Permettere a mysql di "inventare per te" gli ID, non sempre saranno in semplice ordine crescente (dipende dalle impostazioni)
```sql
# sintassi specifica di mysql
create table name (id int primary key auto_increment, ecc...) # Crea una tabella con PK nel db
```
# Select statement (focus)
```sql
select id, name from users where id=1 # Condizione sulla row
select count(*) from users # conta il numbero di utenti
```
# General
## Importing and exporting data
- Alternative grafiche ed intuitive
### CLI
```bash
mysqldump -uroot -hlocalhost database # export dello script schema + dati del database
# nota meglio usarlo in combo con l'operatore bash > per salvare l'output in un file
mysql -uroot -hlocalhost newdbname < backup.sql # import e creazione del nuovo database
```
# Operators
Come sempre vedere la documentazione per maggiori dettagli<br>
Usati nel **where**
> per un confronto con un null è consigliato usare is, is not o isnull
- **like**<br>
  verifica se il valore string contiene quel valore
  dono like "%on" la stringa non soddisfa il like; "%on%" la stringa soddisfa il like
## Logical operators
- AND
- OR
- NOT
- XOR (exclusive or, una delle due deve per forza essere falsa)
