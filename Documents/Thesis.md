# _Micro-Service Architecture_ och _golang_

Erika Bladh-Öström
2024

## Sammanfattning

## Inledning

Vi har under tidigare kurser i den här utbildningen lärt oss om
vissa arkitekturella strukturer, framför allt _Ports and Adapters_
och _clean architecture_. Dessa strukturer är tänkta att genom ökad 
modularitet göra applikationen lättare att utökas, underhållas och
testas. Men resultaten av dom struktuer vi studerat hittills är
en monolitisk applikation, det vill säga en stort applikation 
som behöver kompileras och köras i sin helhet. Det jag vill undersöka 
är _Micro-service_ arkitektur, där olika tjänster kan kompileras och
köras fristående från varandra. Detta kan ses som en vidareutveckling
av mönstret _Ports and Adapters_ (även känt som _Hexagonal architecture_). 

Jag har valt att för mitt utforskande arbeta med _golang_, ett
programmeringsspråk som av tre huvudsakliga anledningar ger
intryck av att vara väl passat för _Micro-service_ arkitetur.
Den första är att det finns ett gott stöd för webb-funktioner
i form av standard bibliotek. Den andra är att koden kompileras till en 
enskild körbar fil vilket kan underlätta driftsättning. 
Den tredje är att dessa applikationer kan utnyttja relativt få
resurser[^1].

Målet med arbetet är att i första hand skapa en övergripande
förståelse för hur en _Micro-service_ arkitektur kan realiseras och
vilka fördelar kontra nackdelar detta innebär i jämförelse med att
utveckla en monolitisk applikation. Arbetet ska också ge grundläggande
kunskaper i utveckling av webbapplikationer med _golang_.

### Innehåll och struktur

- [Sammanfattning](#sammanfattning)
- [Inledning](#inledning)
  - [Innehåll och struktur](#innehåll-och-struktur): Du är här. :)
- [Teori och metod](#teori-och-metod): 
  - [Applikationskoncept](#applikationskoncept)
  - [Arkitektur](#arkitektur)
  - [Driftsättning](#driftsättning)
  - [Utveckling](#utveckling)
- [Resultat](#resultat)
  - [Applikationen](#applikationen)
  - [Driftsättning](#driftsättning-1)
- [Diskussion](#diskussion)
- [Slutsatser](#slutsatser)
- [Referenser](#referenser)
- [Bilagor](#bilagor)

## Teori och Metod

### Applikationskoncept

Den applikation jag tänker arbeta mot är inspirerad av [_Atabook_](https://atabook.org/) och
[_Smart Guestbook_](https://smartgb.com/). Dessa är webbapplikationer som tillhandahåller
gästböcker för hemsidor, bloggar eller sociala medier, och riktar sig främst till privatpersoner
som driver hemsidor i hobby syfte. Jag har också kollat på [_Cuter Counter_](https://www.cutercounter.com/)
samt [_Form Taxi_](https://form.taxi/en) för inspiration till potentiella tjänster utöver gästböcker.

### Arkitektur

Innan jag påbörjar utvecklingsarbetet så har jag arbetat fram en grov skiss på vad jag tänker
arbeta mot, men då detta bara är en utgångspunkt så kommer jag tillåta mig själv att
vid behov modifiera eller utöka den under arbetets gång.

För min arkitektur tänker jag försöka utgå ifrån den referensarkitektur som presenteras av
Mehmet Söylemez och andra i _Microservice reference architecture design: A multi-case study_[^2].
Referensarkitekturen inkluderar en meddelandeplattform och en komponent för att cacha data
men jag kommer initialt att undvika dessa för att hålla komplexiteten av min applikation
lägre. 

Centrum för _Micro-service_ applikationer brukar bestå av ett _API Gateway_, det vill
säga en knytpunkt för programmeringsgränssnitt genom vilket alla anrop till bakgrundstjänster
går. Utöver _API Gateway_ kollar jag initialt kollar på att implementera följande tjänster:

- Gästböcker, som hanterar nya meddelanden samt bör låta gästbokens ägare
administerara dessa
- Användare, som hanterar registrering av nya användare samt bör låta dessa
updatera sina profildetaljer
- Behörighet, som bör se till att behöriga användare får tillgång
till rätt information och funktioner

![Den övergripande arkitekturella designen av tjänster som har varit min utgångspunkt.](Images/Design-Sv.png)

### Driftsättning

Utvecklingen av _Micro-Service_ applikationer framgår som tätt kopplad till driftsättningen,
till exempel så innehåller referensarkitekturen _load balancer_, _service registry_ och
_service orchestrator_. Min intuition är att jag kan ge mig själv tillgång till dessa genom
att använda ArgoCD och kubernetes för min driftsättning. I och med det så kommer jag även att
förlita på mig på docker för att inkapsla dom komponenter jag utvecklar. Att driftsätta med
kubernetes innebär också att jag kommer kunna ha en utvecklingsmiljö lokalt som till stor del kan
se ut som den miljö i vilken applikationen kommer köra när den är i drift.

### Utveckling

I själva utvecklingsarbetet så kommer jag luta mig mot många verktyg och bibliotek. Git har en
självklar plats under utvecklingsarbetet för att sköta versionshantering. Jag kommer använda mig
av Docker och _air_ för att köra applikationen under utvecklingen, detta kommer låta mig se
hur den komponent jag utvecklar fungerar i det större sammanhanget genom att kontinuerligt
lyfta in den updaterade koden i en utvecklings miljö.

För att hantera _routing_ och nätverksförfrågningar så komer jag använda mig av
_gin_ vilket är ett bibliotek som är tänkt att underlätta utvecklingen av webapplikationer.

För persistens så kommer jag arbeta med mongodb då jag redan är bekant med hur jag kan driftsätta
en sådan databas i kubernetes.

## Resultat

### Applikationen

Applikationen kom i slutändan att bestå av följande mikrotjänster: 

- Användartjänst
- Gästbokstjänst
- Behörighetstjänst

Dessa har driftsatts separat och dom ansluter till varandra och bildar tillsammans applikationen som helhet. Nedan är mer detaljerade beskrivningar av tjänsterna.

#### Användar-, Gästboks- och Behörighets-tjänst

Dessa är tjänster som över HTTP internt exponerar ett programmeringsgränssnitt. Dom är driftsatta tillsammans med en mongo databas som dom ansluter till för att läsa och persistera data.

#### Programmeringsgränssnittsknytpunkt

Denna tjänst exponerar över HTTP ett programmeringsgränssnitt med ändpunkter inom två kategorier. Dels en mer stängd som enbart tillåter anslutningar från applikationens registrerade domän och dels en mer öppen som tillåter anslutningar från den domän som en gästbok skapats för (det vill säga, den försöker stoppa anslutningar från t.ex. _shady.zone_ om gästboken den försöker modifiera är skapad med t.ex. _my-shiny.website_.) Detta resultat har jag försökt uppnå genom att tillämpa olika CORS-regler beroende på ändpunkt.

#### Startsida och Administrationsgränssnitt

Dessa två var i den arkitekturskiss jag utgick ifrån två separata tjänster, men under utvecklingsarbetet så slog jag ihop dom till en tjänst under namnet _Frontend_. Jag kan i efterhand identifiera att det jag skrivit är en MVC-applikation där _gin_ ligger i bakgrunden och agerar _controller_, jag har skapt html-mallar som agerar som _view_ och informationen i mallarna fylls ut av en _struct_ som agerar _model_.

Jag arbetated mer med grafiska gränsnittet här än vad jag initialt hade planerat, framförallt för att jag fastnade i att utforska hur html-mallarna kunde utnyttjas för att definiera olika delar av det grafiska gränsnittet i olika filer (t.ex. en fil som definierar navigationen (_navbar.gohtml_) och en annan som innehåller CSS (_style.gohtml_))

#### Exempel gästbok

Min lösning innehåller också en html fil med exempel på hur en gästbok kan integreras i användarens hemsida. I det javaskript den filen innehåller behöver gästbokens identifikationssträng infogas så att den matchar den domän där gästboken kommer leva.

### Driftsättning

Jag har driftsatt min applikation under domänen guestbooks.online (observera att den antagligen inte finns tillgänglig där längre) och jag har för att demonstrera funktionalitet även gjort exempel koden tillgänglig på en separat domän (superellips.com).

#### Docker

Mitt arbete har sen dom första dagarna innehållit en strategi för att inkapsla applikationen, detta då jag i min utvecklingsmiljö utnyttjat _air_ och _delve_ för att direkt kunna testa förändringar i koden i en miljö som försöker efterlikna driftmiljön. All inkapsling sker med linux distributionen alpine som bas då detta resulterar i en liten avbildning med en relativt liten storlek. Mina _Dockerfile_ innehåller fyra steg:

##### Base

Detta steg ställer in vissa miljövariabler som senare används av go för att kompilera koden.

##### Dev

Detta steg utgår från _base_ och bygger ut det med _air_ och _delve_, dessa utnyttjas för att känna av förändringar i kodbasen och kompilera den på nytt. I min utvecklingsmiljö så monters en mapp lokalt in som en volym dit koden kompileras och där den körs ifrån.

##### Builder

Detta steg kompilerar koden för driftsättning.

##### Prod

Detta steg kopierar den kompilerade produktionskoden från _builder_ och kör den. Den kompilerade koden behöver inte att några ytterligare paket installeras och utgår därför direkt ifrån alpine.

#### Docker compose

I min utvecklingsmiljö så körs resultaten av stegen _dev_ tillsammans med docker compose. I den fil som konfigurerar compose så definieras även relevanta miljövariabler samt körs kontainrar med databaser för att testa. Detta har inneburit att jag i det kontinuerliga arbetat med applikationen som helhet har kunnat få insikter kring vad som sker i bakgrunden. Att anslutningssträngar för databaser och addresser till tjänster styrs av miljövariabler har inneburit att förflyttningen från docker compose till kubernetes har skett väldigt smärtfritt.

#### Git, Github Actions och Container Registry

Varje applikation har ett eget repository som är kopplat mot Github och har var sitt flöde som körs automatiskt. Tjänsternas flöden bygger en ny docker avbildning som sedan laddas upp till Github Container registry varifrån dom kan hämtas in till produktionsmiljön.

Jag har i mitt kontinuerliga arbete enbart använt mig av privata repositories och registries på Github, detta för att jag gärna ville få en känsla för vilka extra steg jag skulle behöva ta för att driftsätta en applikation där källkoden inte kan publiceras offentligt. I [avsnitett bilagor](#privata-repositories-och-kontaineravbildningar) så har jag försökt sammanfatta dom steg som varit nödvändiga att ta för att detta skall fungera.

#### Kubernetes

För driftsättning så har jag provisionerat ett kuberneteskluster på [Linode](https://www.linode.com/) bestående av en nod. Mina beskrivningar i [bilagorna](#bilagor) är färgade av detta och kommandon kan behöva modifieras för att fungera med ett kluster tillhandahållet av någon annan.

#### ArgoCD

För att genomföra kontinuerlig driftsättning så har jag i mitt kluster installerat ArgoCD. I vilket jag har definierat varje tjänst som en separat applikation.

![Skärmdump av webbgränsnittet för ArgoCD som visar tjänsterna driftsatta som separata applikationer. Alla tjänsterna visas som hälsosamma och synkroniserade.](Images/ArgoCD-applikationer.png)

Dom applikationer som definierats i ArgoCD består i sin tur av flera olika komponenter, i bilden nedan är dom komponenter som programmeringsgränssnittsknytpuntken består av. Dom tjänster som ansluter till en databas har även här sin tillhörande databasinstans.

![Skärmdump av webbgränsnittet för ArgoCD som visar status och komponenter för programmeringsgränssnittsknytpunkten som exempel på hur applikationerna är driftsatta.](Images/ArgoCD-gateway.png)

## Diskussion

## Slutsatser

## Referenser

- Pereira, R., Couto, M., Ribeiro, F., Rua, R., Cunha, J., Fernandes, J. P., & Saraiva, J. (2021) Ranking programming languages by energy efficiency. (Figur 7)
- Mehmet S., Bedir T., Ayça K. T. (2023) Microservice reference architecture design: A multi-case study (s. 69)

[^1]: Pereira, R., Couto, M., Ribeiro, F., Rua, R., Cunha, J., Fernandes, J. P., & Saraiva, J. (2021) Ranking programming languages by energy efficiency. (Figur 7)
[^2]: Mehmet S., Bedir T., Ayça K. T. (2023) Microservice reference architecture design: A multi-case study (s. 69)

## Bilagor

### Privata repositories och kontaineravbildningar


