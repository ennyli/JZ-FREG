# Javazone2023-FREG
Folkeregisteret konkurranse på JavaZone 2023 i Oslo.
---

### Basic kommandoer for å starte opp Docker og Neo4j:
Installer Docker (macOS)
```
brew install docker
brew install --cask docker\
```
Start Docker
```
systemctl start docker\
```
Start opp image (i bakgrunnen) (med porter) (uten passord auth)
```
docker run -d -p=7474:7474 -p=7687:7687 -v=$HOME/neo4j/data:/data --env NEO4J_AUTH=none neo4j
```
Kopier fil fra maskin til container
```
 docker cp <file.csv> <containerId>:/var/lib/neo4j/import
```
Start opp container og gå inn i image
Gå ut av container
```
docker exec -it <containerId> /bin/bash
exit
```
Sjekk alle/aktive containere
```
docker ps -a
```
Sjekk alle/aktive containere
```
docker images -a
```
Start container (opp igjen)
Stop container
```
docker start <containerId>
docker stop <containerId>
```

* Kjører lokalt: http://localhost:7474/browser/

## Import av data til Neo4j UI
_(importer Persondata først, så relasjonene)_
### Persondata
```
LOAD CSV WITH HEADERS FROM "file:///personer.csv" AS row
create(p:Person {foedselsnummer: row.fnummer, personstatus:row.personstatus,kjoenn:row.kjoenn,foedselsdato:date(row.foedselsdato),sivilstand:row.sivilstand,navn:row.fornavn + " " + row.etternavn, postnummer:row.postnummer})
```
### Relasjoner
**FAMILIERELASJON** 
_(denne tar litt lengre tid å laste inn)_
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

### Annet
* GitHub Ronny: https://github.com/ronnyma/jz_nta.
* Cheatsheet Neo4j Enterprise: https://neo4j.com/docs/cypher-cheat-sheet/5/auradb-enterprise/
