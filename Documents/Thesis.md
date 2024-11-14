# _Micro-Service Architecture_ och _golang_

Erika Bladh-Öström
2024

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

## Teori

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
men jag kommer intiallt att undvika dessa för att hålla komplexiteten av min applikation
lägre. 

Centrum för _Micro-service_ applikationer brukar bestå av ett _API Gateway_, det vill
säga en knytpunkt för programmeringsgränssnitt genom vilket alla anrop till bakgrundstjänster
går. Utöver _API Gateway_ kollar jag initiallt kollar på att implementera följande tjänster:

- Gästböcker, som hanterar nya meddelanden samt bör låta gästbokens ägare
administerara dessa
- Användare, som hanterar registrering av nya användare samt bör låta dessa
updatera sina profildetaljer
- Behörighet, som bör se till att behöriga användare får tillgång
till rätt information och funktioner

![Den övergripande arkitekturella designen av tjänster.](Images/Design-Sv.png)

### Driftsättning

Utvecklingen av _Micro-Service_ applikationer framgår som tätt kopplad till driftsättningen,
till exempel så innehåller referensarkitekturen _load balancer_, _service registry_ och
_service orchestrator_. Min intuition är att jag kan ge mig själv tillgång till dessa genom
att använda ArgoCD och kubernetes för min driftsättning. I och med det så kommer jag även att
förlita på mig på docker för att inkapsla dom komponenter jag utvecklar. Att driftsätta med
kubernetes innebär också att jag kommer kunna ha en testmiljö lokalt som till stor del kan
se ut som den miljö i vilken applikationen kommer köra när den är i drift.


## Metod

Jag kommer försöka uppnå arbetets mål genom att arbeta med praktiskt
utvecklingsarbete och driftsättning av en applikation. Detta kommer
komplementaras med läsning av visst tillgängligt material.

TODO: Kanske skriva något här om git?
TODO: Kanske det är här som ramverk osv hör hemma också? T.ex. _gin_

## Diskussion

[^1]: Pereira, R., Couto, M., Ribeiro, F., Rua, R., Cunha, J., Fernandes, J. P., & Saraiva, J. (2021) Ranking programming languages by energy efficiency. (Figur 7)
[^2]: Mehmet S., Bedir T., Ayça K. T. (2023) Microservice reference architecture design: A multi-case study (s. 69)
