# SDG berichtuitwisselingen

Dit document beschrijft de berichtuitwisselingen tussen de Nationale Portalen en de decentrale overheden. De berichtuitwisselingen zijn uitgewerkt in sequence-diagrammen en vervolgens zijn de benodigde API's ontworpen op basis van het SDG informatiemodel.

Doelstelling van dit document is tweeledig:

- verifieren van het SDG informatiemodel. Bevindingen leiden tot aanpassing over en weer
- discussiedocument voor het afstemmen van de benodigde APIS's


## Openstaande vragen over SDG informatiemodel

1\. Welke organisatie-rollen zijn relevant voor de decentrale overheden?

> Het SDG informatiemodel kent de volgende organisatie-rollen:
>
> - Organisatie voor ondersteuning
> - Organisatie verantwoordelijk voor de decentrale informatie
> - Bevoegd gezag verantwoordelijk voor de procedure
>
> Voor zover nu bekend geldt voor een decentrale overheid dat deze alle 3 de rollen zelf vervult.
>
> - Welk organisatietype gaan we gebruiken als identificatie van de DO?
> - Welke organisatietypen gaan we retourneren in antwoorden?

2\. Er is geen attribuut contactgegevens. Deze zijn impliciet onderdeel van de 3 organisatie-rollen

> Het SDG informatiemodel is hier niet compleet. In de invoervoorziening is dit nu als volgt uitgewerkt:
>
> - een organisatie heeft contactgegevens
>   - e-mail
>   - website
>   - telefoon
>   - en één of meerdere locaties
>     - bezoekadres
>     - openingstijden
>
> Bij een product wordt aangegeven op welke locatie(s) deze beschikbaar is

## Specifieke teksten berichtuitwisselingen

De burger of ondernemer heeft via een search engine of YourEurope het product op een Nationale Portaal gevonden. De burger of ondernemer kan van dit product vervolgens de specifieke teksten van een decentrale overheid opvragen. Het Landelijk portaal haalt deze gegevens op bij de SDG product API bij de invoorvoorziening (van de DO).

Alle gegevens zijn nu beschikbaar en het Landelijk portaal maakt de SDG productpagina op.

Een SDG productpagina bevat conform SDG verordening SDG tags waardoor de pagina vindbaar en herkenbaar is voor het Your Europe portaal.

### Raadplegen NP met live teksten

```mermaid
sequenceDiagram %% diagram
 autonumber
  %% participant
    participant B as Burger of <br>ondernemer
    participant N as Landelijke portalen
    participant A as SDG product API <br>(invoervoorziening)

    B->>N: Raadplegen generieke productbeschrijving
    activate N
        N->>A: Opvragen status van specifieke teksten
        activate A
          A->>N: Per DO de status van de specifieke tekst
        deactivate A
        N->>N: toevoegen SDG tags
      N->>B: Generieke productpagina
    deactivate N

    loop raadplegen één of meerdere DO

      B->>N: Raadplegen specifieke product van DO
      activate N
          N->>N: Selecteer DO,<br> toon DO suggesties op basis van status
          N->>A: ophalen specifieke tekst DO
          activate A
            A->>N: specifieke tekst en contactgegevens
          deactivate A
            N->>N: toevoegen SDG location tag
          N->>B: Specifieke productpagina
      deactivate N

    end
```

### Raadplegen NP met cache teksten

Het ondernemersplein heeft aangegeven de specifieke teksten te willen cachen. De specifieke teksten in het cache worden periodiek geactualiseerd.

De SDG product API zal hiervoor specifieke bulk opvragingen ondersteunen


```mermaid
sequenceDiagram %% diagram
  %% participant
    participant B as Burger of <br>ondernemer
    participant N as Landelijke portalen
    participant A as Producten API <br>(invoervoorziening)

    loop periodiek bijwerken cache

      N->>A: geef gewijzigde specifieke teksten vanaf datum x
      activate A
        A->>N: per DO de gewijzigde specifieke teksten
      deactivate A
    end

    B->>N: Raadplegen generieke productbeschrijving
    activate N
      N->>N: toevoegen per DO status specifieke teksten
      N->>N: toevoegen SDG tags
      N->>B: Generieke productpagina
    deactivate N

    loop raadplegen één of meerdere DO

      B->>N: Raadplegen specifieke product van DO
      activate N
        N->>N: Selecteer DO,<br> toon DO suggesties op basis van status
        N->>N: specifieke tekst en contactgegevens
        N->>N: toevoegen SDG location tag
        N->>B: Specifieke productpagina
      deactivate N
    end
```

### Invoervoorziening API

#### Opvragen status van specifieke teksten

Geef van het gevraagde product een lijst van de decentrale overheden en per DO of het product beschreven is.

- Request:
  - UPN
- Reply
  - UPN
  - alle DO
    - registratiestatus (wel/niet beschreven)
    - registratiestatusToelichting
    - Als status wel beschreven
      - ProductAanwezig (ja/nee)
      - ProductAanwezigToelichting

#### Opvragen specifieke tekst

Geef de specifieke tekst van een organisatie in een bepaalde taal en voor een bepaalde doelgroep

- Request:
  - Organisatie, UPN, Taal, Doelgroep
- Reply
  - VerantwoordelijkeOrganisatie (of BevoegdeOrganisatie??)
    - **contactgegevens**
  - ProductAanwezig
  - ProductAanwezigToelichting
  - NPSpecifiekeLink
  - **locatie**
  - Alle Productvariant
    - ProductTitelDecentraal
    - SpecifiekeTekst
    - VerwijzingLinks
    - DatumWijziging
  - Productvariant (specifiek procedure art 10)
    - BeschikbareTalen
    - ProcedureBeschrijving
    - Vereisten
    - Bewijs
    - BezwaarEnBeroep
    - KostenEnBetaalmethoden
    - UitersteTermijn
    - WTDBijGeenReactie
    - DecentraleProcedureLink

#### Actualiseren kopie specifieke teksten

Geef van alle organisaties de gewijzigde specifieke teksten waarvoor geldt DatumWijziging > DatumVanaf

- Request:
  - DatumVanaf, Doelgroep
- Reply
  - alle organisaties
    - VerantwoordelijkeOrganisatie (of BevoegdeOrganisatie??)
      - **contactgegevens**
    - alle Product (specifiek informatie art 9)
      - UPN_URI
      - Doelgroep
      - Alle talen
        - Taal
        - ProductAanwezig
        - ProductAanwezigToelichting
        - NPSpecifiekeLink
        - **locatie**
        - Alle Productvariant
          - ProductTitelDecentraal
          - SpecifiekeTekst
          - VerwijzingLinks
          - DatumWijziging
        - Productvariant (specifiek procedure art 10)
          - BeschikbareTalen
          - ProcedureBeschrijving
          - Vereisten
          - Bewijs
          - BezwaarEnBeroep
          - KostenEnBetaalmethoden
          - UitersteTermijn
          - WTDBijGeenReactie
          - DecentraleProcedureLink

## Generieke teksten berichtuitwisselingen

De invoervoorziening vraagt de generieke teksten op om deze te tonen in de invoervoorziening. De generieke tekst wordt getoond aan de gemeente redactie bij het schrijven van een specifieke tekst.

### Redactie specifieke teksten met live generieke teksten

```mermaid
sequenceDiagram %% diagram
 autonumber
  %% participant
    participant R as Redactie
    participant I as Invoervoorziening
    participant A as Generieke tekst API <br>(Nationaal Portaal)

    R->>I: Beheren specifieke tekst
    activate I
        I->>A: Opvragen generieke tekst
        activate A
          A->>I: generieke tekst
        deactivate A
        I->>I: invullen referentie tekstblokken
        I->>I: invullen specifieke tekstblokken
      I->>R: formulier specifieke tekst
    deactivate I
```

### Redactie specifieke teksten met cache generieke teksten

```mermaid
sequenceDiagram %% diagram
 autonumber
  %% participant
    participant R as Redactie
    participant I as Invoervoorziening
    participant A as Generieke tekst API <br>(Nationaal Portaal)

    loop periodiek bijwerken cache

      I->>A: geef gewijzigde generieke teksten vanaf datum x
      activate A
        A->>I: gewijzigde generieke teksten
      deactivate A
    end

    R->>I: Beheren specifieke tekst
    activate I
        I->>I: invullen generieke tekst
        I->>I: invullen referentie tekstblokken
        I->>I: invullen specifieke tekstblokken
      I->>R: formulier specifieke tekst
    deactivate I
```

### Opvragen generieke tekst

Geef de generieke tekst van een product

- Request:
  - UPN_URI
- Reply
  - VerantwoordelijkeOrganisatie
  - Product (generiek)
    - Alle talen
      - Taal
      - ProductTitel
      - GeneriekeTekst
      - KorteOmschrijving
      - VerwijzingLinks
      - NationaleLink
      - DatumCheck

### Actualiseren generieke tekst

Geef de gewijzigde generieke teksten waarvoor geldt DatumCheck > DatumVanaf

- Request:
  - DatumVanaf
- Reply
  - VerantwoordelijkeOrganisatie
  - Alle producten
    - Product (generiek)
      - UPN_URI
      - Alle talen
        - Taal
        - ProductTitel
        - GeneriekeTekst
        - KorteOmschrijving
        - VerwijzingLinks
        - NationaleLink
        - DatumCheck

## Mermaid diagrammen

De diagrammen in dit document zijn gemaakt in Mermaid. Mermaid maakt het mogelijk om diagrammen in code te beschrijven, waarna deze dynamisch gerendered wordt.

Een browser ondersteund het renderen van mermaid niet standaard, maar er zijn hier extensies voor beschikbaar.
In Chrome/Vivaldi kun je gebruik maken van bijvoorbeeld de extensie *mermaid-diagrams*.

Om zelf met mermaid te experimenteren: zie [the mermaid live editor](https://mermaid-js.github.io/mermaid-live-editor)