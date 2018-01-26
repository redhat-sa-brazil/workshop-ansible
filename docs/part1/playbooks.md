# Ansible Playbooks

E finalmente chegamos nos Playbooks! Vamos discutir como escrever automações mais complexas, com multiplas etapas e controles, usando os componentes abordados nas sessões anteriores.

## Sintaxe Básica

Conforme já falamos, os **Ansible Playbooks** são escritos usando **YAML**, uma linguagem de marcação (não de programação). Ao invés de começarmos pequeno, vamos olhar um exemplo mais complexo e ir explicando aos poucos cada sessão: 

```yaml
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  tasks:
    - name: Garanta que o Apache está instalado.
      yum:
        name: httpd
        state: latest
    - name: Escreva o arquivo de configuração.
      template:
        src: httpd.j2
        dest: /etc/httpd/conf/httpd.conf
      notify:
        - restart apache
    - name: Garanta que o serviço está habilitado e iniciado.
      service:
        name: httpd
        state: started
        enabled: yes
  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```

No ponto de vista de sintaxe, o YAML é bem simples. Basicamente temos uma estrutura de **chave: valor** que usam indentação para definir escopo. Os valores podem ser **números**, **textos** e **listas**.

Todo Ansible Playbook inicia com a marcação `---`. Na sequencia começa uma lista (prefixo `-`) de escopos de automação, chamados de Plays. Dentro de um Playbook podemos ter diversos Plays, com escopos e tarefas distintas. Isso traz uma flexibilidade de poder fazer automações orquestradas. Cada **Play** define os sistemas alvos (`host`) e varíáveis (`vars`).

```yaml
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  tasks:
(...)
```
Além das variáveis de inventário (discutido na sessão anterior) e as variáveis de playbook, o Ansible PLaybook executa, de forma implícita, um módulo especial chamado `setup` antes das tarefas. Esse módulo coleta uma série de informações e populam variáveis. Essas variáveis podem ser acessadas via `{{ nome_variável }}` no Playbook.

Para ver quais variáveis são coletadas pelo modulo `setup`, use:

```bash
$ ansible -i inventario -m setup webserver.acme.com
```

Voltando ao Playbook, em seguida temos a chave `tasks` que tem como valor uma lista de tarefas a serem executadas. Cada tarefa é composta por um `name` (opicional), um módulo e seus argumentos. Por exemplo, vamos olhar a primeira tarefa da lista:

```yaml
    - name: Garanta que o Apache está instalado.
      yum:
        name: httpd
        state: latest
```

Estamos usando o módulo `yum`, com argumento `name` com valor `httpd` (nome do pacote Apache Web Server) e `state` com valor `latest` (instalado com a última versão). Esse padrão de construção segue igual para as outras tarefas.

No final, temos uma chave opcional `handlers` cujo o valor também é uma lista. Eles são similares às tarefas, mas só são chamados caso uma tarefa (da lista de `tasks`), que gere notificações, tenha sido executada. Essa é uma forma de criarmos assincronismo e evitar a repetição de tarefas iguais ou condicionais complexos. Os `handlers` só são executados uma vez, independente de quantas vezes são acionados, ao final da execução das `tasks`.

```yaml
(...)
    - name: Escreva o arquivo de configuração.
      template:
        src: httpd.j2
        dest: /etc/httpd/conf/httpd.conf
      notify:
        - restart apache
(...)
  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```

Os handlers são acionados pela chave `notify` e valor o nome do handler. No nosso caso, o nome é `restart apache`. Ao acionar handlers, podemos acionar vários e por isso o valor é uma lista.

## Execução

Para executar os Ansible Playbooks usamos uma sintaxe similar ao Ansible Ad-Hoc.

```bash
ansible-playbook -i inventario playbook.yaml
```

Perceba que não precisamos usar todos aqueles parâmetros que usamos com o Ansible Ad-Hoc pois eles agora estão incorporados ao Ansible Playbook. Entretanto algumas opções ainda são importantes:

* **-b** *indica que o Ansible Engine precisa subir de privilégio (padrão usa `sudo`) pois o usuário usado para conexão (`ansible_user`) não é privilegiado.*
* **-K** *indica que o Ansible Engine vai pedir a senha para executar a subida de privilégio (se o `sudo` demandar senha).*
* **-C** *indica que o Ansible Engine vai rodar sem realizar mudanças, apenas validar o que deveria ser alterado (dry-run).*

A saída da execução do Ansible Playbook, conforme o comando acima, seria dessa forma:

```bash
PLAY [webservers] ****************************************************************************

TASK [Gathering Facts] ***********************************************************************
ok: [webserver.acme.com]
ok: [jboss.acme.com]

TASK [Garanta que o Apache está instalado.] **************************************************
changed: [webserver.acme.com]
changed: [jboss.acme.com]

TASK [Escreva o arquivo de configuração.] ****************************************************
ok: [webserver.acme.com]
ok: [jboss.acme.com]

TASK [Garanta que o serviço está habilitado e iniciado.] *************************************
changed: [jboss.acme.com]
changed: [webserver.acme.com]

PLAY RECAP ***********************************************************************************
jboss.acme.com      : ok=4    changed=2    unreachable=0    failed=0   
webserver.acme.com  : ok=4    changed=2    unreachable=0    failed=0  
```

Cada linha indica a execução em um alvo, que pode ter 4 tipos de resultados:

* **ok:** *indica que não foi necessário realizar mudanças na tarefa.*
* **changed:** *indica que foi necessário realizar mudanças e a tarefa foi realizada com sucesso.*
* **failed:** *indica que um erro foi gerado na execução da tarefa.*
* **unreachable:** *indica que o Ansible não conseguiu conectar no alvo.*

## Controles de Execução

No Ansible Engine existem diversas formas de controlar o fluxo de execução das tarefas de automação dentro de um Ansible Playbook. Por simplicidade, vamos abordar dois tipos: **condicionais** e **laços**.

### Condicionais

Podemos usar uma palavra chave nas tarefas chamada `when`, que cria uma condição para aquela ser executada. Dessa forma conseguimos usar variáveis na decisão de execução da tarefa:

```yaml
    - name: Garanta que o Apache está instalado.
      yum:
        name: httpd
        state: latest
      when: ansible_memtotal_mb >= 1024
```

A variável `ansible_memtotal_mb` não foi declarada explicitamente, mas coletada pelo módulo `setup` na tarefa listada como `Gathering Facts`.

Se quisermos criar condicionais que capturam a saída de tarefas anteriores, fazemos uso da palavra chave `register`. Vamos olhar esse exemplo:

```yaml
    - name: Garanta que o Apache está instalado.
      yum:
        name: httpd
        state: latest
      register: output_yum
    - name: Garanta que o serviço está habilitado e iniciado.
      service:
        name: httpd
        state: restarted
        enabled: yes
      when: output_yum.changed
```

Aqui só vamos executar a segunda tarefa se a primeira teve alguma mudança. Ou seja, só reiniciaremos o serviço `httpd` na instalação inicial, ou quando houver atualizações.

### Laços

Esses são os mais simples. Eles permitem iterar uma lista de valores e reaproveita-los na execução de uma tarefa. A primeira construção mais comum é usando o `with_items`, conforme o exemplo abaixo:

```yaml
    - name: Garanta que o Apache está instalado.
      yum:
        name: "{{ items }}"
        state: latest
      with_item:
        - httpd
        - vim
        - httpd-tools
```

Aqui estamos substituindo o `{{ item }}` para cada item da lista em `with_items`. Dessa forma essa tarefa será executada 3 vezes, poupando tempo de escrita e deixando o Ansible Playbook mais conciso.

A segunda construção mais comum é usando o `with_sequence`, conforme o próximo exemplo:

```yaml
    - name: Garanta que o Apache está instalado.
      lineinfile:
        path: /etc/hosts
        line: "192.168.0.{{ item }} server{{ item }} server{{ item }}.acme.com" 
      with_sequence: start=01 end=05
```

Aqui estamos criando 5 entradas no arquivo `/etc/hosts` seguindo a construção especificada em `line`.

## Exercícios

**1)** Crie e execute um Ansible Playbook para todos os servidores capaz de:
  * Instalar os pacotes **vim**.
  * Garantir que o **SELinux** está como **Enforcing**.
  * Garantir que o serviço **tuned** está habilitado no boot e inciado.
  * Configurar o arquivo `/etc/motd` tem a mensagem **Bem-vindo ao servidor {{ machine_type }}**, sendo `machine_type` a variável de inventário.

```bash
cat <<EOF > meuplaybook.yaml
---
- hosts: all
  tasks:
    - name: Pacote vim deve estar presente
      yum:
        name: vim
        state: latest
    
    - name: SELinux deve estar habilitado e enforcing
      selinux:
        policy: targeted
        state: enforcing

    - name: Serviço Firewalld deve estar habilitado e rodando
      service:
        name: tuned
        enabled: yes
        state: started

    - name: Define MOTD padrão
      copy: 
        content: 'Bem-vindo ao servidor {{ machine_type }}'
        dest: /etc/motd
EOF

ansible-playbook meuplaybook.yaml
```

**2)** Modifique e execute novamente o playbook para sobrescrever a variável `machine_type` com `Red Hat Enterprise Linux 7`:

```bash
cat <<EOF > meuplaybook.yaml
---
- hosts: all
  vars:
    machine_type: Red Hat Enterprise Linux 7
  tasks:
    - name: Pacote vim deve estar presente
      yum:
        name: vim
        state: latest
    
    - name: SELinux deve estar habilitado e enforcing
      selinux:
        policy: targeted
        state: enforcing

    - name: Serviço Firewalld deve estar habilitado e rodando
      service:
        name: tuned
        enabled: yes
        state: started

    - name: Define MOTD padrão
      copy: 
        content: 'Bem-vindo ao servidor {{ machine_type }}'
        dest: /etc/motd
EOF
```

Dessa vez vamos adicionar a opção `--diff` para ver o que vai ser alterado no `/etc/motd`:

```bash
ansible-playbook --diff meuplaybook.yaml
```

**3)** Modifique novamente o playbook para incluir no `/etc/motd` o hostname da máquina alvo:

```bash
# cat <<EOF > meuplaybook.yaml
---
- hosts: all
  vars:
    machine_type: Red Hat Enterprise Linux 7
  tasks:
    - name: Pacote vim deve estar presente
      yum:
        name: vim
        state: latest
    
    - name: SELinux deve estar habilitado e enforcing
      selinux:
        policy: targeted
        state: enforcing

    - name: Serviço Firewalld deve estar habilitado e rodando
      service:
        name: tuned
        enabled: yes
        state: started

    - name: Define MOTD padrão
      copy: 
        content: 'Bem-vindo ao servidor {{ ansible_hostname}} ({{ machine_type }})'
        dest: /etc/motd
EOF
```

Execute o playbook com o `--diff`:

```bash
ansible-playbook --diff meuplaybook.yaml
```

**4)** Agora altere o playbook para incluir a instalação adicional dos pacotes `vim` e `wget`, e execute novamente:

```bash
cat <<EOF > meuplaybook.yaml
---
- hosts: all
  vars:
    machine_type: Red Hat Enterprise Linux 7
  tasks:
    - name: Pacotes bash-completion, vim e wget devem estar presentes
      yum:
        name: '{{ item }}'
        state: latest
      with_items:
        - bash-completion
        - vim
        - wget
    
    - name: SELinux deve estar habilitado e enforcing
      selinux:
        policy: targeted
        state: enforcing

    - name: Serviço Firewalld deve estar habilitado e rodando
      service:
        name: tuned
        enabled: yes
        state: started

    - name: Define MOTD padrão
      copy: 
        content: 'Bem-vindo ao servidor {{ ansible_hostname}} ({{ machine_type }})'
        dest: /etc/motd
EOF

ansible-playbook meuplaybook.yaml
```

**5)** Crie uma condição que o `/etc/motd` só vai ser configurado se a máquina gerenciado tiver mais do que 4GB de memória total:

```bash
cat <<EOF > meuplaybook.yaml
---
- hosts: all
  vars:
    machine_type: Red Hat Enterprise Linux 7
  tasks:
    - name: Pacotes bash-completion, vim e wget devem estar presentes
      yum:
        name: '{{ item }}'
        state: latest
      with_items:
        - bash-completion
        - vim
        - wget

    - name: Serviço Firewalld deve estar habilitado e rodando
      service:
        name: firewalld
        enabled: yes
        state: started

    - name: Define MOTD padrão
      copy: 
        content: 'Bem-vindo ao servidor {{ ansible_hostname}} ({{ machine_type }})'
        dest: /etc/motd
      when:
        - ansible_memtotal_mb >= 4096
EOF

# ansible-playbook meuplaybook.yaml
```

**6)** Modifique o playbook e adicione duas tarefas adicionais:
  * Garantir que o pacote `chrony` esta instalado.
  * Caso ele seja instalado, garanta que o serviço `chronyd` será habilitado e iniciado.

```bash
cat <<EOF > meuplaybook.yaml
---
- hosts: all
  vars:
    machine_type: Red Hat Enterprise Linux 7
  tasks:
    - name: Pacotes bash-completion, vim e wget devem estar presentes
      yum:
        name: '{{ item }}'
        state: latest
      with_items:
        - bash-completion
        - vim
        - wget

    - name: Pacote Chrony deve estar instalado
      yum:
        name: chrony
        state: latest
      register: chronytask

    - name: Serviço Chrony deve estar habilitado e rodando
      service:
        name: chronyd
        enabled: yes
        state: started
      when: chronytask.changed

    - name: Serviço Firewalld deve estar habilitado e rodando
      service:
        name: firewalld
        enabled: yes
        state: started

    - name: Define MOTD padrão
      copy: 
        content: 'Bem-vindo ao servidor {{ ansible_hostname}} ({{ machine_type }})'
        dest: /etc/motd
      when:
        - ansible_memtotal_mb >= 4096
EOF

ansible-playbook meuplaybook.yaml
```

Se rodarmos mais uma vez?

```bash
ansible-playbook meuplaybook.yaml
```

**7)** Refatore o exercício anterior para usar `handlers` ao invés de condicional. Antes, de executar novamente o playbook, remova o pacote `chrony` de todos os servidores:

```bash
ansible -m yum -a 'name=chrony state=absent' all

cat <<EOF > meuplaybook.yaml
---
- hosts: all
  vars:
    machine_type: Red Hat Enterprise Linux 7
  tasks:
    - name: Pacotes bash-completion, vim e wget devem estar presentes
      yum:
        name: '{{ item }}'
        state: latest
      with_items:
        - bash-completion
        - vim
        - wget

    - name: Pacote Chrony deve estar instalado
      yum:
        name: chrony
        state: latest
      notify:
        - Serviço Chrony deve estar habilitado e rodando

    - name: Serviço Firewalld deve estar habilitado e rodando
      service:
        name: firewalld
        enabled: yes
        state: started

    - name: Define MOTD padrão
      copy: 
        content: 'Bem-vindo ao servidor {{ ansible_hostname}} ({{ machine_type }})'
        dest: /etc/motd
      when:
        - ansible_memtotal_mb >= 4096

  handlers:
    - name: Serviço Chrony deve estar habilitado e rodando
      service:
        name: chronyd
        enabled: yes
        state: started
EOF

ansible-playbook meuplaybook.yaml
```

Mais uma vez?

```bash
ansible-playbook meuplaybook.yaml
```