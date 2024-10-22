
Estágio de Scan DAST com OWASP ZAP

Visão Geral
Este projeto implementa um estágio de DAST (Dynamic Application Security Testing) usando OWASP ZAP no pipeline CI/CD do GitLab. O estágio dast_scan realiza uma verificação automática em aplicações web para detectar vulnerabilidades, garantindo que falhas críticas sejam identificadas antes da implantação.

Funcionalidades Principais
OWASP ZAP: Ferramenta open-source para testes de segurança em aplicações web.
Integração CI/CD GitLab: Verificações contínuas de segurança durante o ciclo de vida de desenvolvimento.
Geração de Relatórios: Relatório de vulnerabilidades gerado em PDF e enviado por e-mail automaticamente.
Suporte a Proxy: Uso de proxy para baixar dependências e enviar e-mails.
Requisitos
Variáveis de Ambiente
Certifique-se de configurar as seguintes variáveis de ambiente no GitLab:

Proxy:

PROXY_DOCKER: URL do proxy HTTP para o Docker.
Configurações SMTP (para envio de e-mail):

HOST_SMTP_ZAP: Endereço do servidor SMTP.
USER_SMTP_ZAP: Nome de usuário para autenticação no SMTP.
PASSWORD_ZAP: Senha de autenticação no SMTP.
REMETENTE_EMAIL_ZAP: E-mail do destinatário para o envio do relatório.
Configurações do ZAP:

DAST_TARGET_URL: URL da aplicação que será alvo do scan.
