# JavaZone Skatteetaten
Skatteetaten / Folkeregisteret på JavaZone i Oslo.

---
### Opprett og start opp Neo4j Docker-container fra kommandolinjen:
Start opp container _-d (i bakgrunnen) -p (med porter) --env (uten passord)_
```
docker run -d -p=7474:7474 -p=7687:7687 -v=$HOME/neo4j/data:/data --env NEO4J_AUTH=none neo4j
```
Kopier filer fra maskinen til container _(husk riktig fil-sti)_
```
docker cp </path/filename.csv> <containerId>:/var/lib/neo4j/import
```
Sjekk alle/ _-a_ aktive containere
<br/>Sjekk alle/ _-a_ aktive images
```
docker ps -a
docker images -a
```
Start ny container og gå inn i den
<br/>Stop container
<br/>Start container _(igjen)_
<br/>Gå ut av container
```
docker exec -it <containerId> /bin/bash
docker stop <containerId>
docker start <containerId>
exit
```
Åpne applikasjonen i nettleseren: http://localhost:7474/browser/

---

## Import av data til Neo4j Dashboard
_Filene kan lastes ned fra **Resources** katalogen i dette prosjektet. Husk å importer **Personer.csv** først, 
deretter relasjonene - ellers blir ikke relasjonene koblet riktig med personene._
### Persondata
```
LOAD CSV WITH HEADERS FROM "file:///personer.csv" AS row
create(p:Person {foedselsnummer: row.fnummer, personstatus:row.personstatus,kjoenn:row.kjoenn,foedselsdato:date(row.foedselsdato),sivilstand:row.sivilstand,navn:row.fornavn + " " + row.etternavn, postnummer:row.postnummer})
```
### Relasjoner
**FAMILIERELASJON**
<br/>_denne filen er stor og bruker lengre tid på å laste inn..._
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
**Hent en personer med foedselsnummer**
<br/>_alle fnr består av 11 siffer_
```cypher
match (n:Person{foedselsnummer:'00000000000'}) return n
```
**Hent en personer med navn**
```cypher
match (n:Person{navn:'Navn Navnesen'}) return n
```
**Telle antall personer**
```
match (p) with count(p) as personer return personer
```
**Telle antall relasjoner**
<br/>_velg type relasjon (familierelasjon, sivilstand, foreldreansvar)_
```cypher
match (:Person) - [r:FAMILIERELASJON] -> (:Person) with count(r) as relasjoner return relasjoner
```
**Gift en personer med en annen**
<br/>_hvis en relasjon blir opprettet en vei, skal den speiles og legges inn for den andre personen også
(kommandoen må kjøres 2 ganger, på hver sin person)_
<br/>_SIVILSTAND_
```cypher
match (d:Person{foedselsnummer:'00000000000'}), (l:Person{foedselsnummer:'12345678901'}) create (d) - [:SIVILSTAND{sivilstand:'GIFT', ergjeldende:TRUE, gyldighetstidspunkt:date('2000-01-01')}] -> (l) 
```
<br/>_FAMILIERELASJON_
<br/>_legger man inn et giftemål bør det også være en kobling mellom dem i familierelasjonen også (også tovveis)_
```cypher
match (d:Person{foedselsnummer:'00000000000'}), (l:Person{foedselsnummer:'12345678901'}) create (d) - [:FAMILIERELASJON{rolle:'EKTEFELLE_ELLER_PARTNER', ergjeldende:TRUE, gyldighetstidspunkt:date('2000-01-01')}] -> (l)
```
**Lag Person**
```cypher
create (p:Person{foedselsnummer:'00000000000', foedselsdato:date('2000-01-01'), personstatus:'BOSATT', navn:'Navn Navnesen', kjoenn:'MANN', postnummer:'1234'})
```
**Slett relasjoner**
<br/>_husk å slette relasjonen begge veier_
```cypher
match (p:Person{foedselsnummer:'00000000000'}) - [r:FAMILIERELASJON{rolle:'BARN'}] -> (q:Person) delete r
```
```cypher
match (p:Person{foedselsnummer:'12345678901'}) <- [r:FORELDREANSVAR{ansvarstype:'FELLES'}] - (q:Person) delete r
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
