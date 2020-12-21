# Import-dati-censuari-in-PostgreSQL/PostGIS

## Breve descrizione dei dati catastali censuari.
Le informazioni descritte in questa sezione derivano dal documento a cura dell'Agenzia dell'Entrate (DOC. ES-23-IS-05) liberamente consultabile all'indirizzo: https://wwwt.agenziaentrate.gov.it/mt/ServiziComuniIstituzioni/ES-23-IS-05_100909.pdf.

Per maggiori dettagli sui servizi riservati ai comuni di può consultare: https://www.agenziaentrate.gov.it/portale/web/guest/schede/fabbricatiterreni/portale-per-i-comuni/servizi-portale-dei-comuni/estrazione-dati-catastali.


I dati censuari sono costituiti da 4 tipi di file:

- file fabbricati (.FAB);

- file terreni (.TER);

- file soggetti (.SOG);

- file titolarità (.TIT).

Ogni tipo di file è costituito da una tabella che può contenere diversi tipi di record. Il collegamento tra i tipi di file è assicurato dalla presenta di chiavi specifiche:

- .FAB/.TER contengono la chiave identificativo immobile;

- .SOG contiene la chiave identificativo soggetto;

- .TIT contiene sia la chiave identificativo immobile che la chiave identificativo soggetto (oltre che la chiave identificativo titolarietà);

## Importazione dei singoli file in PostgreSQL/PostGIS - .FAB (in costruzione).

## Importazione dei singoli file in PostgreSQL/PostGIS - .TER.
Il file è costituito da 4 differenti tipi record. La particella è identificata attraverso il campo IDENTIFICATIVO IMMOBILE. La presenza di diversi tipi di record può creare delle righe duplicate per ogni particella.

- TIPO RECORD 1: contiene le caratteristiche della particella. E' il record di interesse che verrà utilizzato per ricostruire il dato spaziale;

- TIPO RECORD 2: deduzioni della particella;

- TIPO RECORD 3: riserva della particella;

- TIPO RECORD 4: porzioni della particella.

#### 1) Creazione della tabella contenente tutti i campi (Per non creare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi).

```sql
CREATE TABLE public.ter(
  pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  codice_amministrativo TEXT,
  sezione TEXT,
  identificativo_immobile INTEGER NOT NULL,
  tipo_immobile TEXT,
  progressivo INTEGER,
  tipo_record INTEGER,
  foglio TEXT,
  numero TEXT,
  denominatore INTEGER,
  subalterno TEXT,
  edificialita TEXT,
  qualita TEXT,
  classe TEXT,
  ettari INTEGER,
  are INTEGER,
  centiare INTEGER,
  flag_reddito TEXT,
  flag_porzione TEXT,
  flag_deduzioni TEXT,
  reddito_dominicale_lire INTEGER,
  reddito_agrario_lire INTEGER,
  reddito_dominicale_euro REAL,
  reddito_agrario_euro REAL,
  data_efficacia_valore_atto TEXT,
  data_registrazione_atti_valore_atto TEXT,
  tipo_nota_valore_atto TEXT,
  numero_nota_valore_atto TEXT,
  progressivo_nota_valore_atto TEXT,
  anno_nota_valore_atto TEXT,
  data_efficacia_registrazione_atto TEXT,
  data_registrazione_registrazione_atto TEXT,
  tipo_nota_registrazione_atto TEXT,
  numero_nota_registrazione_atto TEXT,
  progressivo_nota_registrazione_atto TEXT,
  anno_nota_registrazione_atto TEXT,
  partita TEXT,
  annotazione TEXT,
  identificativo_mutazione_iniziale TEXT,
  identificativo_mutazione_finale TEXT
)
```

#### 2) Importazione dei dati.
Convertire il file .TER in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibbile che i CSV abbiano l'header definito come da [esempio.csv](csv/TER.csv)

#### 3) Creazione della vista contenente solo il TIPO RECORD 1.

```sql
CREATE OR REPLACE VIEW public.ter_1 AS
  SELECT * FROM ter
  WHERE tipo_record = '1'
```

### Pulizia della vista ter_1 (opzionale).
La Vista risultante dalla selezione del tipo_record = '1' può essere ulteriormente "pulita" eliminando quelle particelle che non hanno titolarità come la particelle soppresse, acque, strade. Le informazioni possono essere ricavate dal campo partita:

partita | descrizione
:------ | :------
1   | aree di enti urbani e promiscui
2   | accessori comuni ad enti rurali e ad enti rurali e urbani
3   | aree di fabbircati rurali, o urbani da accertare, divisi in subalterni
4   | acque esenti da estimo
5   | strade pubbliche
0   | particelle soppresse

```sql
CREATE OR REPLACE VIEW public.ter_1_clean AS
	SELECT * FROM ter_1
	WHERE ter_1.partita IS DISTINCT FROM '0' AND ter_1.partita IS DISTINCT FROM '4' AND ter_1.partita IS DISTINCT FROM '5'
```

## Importazione dei singoli file in PostgreSQL/PostGIS - .SOG.
Il file è costituito da 2 differenti tipi record. Il soggetto è identificato attraverso il campo IDENTIFICATIVO SOGGETTO.

- TIPO RECORD P: intestato a persona fisica;

- TIPO RECORD G: intestato a persona giuridica;

Per una corretta gestione del file è necessario suddividere il file .SOG in due tabella, una per ogni record. Tale operazione si può effettuare in excel, calc, ecc.

##### 1) Creazione della tabella per i soggetti fisici sogP, contenente tutti i campi (Per non crare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi);

```sql
CREATE TABLE public.sogP
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_amministrativo TEXT,
	sezione	TEXT,
	identificativo_soggetto	INTEGER NOT NULL,
	tipo_soggetto TEXT,
	cognome TEXT,
	nome TEXT,
	sesso TEXT,
	data_nascita TEXT,
	codice_amministratvio_comune_nascita TEXT,
	codice_fiscale	TEXT,
	indicazioni_supplementari TEXT
)
```

#### 2) Importazione dei dati.
Convertire il file .TER in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibbile che i CSV abbiano l'header definito come da [esempio.csv](csv/SOGP.csv)

https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

#### 3) Creazione della tabella per i soggetti giuridici sogG, contenente tutti i campi (Per non creare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi);

```sql
CREATE TABLE public.sogG
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_amministrativo TEXT,
	sezione TEXT,
	identificativo_soggetto  integer NOT NULL,
	tipo_soggetto TEXT,
	denominazione TEXT,
	codice_amministrativo_sede TEXT,
	codice_fiscale TEXT
)
```
#### 4) Importazione dei dati.
Convertire il file .TER in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibbile che i CSV abbiano l'header definito come da [esempio.csv](csv/SOGG.csv)

## Importazione dei singoli file in PostgreSQL/PostGIS - .TIT.
Il file contiene un unico tipo di record.

#### 1) Creazione della tabella titolarità contenente tutti i campi (Per non creare problemi durante l'importazione è stato scelto di importare alcuni campi numerici come testi).

```sql
CREATE TABLE public.tit
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_amministrativo TEXT,
	sezione TEXT,
	identificativo_soggetto INTEGER NOT NULL,
	tipo_soggetto TEXT,
	identificativo_immobile INTEGER NOT NULL,
	tipo_immobile TEXT,
	codice_diritto TEXT,
	titolo_non_codificato TEXT,
	quota_numeratore_possesso INTEGER,
	quota_denominatore_possesso INTEGER,
	regime TEXT,
	soggetto_riferimento TEXT,
	data_validita_atto_generato TEXT,
	tipo_nota_generato TEXT,
	numero_nota_generato TEXT,
	progressivo_nota_generato TEXT,
	anno_nota_generato TEXT,
	data_registrazione_atti_generato TEXT,
	partita TEXT,
	data_validita_atto_concluso TEXT,
	tipo_nota_concluso TEXT,
	numero_nota_concluso TEXT,
	progessivo_nota_concluso TEXT,
	anno_nota_concluso TEXT,
	data_registrazione_atti_concluso TEXT,
	identificativo_mutazione_iniziale TEXT,
	identificativo_mutazione_finale TEXT,
	identificativo_titolarita INTEGER NOT NULL
)
```

#### 2) Importazione dei dati.
Convertire il file .TER in .CSV utilizzando excel, calc, ecc.. Utilizzare la funzione di PgAdmin per l'importazione dei CSV come spiegato al seguente link:
https://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/

Per evitare errori è preferibbile che i CSV abbiano l'header definito come da [esempio.csv](csv/TIT.csv)

## Creazione delle tabella aggiuntive per la codifica dei codici.
Può risultare utile creare alcune tabelle per la codifica dei codici contenuti all'interno del record descrizione particelle.

#### Tabella delle qualità colturali.
```sql
CREATE TABLE public.qualita
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	id_qualita TEXT,
	descrizione TEXT
)
```

Per inserire i valori utilizzare la funzione di PgAdmin per l'importazione dei CSV e utilizzare il file [qualita.csv](csv/qualita.csv)

#### Tabella delle partite speciali.

```sql
CREATE TABLE public.partite_speciali_fabbricati
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_partita TEXT,
	descrizione TEXT
);
INSERT INTO public.partite_speciali_fabbricati (codice_partita, descrizione)
VALUES
('0', 'beni comuni censibili'),
('A', 'beni coumni non censibili'),
('R', 'fabbircati rurali'),
('C', 'unità immobiliari soppresse');

CREATE TABLE public.partite_speciali_terreni
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_partita TEXT,
	descrizione TEXT
);
INSERT INTO public.partite_speciali_terreni (codice_partita, descrizione)
VALUES
('1', 'aree di enti urbani e promiscui'),
('2', 'accessori comuni ad enti rurali e ad enti rurali e urbani'),
('3', 'aree di fabbircati rurali, o urbani da accertare, divisi in subalterni'),
('4', 'acque esenti da estimo'),
('5', 'strade pubbliche'),
('0', 'particelle soppresse')
```

### Tabella dei codici dirrito.

```sql
CREATE TABLE public.codici_diritto
(
	pk_id INTEGER PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
	codice_diritto TEXT,
	descrizione TEXT
);
```
Per inserire i valori utilizzare la funzione di PgAdmin per l'importazione dei CSV e utilizzare il file [diritto.csv](csv/diritto.csv)

## Creazione delle relazioni tra i tipi di file: soggetti persone fisicihe (sogp) e titolarità (tit).
Ogni immobile (particella o fabbricato) può appartenere a più titolari. Per gestire questa relazione (uno a molti) è possibile utilizzare le funzioni di aggregazione. In questo specifico caso è la scelta è ricaduta sulla creazione di un json che contiene i diversi titolari appartenenti ad un dato immobile. Il vantaggio di utilizzare il json è che questo è interrogabile. La creazione della relazione viene fatta in due step.

1) Creazione della vista. La relazione del tipo uno a molti viene esplicitata tramite il join. Il risultato duplicherà le righe relative all'immobile che appartiene a più soggetti.

```sql
CREATE OR REPLACE VIEW  tit_sogp AS SELECT
	row_number() OVER ()::integer AS gid,
	tit.identificativo_immobile,
	tit.tipo_immobile,
	tit.identificativo_soggetto,
	tit.tipo_soggetto,
	dir.descrizione as diritto,
	concat(tit.quota_numeratore_possesso, '/', tit.quota_denominatore_possesso) AS quota,
	sogp.cognome,
	sogp.nome,
	sogp.codice_fiscale,
	sogp.data_nascita
	FROM tit tit
	JOIN sogP ON tit.identificativo_soggetto = sogp.identificativo_soggetto
	JOIN codici_diritto dir ON tit.codice_diritto = dir.codice_diritto
```

2) Creazione della vista aggregata. Viene creata la colonna soggetto che contiene in un'unica riga tutti i titolari dell'immobile.

```sql
CREATE OR REPLACE VIEW tit_sogp_json AS SELECT
	row_number() OVER ()::integer AS gid,
	identificativo_immobile,
	tipo_immobile,
	json_agg
	(
		json_build_object
			(
				'identificativo_soggetto', identificativo_soggetto,
            	'cognome', cognome,
            	'nome', nome,
				'codice_fiscale', codice_fiscale,
				'data_nascita', data_nascita,
				'tipo_soggetto', tipo_soggetto,
				'quota', quota,
				'diritto', diritto
			)
	) as soggetto
FROM tit_sogp
GROUP by identificativo_immobile, tipo_immobile, tipo_soggetto
```

## Creazione delle relazioni tra i tipi di file: soggetti persone fisicihe_titolarità (tit_sogp_json) e immobili.
La creazione della relazione viene fatta in due step.

```sql
create or replace view tabella_join_soggetti_fisici AS
SELECT row_number() OVER ()::integer AS gid,
t.identificativo_immobile as identificativo_immobile_ter,
t.foglio,
t.numero,
t.denominatore,
t.descrizione_qualita,
t.descrizione_partita,
t.classe,
t.ettari,
t.are,
t.centiare,
t.fg_plla,
j.identificativo_immobile as identificativo_immobile_tit,
j.tipo_soggetto,
j.soggetto
FROM ter_fg_plla t
LEFT JOIN immobile_soggetto_pfisica_json j ON t.identificativo_immobile = j.identificativo_immobile
```

## Creazione delle relazioni tra i tipi di file: soggetti persone giuridiche (sogG) e titolarità (tit) (in costruzione).

## Creazione delle relazioni tra le geometrie catastali e i dati censuari (in costruzione).
