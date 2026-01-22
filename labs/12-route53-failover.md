#  Route 53 - Health Check e Roteamento de Failover

[![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=flat-square&logo=amazon-aws)](https://aws.amazon.com/)
[![Route 53](https://img.shields.io/badge/Service-Route%2053-blueviolet?style=flat-square)](https://aws.amazon.com/route53/)
[![License](https://img.shields.io/badge/License-All%20Rights%20Reserved-red?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)]()
[![Duration](https://img.shields.io/badge/Duration-45min-blue?style=flat-square)]()

Laboratório prático sobre configuração de health checks e roteamento de failover utilizando Amazon Route 53.

---

##  Sobre o Laboratório

Este laboratório documenta a configuração de monitoramento de integridade e implementação de roteamento de failover no Amazon Route 53, incluindo notificações por e-mail quando endpoints HTTP apresentam problemas.

**Duração:** 45 minutos  
**Nível de dificuldade:** Intermediário  
**Desafios encontrados:** Durante a execução deste laboratório, foram enfrentadas dificuldades relacionadas à configuração correta das health checks e à sincronização com as notificações SNS.

##  Objetivos

Ao concluir este laboratório, você será capaz de:

- Configurar uma health check do Route 53 para monitorar endpoints HTTP
- Implementar notificações por e-mail quando a integridade de um endpoint deixar de ser íntegro
- Configurar o roteamento de failover no Route 53
- Compreender o funcionamento do DNS failover em cenários de alta disponibilidade


##  Serviços AWS Utilizados

- **Route 53** - Serviço de DNS e gerenciamento de tráfego
- **SNS** - Simple Notification Service (para alertas por e-mail)
- **CloudWatch** - Monitoramento de health checks
- **EC2** - Instâncias para endpoints de teste (implícito)


##  Pré-requisitos

- Conta AWS ativa
- Conhecimentos básicos de DNS
- Familiaridade com o console AWS
- Endereço de e-mail válido para receber notificações
- (Opcional) Domínio registrado no Route 53


##  Resumo da Implementação

Este laboratório envolve as seguintes etapas principais:

1. **Configuração de Health Check**
   - Criação de health check HTTP/HTTPS
   - Definição de parâmetros de monitoramento
   - Configuração de intervalos e thresholds

2. **Integração com SNS**
   - Criação de tópico SNS
   - Configuração de subscrição por e-mail
   - Vinculação do health check ao alarme

3. **Roteamento de Failover**
   - Configuração de registro primário
   - Configuração de registro secundário
   - Teste de failover automático


##  Desafios Encontrados

Durante a execução deste laboratório, foram identificados os seguintes desafios:

- **Sincronização de notificações:** A configuração correta do tópico SNS e a confirmação da subscrição por e-mail exigiram atenção especial
- **Propagação de DNS:** O tempo de propagação dos registros DNS impactou os testes de failover
- **Configuração de thresholds:** Ajustar os valores ideais para detecção de falhas sem gerar falsos positivos

Estas dificuldades foram superadas através de revisão da documentação oficial e testes iterativos.

##  Conclusão

Ao finalizar este laboratório, foram alcançados os seguintes resultados:

-  Health check do Route 53 configurado com sucesso para monitorar endpoints HTTP
-  Sistema de notificações por e-mail implementado e funcional
-  Roteamento de failover operacional, garantindo alta disponibilidade
-  Compreensão prática dos mecanismos de DNS failover da AWS

**Tempo total de execução:** 45 minutos

**Principais aprendizados:**
- Importância do monitoramento proativo de infraestrutura
- Implementação de alta disponibilidade com Route 53
- Integração entre serviços AWS (Route 53 + SNS + CloudWatch)
- Estratégias de disaster recovery baseadas em DNS

---

## ⚠️ Avisos Importantes

- Este laboratório é destinado apenas para fins educacionais e de documentação
- Sempre revise os custos antes de implementar recursos na AWS
- Lembre-se de excluir recursos após os testes para evitar cobranças
- Nunca compartilhe credenciais ou informações sensíveis
- Health checks do Route 53 podem gerar custos adicionais

---

##  Licença

© 2026 Amazon Web Services, Inc. ou suas afiliadas. Todos os direitos reservados.

Este conteúdo é fornecido como material educacional baseado em laboratórios práticos da AWS.

---

##  Sobre

Documentação criada durante processo de aprendizado em Cloud Computing com Amazon Web Services.

**Data de conclusão:** Janeiro de 2026
