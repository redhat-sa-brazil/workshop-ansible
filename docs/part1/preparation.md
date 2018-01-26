# Preparação

Nesse capítulo vamos abordar como preparar o ambiente da nossa máquina de controle para executar nossas automações com o Ansible Engine. Vamos discutir como modificar os parâmetros do motor de automação, assim como definir quais são nossas máquinas gerenciadas.

## Configuração

Ao instalar o Ansible Engine na **máquina de controle**, foi criado um arquivo de configuração que dita as características de funcionamento padrão do motor de automação. Esse arquivo é no formato **INI**, que é basicamente composto por **chave = valor** e separadores de sessão **[categoria]**.

Vamos dar uma olhada no seu conteúdo:

```bash
# less /etc/ansible/ansible.cfg
```

**Não é recomendado alterar os valores nesse arquivo, pois ele afeta o comportamento para todos os usuários da máquina de controle.** Para facilitar a customização das características, o Ansible Engine busca por configurações em localidades mais específicas, seguindo do menos ao mais preferível:

* **~/.ansible.cfg -** *Na raiz do diretório home do usuário que está invocando o Ansible Engine.*
* **./ansible.cfg -** *Mesmo diretório de onde o Ansible Engine está sendo invocado.*

Você poderia também usar **variáveis de ambiente** para alterar as características padrão, mas não vamos abordar essa alternativa.

## Inventários

Inventários são listas de **endereços IP ou FQDN** que o Ansible Engine usa para determinar as máquinas gerenciadas e seus grupos, como alvo para executar as suas automações.

Existem dois tipos de inventários:

* **Estático:** *Esse é o tipo de inventário mais simples, textual, no formato INI (como o ansible.cfg). Por exemplo, temos o seguinte inventário:*

```ini
[webservers]
webserver.acme.com
jboss.acme.com

[dbservers]
db.acme.com

[production]
support.acme.com

[production:children]
webservers
dbservers
```

Grupos são definidos por `[grupo]`, podendo uma máquina gerenciada (host) pertencer a múltiplos grupos. Além disso, podemos ter grupos dentro de grupos usando `[grupo:children]` para definir o supergrupo.

* **Dinâmicos:** *Esse tipo de inventário é gerado por aplicações, scripts ou sistemas externos, no formato JSON. Ele é muito usado quando o inventário é muito volátil, sendo difícil de gerenciar manualmente. Por exemplo, poderíamos ter um CMDB gerando o seguinte inventário, identico ao exemplo anterior:*

```json
{
    "webservers": ["webserver1.acme.com", "webserver2.acme.com"],
    "dbservers": ["db.acme.com"],
    "production": {
        "hosts": ["support.acme.com"],
        "children": ["webservers", "dbservers"]
    }
}
```

## Variáveis

Variáveis são construções fundamentais para a criação de automações reaproveitáveis. Elas podem servir como parametrização de acordo com características dos sistemas, ou equipamentos.

No ponto de vista de inventário, temos possibilidade de criar dois tipos de variáveis:

* **Host Vars:** *Variáveis que são individuais aos sistemas. Elas podem ser definidas dentro do inventário, como exemplificado abaixo, ou em um diretório seguindo a convenção `./host_vars/hostname`.*

```ini
[webservers]
webserver.acme.com ansible_user=root
jboss.acme.com ansible_user=root

[dbservers]
db.acme.com ansible_user=root type=postgres

[production]
support.acme.com ansible_user=admin

[production:children]
webservers
dbservers
```

* **Group Vars:** *Variáveis que são aplicáveis a todos os sistemas de um grupo. Assim como o tipo anterior, elas podem ser definidas dentro do inventário, como exemplificado abaixo, ou em um diretório seguindo a convenção `./group_vars/hostname`.*

```ini
[webservers]
webserver.acme.com
jboss.acme.com

[dbservers]
db.acme.com type=postgres

[production]
support.acme.com ansible_user=admin

[production:children]
webservers
dbservers

[all:vars]
ansible_user=root
ansible_connection=ssh
```

## Exercícios

**1)** Conectado na sua **máquina de controle**, crie um diretório chamado `workshop` e use-o como raiz para os próximos exercícios:

```bash
mkdir -p ~/workshop
cd ~/workshop
```

**2)** Escreva um arquivo de inventário onde:
  * **webserver**, **jboss**, **dbserver** do domínio **lab.redhat.com**
  * os dois primeiros sistemas devem estar no grupo **web**
  * o segundo deve estar no grupo **jboss**
  * o terceiro sistema deve estar no grupo **db**
  * o grupo **linux** deve conter todos os sistemas dos outros grupos

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
EOF
```

**3)** Usando arquivos de configuração, defina que:
  * Não vamos validar autenticidade de novas chaves SSH;
  * Vamos usar até 2 conexões simultâneas;

```bash
cat <<EOF > ./ansible.cfg
[defaults]
forks = 2
host_key_checking = False
EOF
```

**4)** Verifique novamente qual o arquivo de configuração que o Ansible Engine irá utilizar após a criação do arquivo do exercício anterior:

```bash
$ ansible --version
ansible 2.4.2.0
  config file = /home/workshop-user/workshop/ansible.cfg
  configured module search path = [u'/home/workshop-user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, May  3 2017, 07:55:04) [GCC 4.8.5 20150623 (Red Hat 4.8.5-14)]
```
