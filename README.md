# app-variaveis-ambiente

Projeto de exemplo para demonstrar o uso de variáveis de ambiente em aplicações Node.js rodando em Kubernetes, utilizando ConfigMap e Secret.

---

## Sumário

- [Descrição](#descrição)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Variáveis de Ambiente](#variáveis-de-ambiente)
- [Docker](#docker)
- [Kubernetes](#kubernetes)
  - [ConfigMap](#configmap)
  - [Secret](#secret)
  - [Deployment](#deployment)
  - [Service](#service)
- [Execução Local](#execução-local)
- [Execução no Kubernetes (k3d)](#execução-no-kubernetes-k3d)
- [Comandos Úteis](#comandos-úteis)
- [Autor](#autor)

---

## Descrição

A aplicação expõe variáveis de ambiente via Node.js, sendo ideal para testes e demonstrações em ambientes de orquestração como Kubernetes. O projeto mostra como utilizar ConfigMap e Secret para injetar variáveis de ambiente em containers.

---

## Estrutura do Projeto

```
app-variaveis-ambiente/
│
├── k8s/
│   ├── configmal.yaml      # ConfigMap com variáveis de ambiente
│   ├── secret.yaml         # Secret com variáveis sensíveis/base64
│   ├── deploymnet.yaml     # Deployment e Service do Kubernetes
│
├── src/
│   ├── Dockerfile          # Dockerfile da aplicação
│   ├── server.js           # Código principal da aplicação (Node.js)
│   ├── package.json        # Dependências do Node.js
│
└── README.md               # Este arquivo
```

---

## Variáveis de Ambiente

A aplicação utiliza as seguintes variáveis de ambiente:

- `APP_NAME` — Nome da aplicação
- `APP_VERSION` — Versão da aplicação
- `APP_AUTHOR` — Nome do autor

Essas variáveis podem ser fornecidas via ConfigMap ou Secret.

---

## Docker

### Build da imagem

```sh
docker build -t oalexandrino/app-variaveis-ambiente:v1 ./src
```

### Teste local

```sh
docker run -e APP_NAME="Minha App" -e APP_VERSION="1.0.0" -e APP_AUTHOR="Seu Nome" -p 3000:3000 oalexandrino/app-variaveis-ambiente:v1
```

---

## Kubernetes

### ConfigMap

Arquivo: `k8s/configmal.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-variaveis-ambiente-config
data:
  APP_NAME: "Aplicação Exemplo via YAML"
  APP_VERSION: "1.0.0"
  APP_AUTHOR: "Olavo Alexandrino"
```

Criação via comando:

```sh
kubectl apply -f k8s/configmal.yaml
```

### Secret

Arquivo: `k8s/secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-variaveis-ambiente-secret
type: Opaque
data:
  APP_NAME: QXBsaWNhw6fDo28gRXhlbXBsbyB2aWEgWU1MIC0gcGVsbyBzZWNyZXQ=
  APP_VERSION: MS4wLjY1OA==
  APP_AUTHOR: T2xhdm8gQWxlYW5kcmlubw==
```

Criação via comando:

```sh
kubectl apply -f k8s/secret.yaml
```

### Deployment

Arquivo: `k8s/deploymnet.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-variaveis-ambiente
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-variaveis-ambiente
  template:
    metadata:
      labels:
        app: app-variaveis-ambiente
    spec:
      containers:
        - name: app-variaveis-ambiente
          image: oalexandrino/app-variaveis-ambiente:v1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
          env:
            - name: APP_NAME
              valueFrom:
                secretKeyRef:
                  name: app-variaveis-ambiente-secret
                  key: APP_NAME
            - name: APP_VERSION
              valueFrom:
                secretKeyRef:
                  name: app-variaveis-ambiente-secret
                  key: APP_VERSION
            - name: APP_AUTHOR
              valueFrom:
                secretKeyRef:
                  name: app-variaveis-ambiente-secret
                  key: APP_AUTHOR
```

### Service

Arquivo: `k8s/deploymnet.yaml` (mesmo arquivo do deployment)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-variaveis-ambiente
spec:
  selector:
    app: app-variaveis-ambiente
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30000
```

---

## Execução Local

1. Instale as dependências:
   ```sh
   cd src
   npm install
   ```

2. Execute a aplicação:
   ```sh
   APP_NAME="Minha App" APP_VERSION="1.0.0" APP_AUTHOR="Seu Nome" node server.js
   ```

---

## Execução no Kubernetes (k3d)

1. **Carregue a imagem no cluster k3d:**
   ```sh
   k3d image import oalexandrino/app-variaveis-ambiente:v1 -c meu-cluster
   ```

2. **Aplique o ConfigMap e Secret:**
   ```sh
   kubectl apply -f k8s/configmal.yaml
   kubectl apply -f k8s/secret.yaml
   ```

3. **Aplique o Deployment e Service:**
   ```sh
   kubectl apply -f k8s/deploymnet.yaml
   ```

4. **Acesse a aplicação:**
   - Descubra o IP do node (ex: `localhost` para k3d).
   - Acesse via: [http://localhost:30000](http://localhost:30000)

---

## Comandos Úteis

- Ver pods:
  ```sh
  kubectl get pods
  ```
- Ver services:
  ```sh
  kubectl get svc
  ```
- Ver logs:
  ```sh
  kubectl logs <nome-do-pod>
  ```
- Excluir recursos:
  ```sh
  kubectl delete -f k8s/deploymnet.yaml
  kubectl delete -f k8s/configmal.yaml
  kubectl delete -f k8s/secret.yaml
  ```

---

## Autor