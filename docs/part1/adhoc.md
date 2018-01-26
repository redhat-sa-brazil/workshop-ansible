# Ansible Ad-Hoc

Agora vamos aprender a forma mais simples de se usar o Ansible Engine para automatizar uma única tarefa nos alvos listados nos nossos inventários. Dessa forma conseguimos usar o motor de automação para resolver problemas pontuais ou automatizar tarefas não estruturadas.

## Execução

Uma vez com o inventário e configurações definidas, podemos começar a explorar o uso do Ansible Engine para automação de tarefas. A forma mais fácil de fazer é através do que chamamos de execução **Ad-Hoc**, onde o Ansible Engine é usado para executar em paralelo a mesma tarefa em diversos alvos do nosso inventário.

Como exemplo, podemos testar a seguinte construção:

```bash
$ ansible -i inventario -u root -k -a 'uname -a' all 
```

Sobre as opções e argumentos, temos:

* **-i** *indica qual o inventário que vamos usar (poderia ser um script/programa com permissão de execução).*
* **-u** *indica qual o usuário que será usado para conectar na máquina gerenciada (alvo).*
* **-k** *indica pro Ansible Engine que vamos conectar por SSH usando senhas (ao invés do padrão que usa certificados).*
* **-a** *indica quais as opções estamos usando no módulo (que como não foi indicado pela opção **-m**, por padrão ele usa o módulo **command**).*
* **all** *indica o padrão (pattern) de alvo que estamos buscando para nossa automação. No caso, todos os sistemas do inventário.*

Algumas opções importantes:

* **-b** *indica que o Ansible precisa subir de privilégio (padrão usa `sudo`) pois o usuário usado para conexão (`ansible_user`) não é privilegiado.*
* **-K** *indica que o Ansible Engine vai pedir a senha para executar a subida de privilégio (se o `sudo` demandar senha).*
* **-l** *permite limitar o escopo de atuação nas máquinas alvos de maneira mais granular (todos do grupo Linux, menos o jboss).*
* **-e** *permite definir, ou sobrescrever, variáveis no momento da execução da automação.*

## Módulos

Conforme explicado na sessão [Introdução](part1/introduction.md), o Ansible Engine vem com as baterias incluídas. Essas baterias são os módulos, que implementam a lógica por traz de uma tarefa de automação.

Dos módulos mais básicos, temos:

* **command:** *Módulo padrão na execução Ad-Hoc, que executa um comando sem o suporte de um emulador de terminal (shell).*
* **shell:** *Módulo que permite o uso de construções como pipe (|) e redirecionadores de saída (>, >>), além de dsponibilizar as variáveis de ambiente do sistema.*
* **raw:** *Módulo que não demanda de suporte ao Python pelo **managed node**. Usado em último caso, quando o alvo é muito limitado ou não atende os pre-requisitos básicos do Ansible.*

Além desses, temos uma infinidade de outros módulos. Podemos obter uma listagem através do comando:

```bash
ansible-doc -l
```

Para obter mais informações sobre um módulo, usamos:

```bash
ansible-doc service
```

Também existe a documentação no formato web (ufa!) de todos os módulos:

[http://docs.ansible.com/ansible/latest/modules_by_category.html](http://docs.ansible.com/ansible/latest/modules_by_category.html)

A partir da documentação, podemos entender quais são os argumentos necessários para o módulo e assim podemos usá-lo na execução ad-hoc:

```bash
ansible -i inventario -k -m service -a 'name=chronyd state=stopped' webservers
```

## Idempotência

O Ansible Engine é uma ferramenta de automação de tarefas que trabalho em cima de estados dos sistemas. Ao usar os módulos, estes buscam avaliar o estado atual do sistema alvo e, se necessário, realiza mudanças. Isso significa que eu posso executar diversas vezes a mesma automação sem gerar inconsistência ou resultados diferentes. Essa característica se chama idempotência.

A maior parte dos módulos implementam suas logicas para que essa característica seja satisfeita. Entretanto, os módulos `command`, `shell` e `raw` não possuem essa característica. Por essa razão, use-os somente quando não existir módulos especializados!

## Exercícios

**1)** Validar se conseguimos alcançar nossas máquinas gerenciadas cadastradas no inventário:

```bash
ansible -m ping -i meuinventario all
```

Porque não funcionou? Porque não temos autenticação por chaves configurado no SSH. Nesse caso, precisamos usar usuário-senha:

```bash
ansible -m ping -k -i meuinventario all
```

Ainda não funcionou? O problema é que por padrão o Ansible Engine usa o mesmo usuário que está chamando a automação. No nosso caso, ele está tentando usar o usuário 'workshop-user', mas ele não existe nas máquinas alvo. Precisamos dizer que queremos usar o `root`:

```bash
ansible -m ping -k -u root -i meuinventario all
```

**2)** Gere uma chave pública para o usuário **cloud-user** com o comando `ssh-keygen`. Depois, através do uso do Ansible Engine via execução ad-hoc, encontre o módulo que adicione essa chave como autorizada para o usuário **root** de todos os sistemas do inventário. Valide executando o primeiro exemplo dessa sessão sem a opção **-k** sob todos os sistemas do inventário.

```bash
ssh-keygen
cat ~/.ssh/id_rsa.pub
(...)
ansible -m authorized_key -i meuinventario -a 'key="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDOyutraKOe3z4cd0vJEuUDUaVKI3MkJf0DU5rQwHiggFvqCU1RMl6tvwvza4xkTA1was/ipnWNBPK56D3y6a7hcvPgv49LMzwqoGVklGBwOTEMzQfwFuH+CRUnY8SggYmb+K6W/cqFcqtKQcI+BTh776bL8v/NMRLiX+Z/SJKzelDtfDOzwmGGSPJr/SIFfBDFYzTICLdNmM4Fme/2gTrOmA5zcWMaCax5Bh2NgiFXr+k5PKw5bb4fQy7ChUQ5KV8pXWjNl/mn+uKwaxGazK9Mdnrl21gQNHKOg4gD28Dy2MxILSTekQW8aOloLhMeK1TT4ODct8/F4ag+Txx28Aeb workshop-user@tower.lab.redhat.com" user=root' -u root -k all
```

Para validar se as chaves estão copiadas corretamente, vamos tentar usar `ping` mas sem especificar senha com `-k`:

```bash
ansible -m ping -u root -i meuinventario all
```

**3)** Para simplificar a execução de automações vamos configurar o nosso inventário padrão no `ansible.cfg` local.

```bash
cat <<EOF >> ./ansible.cfg
[defaults]
forks = 20
host_key_checking = False
inventory = meuinventario
EOF
```

Também vamos modificar nosso inventário e definir a variável `ansible_user = root` para todos as máquinas alvos:

```bash
cat <<EOF > ./meuinventario
[web]
webserver.lab.redhat.com
jboss.lab.redhat.com

[db]
dbserver.lab.redhat.com

[jboss]
jboss.lab.redhat.com

[linux:children]
web
db
jboss

[all:vars]
ansible_user = root
EOF
```

Vamos tentar usar o `ping` mas agora sem especificar mais nada além do alvo e módulo:

```bash
ansible -m ping all
```

**4)** Rode a mesma tarefa que o exercício anterior, mas evitando o host `jboss.lab.redhat.com`:

```bash
ansible -m ping all -l '!jboss.lab.redhat.com'
```

**5)** Instale pacote 'bash-completion', e garanta que o serviço `firewalld` está configurado e habilitado em todos os servidores Linux:

```bash
ansible -m yum -a 'name=bash-completion state=latest' linux
ansible -m service -a 'name=firewalld enabled=yes state=started' linux
```

Você percebeu que algumas tarefas não mudaram os estados das máquinas alvo?

**6)** Modifique o inventário para incluir uma variável `machine_type` que vai ser Linux para todos os servidores, exceto para o `jboss.lab.redhat.com` que será `jboss`:

```bash
cat <<EOF > ./meuinventario
[web]
webserver.lab.redhat.com
jboss.lab.redhat.com

[db]
dbserver.lab.redhat.com

[jboss]
jboss.lab.redhat.com machine_type=jboss

[linux]
jboss.lab.redhat.com

[linux:children]
web
db

[all:vars]
ansible_user = root

[linux:vars]
machine_type=linux
EOF
```

**7)** Execute uma tarefa de automação que use o valor da variável `machine_type` para cada configurar o `/etc/motd` com a mensagem `Seja bem-vindo ao servidor (<tipo>)`:

```bash
ansible -m copy -a 'content="Seja bem-vindo ao servidor ({{ machine_type }})" dest=/etc/motd' all
```

Use o módulo `command` e valide com `cat` o conteúdo do arquivo `/etc/motd`:

```bash
ansible -a 'cat /etc/motd' all
```

**8)** Execute a mesma automação, mas faça com que a variável seja sobrescrita com o valor `workshop` para todos os alvos:

```bash
ansible -m copy -a 'content="Seja bem-vindo ao servidor ({{ machine_type }})" dest=/etc/motd' -e machine_type=workshop all
```

Use novamente  o módulo `command` e valide com `cat` o conteúdo do arquivo `/etc/motd`:

```bash
ansible -a 'cat /etc/motd' all
```