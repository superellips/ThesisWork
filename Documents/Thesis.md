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
av mönstret _Ports and Adapters_. 

Jag har valt att för mitt utforskande arbeta med _golang_, ett
programmeringsspråk som av tre huvudsakliga anledningar ger
intryck av att vara väl passat för _Micro-service_ arkitetur.
Den första är att det finns ett gott stöd för webb-funktioner
i form av standard bibliotek. Den andra av är att koden
kompileras till en körbar fil vilket kan underlätta driftsättning. 
Den tredje är att dessa applikationer kan utnyttja relativt få
resurser[^1].

Målet med arbetet är att i första hand skapa en övergripande
förståelse för hur en _Micro-service_ arkitektur kan realiseras och
vilka fördelar kontra nackdelar detta innebär i jämförelse med att
utveckla en monolitisk applikation. Arbetet ska också ge grundläggande
kunskaper i utveckling av webbapplikationer med _golang_.

[^1]: Pereira, R., Couto, M., Ribeiro, F., Rua, R., Cunha, J., Fernandes, J. P., & Saraiva, J. (2021). Ranking programming languages by energy efficiency. (Figur 7)

## 

