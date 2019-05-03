## MySql cookbook by hangull
> Guida aggiornata alla versione 5.7

**SQL** = Structured Query Language<br>
**CRUD** = Create. Retrieve. Update. Delete<br>
**ACID** = Atomicity. Consistency. Isolation. Durability<br>
- **Atomicity** - Il concetto più fondamentale dei 4<br>
  l'esecuzione della transazione deve essere o totale o nulla, non sono ammesse esecuzioni parziali.
- **Consistency**<br>
  quando inizia una transazione il database si trova in uno stato coerente e quando la transazione termina il database deve essere in un altro stato coerente, ovvero non deve violare eventuali vincoli di integrità, quindi non devono verificarsi contraddizioni (inconsistenza dei dati ) tra i dati archiviati nel DB.
- **Isolation**<br>
  ogni transazione deve essere eseguita in modo isolato e indipendente dalle altre transazioni, l'eventuale fallimento di una transazione non deve interferire con le altre transazioni in esecuzione.
- **Durability**<br>
  detta anche **persistenza**, si riferisce al fatto che una volta che una transazione abbia richiesto un commit work, i cambiamenti apportati non dovranno essere più persi. Per evitare che nel lasso di tempo fra il momento in cui la base di dati si impegna a scrivere le modifiche e quello in cui li scrive effettivamente si verifichino perdite di dati dovuti a malfunzionamenti, vengono tenuti dei registri di log dove sono annotate tutte le operazioni sul DB.
> InnoDB = ACID; MyISAM = not ACID

Table of Contents
---
---
- [Clients](#clients)
- [mysql CLI commands](#mysql-cli-commands)
  - [Engine oriented commands](#engine-oriented-commands)
  - [Modes](#modes)
  - [Ofted used even by the server that communicate with the DB (CRUD)](#ofted-used-even-by-the-server-that-communicate-with-the-db-crud)
  - [Altering existing schemas](#altering-existing-schemas)
    - [Adding indexes](#adding-indexes)
- [Null and Not Null](#null-and-not-null)
- [Primary Keys](#primary-keys)
  - [Auto Increment](#auto-increment)
- [Select statement (focus)](#select-statement-focus)
  - [Inline views](#inline-views)
  - [Subqueries](#subqueries)
- [General](#general)
  - [Importing and exporting data](#importing-and-exporting-data)
    - [CLI](#cli)
  - [Aggregate functions](#aggregate-functions)
  - [Operators](#operators)
    - [like](#like)
    - [Logical operators](#logical-operators)
  - [Variables](#variables)
- [Data Types](#data-types)
  - [char vs varchar](#char-vs-varchar)
  - [bit](#bit)
  - [blob](#blob)
  - [enum](#enum)
- [Foreign Keys](#foreign-keys)
  - [CONSTRAINTS: Cascade and Restrict](#constraints-cascade-and-restrict)
- [ER Diagrams](#er-diagrams)
  - [One to many](#one-to-many)
  - [Many to many](#many-to-many)
- [Joins and cartesian product](#joins-and-cartesian-product)
  - [Cartesian Product](#cartesian-product)
  - [Inner Join](#inner-join)
  - [Left and Right Outer Joins](#left-and-right-outer-joins)
  - [Self join](#self-join)
- [Creating users](#creating-users)
- [Creating views](#creating-views)
- [Transactions](#transactions)
  - [Exclusive Table Locks (locking tables)](#exclusive-table-locks-locking-tables)
  - [Transactions (non MyISAM)](#transactions-non-myisam)
    - [Row locking and isolation (InnoDB)](#row-locking-and-isolation-innodb)
  - [ACID Isolation levels](#acid-isolation-levels)
# Clients
- mysql CLI
- PhpMyAdmin
- MySqlWorkbench
# mysql CLI commands
Comandi usabili anche dalle soluzioni grafiche (che forniscono anche una console)
```sql
show databases; /* Mostra i database sul server mysql */
create database name; /* Crea un nuovo db */
use name; /* Usa il database 'name' per i successivi comandi */
create table name (column1 bit, column bool default false, ecc...); /* Crea una tabella nel db */
drop table name; /* Elimina la tabella nel db */
/* Esempi di tipi di dati:
text/longtext ecc. (simile a BLOB; per conservare testi molto lunghi), int, char(7)... */
show tables; /* Elenco delle tabelle nel db */
show index in tablename; /* Mostra gli indici della tabella (vedere sotto per la definizione di indice) */
desc tablename; /* Dettagli sulla struttura della tabella */
quit /* Esce dalla CLI */
select year(now()), time(now()), date(now()); /* query che restituisce i dati sull'orario */
select 4*4; /* (16) è possibile fare dei calcoli aritmetici in sql (non è mai molto utile, forse in alcuni casi se si vogliono sommare ecc. due colonne...) */
```
## Engine oriented commands
Volendo le impostazioni, relative agli engine e non, possono anche essere impostate a mano dal file **my.cnf**
```sql
show engines; /* mostra gli engine, ovvero i moduli che costruiscono le tabelle e decidono cosa posso e non posso fare */
/* nota: MyISAM è molto outdated, non supporta transctions e foreign keys per esempio */
/* non avendo le transactions però è più veloce */
show table status; /* Dati tecnici sull tabelle del database */
create table name (column char(10)) engine="MYISAM"; /* Crea una tabella nel db specificando un engine */
set default_storage_engine="MYISAM"; /* Cambia l'engine di default */
```
## Modes
Sono delle impostazioni, possono avere valori globali o locali<br>
Se le si setta da CLI o comunque senza modificare il file di config il cambiamento non è salvato per le prossime sessioni
```sql
/* La seguente sintassi è molto specifica di mysql, per altre modes vedere documentazione */
select @@GLOBAL.SQL_MODE, @@SESSION.SQL_MODE;
select @@SESSION.SQL_SAFE_UPDATES; /* boolean che blocca operazioni pericolose se settato ad 1 */
set sql_safe_updates=0; /* permette l'aggiornamento di più di un campo alla volta */
set sql_mode = "STRICT_ALL_TABLES"; /* Setta assieme sia global che session values a "STRICT_ALL_TABLES" */
```
## Ofted used even by the server that communicate with the DB (CRUD)
```sql
insert into table (column) values ("text", ecc...); /* Inserisce valori nella tabella (C) */
/* nota: se non specifico tutte le colonne le altre sono messe a null o se sono valori non nullabili la gestione del caso varia a seconda del mode impostato (strict_all_tables fa fallire l'inserimento) */
insert into table (column) values ("text", ecc...), ("text2", ecc...), ... /* Inserimento di molteplici valori nella stessa query */
select * from table; /* Seleziona tutti i valori della tabella (R) */
update users set name = 'bill' where id=6; /* Aggiorna i valori delle righe che rispettano la condition (U) */
delete from table; /* Cancella tutta la tabella (fallisce se si è in SQL_SAFE_UPDATES = 1) (D) */
delete from table where id=1; /* funziona in ogni caso essendo che ne cancella solo 1 alla volta (D) */
```
## Altering existing schemas
```sql
alter table user add column email varchar(50) not null; /* aggiunge una colonna (in fondo) */
alter table user add column newcol first; /* aggiunge newcol in cima alle colonne */
alter table user add column newcol after id; /* aggiunge la colonna dopo la colonnas pecificata */
alter table user drop column newcol; /* rimuove colonna */
/* ADDING FOREIGN KEY */
alter table book add constraint fk_library foreign key (library) references library(id);
```
### Adding indexes
Aggiungendo un indice ad una tabella noi aggiungiamo una copia sortata in base a quel campo, questo rende le ricerche in base a quel campo davvero molto più veloci:
```sql
select * from music where band = "tot" /* 0.2sec */
alter table music add index idx_band(band) /* Aggiunta di un indice sulla colonna */
select * from music where band = "tot" /* 0.0002sec */
alter table music drop index idx_band /* index rimosso */

alter table music add index idx_band(band, song, ...) /* Multiple columns */
select * from music where band = "tot" /* 0.0002 */
select * from music where band = "tot" and song = "tot" /* 0.00021 */
select * from music where song = "tot" /* 0.2 l'index ha priorità su band! */
/* se si ha un index su (a, b, c) senza a e b la ricerca di c sarà comunque lenta */
```
# Null and Not Null
Per evitare che un valore assuma il valore nullo lo si deve specificare nella creazione
```sql
create table name (column typeofcolumn not null, ecc...); /* Crea una tabella nel db */
```
# Primary Keys
Campo univoco e su cui ci si basa per l'indexabilità
```sql
create table name (id int primary key, ecc...); /* Crea una tabella con PK nel db */
```
## Auto Increment
Permettere a mysql di "inventare per te" gli ID, non sempre saranno in semplice ordine crescente (dipende dalle impostazioni)
```sql
/* sintassi specifica di mysql */
create table name (id int primary key auto_increment, ecc...); /* Crea una tabella con PK nel db */
```
# Select statement (focus)
Concetti chiave
- where
- aggregate functions
- order by
- limit
- distinct
- group by - having
- union - union all
```sql
select id, name as nome from users where id=1; /* Condizione sulla row + naming alternativo al campo 'name' */
select count(*) from users; /* conta il numbero di utenti */
select * from users order by name, age desc; /* Ordino gli utenti per nome (asc by default) e in caso di nome uguale ordino per età decrescente */
select * from users limit 1000; /* Seleziona i primi 1000 utenti (offset a 0 by default) */
select * from users limit 500. 1000; /* Seleziona 1000 utenti dopo i primi 500 (offset a 500) */
select distinct name, age from users; /* Seleziona gli utenti non ripetendo i valori se ho delle righe con età E NOME uguali */
select count(distinct name) from users; /* Conta i differenti nomi memorizzati */
select gender, count(*), avg(weight) from survey group by gender; /* Stampa il peso medio di ogni sesso, e specifica il numero di esemplari del sesso */
explain <query> /* non esegue la query ma spiega come viene eseguita */
```
**having**, usata per porre condizioni con le **aggregate functions** (non supportate da where)
```sql
select gender, avg(weight) from survey where avg(weight) > 100 group by gender; /* error */
select gender, avg(weight) from survey having avg(weight) > 100 group by gender; /* correct */
```
naming recap
```sql
select name as nome from users u;
/* naming della colonna + naming della tabella (utile ad esempio per fare u.id al posto di users.id in query composte) */
```
**Union and union all**
```sql
select * from users where id <= 2
union
select * from users where id > 1
/* unisce i risultati delle due query, eliminando i duplicati (e.g. id=2 sarebbe un duplicato) */
/* peggiori performance, a causa del controllo */

select * from users where id <= 2
union
select * from users where id > 1
/* unisce i risultati delle due query, tenendo i duplicati (e.g. id=2 sarebbe un duplicato) */
/* migliori performance, assenza del controllo */
```
## Inline views
è possibile generare "on the fly" delle tabelle da usare in una query inline, in questo modo:
```sql
select * from (select id, name from users where id > 5) as users_grater_5
where name like "%a" /* e la * stamperà solo id e name */
```
## Subqueries
In mysql è possibile fare delle query innestate tramite l'utilizzo di **in**
```sql
select * from users where id in (1, 2, 3) /* in usato con una lista di valori */
select * from users where id in (select id from users) /* subquery di esempio (equivalente ad una semplice select *) */
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
### like
  verifica se il valore string contiene quel valore:<br>
  **dono like "%on"** la stringa non soddisfa il like; **"%on%"** la stringa soddisfa il like
### Logical operators
- AND
- OR
- NOT
- XOR (exclusive or, una delle due deve per forza essere falsa)
## Variables
Esempio utile di utilizzo di una variabile in una query
```sql
/* dichiarazione ed inizializzazione (SQL server necessita di una inizializzazione diversa) */
set @user = 'handgull';
/* query che utilizza la variabile */
select * from users where name = @user

/* Settare variabili all'interno di una select */
select @max := max(value), @min := min(value) from values; /* struttura simile, se non per i due punti */
/* eseguendo la query stampo i risultati come se fose una normale query ed in più salvo i valori in delle variabili */
```
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
insert into table (bitfield) values (b'00000111'); /* Inserisco un valore binario (7) */
select bin(bitfield) from table;
/* mi restituisce in formato binario il campo 'bitfield'. (111) senza bin() sarebbe stato 7 */
```
## blob
**blob** = Binary Large Object<br>
tramite questo tipo si possono conservare grandi file nel db, ad esempio immagini o file pdf, spesso è consigliabile conversare file di questo tipo non in un server sql ma su un server apposito, conservando solo le URI dei file.
## enum
Utile per limitare i possibili valori di un campo
```sql
create table name (col enum('hot', 'cold') default 'hot'); /* Crea una tabella con enumerator e default value */
/* In questo caso solo 'hot' e 'cold' sono valori possibili per il campo col */
```
# Foreign Keys
Servono per collegare le tabelle tra di loro, (in realtà potrei collegarle anche senza, ma grazie alle foreign keys viene garantita **l'integrità**).
```sql
create table name (id int primary key auto_increment, id_user, foreign key (id_user) references user(id));
```
## CONSTRAINTS: Cascade and Restrict
Di default una foreign key ha restrict sia on delete che on update
```sql
/* Con restrict io impedisco l'eliminazione/modifica dell'id di righe referenziate in altre tabelle */
create table name (id int primary key auto_increment, id_user, foreign key (id_user) references user(id) on delete restrict on update restrict);
/* Con cascade io permetto l'eliminazione/modifica dell'id di righe referenziate in altre tabelle, cancellando/modificando anche le righe che le referenziano */
create table name (id int primary key auto_increment, id_user, foreign key (id_user) references user(id) on delete cascade on update cascade);
```
> NOTA: per altre costraints delle foreign key guardare la documentazione (RESTRICT | CASCADE | SET NULL | NO ACTION)
# ER Diagrams
Diagrammi per progettare la struttura del DB, la workbench ha un sistema che dallo schema ER genera sql, et viceversa (reverse enginering)
## One to many
Foreign key messa in una delle due tabelle
## Many to many
Foreign key messe in una tabella a parte:
```sql
create table person_product (id_person int not null, id_product int not null,
foreign key (id_person) references person(id),
foreign key (id_product) references product(id));
/* spesso in questo caso uso una multiple join */
```
# Joins and cartesian product
## Cartesian Product
```sql
/* prodotto cartesiano, ogni riga della prima tabella viene combinata con ogni riga della seconda */
select * from person, address;
```
## Inner Join
```sql
/* join funzionante ma non fatta nel modo migliore (deprecato in alcuni db, non in mysql)*/
select * from person p, address a where p.id_address = a.id;
/* inner join nel modo migliore e più leggibile e più difficile da sbagliare */
select * from person p join address a on p.id_address = a.id;
select * from person p inner join address a on p.id_address = a.id; /* Identico, inner è usato di default nelle join */
select * from person p inner join address a on p.id_address = a.id join data d on p.id_data = d.id ... /* Multiple join */
/* NOTA: volendo si possono combinare join, left join ecc. */
```
## Left and Right Outer Joins
La left join stampa oltre alle righe che rispettano la condizione, anche le righe (della tabella di sinistra) che non hanno nessuna corrispondenza (null in ogni colonna della tabella di destra).<br>
Stessa cosa vale per la right join, vengonostampate tutte le righe dela tabella di destra e dove non si ha una corrispondenza si ha null in ogni colonna della tabella di sinistra
```sql
select * from person p left join address a on p.id_address = a.id; /* left outer join (outer è opzionale) */
select * from person p right outer join address a on p.id_address = a.id; /* Right outer join (outer è opzionale) */
```
## Self join
**JOB INTERVIEW QUESTION**: come capire quali posti sono liberi ed hanno affianco un posto libero:
```sql
select s1.id from seats s1 join seats s2 on s2.id = s1.id+1 where s1.free = true and s2.free = true;
```
> NOTA: premettendo che gli id sono trutti adiacenti e l'auto_increment è di 1 (in un caso reale è meglio evitare una situazione del genere)
# Creating users
Aavere un utente diverso da root e con diritti diversi è utile nel caso di un hacker, in questo modo i danni possono essere minori
```sql
create user 'username'@'localhost' identified by 'password' /* Il nuovo utente non ha permessi */
grant all privileges database_name.table_name to 'username'@'localhost'
flush privileges /* refresha i privilegi, ma bisogna avere il diritto RELOAD per farlo */
/* *.* for all databases/tables */
select * from mysql.users /* Lista con informazioni di tutti gli utenti */
drop user username@localhost /* cancella utente */
/* GRANT SPECIFIC PRIVILEGES (guardare la documentazione per l'elenco completo)*/
grant select database.* to username@localhost /* sa il diritto di fare select sul database */
```
# Creating views
Creando delle view noi abbiamo delle viste, che si possono **aggiornare** se coinvolgono SOLO una tabella, una vista in pratica è una simil-tabella, la sua struttura è definita da una query, e può coinvolgere n tabelle:
```sql
show full tables /* mostra tabelle e views specificando cosa è cosa */
create view newview as select * from person p join address a on p.id_address = a.id;
create algorithm=merge view newview as select * from person p join address a on p.id_address = a.id; /* algoritmo più efficente, prende i dati direttamente dalla query */
create algorithm=temptable ... /* meno efficente, crea una tabella temporanea e fa le operazioni su quella */
create algorithm=undefined ... /* usa la miglior soluzione */

create view as select * from users where id > 100 with check option /* controlla che gli inserimenti futuri rispettino la condizione */
```
> NOTA: se inserisco nella view inserisco anche nella table, solo se però gli altri valori sono nullable (se la view ha algorithm=merge, se no i valori cambiano solo nella view)

> NOTA: un bel vantaggio delle views sarebbe garantire i privilegi all'user solo della views e non della tabella intera 
# Transactions
## Exclusive Table Locks (locking tables)
I table lock sono molto simili alle transaction, il risultato infatti è una specie di "fake transaction"<br>
Sono utili ad esempio se l'engine non supporta le transaction. (e.g. MYISAM), se si usa INNODB i table lock non sono la miglior soluzione.<br>
Perchè bloccare una tabella? Nel caso in cui ho delle modifiche pesanti (write) o voglio evitare modifiche da qualsiasi connessione (read).<br><br>
Esistono 2 tipi di lock READ e WRITE:
- **read**<br>
  la connessione che ottiene un lock in lettura ha la garanzia che nessuno (nemmeno lei) potrà compiere operazioni di aggiornamento sulla tabella sino a quando il lock non verrà rilasciato: **nessuno può modificare i dati, ma tutti hanno facoltà di accedervi** in lettura
- **write**<br>
  la connessione che ottiene un lock in scrittura ha la possibilità di leggere e modificare i dati in esclusiva; sino a quando il lock non verrà rilasciato, infatti, nessun altro potrà accedere a quei dati **ne in lettura ne in scrittura**
```sql
lock tables tabel1 write, table2 read, ... /* Posso bloccare n tabelle */
unlock tables /* Libero tutte le tabelle bloccate */
```
## Transactions (non MyISAM)
Usando InnoDB di default le transaction si autocommittano in caso di successo, per studiarne il funzionamento disattiviamo l'autocommit:
```sql
set autocommit=0; /* evito che le transactions si committino in caso di successo */
insert into books (name) values ("name1");
insert into books (name) values ("name2");
select * from books /* I nuovi valori sono visibili, ma solo in questa connessione */
commit; /* applico i cambiamenti */
set autocommit=1;

/* in alternativa posso bloccare l'autocommit solo su una transaction avviandola esplicitamente: */
start transaction; /* evito che questa transactions si committi in caso di successo */
insert into books (name) values ("name1");
savepoint first_insert; /* salvo la situazione fino a qui */
insert into books (name) values ("name2");
select * from books /* I nuovi valori sono visibili, ma solo in questa connessione */
rollback to first_insert; /* torno alla situazione salvata */
rollback; /* annullo tutti i cambiamenti */
```
### Row locking and isolation (InnoDB)
Un buon caso d'uso: **account transfer**<br>
Devo trasferire del denaro da un account all'altro, e gli account sono, giustamente, nella stessa tabella. Le transactions sono essenziali per garantire l'atomicità dell'operazione (evitare che dei soldi spariscano o compaiano a causa di un crash). soluzione? **row locking**
- **row write lock esplicito** (vedi sotto)
- **row read lock esplicito (aka share lock)**
```sql
select id from libraries where name=@name lock in share mode
```
## ACID Isolation levels
Le transactions applicano il row lock, cioè si applica a specifiche row della tabella, nello specifico il lock si applica **agli index delle row**. Questo rende InnoDB più efficente di MyISAM se diverse transactions modificano diverse rows, non bloccando l'intera tabella.
> NOTA: per non bloccare l'intera tabella le select fatte nella transaction devono coinvolgere quindi dei campi indexati!
```sql
select * from users where name="handgull" /* se name non è indexato, il lock è globale a tutta la tabella */
```
```sql
select @@session.tx_isolation /* Per sapere qual'è il livello di isolazione ACID in uso */
set session transaction isolation level repeatable read; /* setta il livello di isolazione */
/* NOTA: può essere settato anche nel file di config, come tutte le variabili di mysql */
```
In ordine di livello di isolazione (dal più alto al più basso) (più basso = meno feature ma più veloce) (i livelli più alti contengono anche i più bassi)
- **Serializable**<br>
  Fornisce un livello di isolazione molto alto: finchè non finisco la transazione le rows coinvolte non possono essere modificate da altre transactions. Spesso overkill.
- **Repeatable read** - Default<br>
  Le query eseguite all'interno della transaction restituiscono sempre lo stesso risultato, anche se le rows fuori dalla transaction sono aggiornate.
> TRICK: se serve un lock sulla scrittura delle rows (come in serializable) si può usare 'for update' dopo la select: (row write lock lock esplicito)
```sql
select balance from accounts where id=@account for update
```
- **Read committed**<br>
  leggo le modifiche che sono state committate da altre transactions (ed influiscono sulla select della transaction)
- **Read uncommitted**<br>
  Più basso livello, è quasi come non usare le transactions: le modifiche da parte delle altre transaction sono viste anche se non le committo (unico vantaggio rispetto al non usarle è il rollback)