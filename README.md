# Javazone2023-FREG
Folkeregisteret konkurranse på JavaZone 2023 i Oslo.

---
### Starte opp Docker og Neo4j fra kommandolinjen:
Start opp image (i bakgrunnen) (med porter) (uten passord auth)
```
docker run -d -p=7474:7474 -p=7687:7687 -v=$HOME/neo4j/data:/data --env NEO4J_AUTH=none neo4j
```
Kopier fil fra maskin til container
```
 docker cp <filename.csv> <containerId>:/var/lib/neo4j/import
```
Start opp container og gå inn i image
Start container (opp igjen)
Stop container
Gå ut av container
```
docker exec -it <containerId> /bin/bash
docker start <containerId>
docker stop <containerId>
exit
```
Sjekk alle/aktive containere
Sjekk alle/aktive images
```
docker ps -a
docker images -a
```
* Åpne applikasjonen i nettleseren: http://localhost:7474/browser/
---

## Import av data til Neo4j UI
_(Filene kan lastes ned fra 'Resources'. Husk å importer 'Persondata' først, så 'relasjonene' - ellers blir ikke relasjonene koblet riktig med personene.)_
### Persondata
```
LOAD CSV WITH HEADERS FROM "file:///personer.csv" AS row
create(p:Person {foedselsnummer: row.fnummer, personstatus:row.personstatus,kjoenn:row.kjoenn,foedselsdato:date(row.foedselsdato),sivilstand:row.sivilstand,navn:row.fornavn + " " + row.etternavn, postnummer:row.postnummer})
```
### Relasjoner
**FAMILIERELASJON** 
_(denne bruker litt lengre tid på å laste inn...)_
```
LOAD CSV WITH HEADERS FROM "file:///relasjoner.csv" AS row
match (p:Person {foedselsnummer: row.fnummer1}),(q:Person {foedselsnummer: row.fnummer2})
create (p)-[:FAMILIERELASJON {rolle: row.fnummer1_rolle,ergjeldende:toBoolean(row.ergjeldende),gyldighetstidspunkt:date(row.gyldighetsdato)}]->(q)
```
**FORELDREANSVAR**
```
LOAD CSV WITH HEADERS FROM "file:///foreldreansvar.csv" AS row
match (p:Person {foedselsnummer: row.ansvarlig}),(q:Person {foedselsnummer: row.ansvarssubjekt})
create (p)-[:FORELDREANSVAR {ansvarstype: row.ansvarstype,ergjeldende:toBoolean(row.ergjeldende),gyldighetstidspunkt:date(row.gyldighetsdato)}]->(q)
```
**SIVILSTAND**
```
LOAD CSV WITH HEADERS FROM "file:///sivilstand.csv" AS row
match (p:Person {foedselsnummer: row.hovedperson}),(q:Person {foedselsnummer: row.ektefelleEllerPartner})
create (p)-[:SIVILSTAND {sivilstand: row.sivilstand,ergjeldende:toBoolean(row.erGjeldende),sivilstandsdato:date(row.sivilstandsdato),myndighet:row.myndighet}]->(q)
```
---
## Cypher queries
**Hent alle personer i databasen**
```cypher
match (n) return n
```
**Hent en personer med fnr**
```cypher
match (n:Person{foedselsnummer:'12312312312'}) return n
```
**Hent en personer med navn**
```cypher
match (n:Person{navn:'Navn Navnesen'}) return n
```
**Telle antall relasjoner** _(velg type)_
```cypher
match (:Person) - [r:FAMILIERELASJON] -> (:Person) with count(r) as relasjoner return relasjoner
```
**Gift en personer med en annen**
<br/>_SIVILSTAND_
```cypher
match (d:Person{foedselsnummer:'98765432101'}), (l:Person{foedselsnummer:'12312312312'}) create (d) - [:SIVILSTAND{sivilstand:'GIFT', ergjeldende:TRUE, gyldighetstidspunkt:date('2010-01-01')}] -> (l) 
```
_FAMILIERELASJON_
```cypher
match (d:Person{foedselsnummer:'98765432101'}), (l:Person{foedselsnummer:'12312312312'}) create (d) - [:FAMILIERELASJON{rolle:'EKTEFELLE_ELLER_PARTNER', ergjeldende:TRUE, gyldighetstidspunkt:date('2010-01-01')}] -> (l)
```
**Lag Person**
```cypher
create (p:Person{foedselsnummer:'10123456789', foedselsdato:date('1923-09-6'), personstatus:'BOSATT', navn:'Navn Navnesen', kjoenn:'MANN', postnummer:'1234'})
```
**Slett en relasjon mellom to personer**
```cypher
match (p:Person{foedselsnummer:'12312312312'}) - [r:FAMILIERELASJON{rolle:'BARN'}] -> (q:Person) delete r
```
```cypher
match (p:Person{foedselsnummer:'12312312312'}) <- [r:FORELDREANSVAR{ansvarstype:'FELLES'}] - (q:Person) delete r
```
**Slett en person med tilhørende relasjoner**
```cypher
match (p:Person{navn:'Navn Navnesen'}) detach delete p
```
**Slett hele tabellen**
```cypher
match (n) detach delete n
```
---
### Annet
* GitHub Ronny: https://github.com/ronnyma/jz_nta.
* Cheatsheet Neo4j Enterprise: https://neo4j.com/docs/cypher-cheat-sheet/5/auradb-enterprise/
