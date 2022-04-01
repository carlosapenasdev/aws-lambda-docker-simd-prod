# AWS Lambda Docker Python Pillow-SIMD / Teste Deploy Fora

Registrar conteúdo de aprendizagem sobre AWS Lambda, utilizando Docker para um runtime python instalando
a lib Pillow-SIMD.

A lib Pillow-SIMD informa ser mais veloz que a lib original (Pillow) por utilizar
libs compiladas para o SO específico.

O uso do docker para gerenciar a função lambda, foi escolhido por facilitar o processo de instalação e manutenção
das dependencias do projeto.

O objetivo final é comparar a performance com diferentes libs de resize e imagens, tendo um mesmo ambiente base para as funções lambda.

## 0. Requisitos
- AWS Cli ([link sobre instalação](https://docs.aws.amazon.com/pt_br/cli/latest/userguide/getting-started-install.html))
```bash
cd /tmp/ && mkdir aws && cd aws \
&& curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
&& unzip awscliv2.zip \
&& sudo ./aws/install \
&& aws --version
```
- Docker ([link sobre instalação](https://docs.docker.com/get-docker/))
```bash
#instalação no ubuntu 20.04
sudo apt update \
&& sudo apt install -y apt-transport-https ca-certificates curl software-properties-common \
&& curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - \
&& sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" \
&& sudo apt update \
&& sudo apt -y install docker-ce \
&& sudo usermod -aG docker ${USER} \
&& su - ${USER}
```
- Criar user/group do IAM para o AWS CLI e SAM ([link](https://docs.aws.amazon.com/pt_br/IAM/latest/UserGuide/getting-started_create-admin-group.html))
    - Caso já possua user criado no IAM, desconsidere o passo acima
    - Precisa ter em mãos AWS IAM Access key id  e AWS IAM Access key Secret
- Configurar o AWS credenciais. ([link](https://docs.aws.amazon.com/pt_br/serverless-application-model/latest/developerguide/serverless-getting-started-set-up-credentials.html))
```
$ aws configure
#AWS Access Key ID [None]: your_access_key_id
#AWS Secret Access Key [None]: your_secret_access_key
#Default region name [None]: your_default_region
#>A função lambda, S3 e ECR precisão estar na mesma região, importante configurar esse passo: EX: us-east-1
#Default output format [None]: your_default_output
```
- AWS Sam ([link sobre instalação](https://docs.aws.amazon.com/pt_br/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html))
```
curl -L https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip > awssam.zip \
&& unzip awssam.zip -d sam-installation \
&& sudo ./sam-installation/install \
&& sam --version
```
-  clonar este repositorio localmente:
```bash
git clone https://github.com/carlossantoswd/aws-lambda-docker-simd-prod.git lambda_prod \
&& cd lambda_prod
```
## 1. Configurar Função
Nesta etapa vamos configurar o projeto personalizando com as informações existentes. Precisamos
alterar o bucket destino das imagens, entre outros.

```
BUCKET_FINAL=nome_do_bucket_para_imgs_redimensionadas
find . -type f -exec sed -i "s/bucket-store-blossv1/${BUCKET_FINAL}/g" {} +
```
## 2. Buildar e Implantar
```bash
sam build && sam deploy --stack-name docker-reven  --resolve-s3 --resolve-image-repos --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_IAM
```

## 3. Criar trigger entre função lambda e bucket
Como utilizaremos um bucket pré existente, o gatilho entre ele e o Lambda precisa ser criado manualmente (O SAM exige que o bucket esteja no arquivo de definição yaml para criar o evento, como já temos o bucket criado, ele teve de ser removido do template)

