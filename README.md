# Trabalho bônus Arquitetura de Microcontainers

Esta página tem como objetivo fornecer os arquivos no formato de [helm-chart](https://helm.sh/) para a infraestrutura de wordpress com banco de dados mysql gerenciado em Kubernetes, é referente a um trabalho bônus do MBA de Arquitetura de Soluções da FIAP da disciplina de Arquitetura de Microcontainers. Esta página e fornecimento do helm-chart pelo github pages foi feito utilizando como base o projeto especificado em [Provision a free personal Helm chart repo using GitHub](https://medium.com/@gerkElznik/provision-a-free-personal-helm-chart-repo-using-github-583b668d9ba4)

## Pré-requisitos

* Helm versão 3 instalado para utilização de charts. Siga as instruções da página [Installing Helm](https://helm.sh/docs/intro/install/)

Um exemplo para sistemas Linux pode ser visto aqui:

```bash
# Baixar o script de instalação do Helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

# Dar permissão de execução
chmod 700 get_helm.sh

# Executar o script de instalação
./get_helm.sh

# Verificar a instalação
helm version
```

## Uso

Com o Helm instalado e configurado corretamente, adicionar o repositório:

```bash
helm repo add 4asor-vmc https://viniciuscornieri.github.io/4asor-mba-aquitetura-de-microcontainers-trabalho-bonus/
```

Se o repositório já tiver sido instalado, executar `helm repo update` para atualizar e pegar as ultimas versões dos pacotes.

Após isso é possível executar `helm search repo 4asor-vmc` para visualizar os charts disponíveis.

## Criação dos charts

Esta seção é a explicação de como foi gerado os charts para esse repositório.


### 1. Criação do diretório padrão do chart do mysql e wordpress
Com o helm já instalado e estando dentro da diretório deste repositório git, foi executado os comandos abaixo para criar a estrutura inicial para os charts do mysql e wordpress:

```bash
#cria diretório charts
mkdir charts

# Criar os charts
helm create charts/mysql
helm create charts/wordpress

# Limpar os templates padrão selecionando yes para apagar os arquivos
rm -rf charts/mysql/templates/*
rm -rf charts/wordpress/templates/*
```

### 2. Estrutura desse projeto

Abaixo contém os arquivos de charts e utilizados pelo github workflows 

```
wordpress-helm/
├── .github/
│   └── workflows/
│       ├── [release.yaml](.github/workflows/release.yaml)
│       └── [test.yaml](.github/workflows/test.yaml)
├── charts/
│   ├── mysql/
│   │   ├── [Chart.yaml](charts/mysql/Chart.yaml)
│   │   ├── [values.yaml](charts/mysql/values.yaml)
│   │   ├── templates/
│   │   │   ├── [NOTES.txt](charts/mysql/templates/NOTES.txt)
│   │   │   ├── [deployment.yaml](charts/mysql/templates/deployment.yaml)
│   │   │   ├── [pvc.yaml](charts/mysql/templates/pvc.yaml)
│   │   │   ├── [secret.yaml](charts/mysql/templates/secret.yaml)
│   │   │   └── [service.yaml](charts/mysql/templates/service.yaml)
│   │   ├── tests/
│   │   │   └── [test-connection.yaml](charts/mysql/templates/tests/test-connection.yaml)
│   └── wordpress/
│       ├── [Chart.yaml](charts/wordpress/Chart.yaml)
│       ├── [values.yaml](charts/wordpress/values.yaml)
│       ├── templates/
│       │   ├── [NOTES.txt](charts/wordpress/templates/NOTES.txt)
│       │   ├── [_helpers.tpl](charts/wordpress/templates/_helpers.tpl)
│       │   ├── [deployment.yaml](charts/wordpress/templates/deployment.yaml)
│       │   ├── [ingress.yaml](charts/wordpress/templates/ingress.yaml)
│       │   ├── [pvc.yaml](charts/wordpress/templates/pvc.yaml)
│       │   └── [service.yaml](charts/wordpress/templates/service.yaml)
│       └── tests/
│           └── [test-connection.yaml](charts/wordpress/templates/tests/test-connection.yaml)
└── [README.md](README.md)
```