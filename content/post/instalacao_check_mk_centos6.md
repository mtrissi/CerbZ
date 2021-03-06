+++
authors = [
  "Mateus",
]

title = "Instalação do Check_MK no CentOS 6"
date = "2020-06-21"
description = "    O check_MK é uma ferramenta, de código aberto, para realizar monitoramento da infraestrutura de TI. Esta postagem é um tutorial para a instalação do Check_MK 1.5 Raw Edition no CentOS 6."

tags = [
  "tutorial",
  "check_mk",
  "centos",
  "centos 6",
  "linux",
  "omd",
]

images = [
  "check_mk_logo.png",
]
+++
# Introdução
&nbsp;&nbsp;&nbsp;&nbsp;O [Check_MK](https://checkmk.com/ "Check_MK's Homepage") é uma ferramenta, de código aberto, para realizar monitoramento da infraestrutura de TI. Oferece dados sobre avaliabilidade e performance dos equipamentos (com métricas e gráficos), alertas dinâmicos e sua interface é amigável e totalmente customizável.

&nbsp;&nbsp;&nbsp;&nbsp;Esta postagem serve de tutorial para a instalação do __Check_MK 1.5 Raw Edition__ no sistema operacional __CentOS 6__.

&nbsp;

# Particionar o diretório /OPT
&nbsp;&nbsp;&nbsp;&nbsp;O diretório ```/opt``` é tradicionalmente usado para softwares de terceiros.

&nbsp;&nbsp;&nbsp;&nbsp;O Check_MK é instalado no path ```/opt/omd```. Se o servidor for usado exclusivamente pelo Check_MK, então é aconselhável que ```/opt``` ou o ```/opt/omd``` tenha uma partição própria.

&nbsp;&nbsp;&nbsp;&nbsp;Vantagens de particionar o ```/opt```:

* Não afeta a estrutura do sistema operacional;
* Todos os arquivos referentes ao Check_MK ficam em um único lugar (facilitando encontrar arquivos de configuração, por exemplo);
* Maior facilidade em redimensionar o disco;
* Possibilidade de utilizar diferentes Sistemas de Arquivos (ext4, JFS, XFS);
* Maior facilidade na hora de realizar o backup.

&nbsp;

# Tempo do sistema
&nbsp;&nbsp;&nbsp;&nbsp;Para que o Check_MK use um horário correto utilizaremos o ```ntpdate```.

```bash
yum check-update
yum -y install ntpdate
ntpdate -u 0.br.pool.ntp.org
```

&nbsp;&nbsp;&nbsp;&nbsp;O __0.br.pool.ntp.org__ é apenas um servidor [NTP](https://pt.wikipedia.org/wiki/Network_Time_Protocol "Network Time Protocol"), pode ser substituído por qualquer outro de sua preferência.

&nbsp;&nbsp;&nbsp;&nbsp;Edite o arquivo da crontab com ```vi /etc/crontab``` e adicione a seguinte linha para que o comando ```ntpdate``` seja executado sozinho 1 vez a cada hora:

```bash
9 * * * * root ntpdate 0.br.pool.ntp.org
```

&nbsp;

# SELinux
&nbsp;&nbsp;&nbsp;&nbsp;O Security-Enhanced Linux é uma arquitetura de segurança que permite que administradores tenham mais controle sobre quem pode acessar o sistema. Usando políticas de segurança, um conjunto de regras que dizem ao SELinux o que pode ou não ser acessado, ele define controles de acesso para aplicações, processos e arquivos em um sistema.

&nbsp;&nbsp;&nbsp;&nbsp;Aqui temos duas opções:

###### Desabilitar SELinux

&nbsp;&nbsp;&nbsp;&nbsp;Podemos desabilitar o SELinux (não aconselhado para ambientes de [produção](https://bsoft.com.br/blog/ambiente-de-producao-e-homologacao "Explicação ambiente de produção")) editando o arquivo ```/etc/selinux/config```.

```bash
vi /etc/selinux/config
```

&nbsp;&nbsp;&nbsp;&nbsp;Troque ```enforcing``` por ```disabled``` e reinicie o servidor com o comando ```reboot```.

&nbsp;

###### Configurar o SELinux

&nbsp;&nbsp;&nbsp;&nbsp;Podemos adicionar regras que permitam o funcionamento do Check_MK usando o [audit2allow](https://linux.die.net/man/1/audit2allow "audit2allow linux man page"). Aqui tem um [tutorial](https://andhersonsilva.wordpress.com/2016/10/04/apresentando-o-audit2allow-para-configurar-politicas-no-selinux/ "Tutorial audit2allow") para o ```audit2allow```.

&nbsp;&nbsp;&nbsp;&nbsp;Para instalar o pacote que contém o audit2allow basta executar o comando:

```bash
yum install policycoreutils-python
```

&nbsp;

# EPEL
&nbsp;&nbsp;&nbsp;&nbsp;Como estamos realizando a instalação do Check_MK 1.5, se faz necessário configurar o repositório EPEL (Extra Packages for Enterprise Linux) para a instalação de determinados pacotes que o Check_MK precisa. Para tanto, basta executar o comando:

```bash
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
yum check-update
```

&nbsp;&nbsp;&nbsp;&nbsp;Assim criamos o repositório EPEL para o CentOS 6. Podemos confirmar isso com o comando:

```bash
cat /etc/yum.repos.d/epel.repo
```

&nbsp;

# Instalação do Check_MK

&nbsp;&nbsp;&nbsp;&nbsp;Agora, com todo o necessário já configurado, é simples:

&nbsp;&nbsp;&nbsp;&nbsp;Realizamos o download do Check_MK (arquivo será salvo com o nome "check_mk-1.5.rpm"):

```bash
curl -L https://checkmk.com/support/1.5.0p24/check-mk-raw-1.5.0p24-el6-38.x86_64.rpm -o check_mk-1.5.rpm
```

&nbsp;&nbsp;&nbsp;&nbsp;Antes de instalar o pacote, faça o download da chave e realize sua importação:

```bash
curl -L https://checkmk.com/support/Check_MK-pubkey.gpg -o Check_MK-pubkey.gpg
rpm --import Check_MK-pubkey.gpg
```

&nbsp;&nbsp;&nbsp;&nbsp;Agora sim, vamos instalar o Check_MK:

```bash
yum install check_mk-1.5.rpm
```

&nbsp;&nbsp;&nbsp;&nbsp;Para verificar se tudo ocorreu corretamente, execute o seguinte comando ```omd version```.

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;Outras versões do check_MK estão disponíveis [aqui](https://checkmk.com/download.php "Página de download do Check_MK").

&nbsp;

# Criação do site

&nbsp;&nbsp;&nbsp;&nbsp;Com o OMD podemos criar nosso site de monitoramento (é possível criar quantos sites quiser). Vamos chamar nosso site de "monitoramento" (plausível?):

```bash
omd create monitoramento
```

![OMD Create](https://www.cerbz.com/images/omd_create.jpg "OMD Create")

&nbsp;&nbsp;&nbsp;&nbsp;Então, vamos iniciar nosso site:

```bash
omd start monitoramento
```

&nbsp;&nbsp;&nbsp;&nbsp;Caso não tenha anotado a senha do site, mude para o usuário monitoramento com ```su monitoramento``` e troque a senha com o comando:

```bash
htpasswd -m ~/etc/htpasswd cmkadmin
```

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;Agora é possível acessar o site recém criado, para tanto utilize seu navegador e acesse ```server-name-or-ip-address/site_name```. Por exemplo, o endereço IP do meu servidor é 192.168.0.177 e o nome do site que criei é monitoramento, logo devo acessar a URL ```192.168.0.177/monitoramento```.

![Check_MK Login Page](https://www.cerbz.com/images/check_mk_login_page.jpg "Check_MK Login Page")

![Check_MK Index Page](https://www.cerbz.com/images/check_mk_index_page.jpg "Check_MK Index Page")

&nbsp;&nbsp;&nbsp;&nbsp;Caso não esteja funcionando, o motivo pode ser o bloqueio do site pelo firewall do CentOS. Desabilite o firewall com o comando ```service iptables stop``` e tente acessar o site novamente.

&nbsp;&nbsp;&nbsp;&nbsp;Funcionando? Agora só é preciso adicionar quais itens você quer monitorar. Futuramente farei uma postagem sobre o assunto.
