# Desafio-docker-Cluster

# Documentação do Codigo # 

Esta documentação fornece uma visão geral e instruções para usar o Vagrantfile e scripts relacionados em seu projeto.

## Índice
- [Introdução](#introdução)
- [Vagrantfile](#vagrantfile)
- [docker.sh](#dockersh)
- [master.sh](#mastersh)
- [worker.sh](#workersh)

## Introdução

Este projeto usa o Vagrant para automatizar a criação e configuração de máquinas virtuais para o seu ambiente de desenvolvimento. O Vagrantfile fornecido define as configurações das máquinas virtuais, e existem vários scripts shell usados para provisionar as máquinas com o Docker e configurar um Docker Swarm.

## Vagrantfile

O Vagrantfile contém a configuração das máquinas virtuais. Ele define três máquinas: `master`, `node01` e `node02`. Cada máquina tem configurações específicas de memória, CPU, IP e imagem.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby  :

machines = {
  "master" => {"memory" => "1024", "cpu" => "1", "ip" => "100", "image" => "bento/ubuntu-22.04"},
  "node01" => {"memory" => "1024", "cpu" => "1", "ip" => "101", "image" => "bento/ubuntu-22.04"},
  "node02" => {"memory" => "1024", "cpu" => "1", "ip" => "102", "image" => "bento/ubuntu-22.04"}
}

Vagrant.configure("2") do |config|

  machines.each do |name, conf|
    config.vm.define "#{name}" do |machine|
      machine.vm.box = "#{conf["image"]}"
      machine.vm.hostname = "#{name}"
      machine.vm.network "private_network", ip: "10.10.10.#{conf["ip"]}"
      machine.vm.provider "virtualbox" do |vb|
        vb.name = "#{name}"
        vb.memory = conf["memory"]
        vb.cpus = conf["cpu"]
      end
      machine.vm.provision "shell", path: "docker.sh"
      
      if "#{name}" == "master"
        machine.vm.provision "shell", path: "master.sh"
      else
        machine.vm.provision "shell", path: "worker.sh"
      end
    end
  end
end
```

Para usar este Vagrantfile, certifique-se de ter o Vagrant instalado em seu sistema. Em seguida, execute o comando `vagrant up` no diretório do projeto para criar e provisionar as máquinas virtuais com base na configuração.

## docker.sh

O script `docker.sh` é usado para provisionar as máquinas virtuais com o Docker. Ele instala o Docker e o Docker Compose.

```bash
#!/bin/bash
curl -fsSL https://get.docker.com | sudo bash
sudo curl -fsSL "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo usermod -aG docker vagrant
```

Este script pode ser executado pelo Vagrant durante o processo de provisionamento para instalar o Docker em cada máquina.

## master.sh

O script `master.sh

` é usado para provisionar a máquina `master` com a inicialização do Docker Swarm.

```bash
#!/bin/bash
sudo docker swarm init --advertise-addr=10.10.10.100
sudo docker swarm join-token worker | grep docker > /vagrant/worker.sh
```

Quando executado na máquina `master`, este script inicializa um Docker Swarm e gera um token para a adesão das máquinas worker. O script `worker.sh` gerado é salvo no diretório `/vagrant`, que pode ser acessado pela máquina host.

## worker.sh

O script `worker.sh` é usado para provisionar as máquinas worker e adicioná-las ao Docker Swarm.

```bash
docker swarm join --token SWMTKN-1-3pj8k0i4tn777jd093ha0yxhgh36hxuef5q5oyg1732rztnfy29ll-a94q0ipwgrjs4xikzyb4yb3n5 10.10.10.100:2377
```

Quando executado em uma máquina worker, este script adiciona a máquina ao Docker Swarm usando o token gerado pelo script `master.sh`.

## Uso

Para usar este projeto, siga estas etapas:

1. Instale o Vagrant em seu sistema.
2. Clone o repositório do projeto no GitHub.
3. Navegue até o diretório do projeto.
4. Execute o comando `vagrant up` para criar e provisionar as máquinas virtuais.
5. Depois que as máquinas estiverem ativas, você poderá acessá-las usando SSH ou qualquer outro método preferido.
6. A máquina `master` é inicializada como um gerenciador do Docker Swarm.
7. As máquinas `worker` são adicionadas ao Docker Swarm como workers.
8. Você pode usar comandos do Docker Swarm para gerenciar seu cluster.

Observe que esta documentação fornece uma visão geral geral, e você pode precisar personalizar as configurações ou scripts para atender aos seus requisitos específicos.

Para obter mais informações sobre o Vagrant, consulte a [documentação oficial](https://www.vagrantup.com/docs).

## Licença

Este projeto está licenciado sob a [Licença MIT](LICENSE).
