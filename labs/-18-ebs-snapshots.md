#  Laboratório AWS - Amazon EBS (Elastic Block Store)

![AWS](https://img.shields.io/badge/AWS-EBS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![License](https://img.shields.io/badge/License-Educational-blue?style=for-the-badge)

##  Sobre o Laboratório

Este repositório contém a documentação completa de um laboratório prático focado no gerenciamento de volumes **Amazon Elastic Block Store (EBS)** integrados com instâncias **Amazon EC2**.

###  Informações Gerais

- **Duração**: Aproximadamente 45 minutos
- **Nível**: Intermediário
- **Plataforma**: Amazon Web Services (AWS)
- **Serviços**: Amazon EBS, Amazon EC2

---

##  Objetivos de Aprendizado

Ao concluir este laboratório, você será capaz de:

-  Criar volumes do Amazon Elastic Block Store (EBS)
-  Anexar volumes EBS a instâncias Amazon EC2
-  Montar volumes EBS no sistema operacional da instância
-  Criar snapshots (cópias de segurança) de volumes EBS
-  Restaurar volumes EBS a partir de snapshots existentes

---

##  Sobre o Amazon EBS

O **Amazon Elastic Block Store (EBS)** é um serviço de armazenamento em bloco de alto desempenho projetado para uso com o Amazon EC2.

### Principais Características

| Recurso | Descrição |
|---------|-----------|
|  **Armazenamento Persistente** | Os dados permanecem mesmo quando a instância EC2 é desligada |
|  **Alta Disponibilidade** | Replicação automática dentro da zona de disponibilidade |
|  **Snapshots** | Backups incrementais armazenados no Amazon S3 |
|  **Flexibilidade** | Volumes podem ser anexados, desanexados e reanexados a instâncias |

---

##  Procedimentos do Laboratório

### Tarefa 1: Criar um Volume EBS

Criação de um novo volume do Amazon EBS que poderá ser anexado a uma instância EC2.

**Passos:**

1. Acesse o console da AWS e navegue até o serviço EC2
2. No painel de navegação, selecione **Volumes** em *Elastic Block Store*
3. Clique no botão **Create Volume**
4. Configure o volume com as especificações desejadas:
   - Tipo de volume (gp3, gp2, io1, etc.)
   - Tamanho (em GB)
   - Zona de disponibilidade (deve ser a mesma da instância EC2)
5. Confirme a criação do volume

---

### Tarefa 2: Anexar o Volume a uma Instância EC2

Anexar o volume criado a uma instância EC2 em execução na mesma zona de disponibilidade.

**Passos:**

1. Selecione o volume criado na lista de volumes
2. Clique em **Actions** > **Attach Volume**
3. Selecione a instância EC2 de destino
4. Anote o nome do dispositivo (ex: `/dev/sdf` ou `/dev/xvdf`)
5. Confirme o anexo do volume

---

### Tarefa 3: Montar o Volume no Sistema Operacional

Após anexar o volume à instância, é necessário montá-lo no sistema operacional.

**Passos:**

1. Conecte-se à instância EC2 via SSH:
   ```bash
   ssh -i sua-chave.pem ec2-user@seu-ip-publico
   ```

2. Verifique os dispositivos de armazenamento disponíveis:
   ```bash
   lsblk
   ```

3. Formate o volume (apenas na primeira vez):
   ```bash
   sudo mkfs -t ext4 /dev/xvdf
   ```

4. Crie um ponto de montagem:
   ```bash
   sudo mkdir /mnt/ebs-volume
   ```

5. Monte o volume:
   ```bash
   sudo mount /dev/xvdf /mnt/ebs-volume
   ```

6. Verifique a montagem:
   ```bash
   df -h
   ```

7. (Opcional) Configure montagem automática no boot editando `/etc/fstab`:
   ```bash
   sudo nano /etc/fstab
   # Adicione a linha:
   /dev/xvdf   /mnt/ebs-volume   ext4   defaults,nofail   0   2
   ```

---

### Tarefa 4: Criar um Snapshot do Volume

Os snapshots são backups incrementais que permitem restaurar volumes ou criar novos volumes baseados em um ponto no tempo específico.

**Passos:**

1. No console EC2, navegue até **Snapshots** em *Elastic Block Store*
2. Clique em **Create Snapshot**
3. Selecione o tipo de recurso: **Volume**
4. Selecione o volume do qual deseja criar o snapshot
5. Adicione uma descrição identificadora (ex: "Backup do volume EBS - 10/02/2026")
6. (Opcional) Adicione tags para organização
7. Confirme a criação do snapshot
8. Aguarde a conclusão do processo (status: completed)

**Características dos Snapshots:**
- São incrementais (apenas os blocos alterados são salvos)
- Armazenados no Amazon S3 (gerenciado automaticamente)
- Podem ser copiados entre regiões
- Servem como base para criar novos volumes

---

### Tarefa 5: Restaurar Volume a Partir de Snapshot

Criar novos volumes a partir de snapshots existentes, permitindo recuperação de dados ou replicação de configurações.

**Passos:**

1. Na lista de snapshots, selecione o snapshot criado anteriormente
2. Clique em **Actions** > **Create Volume from Snapshot**
3. Configure as especificações do novo volume:
   - **Tipo de volume**: escolha conforme necessidade de performance
   - **Tamanho**: pode ser igual ou maior que o snapshot original
   - **Zona de disponibilidade**: selecione a zona desejada
   - **Criptografia**: opcionalmente habilite
4. Adicione tags se necessário
5. Confirme a criação do volume baseado no snapshot
6. Aguarde a criação (status: available)
7. Anexe e monte o novo volume seguindo as Tarefas 2 e 3

---

##  Resultados Esperados

Ao concluir este laboratório com sucesso, você terá:

-  Criado e gerenciado volumes EBS no ambiente AWS
-  Anexado volumes a instâncias EC2 em execução
-  Configurado e montado volumes no sistema operacional Linux
-  Implementado estratégia de backup usando snapshots
-  Restaurado volumes a partir de backups (snapshots)

---

##  Dicas e Boas Práticas

### Segurança
-  Utilize criptografia em volumes que contenham dados sensíveis
-  Gerencie chaves de criptografia via AWS KMS
-  Configure políticas IAM apropriadas para acesso aos volumes

### Performance
-  Escolha o tipo de volume adequado para sua carga de trabalho:
  - **gp3/gp2**: Uso geral (ótimo custo-benefício)
  - **io2/io1**: Alta performance e IOPS provisionados
  - **st1**: Throughput otimizado (big data)
  - **sc1**: Acesso infrequente (arquivamento)

### Custos
-  Delete volumes não utilizados
-  Monitore o uso e otimize o tamanho dos volumes
-  Estabeleça políticas de retenção para snapshots antigos
-  Use Amazon Data Lifecycle Manager para automatizar snapshots

### Backup
-  Estabeleça uma rotina regular de snapshots
-  Considere copiar snapshots críticos para outras regiões
-  Teste a restauração periodicamente
-  Use tags consistentes para organizar snapshots

---

##  Comandos Úteis

### Verificar Volumes Anexados
```bash
lsblk
df -h
sudo fdisk -l
```

### Verificar Sistema de Arquivos
```bash
sudo file -s /dev/xvdf
```

### Desmontar Volume
```bash
sudo umount /mnt/ebs-volume
```

### Verificar UUID de um Volume
```bash
sudo blkid
```

### Expandir Sistema de Arquivos (após aumentar tamanho do volume)
```bash
# Para ext4
sudo resize2fs /dev/xvdf

# Para xfs
sudo xfs_growfs -d /mnt/ebs-volume
```

---

##  Recursos Adicionais

### Documentação Oficial AWS
- [Amazon EBS Documentation](https://docs.aws.amazon.com/ebs/)
- [Amazon EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [EBS Volume Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)
- [EBS Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html)

### Tutoriais e Guias
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [EBS Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-best-practices.html)

---

##  Avisos Importantes

> **Atenção**: Lembre-se de encerrar recursos AWS após concluir o laboratório para evitar cobranças desnecessárias.

### Limpeza de Recursos

1. Desmonte o volume da instância EC2
2. Desanexe o volume da instância
3. Delete volumes não utilizados
4. Delete snapshots desnecessários
5. Encerre instâncias EC2 se não forem mais necessárias

---

##  Notas

- Amazon Web Services, AWS, Amazon EBS e Amazon EC2 são marcas registradas da Amazon.com, Inc. ou suas afiliadas
- Este material é destinado apenas para fins educacionais e de treinamento
- Os procedimentos e práticas descritos são baseados nas recomendações oficiais da AWS

---


##  Licença

Este projeto é disponibilizado para fins educacionais. O conteúdo é baseado em materiais e práticas recomendadas pela AWS.

---

##  Autor

Documentação elaborada e organizada para facilitar o aprendizado e servir como referência para estudos de AWS.

**Kaylane Kimberly**
**Data de criação**: Fevereiro de 2026

---

##  Agradecimentos

Agradecimentos à Amazon Web Services por fornecer excelente documentação e recursos educacionais que tornaram possível a criação deste laboratório.

---

<div align="center">

**Feito com  para a comunidade de aprendizado AWS**

[⬆ Voltar ao topo](#-laboratório-aws---amazon-ebs-elastic-block-store)

</div>
