# Cheatsheet — Computação EC2

## Famílias de Instâncias EC2

| Família | Otimização | Exemplos | Casos de uso |
|---|---|---|---|
| M | Geral (balanced) | m5, m6i, m6a, m7g | Web servers, apps, dev environments |
| C | CPU (compute) | c5, c6i, c6g, c7g | HPC, batch, ML inference, encoding |
| R | Memória | r5, r6i, r6g, x1e | Redis, Elasticsearch, SAP HANA, in-memory DB |
| I | I/O local (NVMe) | i3, i4i | NoSQL DB (Cassandra, MongoDB), OLTP alto IOPS |
| D | Dense storage | d3 | Data warehousing, HDFS |
| P | GPU (NVIDIA) | p3, p4 | ML training, HPC com CUDA |
| G | GPU geral | g4dn, g5 | ML inference, gaming, renderização |
| T | Burstable | t3, t3a, t4g | Ambientes dev, sites baixo tráfego, microserviços |
| Inf | AWS Inferentia | inf1, inf2 | ML inference custom chip |
| Hpc | HPC | hpc6a | HPC de alta performance com EFA |

---

## Opções de Compra EC2

| Opção | Desconto | Flexibilidade | Melhor para |
|---|---|---|---|
| On-Demand | Nenhum | Total | Workloads irregulares, teste |
| Standard RI (1/3 anos) | Até 72% | Baixa (família/região/OS fixos) | DB PostgreSQL production estável |
| Convertible RI (1/3 anos) | Até 54% | Média (pode trocar atributos) | Workloads que podem mudar tipo |
| Compute Savings Plans | Até 66% | Alta (qualquer família/região) | Workloads que movem entre tipos |
| EC2 Instance Savings Plans | Até 72% | Baixa (família+região fixos) | Similar à Standard RI |
| Spot | Até 90% | Alta (mas interrompível) | Batch, ML, render, jobs tolerantes |
| Dedicated Host | — | Baixa | Licença por socket/core (BYO-L) |
| Dedicated Instance | 10% sobre | Baixa | Compliance (hardware exclusivo) |
| Capacity Reservation | Nada | Total | Garantir capacidade sem contr. |

---

## EBS — Tipos de Volume

| Tipo | Categoria | IOPS | Throughput | Boot | Uso típico |
|---|---|---|---|---|---|
| gp3 | SSD geral | 3.000–16.000 | 125–1.000 MB/s | ✅ | Boot, apps gerais, padrão |
| gp2 | SSD geral | 100–16.000 (3/GB) | 128–250 MB/s | ✅ | Legado (prefira gp3) |
| io2 | SSD IOPS | Até 64.000 (io2 BE: 256K) | 4.000 MB/s | ✅ | OLTP crítico, Oracle, SQL |
| io1 | SSD IOPS | Até 64.000 | 1.000 MB/s | ✅ | Legado do io2 |
| st1 | HDD throughput | N/A | 40–250 MB/s por TB (máx 500) | ❌ | Logs, Kafka, data warehouse sequencial |
| sc1 | HDD cold | N/A | 12–80 MB/s por TB | ❌ | Dados raramente acessados, backup |

> **Nota:** io2 Block Express requer instâncias do tipo `io2 BE`. Multi-attach disponível apenas em io1/io2 em instâncias Nitro.

---

## Placement Groups — Comparativo

| Tipo | Topologia | Limite | Latência | Uso |
|---|---|---|---|---|
| Cluster | Mesmo rack, mesma AZ | Sem limite prático | Mais baixa (<1ms, 10Gbps+) | HPC, MPI, analytics de alta velocidade |
| Spread | Rack diferente por instância | 7 instâncias por AZ | Alta (racks distintos) | Alta disponibilidade, falha isolada |
| Partition | Grupo de instâncias por rack | 7 partições por AZ | Moderada | HDFS, Cassandra, Kafka (ciente de partição) |

---

## Ciclo de Vida EC2

```
Pending → Running ──┬─→ Stopping → Stopped → Terminated
                    │         (EBS preservado)
                    ├─→ [Hibernate] → Hibernating → Stopped (RAM salva no EBS)
                    │
                    └─→ Shutting-down → Terminated
                                        (EBS root deletado se deleteOnTermination=true)
```

| Estado | Cobra instância | Cobra EBS | RAM preservada |
|---|---|---|---|
| Running | Sim | Sim | Sim |
| Stopped | Não | Sim | Não |
| Hibernated | Não | Sim (RAM no root) | Sim |
| Terminated | Não | Não (geralmente) | Não |

---

## IMDSv1 vs IMDSv2

| Característica | IMDSv1 | IMDSv2 |
|---|---|---|
| Autenticação | Nenhuma (GET direto) | Token (PUT → GET com header) |
| Proteção SSRF | ❌ Vulnerável | ✅ Protegido |
| Configuração | `HttpTokens: optional` | `HttpTokens: required` |
| Token TTL | N/A | 1s a 21600s (6h) |
| Recomendação AWS | Deprecado | ✅ Use sempre |

Endpoints importantes:
```
http://169.254.169.254/latest/meta-data/instance-id
http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
http://169.254.169.254/latest/meta-data/placement/availability-zone
http://169.254.169.254/latest/user-data
```

---

## AMIs — Dicas de Prova

- Copiar AMI para outra região → especifique CMK de destino para encriptar
- Compartilhar AMI encriptada → deve compartilhar a CMK também  
- AMI baseada em snapshot → o snapshot fica na sua conta se você criou a AMI
- AMI pública da AWS Marketplace → pode usar mas não compartilhar o snapshot base
- HVM é o único tipo de virtualização em instâncias modernas (PV é legado)

---

## Spot — Estratégias do Spot Fleet

| Estratégia | Comportamento | Quando usar |
|---|---|---|
| `lowestPrice` | Escolhe pool de menor preço | Batch de custo crítico |
| `capacityOptimized` | Escolhe pool com maior capacidade disponível | Reduzir interrupções |
| `diversified` | Distribui por todos os pools definidos | Resiliência máxima |
| `priceCapacityOptimized` | Combina preço e capacidade | Recomendado padrão |

---

## Dicas de Prova (EC2)

- **gp3** é o padrão atual — mais barato e mais flexível que gp2
- **io2 Block Express** para requisitos > 64.000 IOPS
- **st1** para throughput sequencial (logs) — NÃO é opção de boot
- **Instance Store** perdido em stop/terminate, mas persistido em restart
- **Cluster PG** → mesma AZ, baixa latência. Se uma instância falha, todas ficam expostas
- **Spread PG** → máximo 7 por AZ — escolher para instâncias críticas isoladas
- IMDSv2 com `HttpTokens: required` bloqueia IMDSv1 — hardened security
- Spot Fleet `capacityOptimized` é preferível para reduzir interrupções vs `lowestPrice`
- **Dedicated Host** é necessário para BYO-L (license mobility) — não Dedicated Instance
- Hibernate: RAM salva no EBS → precisa de EBS encriptado + espaço ≥ RAM
