
<h2>LO §5.2.2</h2>
Flow voor een Ii01 bericht


Variabelen

```yaml
id: Ii01
namespace: rvig.sent

variables:
  host: host.docker.internal
  type_bericht: Ii01
  antwoord: NIET_AANWEZIG
  gevonden: MINISTERIEEL_BESLUIT
```

Taken

```yaml
tasks:
  - id: initieer_berichtenstroom
    type: io.kestra.core.tasks.log.Log
    message: Initieer Ii01 bericht
  - id: stuur_Ii01_bericht
    type: io.kestra.plugin.fs.http.Request
    uri: https://marc4gov.pythonanywhere.com/initiatie/Ii01
  - id: stuur_pl
    type: io.kestra.core.tasks.flows.Parallel
    tasks:
      - id: controeer_pl
        type: io.kestra.core.tasks.flows.Sequential
        tasks:
          - id: haal_pl_op
            type: "io.kestra.plugin.mongodb.Find"
            connection:
              uri: "mongodb://{{vars.host}}:27017/?authSource=admin"
            database: "local"
            collection: "pl"
            filter:
              _id:
                $oid: 657f5a7b260794696f111bcb
          - id: doe_bestandscontrole
            type: io.kestra.plugin.fs.http.Request
            uri: https://marc4gov.pythonanywhere.com/initiatie/controle_ok
          - id: check_bestand
            type: io.kestra.core.tasks.log.Log
            message: Resultaat is {{outputs.doe_bestandscontrole.body | jq('.bericht') | first }}
      - id: check_bericht
        type: io.kestra.core.tasks.log.Log
        message: Dit is een {{ outputs.stuur_Ii01_bericht.body | jq('.bericht') | first }} bericht
  - id: stuur_antwoord
    type: io.kestra.core.tasks.flows.Switch
    value: "{{ vars.antwoord}}"
    cases: 
      NIET_UNIEK:
        - id: niet_uniek
          type: io.kestra.core.tasks.log.Log
          message: Niet uniek
      NIET_AANWEZIG:
        - id: niet_aanwezig
          type: io.kestra.core.tasks.log.Log
          message: Niet aanwezig
    defaults:
      - id: gevonden
        type: io.kestra.core.tasks.flows.Switch
        value: "{{ vars.gevonden}}"
        cases: 
          OVERLEDEN:
            - id: overleden
              type: io.kestra.core.tasks.log.Log
              message: Overleden
          GEBLOKKEERD:
            - id: geblokkeerd
              type: io.kestra.core.tasks.log.Log
              message: Geblokkeerd
          MINISTERIEEL_BESLUIT:
            - id: ministerieel_besluit
              type: io.kestra.core.tasks.log.Log
              message: Ministerieel Besluit
          GEËMIGREERD:
            - id: geëmigreerd
              type: io.kestra.core.tasks.log.Log
              message: Geëmigreerd
        defaults:
          - id: actueel
            type: io.kestra.core.tasks.log.Log
            message: Actueel
```