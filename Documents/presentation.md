---
marp: true
theme: gaia
class:
    - lead
    - invert
paginate: true
---

# Mikrotjänster och golang

---

![bg 75%](./Images/golang-thumb.png)

<!-- 
Notes: 
- 2009
- Rob Pike, Robert Griesemer, Ken Thompson
- BSD license (opensource) 
-->

---

## Go

```golang
func loadPage(title string) (*Page, error) {
	filename := "/app/pages/" + title + ".html"
	body, err := os.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}
```

<!-- 
Notes:
- hittills: lätt att komma in i men svårt att bemästra
- upplever det ganska annorlunda jämfört med C#
-->

---

```Dockerfile
FROM base AS dev
WORKDIR /app

RUN |
    go install github.com/air-verse/air@latest && \
    go install github.com/go-delve/delve/cmd/dlv@latest

COPY go.mod go.sum ./
RUN go mod download

EXPOSE 8080
EXPOSE 2345

CMD [ "air", "-c", ".air.toml" ]
```

<!-- 
Notes:
- alla verktyg verkar finnas MEN är mer DIY
- lätt att kontaineriser med docker
-->

---

## Applikationskoncept

Tjänst för att skapa och hantera gästböcker.

---

## Arkitektur

!["Min arkitektur referens."](./Images/Draft-Design.png)


<!-- 
Notes: 
- Referensarkitektur 
- API Gateway eller mer tjänst-till-tjänst kommunikation
-->

--- 

![bg 75%](./Images/k8s-thumb.png)

<!-- 
Notes:
- vissa delar av referensarkitekturen känns som att dom är inbyggda i k8s
- t.ex. lastbalancering och service discovery
-->
---

## Demo

---

## Tankar

---

## Frågor
