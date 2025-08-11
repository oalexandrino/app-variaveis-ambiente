# app-variaveis-ambiente

Essa aplicação tem como objetivo ser um exemplo para aulas relacionadas ao uso e configuração de variáveis de ambiente.

Variáveis de ambiente da aplicação:

APP_NAME => Nome da aplicação

APP_VERSION => Versão da aplicação

APP_AUTHOR => Nome do autor

Comando configmap:

kubectl create configmap app-variaveis-ambiente-config --from-literal=APP_NAME="Aplicação Exemplo via YAML" --from-literal=APP_VERSION="1.0.0" --from-literal=APP_AUTHOR="Olavo Alexandrino"
