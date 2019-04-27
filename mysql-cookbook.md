## MySql cookbook by hangull
**SQL** = Structured Query Language<br>
**CRUD** = Create. Retrieve. Update. Delete<br>

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
  - [Aggregate functions](#aggregate-functions)
  - [Operators](#operators)
    - [Logical operators](#logical-operators)
- [Data Types](#data-types)
  - [char vs varchar](#char-vs-varchar)
  - [bit](#bit)
  - [blob](#blob)
  - [enum](#enum)
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
create table name (column1 bit, column bool default false, ecc...) # Crea una tabella nel db
drop table name # Elimina la tabella nel db
# Esempi di tipi di dati:
# text/longtext ecc. (simile a BLOB; per conservare testi molto lunghi), int, char(7)...
show tables; # Elenco delle tabelle nel db
desc tablename; # Dettagli sulla struttura della tabella
quit # Esce dalla CLI
select year(now()), time(now()), date(now()) # query che restituisce i dati sull'orario
select 4*4 # (16) è possibile fare dei calcoli aritmetici in sql (non è mai molto utile, forse in alcuni casi se si vogliono sommare ecc. due colonne...)
```
## Engine orriented commands
Volendo le impostazioni, relative agli engine e non, possono anche essere impostate a mano dal file **my.cnf**
```sql
show engines; # mostra gli engine, ovvero i moduli che costruiscono le tabelle e decidono cosa posso e non posso fare
# nota: MyISAM è molto outdated, non supporta transctions e foreign keys per esempio
show table status # Dati tecnici sull tabelle del database
create table name (column char(10)) engine="MYISAM" # Crea una tabella nel db specificando un engine
set default_storage_engine="MYISAM" # Cambia l'engine di default
```
## Modes
Sono delle impostazioni, possono avere valori globali o locali<br>
Se le si setta da CLI o comunque senza modificare il file di config il cambiamento non è salvato per le prossime sessioni
```sql
# La seguente sintassi è molto specifica di mysql, per altre modes vedere documentazione
select @@GLOBAL.SQL_MODE, @@SESSION.SQL_MODE
select @@SESSION.SQL_SAFE_UPDATES # boolean che blocca operazioni pericolose se settato ad 1
set sql_safe_updates=0 # permette l'aggiornamento di più di un campo alla volta
set sql_mode = "STRICT_ALL_TABLES" # Setta assieme sia global che session values a "STRICT_ALL_TABLES"
```
## Ofted used even by the server that communicate with the DB
```sql
insert into table (column) values ("text", ecc...) # Inserisce valori nella tabella (C)
# nota: se non specifico tutte le colonne le altre sono messe a null o se sono valori non nullabili la gestione del caso varia a seconda del mode impostato (strict_all_tables fa fallire l'inserimento)
select * from table # Seleziona tutti i valori della tabella (R)
update users set name = 'bill' where id=6 # Aggiorna i valori delle righe che rispettano la condition (U)
delete from table # Cancella tutta la tabella (fallisce se si è in SQL_SAFE_UPDATES = 1) (D)
delete from table where id=1 # funziona in ogni caso essendo che ne cancella solo 1 alla volta (D)
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
select * from users order by name, age desc # Ordino gli utenti per nome (asc by default) e in caso di nome uguale ordino per età decrescente
select * from users limit 1000 # Seleziona i primi 1000 utenti (offset a 0 by default)
select * from users limit 500. 1000 # Seleziona 1000 utenti dopo i primi 500 (offset a 500)
select distinct name, age from users # Seleziona gli utenti non ripetendo i valori se ho delle righe con età E NOME uguali
select count(distinct name) from users # Conta i differenti nomi memorizzati
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
## Aggregate functions
- count()
- max()
- min()
- avg()
- ...
> Vedi documentazione
## Operators
Come sempre vedere la documentazione per maggiori dettagli<br>
Usati nel **where**
> per un confronto con un null è consigliato usare is, is not o isnull
- **like**<br>
  verifica se il valore string contiene quel valore
  dono like "%on" la stringa non soddisfa il like; "%on%" la stringa soddisfa il like
### Logical operators
- AND
- OR
- NOT
- XOR (exclusive or, una delle due deve per forza essere falsa)
# Data Types
## char vs varchar
**char(5)** -> alloco SEMPRE 5 caratteri anche se non mi servirebbero tutti<br>
**varchar(5)** -> alloco fino a 5 caratteri, usando solo quelli necessari
> Con **varchar** ho dei vantaggi in fatto di memoria risparmiata, con **char** ho dei vantaggi in fatto di velocità di risposta, avendo dei campi di dimensione fissa rendo più efficente l'invio dei **record**.

> Ad esempio nel caso di un nome, che varia molto, è pù vantaggioso usare varchar. Nel caso di uno zip code/codice postale è più vantaggioso l'utilizzo di char
## bit
bit spesso è usato per i flag (senza parentesi appunto ha solo 2 valori: 1 e 0).<br>
> NOTA: sarebbe comunque meglio usare bool che è un sinonimo di **tinyint(1)**

Posso però assegnare più di un bit e.g. **bit(8)**<br>
```sql
insert into table (bitfield) values (b'00000111') # Inserisco un valore binario (7)
select bin(bitfield) from table
# mi restituisce in formato binario il campo 'bitfield'. (111) senza bin() sarebbe stato 7
```
## blob
**blob** = Binary Large Object<br>
tramite questo tipo si possono conservare grandi file nel db, ad esempio immagini o file pdf, spesso è consigliabile conversare file di questo tipo non in un server sql ma su un server apposito, conservando solo le URI dei file.
## enum
Utile per limitare i possibili valori di un campo
```sql
create table name (col enum('hot', 'cold') default 'hot') # Crea una tabella con enumerator e default value
# In questo caso solo 'hot' e 'cold' sono valori possibili per il campo col
```