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

