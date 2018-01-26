# Pré-requisitos

## Cliente

Para você conseguir aproveitar o conteúdo desse workshop, você deveria ter pelo menos um computador (Windows, Linux ou OSX) com acesso à internet. Esse computador será usado como cliente para acessar o ambiente usados nos exercícios. Por isso, você vai precisar:

1. **Editor de texto com suporte a YAML**. *Recomendamos [Atom](https://atom.io/), [Visual Studio Code](https://code.visualstudio.com/) ou [Sublime](https://www.sublimetext.com/). Sem dúvida você pode usar [Vim](http://www.vim.org/) ou [GNU Emacs](https://www.gnu.org/software/emacs/) também!*
2. **Cliente de SSH**. *Para Linux ou OSX, não precisa fazer mais nada pois seu sistema já tem o cliente nativo. Para Windows, recomendamos o [Putty](http://www.putty.org/).*

## Laboratório

Além do computador, precisamos de um ambiente de testes para validar as nossas automações. Esse ambiente deve conter, pelo menos 5 máquinas virtuais:

<p align="center">
![ansible_architecture](media/lab_diagram.png)
</p>

* **Máquina de Suporte (support.lab.redhat.com):** *É uma máquina complementar que servirá como repositório de pacotes local.*
* **Máquina Tower (tower.lab.redhat.com):** *Conforme vamos ver no workshop, precisamos de uma máquina de controle para executar nossas automações. Essa máquina também servirá para hospedar o Ansible Tower (AWX).*
* **Máquinas Webserver/DBServer/JBoss (webserver/dbserver/jboss.lab.redhat.com):** *Máquinas que servirão como servidores Linux (RHEL/CentOS), alvos das nossas automações.*

Essas máquinas devem aceitar login como **root** e senha **Redhat1!@2018**, exceto a máquina **tower.lab.redhat.com**, que tem login **workshop-user**. 

**Se você estiver fazendo esse workshop em um evento Red Hat Brasil, esse laboratório (LATAM-SA-BR-AnsibleWorkshop-v2) já estará pronto e disponível!** Caso você esteja realizando esse workshop sozinho, você pode construir esse laboratório usando máquinas virtuais, através de ferramentas como [Virt-Manager](https://virt-manager.org/), [Oracle VM VirtualBox](https://www.virtualbox.org/) ou [VMware Workstation Player](https://www.vmware.com/products/workstation-player.html). 