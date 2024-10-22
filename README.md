# Estágio de Scan DAST com OWASP ZAP

## Visão Geral

Este projeto implementa um estágio de **DAST (Dynamic Application Security Testing)** usando **OWASP ZAP** no pipeline CI/CD do GitLab. O estágio `dast_scan` realiza uma verificação automática em aplicações web para detectar vulnerabilidades, garantindo que falhas críticas sejam identificadas antes da implantação.

## Funcionalidades Principais

- **OWASP ZAP**: Ferramenta open-source para testes de segurança em aplicações web.
- **Integração CI/CD GitLab**: Verificações contínuas de segurança durante o ciclo de vida de desenvolvimento.
- **Geração de Relatórios**: Relatório de vulnerabilidades gerado em PDF e enviado por e-mail automaticamente.
- **Suporte a Proxy**: Uso de proxy para baixar dependências e enviar e-mails.

## Requisitos

### Variáveis de Ambiente

Certifique-se de configurar as seguintes variáveis de ambiente no GitLab:

#### Proxy:

- `PROXY_DOCKER`: URL do proxy HTTP para o Docker.

#### Configurações SMTP (para envio de e-mail):

- `HOST_SMTP_ZAP`: Endereço do servidor SMTP.
- `USER_SMTP_ZAP`: Nome de usuário para autenticação no SMTP.
- `PASSWORD_ZAP`: Senha de autenticação no SMTP.
- `REMETENTE_EMAIL_ZAP`: E-mail do destinatário para o envio do relatório.

#### Configurações do ZAP:

- `DAST_TARGET_URL`: URL da aplicação que será alvo do scan.

## Configuração do Pipeline

Aqui está o arquivo `.gitlab-ci.yml` para configurar o estágio `dast_scan`:

```yaml
dast_scan:
  stage: dast_scan
  image: maven:3.8.5-openjdk-11-slim
  before_script: [] 
  script:
    - export http_proxy=$PROXY_DOCKER
    - export https_proxy=$PROXY_DOCKER
    - apt-get update
    - apt-get -y install wget wkhtmltopdf msmtp
    - wget https://github.com/zaproxy/zaproxy/releases/download/v2.15.0/ZAP_2.15.0_Linux.tar.gz
    - mkdir zap
    - tar -xvf ZAP_2.15.0_Linux.tar.gz
    - cd ZAP_2.15.0
    - ./zap.sh -cmd -quickurl $DAST_TARGET_URL -quickprogress -quickout ../zap_report.html -config database.reset=true
    - cd ..
    - wkhtmltopdf zap_report.html zap_report.pdf

    # Configurações para envio de e-mail
    - |
      echo "account gitlab" > /etc/msmtprc
      echo "host $HOST_SMTP_ZAP" >> /etc/msmtprc
      echo "port 587" >> /etc/msmtprc
      echo "auth on" >> /etc/msmtprc
      echo "user $USER_SMTP_ZAP" >> /etc/msmtprc
      echo "password $PASSWORD_ZAP" >> /etc/msmtprc
      echo "tls on" >> /etc/msmtprc
      echo "tls_starttls on" >> /etc/msmtprc
      echo "tls_certcheck off" >> /etc/msmtprc
      echo "from $USER_SMTP_ZAP" >> /etc/msmtprc
      echo "logfile /var/log/msmtp.log" >> /etc/msmtprc
      echo "account default : gitlab" >> /etc/msmtprc
    - chmod 600 /etc/msmtprc

    # Enviar e-mail com o relatório
    - |
      {
        current_time=$(date "+%Y-%m-%d")
        echo "Subject: Relatório DAST gerado em $current_time para o site $DAST_TARGET_URL."
        echo "To: $REMETENTE_EMAIL_ZAP"
        echo "Content-Type: multipart/mixed; boundary=\"boundary\""
        echo ""
        echo "--boundary"
        echo "Content-Type: text/plain; charset=\"utf-8\""
        echo ""
        echo "O relatório do DAST está em anexo. O site que foi escaneado é: $DAST_TARGET_URL."
        echo ""
        echo "--boundary"
        echo "Content-Type: application/pdf; name=\"zap_report.pdf\""
        echo "Content-Transfer-Encoding: base64"
        echo "Content-Disposition: attachment; filename=\"zap_report.pdf\""
        echo ""
        base64 zap_report.pdf
        echo "--boundary--"
      } | msmtp -a gitlab -t

  artifacts:
    paths:
      - zap_report.pdf
    expire_in: 1 day
  tags:
    - docker
  when: on_success
