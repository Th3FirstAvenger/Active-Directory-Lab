# Desplegar el Laboratorio
https://github.com/Th3FirstAvenger/Active-Directory-Lab/


# Instalación infraestructura
## Dependencias para las herramientas
```bash
sudo apt update
sudo apt install tmux proxychains4 golang docker.io python3-pip ruby ruby-dev python3.8-venv -y
```

## Impacket

```bash
git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
cd /opt/impacket
python3 -m pip install .
```
https://github.com/SecureAuthCorp/impacket

## Evil-WinRM
```bash
gem install evil-winrm
```
https://github.com/Hackplayers/evil-winrm

## Crackmapexec 
```bash
python3 -m pip install pipx
pipx ensurepath
pipx install crackmapexec
export PATH=$PATH:/root/.local/bin
```
https://github.com/byt3bl33d3r/CrackMapExec/wiki/Installation

## Chisel

```bash
curl -L https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz -o chisel.gz;gzip -d
chisel.gz;chmod +x chisel
```
https://github.com/jpillora/chisel

## Empire: 

```bash
git clone --recursive https://github.com/BC-SECURITY/Empire.git /root/Empire 
```

Modificar el fichero 

Preparar entorno de trabajo. 
_vim Empire/empire/client/config.yaml_

Añadir ip pública para conectarse al servidor. 


## TMUX

```bash
tmux new -s LAB
```
> | 0 - C2 | 1 - PIVOT | 2 - LATERAL | 3 - OTHER |

Desde Others se crea un directorio en la raiz llamado uploads

```bash
mkdir ~/uploads
```

Iniciamos un servicio http con python a este directorio

```bash
python3 -m http.server 8888 --dir ~/uploads
```
