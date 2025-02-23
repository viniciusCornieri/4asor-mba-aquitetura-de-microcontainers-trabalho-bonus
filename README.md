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

### 2. Criação do template para o Mysql

O chart do mysql foi criado utilizando os seguintes arquivos:

#### 2.1 charts/mysql/templates/deployment.yaml

Aquivo que irá configurar o deployment do mysql no kubernetes, ele configura a imagem o password root e de usuário, e também o volume persistente para os pods do mysql.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-mysql
  labels:
    app: mysql
    release: {{ .Release.Name }}
spec:
  selector:
    matchLabels:
      app: mysql
      release: {{ .Release.Name }}
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
        release: {{ .Release.Name }}
    spec:
      containers:
      - name: mysql
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Release.Name }}-mysql-secret
              key: mysql-root-password
        - name: MYSQL_DATABASE
          value: {{ .Values.mysql.database }}
        - name: MYSQL_USER
          value: {{ .Values.mysql.user }}
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Release.Name }}-mysql-secret
              key: mysql-password
        ports:
        - name: mysql
          containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: {{ .Release.Name }}-mysql-pvc
```

#### 2.2 charts/mysql/templates/service.yaml

Aquivo que irá configurar o service do mysql no kubernetes.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-mysql
  labels:
    app: mysql
    release: {{ .Release.Name }}
spec:
  ports:
  - port: 3306
    targetPort: mysql
    protocol: TCP
    name: mysql
  selector:
    app: mysql
    release: {{ .Release.Name }}
  clusterIP: None
```

#### 2.3 charts/mysql/templates/secret.yaml

Aquivo que irá configurar o secret do mysql no kubernetes, configurando as senhas do root do mysql e do usuario

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-mysql-secret
  labels:
    app: mysql
    release: {{ .Release.Name }}
type: Opaque
data:
  mysql-root-password: {{ .Values.mysql.rootPassword | b64enc }}
  mysql-password: {{ .Values.mysql.password | b64enc }}
```

#### 2.4 charts/mysql/templates/pvc.yaml

Aquivo que irá configurar o volume persistente do mysql no kubernetes.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ .Release.Name }}-mysql-pvc
  labels:
    app: mysql
    release: {{ .Release.Name }}
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: {{ .Values.storage.size }}
  storageClassName: {{ .Values.storage.storageClass }}
```


#### 2.5 charts/mysql/templates/NOTES.txt

Aquivo que com a mensagem que irá aparecer ao instalar esse chart do mysql.

```
MySQL has been deployed successfully!

Connection details:
  Host: {{ .Release.Name }}-mysql
  Port: 3306
  Database: {{ .Values.mysql.database }}
  User: {{ .Values.mysql.user }}

To connect to your database:
  kubectl run -it --rm --image=mysql:8.0 --restart=Never mysql-client -- mysql -h {{ .Release.Name }}-mysql -u {{ .Values.mysql.user }} -p{{ .Values.mysql.password }}
```

#### 2.6 charts/mysql/templates/tests/test-connection.yaml

Aquivo que para o helm validar que o chart foi aplicado com sucesso.
[tests/test-connection.yaml](charts/mysql/templates/tests/test-connection.yaml)