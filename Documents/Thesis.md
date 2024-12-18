# _Micro-Service Architecture_ och _golang_

Erika Bladh-Öström

18-12-2024

## Abstract/Sammanfattning

### English

The work has been an ongoing process rather than something with a clear endpoint. The application is not complete, but I have laid a solid foundation for further development, particularly by choosing Kubernetes for deployment early on. A well-functioning development environment was created, which facilitated both deployment and troubleshooting.

Two areas received more focus than planned: the presentation layer, where the MVC structure with Gin and HTML/templates provided deeper understanding, and security around the repository and container registry. Deploying the application under a registered domain was also an educational experience in the context of Kubernetes.

I find microservice architecture overly complex for a solo developer. A monolithic design with Domain Driven Design would have been a simpler starting point. However, Kubernetes and Docker have proven to be highly useful, and Go feels well-suited for web applications due to its strong library support and simplicity.

There are advantages to microservices, such as scalability and the ability for larger teams to work on different services in parallel, but these benefits were not relevant to my work.

### Svenska

Arbetet har varit en pågående process snarare än något med en tydlig slutpunkt. Applikationen är inte färdig, men jag har lagt en solid grund för vidare utveckling, särskilt genom att tidigt välja Kubernetes för driftsättning. En välfungerande utvecklingsmiljö skapades, vilket underlättade både driftsättning och felsökning.

Två områden fick mer fokus än planerat: presentationslagret, där MVC-strukturen med Gin och HTML/templates gav djupare förståelse, samt säkerhet kring repository och container registry. Att driftsätta under en registrerad domän var också lärorikt i Kubernetes-kontext.

Jag upplever mikrotjänstarkitektur som överkomplicerad för en ensam utvecklare. En monolitisk design med Domain Driven Design hade varit en enklare utgångspunkt. Kubernetes och Docker har dock varit mycket användbara, och Go känns väl lämpat för webbapplikationer tack vare starkt biblioteksstöd och enkelhet.

Fördelar med mikrotjänster finns, så som skalbarhet och möjligheten att i större team arbeta parallellt med olika tjänster, men dessa fördelar var inte aktuella för mitt arbete.

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

Jag har valt att för det här examensarbetet arbeta med _golang_, ett
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

- [Abstract/Sammanfattning](#abstractsammanfattning)
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
- [Reflektion och Diskussion](#reflektion-och-diskussion)
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


__Code from__ `./Frontend/main.go`:

```golang
// Function to render complete html-template
// Note: The last template file to parse
//       is being choosen dynamically by
//       the Page model "p"
func renderTemplate(w http.ResponseWriter, p *Page) {
	t := template.Must(template.ParseFiles(
		"/app/templates/default.gohtml",
		"/app/templates/navbar.gohtml",
		"/app/templates/style.gohtml",
		"/app/templates/footer.gohtml",
		"/app/pages/"+p.Template+".gohtml"))
	t.Execute(w, p)
}
```

__Code from__ `./Frontend/templates/navbar.gohtml`:

```html
<!-- Template of the navigation bar. This is included in the main template. -->
{{ define "navbar" }}
<nav id="navbar" style="margin-bottom: 0px; margin-top: 10px;">
<ul>
        <li><a href="/">Home</a></li>
        {{ if .Authenticated }}
        <li><a href="/user/{{ .UserId }}">{{ .UserName }}</a></li>
        <li><a href="/logout">Logout</a></li>
        {{ else }}
        <li><a href="/register">Register</a></li>
        <li><a href="/login">Login</a></li>
        {{ end }}
        <li><a href="/about">About</a></li>
</ul>
</nav>
{{ end }}
```


__Code from__ `./Frontend/templates/default.gohtml`:

```html
<!-- The main template in which other definitions are being included. -->
<!DOCTYPE html>
<head>
    <title>{{ .Title }}</title>
    {{ template "style" . }}
</head>
<body>
<div id="container">

{{ template "navbar" . }}
          
<div id="flex">
    <main>
        {{ template "content" . }}
    </main>
</div>
{{ template "footer" . }}
</div>
</body>
```
#### Exempelgästbok

Min lösning innehåller också en html-fil med exempel på hur en gästbok kan integreras i användarens hemsida. I det javaskript filen innehåller behöver värdet för `BASE_URL` modifieras för att matcha gästbokens identifikationssträng samt domännamnet där tjänsten kör.

```js
const BASE_URL = 'https://api.<HOST_NAME>/public/messages/<GUESTBOOK_ID>';
```

### Driftsättning

Jag har driftsatt min applikation under domänen guestbooks.online (observera att den antagligen inte finns tillgänglig där längre) och jag har för att demonstrera funktionalitet även gjort exempel koden tillgänglig på en separat domän (superellips.com).

#### Docker

Mitt arbete har sen dom första dagarna innehållit en strategi för att inkapsla applikationen, detta då jag i min utvecklingsmiljö utnyttjat _air_ och _delve_ för att direkt kunna testa förändringar i koden i en miljö som försöker efterlikna driftmiljön. All inkapsling sker med linux distributionen alpine som bas då detta resulterar i en liten avbildning med en relativt liten storlek. Mina _Dockerfile_ innehåller fyra steg:

##### Base

Detta steg ställer in vissa miljövariabler som senare används av go för att kompilera koden.

```Dockerfile
FROM golang:1.23-alpine AS base
WORKDIR /app

ENV GO111MODULE="on"
ENV GOOS="linux"
ENV CGO_ENABLED=0

RUN apk update && apk add --no-cache ca-certificates curl tzdata git && update-ca-certificates
```

##### Dev

Detta steg utgår från _base_ och bygger ut det med _air_ och _delve_, dessa utnyttjas för att känna av förändringar i kodbasen och kompilera den på nytt. I min utvecklingsmiljö så monters en mapp lokalt in som en volym dit koden kompileras och där den körs ifrån.

```Dockerfile
FROM base AS dev
WORKDIR /app

RUN go install github.com/air-verse/air@latest && go install github.com/go-delve/delve/cmd/dlv@latest

COPY go.mod go.sum ./
RUN go mod download

EXPOSE 8080
EXPOSE 2345

CMD [ "air", "-c", ".air.toml" ]
```

##### Builder

Detta steg kompilerar koden för driftsättning.

```Dockerfile
FROM base AS builder
WORKDIR /app

COPY go.mod go.sum .air.toml ./
RUN go mod download && go mod verify

COPY *.go ./
RUN go build -o ./<SERVICE_NAME>
```

##### Prod

Detta steg kopierar den kompilerade produktionskoden från _builder_ och kör den. Den kompilerade koden behöver inte att några ytterligare paket installeras och utgår därför direkt ifrån alpine.

```Dockerfile
FROM alpine:latest AS prod

ENV GIN_MODE="release"

COPY --from=builder /app/<SERVICE_NAME> /usr/local/bin/<SERVICE_NAME>
EXPOSE 8080

ENTRYPOINT [ "/usr/local/bin/<SERVICE_NAME>" ]
```

#### Docker compose

I min utvecklingsmiljö så körs resultaten av stegen _dev_ tillsammans med docker compose. I den fil som konfigurerar compose så definieras även relevanta miljövariabler samt körs kontainrar med databaser för att testa. Detta har inneburit att jag i det kontinuerliga arbetat med applikationen som helhet har kunnat få insikter kring vad som sker i bakgrunden. Att anslutningssträngar för databaser och addresser till tjänster styrs av miljövariabler har inneburit att förflyttningen från docker compose till kubernetes har skett väldigt smärtfritt.

#### Git, Github Actions och Container Registry

Varje applikation har ett eget repository som är kopplat mot Github och har var sitt flöde som körs automatiskt. Tjänsternas flöden bygger en ny docker avbildning som sedan laddas upp till Github Container registry varifrån dom kan hämtas in till produktionsmiljön.

Jag har i mitt kontinuerliga arbete enbart använt mig av privata repositories och registries på Github, detta för att jag gärna ville få en känsla för vilka extra steg jag skulle behöva ta för att driftsätta en applikation där källkoden inte kan publiceras offentligt. I [avsnitett bilagor](#privata-repositories-och-kontaineravbildningar) så har jag försökt sammanfatta dom steg som varit nödvändiga att ta för att detta skall fungera.

#### Kubernetes

För driftsättning så har jag provisionerat ett kuberneteskluster på [Linode](https://www.linode.com/) bestående av en nod. Mina beskrivningar i [bilagorna](#bilagor) är färgade av detta och kommandon kan behöva modifieras för att fungera med ett kluster tillhandahållet av någon annan.

Nedan finns en beskrivning av hur tjänsterna driftsätts men min lösning innehåller även en ingress och provisioneringen av den har skett utanför ArgoCD, ett försök att beskriva mitt tillvägagångssätt finns i [bilagorna](#ingress-och-tls-certifikat).

#### ArgoCD

För att genomföra kontinuerlig driftsättning så har jag i mitt kluster installerat ArgoCD. I vilket jag har definierat varje tjänst som en separat applikation.

![Skärmdump av webbgränsnittet för ArgoCD som visar tjänsterna driftsatta som separata applikationer. Alla tjänsterna visas som hälsosamma och synkroniserade.](Images/ArgoCD-applikationer.png)

Dom applikationer som definierats i ArgoCD består i sin tur av flera olika komponenter, i bilden nedan är dom komponenter som programmeringsgränssnittsknytpuntken består av. Dom tjänster som ansluter till en databas har även här sin tillhörande databasinstans.

![Skärmdump av webbgränsnittet för ArgoCD som visar status och komponenter för programmeringsgränssnittsknytpunkten som exempel på hur applikationerna är driftsatta.](Images/ArgoCD-gateway.png)

## Reflektion och Diskussion

Det har under utbildningen tydliggjorts många gånger att utveckling av mjukvara är ett kontinuerligt arbete som ofta saknar en definitiv slutpunkt och så även upplever jag det arbete jag har genomfört senaste veckorna. Jag blev inte _färdig_ med applikationen och det var inte heller mitt mål att ha en _färdig_ applikation nu när examensarbetet ska avslutas. Jag upplever alltså den applikation som jag producerat som inkomplett. Men det jag känner att jag har uppnått är en bra grund för fortsatt arbete, framförallt genom att jag tidigt identifierade att jag ville nyttja kubernetes för driftsättning. Det ledde till att jag tidigt, med inspiration i [en lösningen av Andrei Dascalu](https://dev.to/andreidascalu/setup-go-with-vscode-in-docker-for-debugging-24ch)[^3], lyckades åstadkomma en utvecklingsmiljö ur vilken jag upplevde det som väldigt lätt att lyfta applikationen till driftmiljö från. Att jag under utvecklingsarbetet även kunde se hur dom olika tjänsterna interagerade med varandra har varit ovärderligt när jag behövt felsöka. Att jag har haft god insyn i gränsnittet mellan tjänsterna upplever jag som väldigt lärorikt.

Det finns två delar av arbetet som jag fokuserat på mer än jag hade planerat för. Den första är presentationslagret, vilket jag inte hade planerat i någon detalj. Jag kände initialt att det hade låg prioritet. När jag i dom senare skedena hade kommit igång med det på riktigt så insåg jag att den lösning bestående av _gin_ och _html/templates_ bildade en i min mening stark grund för en applikation som följer MVC, jag upplever det som att mina tidigare kunskaper fördjupades av den insikten. Den andra är driftsättningen inom vilken jag arbetade mer med att hålla _repository_ och _container registry_ privata än vad jag hade planerat för. Min motivation var att försöka förstå i större detalj hur jag kunde lösa den utmaningen. Utöver det så registrerade jag även en domän under vilken jag driftsatte applikationen. Även det var lärorikt med kubernetes som kontext. 

Rent konkret så vill jag beskriva min slutgiltiga lösning som fungerande, den grundläggande funktionalitet jag eftersökte finns trots avsaknad av vissa viktiga kvalitativa attribut närvarande. Jag upplever att jag lagt en god grund för att kontinuerligt leverera uppdaterade versioner av applikationen. Jag upplever att jag har fått grundläggande kunskaper i utveckling av webbapplikationer med go.

## Slutsatser

Med min utgångspunkt för applikationen så har jag upplevt miktrotjänstarkitektur som begränsande och onödigt komplext. Att som ensam utvecklare behöva underhålla och arbeta i ett flertal separata kodbaser har inneburt visst repetitivt arbete där väldigt liknande kod producerats på flera ställen samt att överblicken av applikationen som helhet har blivit grumligare. Min slutsats är att en monolitisk applikation utvecklad med _Domain Driven Design_ hade inneburit en bättre utgångspunkt då min intuition är att det ur en sådan hade gått att extrahera viss funktionalitet till separata tjänster.

Kubernetes och Docker upplever jag som väldigt passande verktyg för miktrotjänster, driftsättning av flera separata tjänster vid sidan av varandra både i driftmiljö och utvecklingsmiljö har varit fördelaktivt och jag upplever att så även hade varit fallet för en monolitisk applikation.

Go som programmeringsspråk för webbapplikationer upplever jag också som väldigt passande. Det har varit lätt att finna information och stödet i form av bibliotek så som _gin_ gjorde det i min mening väldigt intuitivt att strukturera upp en applikation enligt MVC-modell.

Fördelar jag kan se med en applikation designad som miktrotjänster är att ett större antal utvecklare kan arbeta separat med dom olika kodbaserna och det är bara dom yttre programmeringsgränssnitten som behöver uprätthållas för att helheten ska fortsätta fungera. Jag upplever också att det kan finnas stora fördelar med att kunna skala bara dom delar av applikationen som ser störst belastning separat från dom andra tjänsterna, om t.ex. gästbokstjänsten skulle se en högre belastning än användartjänsten så behöver inte applikationen som helhet skalas upp utan bara dom tjänster som är belastade.

## Referenser

### Verktyg

- [Delve](https://github.com/go-delve/delve)
- [Air](https://github.com/air-verse/air)

### Studiematerial

- Andrei Dascalu, Setup go with vscode in docker for debugging, https://dev.to/andreidascalu/setup-go-with-vscode-in-docker-for-debugging-24ch
- Mehmet S., Bedir T., Ayça K. T. (2023) Microservice reference architecture design: A multi-case study (s. 69)

### Faktapåståenden

- Pereira, R., Couto, M., Ribeiro, F., Rua, R., Cunha, J., Fernandes, J. P., & Saraiva, J. (2021) Ranking programming languages by energy efficiency. (Figur 7)


[^1]: Pereira, R., Couto, M., Ribeiro, F., Rua, R., Cunha, J., Fernandes, J. P., & Saraiva, J. (2021) Ranking programming languages by energy efficiency. (Figur 7)
[^2]: Mehmet S., Bedir T., Ayça K. T. (2023) Microservice reference architecture design: A multi-case study (s. 69)
[^3]: Andrei Dascalu, Setup go with vscode in docker for debugging, https://dev.to/andreidascalu/setup-go-with-vscode-in-docker-for-debugging-24ch

## Bilagor

### Privata repositories och kontaineravbildningar

#### Tjänster och verktyg

- Linode Kubernetes Engine
- Linode CLI
- ArgoCD & ArgoCD CLI
- kubectl
- jquery

#### Installera och konfigurera ArgoCD

##### På linodes hemsida

1. Navigera till https://cloud.linode.com/kubernetes/clusters
2. Tryck _Create_
3. Mina val är enligt följande:
    - _Label:_ linode-1
    - _Region:_ se-sto (stockholm)
    - _HA Control Plane:_ no
    - _Control Plane ACL:_ disabled
    - _Node Pool:_
        - 1x Shared CPU - Linode 2GB
4. Tryck på _Create Cluster_

##### I en terminal

1. Generera en kubernetes konfigurationsfil med: `linode lke kubeconfig-view $(linode lke clusters-list --json | jq .[0].id) --text | sed -n '2 p' | base64 --decode > k8s-config.yaml`
2. Installera ArgoCD med: `kubectl apply --kubeconfig k8s-config.yaml -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`
3. När ArgoCD är installarat, gör _argocd-server_ tillgängligt på `localhost:8080` med _port-forward:_ `kubectl --kubeconfig k8s-config.yaml -n argocd port-forward svc/argocd-server 8080:443`
4. Logga in argocd-cli med: `argocd login --kube-context k8s-config.yaml --insecure --username admin --password $(argocd admin initial-password --kubeconfig=k8s-config.yaml --kube-context=$(kubectl config current-context --kubeconfig k8s-config.yaml) -n argocd | sed -n '1 p') localhost:8080`
5. Updatera lösenordet med: `argocd account update-password --current-password=$(argocd admin initial-password --kubeconfig=k8s-config.yaml --kube-context=$(kubectl config current-context --kubeconfig k8s-config.yaml) -n argocd | sed -n '1 p')`
6. Lägg till anslutningsuppgiter för repositories (körs en gång för varje repositry): `argocd repo add git@github.com:<GH_USER>/<REPO_NAME>.git --insecure-ignore-host-key --ssh-private-key-path </PATH/TO/SSH_KEY>`
7. Skapa en accesskey för ghcr.io (github container registry) och lägg till den som en hemlighet i kubernetes: `kubectl --kubeconfig k8s-config.yaml create secret docker-registry regcred --docker-server=ghcr.io --docker-username=<GH_NAME> --docker-password <ACCESS_TOKEN> --docker-email=<EMAIL>`
8. Skapa applikationer i ArgoCD (körs en gång för varje repository): `cd <REPO_FOLDER> && argocd app create <APP_NAME> --repo=$(git remote get-url origin) --path='./manifests' --auto-prune --dest-server https://kubernetes.default.svc --dest-namespace guestbook --sync-policy auto --sync-option='CreateNamespace=true' --self-heal --project default --kube-context=$(kubectl config current-context --kubeconfig k8s-config.yaml) && cd ..`

### Ingress och TLS certifikat

1. Skapa ett manifest för ingress (`ingress.yaml`) med följande innehåll och där `<HOST_NAME>` är domännamnet som tjänsterna kommer finnas tillgängliga under (i mitt faill `guestbooks.online`):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: application-ingress
  namespace: guestbook
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - api.<HOST_NAME>
    - www.<HOST_NAME>
    - <HOST_NAME>
    secretName: guestbook-tls
  rules:
  - host: api.<HOST_NAME>
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: apigateway-service
            port:
              number: 80
  - host: www.<HOST_NAME>
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: <HOST_NAME>
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

2. Skapa den hemlighet i kubernetes som förväntas (`guestbook-tls`), modifiera `path/to/` för nyckel och certifikat så att dessa matchar dom filer du har: `kubectl --kubeconfig k8s-config.yaml create secret tls guestbook-tls -n guestbook --cert=path/to/tls.crt --key=path/to/tls.key`
3. Applicera manifestet för ingress med: `kubectl --kubeconfig k8s-config.yaml apply -f ingress.yaml`
