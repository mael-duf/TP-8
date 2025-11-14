# TP 8

## Étape 1 — Configuration IP de base

### IP et masque :
#### fw (vers lan_int) : 192.168.10.1/24
<img width="1042" height="88" alt="image" src="https://github.com/user-attachments/assets/d39d7b6f-6190-46b2-9994-bbe6e336966e" />

#### fw (vers dmz) : 192.168.20.1/24
<img width="1130" height="91" alt="image" src="https://github.com/user-attachments/assets/cdaefa27-b84a-4d27-b93f-157301658103" />

#### lan_int : 192.68.10.2/24
<img width="1191" height="92" alt="image" src="https://github.com/user-attachments/assets/4648b039-8f90-4845-aedf-65e5312c88c9" />

#### dmz : 192.168.20.2/24
<img width="1105" height="92" alt="image" src="https://github.com/user-attachments/assets/ea2a3e1a-e484-4c06-a499-a5aebe21af63" />

### Ping de lan-cli vers l'ip lan_int de fw
<img width="780" height="376" alt="image" src="https://github.com/user-attachments/assets/c89b71d2-18e9-4667-a968-46f1d6ef6b43" />

### Ping de dmz-srv vers l'ip dmz de fw
<img width="727" height="354" alt="image" src="https://github.com/user-attachments/assets/1c000f6b-d29e-4ed7-9cfa-da6d410ff13f" />

## Étape 2 — Routage simple entre LAN et DMZ

### Sur fw j'ai fait la commande ```sudo sysctl -w net.ipv4.ip_forward=1``` pour activer le routage IPV4 temporairement

### Ping de dmz-srv depuis lan-cli : 
<img width="662" height="349" alt="image" src="https://github.com/user-attachments/assets/6d4b61a1-2059-447a-8227-17c6a44b36f4" />

### Ping de lan-cli depuis dmz-srv :
<img width="718" height="353" alt="image" src="https://github.com/user-attachments/assets/0e248849-1d05-43b6-b0c0-f674e671f2f2" />

## Étape 3 — Mise en place d’un serveur dans la DMZ

### Sur dmz-srv :
```
sudo apt update
sudo apt install apache2 -y
sudo systemctl status apache2
```
<img width="1851" height="384" alt="image" src="https://github.com/user-attachments/assets/f2a932a6-702f-497c-a4b0-64fb9dc931d1" />

### Activer le routage sur lan-cli : ```sudo sysctl -w net.ipv4.ip_forward=1```

### Test les connectiviter
#### Sur lan-cli :
```
ping 192.168.10.1
ping 192.168.20.2
```
#### Sur dmz-srv :
```
ping 192.168.20.1
ping 192.168.10.2
```

### Test l'accès au serveur web Apache
#### Sur lan-cli :
```
wget http://192.168.20.2
```
<img width="1811" height="223" alt="image" src="https://github.com/user-attachments/assets/505ab78c-4a54-4f9d-a0cf-b3e2cb2992cf" />

## Étape 4 — Introduction du pare-feu

### Pour le pare feux , dans le terminal de fw :

```
sudo iptables -P INPUT DROP  // Définit la politique par défaut : tout trafic entrant vers fw est bloqué
sudo iptables -P OUTPUT DROP  // Tout trafic sortant depuis fw est bloqué par défaut
sudo iptables -P FORWARD DROP  // Tout transit (trafic routé entre LAN et DMZ) est bloqué par défaut

sudo iptables -A INPUT -i lo -j ACCEPT  // Autorise tous les flux sur l'interface locale (loopback)
sudo iptables -A OUTPUT -o lo -j ACCEPT  // Autorise tous les flux sortants sur l'interface locale

sudo iptables -A FORWARD -p icmp -s 192.168.10.0/24 -d 192.168.20.0/24 -j ACCEPT  // Autorise les pings (ICMP) du LAN vers le DMZ
sudo iptables -A FORWARD -p tcp --dport 80 -s 192.168.10.0/24 -d 192.168.20.2 -j ACCEPT // Autorise le trafic HTTP (port 80) LAN vers le serveur web DMZ
sudo iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT // Autorise les réponses aux connexions établies (retour HTTP/ping, etc.)
```

### Tests
#### Depuis lan-cli ping dmz-srv et accès au service :
<img width="1799" height="499" alt="image" src="https://github.com/user-attachments/assets/893ed24d-f583-4214-8a3d-4ff55c85e92c" />

#### Depuis dmz-srv ping lan-cli :
<img width="847" height="131" alt="image" src="https://github.com/user-attachments/assets/a9e97181-6a1b-47e8-8a65-19ff702e2139" />

### Il est bien bloquer comme indiquer dans le pare feu

## Étape 5 — Pare-feu plus strict + logs et observation

### Sur fw, « tout est bloqué sauf ce que j’autorise » :
```
sudo iptables -P INPUT DROP  // Tout ce qui arrive sur fw est bloqué par défaut
sudo iptables -P OUTPUT DROP   // Tout ce qui sort de fw est bloqué par défaut
sudo iptables -P FORWARD DROP // Aucun routage LAN<->DMZ n'est autorisé par défaut
```

### Autoriser uniquement : 
- ping dmz-srv depuis lan-cli
- accès au service lan-cli
- administration de dmz-srv depuis fw

```
sudo iptables -A INPUT -i lo -j ACCEPT    // Autorise tout ce qui est local (loopback)
sudo iptables -A OUTPUT -o lo -j ACCEPT  // Pareil

sudo iptables -A FORWARD -p icmp -s 192.168.10.0/24 -d 192.168.20.0/24 -j ACCEPT  // Autorise le ping du LAN vers la DMZ
sudo iptables -A FORWARD -p tcp --dport 80 -s 192.168.10.0/24 -d 192.168.20.2 -j ACCEPT // Autorise HTTP du LAN vers DMZ

sudo iptables -A OUTPUT -p tcp --dport 22 -d 192.168.20.2 -j ACCEPT  // Autorise SSH sortant fw→dmz-srv (inutile si pas demandé)
sudo iptables -A INPUT -p tcp --sport 22 -s 192.168.20.2 -j ACCEPT    // Autorise retour du SSH dmz-srv→fw
```

### Ajouter une règle de log pour les paquets refusés
```
sudo iptables -A FORWARD -j LOG --log-prefix "[FW DROP] " --log-level 4   // Envoie une trace dans /var/log/syslog pour chaque paquet refusé
```

### Tests
#### Sur lan-cli ping et accès au service de dmz :
<img width="1782" height="518" alt="image" src="https://github.com/user-attachments/assets/90701eb9-ba15-4c8b-bd71-a9042902a235" />

#### Sur lan-cli, un port non autoriser :
<img width="780" height="189" alt="image" src="https://github.com/user-attachments/assets/f1c74ff9-8587-4153-b350-c049567f263f" />

#### Logs des paquet bloquer :
<img width="1273" height="84" alt="image" src="https://github.com/user-attachments/assets/d6a72def-d9d8-4aa2-81b7-60adb78afc24" />

### Pour expliquer, j'ai bloquer tous depuis le pare feu et après j'ai autoriser seulement ce qui étais demander.

## Étape 6 — Scénarios bonus 

## Je choisis le scénario 3 : Mini-DNS local pour la DMZ

### Je commence par ```sudo nano /etc/hosts``` sur lan-cli
### J'ai rajouter la ligne ```192.168.20.2   srv.tp.local```
### Enregistrez puis quitter le nano
### Et enfin ```wget http://srv.tp.local```
<img width="1794" height="280" alt="image" src="https://github.com/user-attachments/assets/ce1a7abe-aacf-464d-ad24-056999c67fe7" />

### Comme sa, on accède a dmz grâce a srv.tp.local au lieu de l'ip
