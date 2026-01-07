# **Kubernetes: Hello from the Cluster**

## **Úvod: Co tahle ukázka demonstruje**

Kubernetes je systém, který za vás:

- spouští kontejnery,

- udržuje je běžící (self-healing),

- a umožňuje je snadno škálovat (přidávat / ubírat kopie).

V téhle jednoduché ukázce běží malý web („Hello from Kubernetes“), který spouštíme *víckrát paralelně* a Kubernetes:

1. vytvoří potřebný počet instancí (Podů),

2. vystaví je přes jednu společnou adresu (Service),

3. znovu vytvoří Pod, který schválně smažeme,

4. umí počet běžících instancí jedním příkazem změnit.



## **0. Start lokálního Kubernetes clusteru (minikube)**

Předpoklady:

- nainstalovaný Docker Desktop

- nainstalované minikube a kubectl

- repozitář, ve kterém jsou soubory deployment.yaml a service.yaml

### Příkazy
```bash
# spuštění lokálního clusteru
minikube start --driver=docker

# ověření, že cluster běží a máme 1 node
kubectl get nodes
```

Co se děje:
Minikube vytvoří malý Kubernetes cluster běžící lokálně (v Dockeru). kubectl se na něj připojí a ukáže dostupné nody a Pody.

## **1. Cluster je prázdný**
```bash
kubectl get pods
kubectl get deployments
kubectl get services
```
Co se děje:
Ověření toho, že zatím v clusteru nic neběží.

## **2. Nasazení aplikace pomocí Deploymentu**

V repozitáři je soubor deployment.yaml, který definuje:

- Deployment hello-deployment

- 2 repliky (2 Pody) malé „hello“ webové aplikace

- kontejner nginxdemos/hello:plain-text na portu 80

Příkazy
```bash
# nasazení Deploymentu ze souboru
kubectl apply -f deployment.yaml

# kontrola, že Deployment existuje
kubectl get deployments

# zobrazení vytvořených Podů
kubectl get pods
```

Co se děje:
Kubernetes vytvoří Deployment, který se postará o to, aby běžely 2 Pody s webovou aplikací. kubectl get pods ukáže konkrétní běžící instance.

## **3. Zpřístupnění aplikace přes Service**

V repozitáři je soubor service.yaml, který definuje:

- Service hello-service

- typ NodePort (otevře port na nodu, typicky 30080)

- selector app: hello-k8s, který propojí Service s našimi Pody

Příkazy
```bash
# vytvoření Service
kubectl apply -f service.yaml

# kontrola, že Service běží
kubectl get services

# zobrazení, na které Pody (IP:port) Service směruje provoz
kubectl get endpoints hello-service
```


Co se děje:
Service získá label-selektorem seznam Podů a vytvoří na ně jednotnou „vstupní bránu“ – jednu adresu/port, přes kterou se na web dostaneme.

## **4. Otevření webu v prohlížeči**

Minikube umí přímo otevřít URL služby v prohlížeči.

Příkazy
```bash
# otevře hello-service v prohlížeči
minikube service hello-service

# případně jen vytiskne URL
minikube service hello-service --url
```

Co se děje:
Minikube najde, na jakém NodePortu naše Service běží, a otevře odpovídající URL. Uvnitř stránky je vypsané i jméno serveru/Podu, takže je vidět, na jakou instanci jsme se trefili.

## **5. Self-healing: smazání Podu a jeho obnova**

Deployment hlídá, že běží správný počet Podů. Když jeden smažeme, měl by se automaticky vytvořit nový.

Příkazy
```bash
# najde aktuální Pody
kubectl get pods

# smažeme jeden z Podů (dosadit konkrétní jméno)
kubectl delete pod <jmeno-podu>

# sledujeme změny, dokud se znovu neobjeví 2 Pody
kubectl get pods
```

Co se děje:
Pod ručně smažeme. Deployment si toho všimne (počet replik už není 2) a vytvoří nový Pod, aby znovu splnil požadovaný stav.

## **6. Škálování: změna počtu běžících kopií**

Počet replik aplikace jde jednoduše měnit – například při různé zátěži.

Příkazy
```bash
# zvětšení počtu replik na 5
kubectl scale deployment hello-deployment --replicas=5

# kontrola Deploymentu a Podů
kubectl get deployments
kubectl get pods

# zmenšení počtu replik na 1
kubectl scale deployment hello-deployment --replicas=1

# opět kontrola
kubectl get pods
```

Co se děje:
Příkazem kubectl scale změníme počet požadovaných replik. Kubernetes podle toho přidá nebo smaže Pody, aby se skutečný stav rovnal požadovanému.

## **Závěr: Co si z demo ukázky odnést**

V téhle ukázce jsme viděli tři základní stavební kameny Kubernetes:

- Pod – jedna běžící instance aplikace.

- Deployment – deklaruje, kolik Podů má běžet, a stará se o jejich životní cyklus (včetně self-healingu).

- Service – poskytuje stabilní adresu, přes kterou se na Pody připojujeme.

**Dohromady umožňují:**

- jednoduše spustit aplikaci,

- vystavit ji ven uživatelům,

- přežít pád jednotlivých instancí,

- a snadno škálovat podle potřeby jedním příkazem.