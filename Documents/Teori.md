Det här är det jag hittills skrivit för delarna metod och teori. Har känt att det varit lite svårt att
separera vilka delar som hör till vilket avsnitt så jag har hittills skrivit dom under samma rubrik.
Mina fingrar har också kliat efter att faktiskt koda så jag har under veckan också påbörjat arbetet
med den tjänst jag kallat för _Gästbookstjänst_.

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

I själva utvecklingsarbetet så har jag lutat mig mot många verktyg och bibliotek. Git har en
självklar plats under utvecklingsarbetet för att sköta versionshantering. Jag har lutat mig
mot Docker och _air_ för att köra applikationen under utvecklingen, detta har låtit mig se
hur den komponent jag utvecklar fungerar i det större sammanhanget genom att kontinuerligt
lyfta in den updaterade koden i den miljön.

För att hantera _routing_ och nätverksförfrågningar så har jag framförallt lutat mig mot
_gin_ vilket är ett bibliotek som underlättar utvecklingen av webapplikationer.

För persistens så har jag då jag redan är bekant med driftsättningen arbetat mot mongodb.

[^2]: Mehmet S., Bedir T., Ayça K. T. (2023) Microservice reference architecture design: A multi-case study (s. 69)
