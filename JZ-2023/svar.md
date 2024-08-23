# Svar
### Spørsmål 1:
_Relasjoner: mor, far, barn, ektefelle_eller_partner_
```cypher
match (p{navn:'Historisk Adventskalender'}) <- [r:FAMILIERELASJON] - (q)
return r.rolle
```
### Spørsmål 2:
_Navn/fødselsnummer: 'Institusjon*Gutt Svak'/13852199983_
```cypher
match (p:Person) - [r:FAMILIERELASJON{rolle:'EKTEFELLE_ELLER_PARTNER'}] - (:Person) with p, count(r) as ektEllerPart where ektEllerPart > 2 return p, ektEllerPart
```
### Spørsmål 3:
_Navn/fødselsnummer: 'Fantastisk Agurktid'/28817597620_
```cypher
match (b:Person) <- [r:FAMILIERELASJON{rolle:'BARN'}] - (v:Person) where v.foedselsdato < b.foedselsdato return b
```
### Spørsmål 4:
_2 personer - navn/fødselsnummer: 'Personlig Geografi'/17898198227 og 'Sorgløs Ost'/12878197559 (far er 'Kursiv Terskel'/01850897731)_
```cypher
match (p:Person) <- [r:FAMILIERELASJON] - (:Person) with p, count(r) as relasjoner where relasjoner > 19 return p, relasjoner
```
### Spørsmål 5:
| a.navn       | b.nvan        | relasjoner        |
|--------------|---------------|-------------------|
| 'Matematisk Sans' | 'Stabil Sans' | 'FAMILIERELASJON' |
| 'Matematisk Sans' | 'Treg Sans' | 'FAMILIERELASJON' |
```cypher
match (a:Person{navn:'Matematisk Sans'}) - [r] -> (b:Person) with *, type(r) as relasjoner return a.navn, b.navn, relasjoner
```
