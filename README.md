Argo CD telepítése lokális Kubernetes clusterre (Alpine Linux)
1. Argo CD telepítése a Kubernetes clusterbe

Ha már rendelkezésre áll egy lokális Kubernetes cluster, akkor a management Alpine Linux gépről adjuk ki a következő parancsokat.

Namespace létrehozása
bash
kubectl create namespace argocd

Argo CD telepítése
bash
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


A telepítés ezzel elindul. A komponensek állapotát az alábbi paranccsal ellenőrizhetjük:

bash
kubectl get pods -n argocd


Forrás: Argo CD – Getting Started

2. Argo CD CLI telepítése Alpine Linuxra

A hivatalos Getting Started útmutató tartalmazza a CLI telepítését is. Az ott található curl-os telepítési folyamat Alpine Linuxhoz igazított változata:

Bináris letöltése
bash
curl -fSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

Futtathatóvá tétele
bash
chmod +x argocd-linux-amd64

Áthelyezés a PATH-ban található könyvtárba
bash
mv argocd-linux-amd64 /usr/local/bin/argocd


Fontos: az mv parancshoz root jogosultság szükséges, mivel a /usr/local/bin könyvtárba általában csak rendszergazdaként lehet írni.

Alternatív megoldásként a letöltésnél rögtön átnevezhetjük a fájlt:

bash
curl -fSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64


Ebben az esetben a CLI az aktuális könyvtárból a ./argocd paranccsal indítható. így nem szükséges a /usr/local/bin könyvtárba másolni.

Telepítés ellenőrzése
bash
argocd version --client


Ha nem a PATH-ban található fájlt használjuk:

bash
./argocd version --client


Forrás: Argo CD – Getting Started

3. Argo CD elérésének beállítása (Port Forwarding)

Mivel lokális Kubernetes clustert használunk, nincs cloud providerünk, amely LoadBalancer szolgáltatást biztosítana.

Ezért a Port Forwarding megoldást választjuk.

3.1. VirtualBox NAT port forwarding szabály

A VirtualBox NAT hálózatán vegyünk fel egy új port forwarding szabályt.

Például:

Beállítás	Érték
Name	ArgoCD
Protocol	TCP
Host IP	127.0.0.1
Host Port	8080
Guest IP	Alpine VM NAT IP-címe
Guest Port	8080

A Guest IP mezőbe az Alpine VM tényleges NAT IP-címét kell írni (például 10.0.2.15), nem a 127.0.0.1 címet.

A Host Port és Guest Port lehet más is, de a példában mindkettő 8080.

3.2. Kubernetes Port Forwarding indítása

Az Alpine SSH termináljában adjuk ki:

bash
./kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443


Ez összeköti az Alpine VM 8080-as portját az Argo CD szerver 443-as portjával.

FONTOS: Ez a parancs lefoglalja a terminált, és futva kell hagyni a kapcsolat fenntartásához. Nyissunk egy új SSH sessiont a további parancsokhoz!

4. Kezdeti admin jelszó lekérése

Az Argo CD automatikusan generál egy kezdeti admin jelszót.

Lekérdezése:

bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


A kezdeti bejelentkezési adatok:

Username: admin
Password: az előző parancs által visszaadott véletlenszerű jelszó.
5. Admin jelszó megváltoztatása

Először jelentkezzünk be az Argo CD CLI segítségével:

bash
argocd login localhost:8080 --insecure


A localhost:8080 a szerver címe. Ha a fenti útmutatót követtük, akkor ez lesz a megfelelő cím.

A kezdeti admin jelszóval jelentkezzünk be.

Ezután módosítsuk a jelszót:

bash
argocd account update-password


A CLI bekéri:

A jelenlegi jelszót.
Az új jelszót.
Az új jelszó megerősítését.

Megjegyzés: Az új jelszónak legalább 8 karakter hosszúnak kell lennie.

6. Bejelentkezés a webes felületre

Nyissuk meg a böngészőben:

https://localhost:8080

Mivel az Argo CD alapértelmezés szerint self-signed tanúsítványt használ, a böngésző biztonsági figyelmeztetést jeleníthet meg.

A tanúsítvány elfogadása után tovább lehet lépni az oldalra.

Bejelentkezési adatok:

Username: admin
Password: az általunk korábban beállított új jelszó.

Ha minden lépést sikeresen végrehajtottunk, elérhető az Argo CD webes felülete!

7. Hasznos ellenőrző parancsok

A telepítés és a működés ellenőrzéséhez:

bash
# Argo CD podok állapota
kubectl get pods -n argocd

# Argo CD service-ek
kubectl get svc -n argocd

# Argo CD CLI verzió
argocd version --client

# Bejelentkezett felhasználó ellenőrzése
argocd account get-user-info


Kész! Az Argo CD telepítve van a lokális Kubernetes clusterben, és a webes felülete elérhető a host gép böngészőjéből.