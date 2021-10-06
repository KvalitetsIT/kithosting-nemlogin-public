# Login med NemLog-in
Denne vejledning beskriver hvordan man som tjenesteudbyder opsætter sin applikation, så den tilbyder login gennem NemLog-in, med Keycloak som broker. Login går gennem KIT's infrastruktur, og man får således mulighed for at logge ind gennem NemLog-in, uden at man skal oprettes som NemID-tjenesteudbyder.

NemLog-in virker som broker for både NemId og MitId

Vejledningen beskriver hvilke skridt man skal udføre for at gennemføre opkoblingen. Der medfølger også et demo-setup, der demonstrerer login.

## Forudsætninger
For at køre setuppet har man brug for at have [Docker Community Edition](https://docs.docker.com/install/) installeret. 

For at etablere login gennem sin applikation, skal applikationen kunne kommunikere over SAML 2.0-protokollen. Det er op til tjenesteudbyderen at vælge, hvordan dette skal gøres. Man kan enten importere et SAML-rammeværk i sin applikation, eller deploye sin applikation bag en SAML-proxy (demo-setuppet fungerer på denne måde). Dette vil uanset hvad medføre, at man skal generere et nøglepar for sin applikation. Til testformål kan der anvendes selvsignerede certiifikater, mens der til produktion skal anvendes CA-udstedte certifikater.

## Det udleverede setup
Setuppet består af en række containere, beskrevet i filen testlogin/docker-compose.yml, samt en række filer, som vist her:

```
.
├── docker-compose.yml
└── echo
    ├── certificates
    │   ├── echo.cert
    │   └── echo.pem
    └── config.json

```

Demo-applikationen, som er beskyttet af login, er en simpel applikation som blot skriver de requests den modtager tilbage til klienten.

Man starter setuppet ved at køre:

```
docker-compose up
```

Når containerne er oppe tilgår man _http://localhost/_ i en browser. Man viderestilles til kithosting og NemLogin, og får vist standard NemLog-in i browseren:

![nemid](images/nemlogin.png)

Login foregår mod NemID's testmiljø. Til login kan man anvende følgende testbruger: tade328/asasas12, med [dette](testusers/tade328.pdf) nøglekort. Når login er gennemført, bliver man videresendt til demo-applikationen, og login er udført:

![echo](images/echo.png)

Hvis man får vist ovenstående skærmbillede, så har man succesfuldt fået setuppet til at køre.

Det er også mulig at test nemlogin opkobling uden benytte test setup'et men ved at tilgå følgende url: 

* [https://loginservice.t0.hosting.kitkube.dk/auth/realms/nemlogin/account/#/personal-info](https://loginservice.t0.hosting.kitkube.dk/auth/realms/nemlogin/account/#/personal-info)

## Testsetup
Testsetup'et består af 3 containere:
* ServiceProvider (samlsp) der benytter kvalitetsit/kitcaddy. Komponenten er en implementation af en SAML ServiceProvider.
* Mongo database (mongodb) der gemmer sessionsdata fra samlsp.
* En "applikation" der viser det request der kommer ind.

kvalitetsit/kitcaddy er en opensource komponent der frit kan benyttes. Alternativt kan applikationen selv udstille sig som SAML service provider.

## Opkobling til Keycloak
For at etablere opkobling til Keycloak, skal der udveksles metadata mellem applikationen og Keycloak. Dette beskrives i de følgende afsnit.

### Generere metadata for applikation
Applikationen skal oprettes i Keycloak, og til dette formål skal der genereres metadata for applikationen. Hvordan dette gøres afhænger af applikationens valgte saml-rammeværk, men kan typisk genereres ved at tilgå en bestemt url, som applikationen udstiller ved hjælp af rammeværket. Når metadata er genereret, sendes det til KIT (kithosting@kvalitetsit.dk).

### Hente metadata fra Keycloak
Applikationen skal sættes op til at genkende Keycloak som identity provider, hvilket gøres ved at installere en metadata-fil for Keycloak i applikationen. Man henter metadata for Keycloak her: 
 
 * [https://loginservice.t0.hosting.kitkube.dk/auth/realms/nemlogin/protocol/saml/descriptor](https://loginservice.t0.hosting.kitkube.dk/auth/realms/nemlogin/protocol/saml/descriptor). 
 
 Metadata-filen skal placeres et sted hvor applikationen kan læse den, hvilket igen afhænger af det valgte saml-rammeværk.

### Test af login
Login kan nu testes ved at følge proceduren beskrevet tidligere.
