**Inhoud**

[TOC]

[the mermaid live editor](https://mermaid-js.github.io/mermaid-live-editor)

## Variant 1: gedeelde invoervoorziening

### Raadplegen op Landelijk portaal

De burger of ondernemer volgt de weg zoals deze door Europa bedoeld is. Via YourEurope wordt de burger verwezen naar het juiste Nederlandse Nationale portaal. Op het Nationale portaal vindt de burger het product en vraagt van dit product de gegevens van een gemeente op. Het Landelijk portaal haalt deze gegevens op bij de API die door de gedeelde invoorvoorziening wordt aangeboden.

Alle gegevens zijn nu beschikbaar en het Landelijk portaal maakt de SDG productpagina op.

Een SDG productpagina bevat conform SDG verordening SDG tags waardoor de pagina vindbaar en herkenbaar is voor het Your Europe portaal.

#### Globale werking API

```mermaid
sequenceDiagram %% diagram
	autonumber
	%% participant
    participant B as Burger of <br>ondernemer
    participant Y as Search engine of<br>Your Europe
    participant N as Landelijke portalen
    participant A as Producten API <br>(invoervoorziening)

    B->>Y: zoeken overheidsinformatie
    activate Y
      Y->>B: verwijzing nationale portalen
    deactivate Y

	rect rgb(245, 245, 245)
        Note left of A: Interactiedeel portalen en producten API

		B->>N: Zoeken productbeschrijving
		activate N
		  N->>B: Generieke productpagina
		deactivate N

		B->>N: Selecteren gemeenten
		activate N
			N->>A: ophalen specifieke tekst gemeente
			activate A
			  A->>N: specifieke tekst
			deactivate A
			N->>B: SDG compliant productpagina
		deactivate N
	end
```

_Het landelijke portaal haalt specifieke teksten op_

#### Werking in meer detail (zonder search engine)

```mermaid
sequenceDiagram %% diagram
	autonumber
  %% participant
    participant B as Burger of <br>ondernemer
    participant N as Landelijke portalen
    participant A as Producten API <br>(invoervoorziening)

    B->>N: Zoeken productbeschrijving
    activate N
        N->>A: ophalen beschikbare specifieke productteksten
        activate A
        Note left of A: Hoe optimaliseren we deze aanroep <br> (volgens API design rules)?
          A->>N: alle gemeenten en indicator beschikbaarheid producttekst <br>(ja, nee, voert product niet )
        deactivate A
      N->>B: Generieke productpagina
    deactivate N

    B->>N: Toon product voor gemeente x
    activate N
        activate N
            N->>N: bepaal gemeente adhv plaatsnaam
        deactivate N
        N->>A: ophalen organisatiegegevens gemeente x
        activate A
          A->>N: lokatie, openingstijden, etc
        deactivate A
        N->>A: ophalen specifieke tekst gemeente x
        activate A
          A->>N: specifieke tekst
        deactivate A
        N->>B: Productpagina gemeente x
    deactivate N
```

#### Werking in meer detail met tags (zonder search engine)

```mermaid
sequenceDiagram %% diagram
  %% participant
    participant B as Burger of <br>ondernemer
    participant N as Landelijke portalen
    participant A as Producten API <br>(invoervoorziening)

    B->>N: Zoeken productbeschrijving
    activate N
        N->>A: ophalen beschikbare specifieke productteksten
        activate A
        Note left of A: Hoe optimaliseren we deze aanroep <br> (volgens API design rules)?
          A->>N: alle gemeenten en indicator beschikbaarheid producttekst <br>(ja, nee, voert product niet )
        deactivate A
        Note left of N: Hoe omgaan met organisatie tags?
        activate N
            N->>N: toevoegen SDG tags
        deactivate N
      N->>B: Generieke productpagina
    deactivate N

    B->>N: Toon product voor gemeente x
    activate N
        activate N
            N->>N: bepaal gemeente adhv plaatsnaam
        deactivate N
        N->>A: ophalen specifieke tekst gemeente x
        activate A
          A->>N: specifieke tekst
        deactivate A
        N->>B: Productpagina gemeente x
    deactivate N
```

```mermaid
sequenceDiagram %% diagram
  %% participant
    participant B as Burger of <br>ondernemer
    participant Y as Your Europe / Google
    participant N as Landelijke portalen
    participant A as Producten API <br>(invoervoorziening)

    B->>Y: zoeken overheidsinformatie
    activate Y
      Y->>B: verwijzing nationale portalen
    deactivate Y

    B->>N: Zoeken productbeschrijving
    activate N
        activate N
            N->>N: toevoegen SDG tags
        deactivate N
    N->>B: Productpagina met generieke producttekst en tags
    deactivate N

    loop per gemeente
      activate B
      B->>B: Selecteren gemeenten
      B->>A: ophalen specifieke tekst gemeente
        activate A
            A->>B: specifieke tekst
        deactivate A
      deactivate B
    end
```

_De specifieke producttekst wordt door de browser van de burger of ondernemer ingevoegd_

<div style="page-break-before: always;"></div>

### Caching in portaal

Voor een goede gebruikservaring zal het portaal enkele gegevens willen cachen. De API zal hiervoor specifieke bulk opvragingen ondersteunen

#### Wijzigingen sinds laatste raadpleging

```mermaid
sequenceDiagram %% diagram
  %% participant
    participant N as Landelijke portalen
    participant A as Producten API <br>(invoervoorziening)

  alt event sourcing (polling)
    N->>A: geef wijzigingen vanaf tijdstip x
    activate A
      A->>N: gelogde events
    deactivate A
  else notificeren
      Note left of A: Notificeren dat er iets gewijzigd is of <br>gelijk gegevens meesturen?
      A->>N: gemeente x heeft specifieke tekst y gewijzigd
    activate N
    N->>A: ontvangstbevestiging
    deactivate N
      Note right of N: Synchroon afhandelen. <br>Asynchroon niet ondersteund in API design rules
  end
```

<div style="page-break-before: always;"></div>

### Raadplegen op gemeentelijke website

De burger of ondernemer heeft een vraag en zoekt met Google naar informatie. Via Google wordt de burger verwezen naar de website van een gemeente.

Een gemeentelijke website kan de teksten van zijn eigen productpagina's dubbel beheren naast de teksten in de invoervoorziening. De gemeente kan er ook voor kiezen om de SDG teksten alleen bij de te houden in de invoervoorziening en deze in te voegen in de eigen pagina's. Deze laatste variant is in het onder staande diagram uitgewerkt

De gemeentelijke website haalt de SDG teksten op bij de invoervoorziening en voegt deze in in de eigen productpagina. Hiermee is deze pagina compleet en wordt deze getoond aan de burger

```mermaid
sequenceDiagram %% diagram
  %% participant
    participant B as Burger of ondernemer
    participant G as Google
    participant W as Website gemeente
    participant C as Lokale CMS
    participant A as Specifieke <br>productteksten API <br>(invoervoorziening)

    B->G: zoeken overheidsinformatie
    activate G
      G->>B: verwijzing gemeente website
    deactivate G
    B->>W: Raadplegen gemeentelijke productpagina
    activate W
          W->>C: ophalen productpagina
          activate C
            C->>W: productpagina
          deactivate C
          W->>A: ophalen <br> producttekst gemeente
          activate A
            A->>W: SDG productteksten
          deactivate A
        W->>B: Gemeentelijke productpagina (niet SDG compliant)
    deactivate W

```

<div style="page-break-before: always;"></div>

## Variant 2: Beheer SDG teksten in lokaal CMS

### Raadplegen op Landelijk portaal

De burger of ondernemer volgt de weg zoals deze door Europa bedoeld is. Via YourEurope wordt de burger verwezen naar het juiste Nederlandse Nationale portaal. Op het Nationale portaal vindt de burger het product en vraagt van dit product de gegevens van een gemeente op. Het Landelijk portaal vraagt de gemeentelijke gegevens op bij de routeervoorziening. De routeervoorziening verwijst het verzoek door naar de API van de gemeente. Bij de gemeente worden de gegevens opgehaald.

Alle gegevens zijn nu beschikbaar en het Landelijk portaal maakt de SDG productpagina op.

Een SDG productpagina bevat conform SDG verordening SDG tags waardoor de pagina vindbaar en herkenbaar is voor het Your Europe portaal.

```mermaid
sequenceDiagram %% diagram
  %% participant
    participant B as Burger of <br>ondernemer
    participant Y as Your Europe
    participant N as Landelijke portalen
    participant R as Routeerservice
    participant A as Producten API <br>(lokaal CMS)

    B->>Y: zoeken overheidsinformatie
    activate Y
      Y->>B: verwijzing nationale portalen
    deactivate Y

    B->>N: Zoeken productbeschrijving
    activate N
      N->>B: Samenvatting product
    deactivate N

    B->>N: Selecteren gemeenten
    activate N
        N->>R: ophalen <br> producttekst gemeente
        activate R
          R->>A: ophalen <br> producttekst gemeente
          activate A
            A->>R: producttekst
          deactivate A
          R->>N: producttekst
        deactivate R
        activate N
            N->>N: toevoegen SDG tags
        deactivate N
        N->>B: SDG compliant productpagina
    deactivate N
```

<div style="page-break-before: always;"></div>

### Raadplegen op gemeentelijke website

De burger of ondernemer heeft een vraag en zoekt met Google naar informatie. Via Google wordt de burger verwezen naar de website van een gemeente.

De productpagina en de SDG teksten worden in het lokale CMS beheerd. De website heeft dus alle informatie voor de productpagina in het eigen CMS beschikbaar. Afhankelijk van het CMS wordt de hele productpagina rechtstreeks ingelezen of wordt ook voor de eigen website gebruik gemaakt van API's (bijvoorbeeld Open Webconcept werkt op deze wijze)

De gemeentelijke website haalt de pagian met de SDG teksten op bij het CMS en toont deze aan de burger

```mermaid
sequenceDiagram %% diagram
  %% participant
    participant B as Burger of <br>ondernemer
    participant G as Google
    participant W as Website gemeente
    participant A as Lokale CMS direct <br>(of via API)

      B->G: zoeken overheidsinformatie
      activate G
        G->>B: verwijzing gemeente website
      deactivate G
      B->>W: Raadplegen gemeentelijke productpagina
      activate W
            W->>A: ophalen <br> producttekst gemeente
            activate A
              A->>W: producttekst
            deactivate A
          W->>B: Gemeentelijke productpagina (niet SDG compliant)
      deactivate W
```
