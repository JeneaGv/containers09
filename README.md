# Optimizarea imaginilor

# Scopul lucrării

Scopul lucrării este de a se familiariza cu metodele de optimizare a imaginilor.

# Sarcina 

Compararea diferitelor metode de optimizare a imaginilor:

<ul type="disk">
<li>Ștergerea fișierelor temporare și a dependențelor neutilizate</li>
<li>Reducerea numărului de straturi</li>
<li>Utilizarea unei imagini de bază minime</li>
<li>Reambalarea imaginii</li>
<li>Utilizarea tuturor metodelor</li>
</ul>

# Execuție

Cream un repozitoriu containers09 și copiem pe computerul nostru. 

În directorul containers09 cream directorul site și plasam în el fișierele site-ului (html,js).

![image](https://github.com/user-attachments/assets/a9e6a3e0-1004-45ab-80e8-ef4f8393bf5b)


Pentru teste de optimizare a imaginilor va fi utilizată imaginea creată în bază Dockerfile.raw:

```
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Cream în folderul containers09 și construim imaginea cu numele mynginx:raw:

![image](https://github.com/user-attachments/assets/6f218cff-64ca-4045-b4f1-3bc057f5fc20)

# Eliminarea dependențelor neutilizate și a fișierelor temporare

Eliminam fișierele temporare și dependențele neutilizate în Dockerfile.clean:

```
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# remove apt cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Asamblam imaginea cu numele mynginx:clean și verificam dimensiunea:

![image](https://github.com/user-attachments/assets/175e9b45-5ccb-489f-b28e-52514fccd549)

![image](https://github.com/user-attachments/assets/72e3be05-1430-4351-b2ba-f1ba8da905e1)

# Minimizarea numărului de straturi

Minimizam numărul de straturi în Dockerfile.few:

```
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y nginx && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Construim imaginea cu numele mynginx:few și verificam dimensiunea:

docker image build -t mynginx:few -f Dockerfile.few .
docker image list

![image](https://github.com/user-attachments/assets/359f89d5-8fa2-4705-a5de-683744611822)

![image](https://github.com/user-attachments/assets/093ac163-44ac-4732-bb76-56a9c8a630ea)

# Utilizarea unei imagini de bază minime

Înlocuim imaginea de bază cu alpine în Dockerfile.alpine:

```
# create from alpine image
FROM alpine:latest

# update system
RUN apk update && apk upgrade

# install nginx
RUN apk add nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Construim imaginea cu numele mynginx:alpine și verificam dimensiunea:

![image](https://github.com/user-attachments/assets/fde08576-37e3-48fe-995d-7e143ff910ab)

![image](https://github.com/user-attachments/assets/5081e029-f904-4ac6-ad16-520bc29a9bc3)

# Repachetarea imaginii

Repachetam imaginea mynginx:raw în mynginx:repack:

docker container create --name mynginx mynginx:raw
docker container export mynginx | docker image import - mynginx:repack
docker container rm mynginx
docker image list

![image](https://github.com/user-attachments/assets/d9fc82b5-def5-49d5-8c00-c9f889ec4dde)

# Utilizarea tuturor metodelor

Cream imaginea mynginx:minx utilizând toate metodele de optimizare. Pentru aceasta definim următorul Dockerfile.min:

```
# create from alpine image
FROM alpine:latest

# update system, install nginx and clean
RUN apk update && apk upgrade && \
    apk add nginx && \
    rm -rf /var/cache/apk/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Construim imaginea cu numele mynginx:minx și verificam dimensiunea. Repachetam imaginea mynginx:minx în mynginx:min:

docker image build -t mynginx:minx -f Dockerfile.min .
docker container create --name mynginx mynginx:minx
docker container export mynginx | docker image import - myngin:min
docker container rm mynginx
docker image list

![image](https://github.com/user-attachments/assets/440d5770-50e7-48fe-a0e1-5a14af58c2db)

Pornire și testare
Verificam dimensiunea imaginilor:

docker image list

![image](https://github.com/user-attachments/assets/4ed8ed4c-3cd5-44fb-8a5e-d64d3183c289)

# Răspunsuri la întrebări

<b>Care metodă de optimizare a imaginilor vi se pare cea mai eficientă?</b>

Cea mai eficientă metodă pare a fi combinația dintre utilizarea unei imagini de bază minime (Alpine în loc de Ubuntu) și repachetarea imaginii. 

Utilizarea Alpine reduce semnificativ dimensiunea inițială a imaginii, deoarece Alpine este concepută special ca o distribuție minimalistă. 

Când această metodă este combinată cu repachetarea (așa cum s-a făcut pentru imaginea mynginx:min), se obțin rezultate optime de reducere a dimensiunii.

<b>De ce curățirea cache-ului pachetelor într-un strat separat nu reduce dimensiunea imaginii?</b>

Curățirea cache-ului pachetelor într-un strat separat nu reduce dimensiunea imaginii deoarece în Docker, fiecare instrucțiune RUN creează un nou strat.

Chiar dacă într-un strat ulterior se șterg fișiere create în straturile anterioare, aceste fișiere rămân fizic prezente în straturile anterioare. 

Docker folosește un sistem de fișiere în straturi, unde straturile sunt imutabile.

Astfel, atunci când se șterge ceva într-un strat nou, se marchează doar ca șters în acel strat, dar datele rămân în straturile anterioare, ocupând spațiu. 

Pentru eficiență, curățirea trebuie făcută în același strat RUN în care s-au instalat pachetele.

<b>Ce este repachetarea imaginii?</b>

Repachetarea imaginii este procesul prin care se creează un container din imaginea existentă, se exportă conținutul acestuia și apoi se importă înapoi ca o nouă imagine. 

Acest proces comprimă toate straturile imaginii inițiale într-un singur strat, eliminând straturile intermediare și datele neutilizate. 

Comanda docker container export exportă sistemul de fișiere al containerului ca un singur arhivă tar, iar docker image import creează o nouă imagine din această arhivă.

Repachetarea este eficientă pentru a elimina "istoricul" nefolosit al unei imagini și pentru a reduce dimensiunea finală.

# Concluzie 

În urma experimentelor de optimizare a imaginilor Docker, am constatat că dimensiunea acestora poate fi redusă semnificativ prin combinarea mai multor tehnici: 

utilizarea unei imagini de bază minimale (Alpine), reducerea numărului de straturi, curățarea cache-ului în același strat în care au fost create fișierele și repachetarea imaginii.

Optimizarea imaginilor Docker este esențială deoarece imaginile mai mici se transferă mai rapid, ocupă mai puțin spațiu de stocare și au o suprafață de atac redusă pentru vulnerabilități. 

Combinația dintre Alpine și repachetare s-a dovedit a fi cea mai eficientă metodă în acest context experimental.
