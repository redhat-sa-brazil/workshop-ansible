# Introdução

Nessa primeira parte do workshop iremos discutir conhecimentos básicos sobre o motor de automação Ansible, também chamado de **Ansible Engine**. Vamos explorar a arquitetura básica dos seus componentes, modos de funcionamento e melhores práticas para criação de automações.

## Motivação

Antes de começarmos a nos aprofundar nas questões técnicas, devemos pensar na motivação. Porque estamos pensando em usar Ansible? A resposta pode ser respondida em três pontos:

* **Não precisa de agentes nas máquinas e dispositivos alvos de automação.** *O Ansible se utiliza de protocolos padrão de gerência remota, como SSH ou WinRM, para se conectar nos alvos e realizar suas automações. A recomendação é ter pelo menos o runtime Python instalado, mas não é mandatório para seu funcionamento.*

* **Linguagem simples e legível das automações e configurações.** *O Ansible usa de arquivos INI e YAML, simplificando o aprendizado e adoção da tecnologia para todos, inclusive por pessoas sem conhecimento de programação.*

* **Flexibilidade para atender diversos casos de uso.** *O Ansible pode ser usado para gestão de configuração de sistemas, orquestração de ambientes, provisionamento de aplicações e automação de redes.*

Além desses pontos, o Ansible possui uma [comunidade](https://www.ansible.com/community) muito ativa e tem sido adotado por corporações pelo mundo. Estamos convencidos?

## Arquitetura

O **Ansible Engine**, ou somente Ansible, é um motor de automação de tarefas, flexível e robusto, escrito em Python, de código 100% aberto. Ele é composto por diversos componentes com funções específicas:

<p align="center">
![ansible_architecture](media/ansible_architecture.png)
</p>

* **Módulos:** *Esses componentes implementam abstrações de atividades específicas, utilizados como peças dentro de uma automação.*
* **Plugins:** *São componentes que podem agregar funcionalidades complementarem ao motor de automação.*
* **API/SDK:** *Interface Python usada para integração com outras soluções sem necessidade de usar cli-scrapping.*
* **Inventários:** *Arquivo textual, gerado manualmente ou por programas externos, que lista um conjunto de nós (servidores, dispositivos de rede, instâncias de nuvem, etc) que serão alvo de automação pelo Ansible.*
* **Playbook:** *Arquivo YAML, escrito por usuários ou administradores de sistemas, que descreve um conjunto de automações, usando módulos, que serão executadas sobre alvos específicos de um inventário.*

## Funcionamento

Uma vez esclarecida, mesmo que superficialmente, a arquitetura do Ansible Engine podemos olhar como a solução é de fato operacionalizada. Quando olhamos nossos sistemas, podemos classifica-los em duas categorias.

* **Máquina de Controle:** *São os sistemas usados como a origem das automações. Normalmente estamos falando de uma máquina Linux/Unix de onde o Ansible Engine vai estabelecer as conexões SSH.*
* **Máquina Gerenciada** : *São os sistemas alvos para as automações do Ansible Engine. Estes são os sistemas/equipamentos listados nos Inventários, como discutido na sessão anterior.*

O Ansible Engine executa suas automações através de uma conexão de gerência, com origem na **Máquina de Controle** e destino a **Máquina Gerenciada**. O uso de protocolos de gerência remota padrão, como SSH e WinRM, permite o Ansible não precisar de agentes.

<p align="center">
![ansible_works](media/ansible_works.png)
</p>

ALém disso, as máquinas gerenciadas pelo Ansible Engine precisam apenas do runtime Python, algo presente nativamente na maioria dos sistemas operacionais modernos.

## Instalação

Conforme explicado anteriormente, o Ansible é escrito em Python e demanda que o *runtime* esteja instalado na **Máquina de Controle**. Entretanto, podemos usar a mágica de **resolução de dependência** e deixar o **Yum** fazer seu trabalho!

Usando o RHEL 7 como **Máquina de Controle**, precisamos ativar o repositório EXTRAS antes de instalar o Ansible Engine:

```bash
sudo subscription-manager repos --enable rhel-7-server-extras-rpms
sudo yum install ansible
```

Caso você esteja usando CentOS 7 como **Máquina de Controle**, você precisa ativar o EPEL antes de instalar o Ansible Engine:

```bash
sudo yum install epel-release
sudo yum install ansible
```

Já no Fedora 27, é mais simples:

```bash
sudo yum install ansible
```

Para validar como o Ansible Engine está instalado no seu ambiente, você pode usar:

```bash
ansible --version
```

## Exercícios

**1)** Conecte na **máquina de controle** do seu laboratório (ex.: tower) por SSH, e verifique se o Ansible Engine está instalado e qual sua versão:

```bash
sudo yum info ansible
ansible --version
```