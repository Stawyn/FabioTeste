# Análise de Desempenho do Sistema de Arquivos EXT4

## Resumo

Este documento apresenta uma análise detalhada do desempenho do sistema de arquivos EXT4 através de testes práticos realizados em ambiente virtualizado. Os testes foram conduzidos utilizando ferramentas padronizadas para avaliação de I/O em sistemas Linux, com foco na medição de throughput de leitura e escrita sequencial.

## 1. Introdução

O sistema de arquivos EXT4 (Fourth Extended File System) é amplamente utilizado em distribuições Linux devido à sua estabilidade e desempenho. Este estudo visa quantificar as características de performance através de benchmarks controlados.

### 1.1 Objetivos

- Avaliar o desempenho de escrita e leitura sequencial do EXT4
- Documentar metodologia de teste reproduzível
- Analisar resultados obtidos com ferramentas `dd` e `fio`

## 2. Metodologia

### 2.1 Ambiente de Teste

**Sistema Operacional:** Ubuntu 24.04.3 Desktop AMD64

**Virtualização:** VirtualBox

**Especificações do Hardware Host:**
```
Processador: AMD Ryzen 9 5900XT 16-Core Processor
Arquitetura: x86_64
CPUs Virtuais: 24 cores
Memória RAM: 15.986 MB (~16GB)
Swap: Desabilitado
```

**Configuração da VM:**
```bash
user@user:~$ lscpu
Architecture:                x86_64
CPU(s):                      24
Vendor ID:                   AuthenticAMD
Model name:                  AMD Ryzen 9 5900XT 16-Core Processor
Virtualization type:         full
```

### 2.2 Preparação do Ambiente de Teste

#### 2.2.1 Criação do Sistema de Arquivos

1. **Criação do diretório de montagem:**
```bash
sudo mkdir -p /mnt/ext4_test
```

2. **Criação da imagem de disco (1GB):**
```bash
sudo dd if=/dev/zero of=/tmp/ext4.img bs=1M count=1024
```

3. **Formatação com EXT4:**
```bash
sudo mkfs.ext4 /tmp/ext4.img
```

4. **Montagem do sistema:**
```bash
sudo mount -o loop /tmp/ext4.img /mnt/ext4_test
```

#### 2.2.2 Verificação da Montagem

```bash
user@user:~$ df -h | grep mnt
/dev/loop9      974M  201M  707M  23% /mnt/ext4_test
```

**Análise:** O sistema de arquivos foi montado com sucesso, apresentando:
- Tamanho total: 974MB
- Espaço utilizado: 201MB (metadados do sistema)
- Espaço disponível: 707MB
- Utilização: 23%

### 2.3 Ferramentas de Teste

#### 2.3.1 DD (Data Dump)
Ferramenta básica para operações de I/O sequencial em blocos.

#### 2.3.2 FIO (Flexible I/O Tester)
Ferramenta avançada para benchmark de I/O com múltiplas configurações.

## 3. Execução dos Testes

### 3.1 Teste de Escrita com DD

**Comando executado:**
```bash
sudo dd if=/dev/zero of=/mnt/ext4_test/testfile bs=1M count=100 oflag=direct
```

**Resultado:**
```
100+0 records in
100+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 0.0853317 s, 1.2 GB/s
```

**Análise:**
- Volume de dados: 104.857.600 bytes (105 MB)
- Tempo de execução: 0.0853317 segundos
- **Throughput de escrita: 1,2 GB/s**

### 3.2 Teste de Leitura com DD

**Comando executado:**
```bash
sudo dd if=/mnt/ext4_test/testfile of=/dev/null bs=1M iflag=direct
```

**Resultado:**
```
100+0 records in
100+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 0.0233496 s, 4.5 GB/s
```

**Análise:**
- Volume de dados: 104.857.600 bytes (105 MB)
- Tempo de execução: 0.0233496 segundos
- **Throughput de leitura: 4,5 GB/s**

### 3.3 Teste Avançado com FIO

**Comando executado:**
```bash
sudo fio --name=write_test --directory=/mnt/ext4_test --rw=write --bs=1M --size=100M --numjobs=1 --runtime=30 --group_reporting
```

**Resultados detalhados:**
```
write_test: (groupid=0, jobs=1): err= 0: pid=6217: Thu Sep 18 07:15:13 2025
  write: IOPS=1162, BW=1163MiB/s (1219MB/s)(100MiB/86msec)
```

**Métricas principais:**
- **IOPS (Input/Output Operations Per Second): 1.162**
- **Bandwidth: 1.163 MiB/s (1.219 MB/s)**
- **Tempo de execução: 86ms**

**Latência:**
```
clat (usec): min=553, max=842, avg=605.04, stdev=51.81
lat (usec): min=558, max=847, avg=616.56, stdev=58.49
```

- Latência média: 605,04 μs
- Latência mínima: 553 μs
- Latência máxima: 842 μs
- Desvio padrão: 51,81 μs

## 4. Análise dos Resultados

### 4.1 Comparação de Performance

| Métrica | DD Escrita | DD Leitura | FIO Escrita |
|---------|------------|------------|-------------|
| Throughput | 1,2 GB/s | 4,5 GB/s | 1,219 GB/s |
| Tempo (100MB) | 85,33ms | 23,35ms | 86ms |
| IOPS | - | - | 1.162 |

### 4.2 Análise Técnica

#### 4.2.1 Performance de Leitura vs Escrita
- **Leitura 3,75x mais rápida** que escrita (4,5 GB/s vs 1,2 GB/s)
- Diferença típica em sistemas de arquivos journaling
- Cache do sistema operacional pode influenciar leituras

#### 4.2.2 Consistência dos Resultados
- Resultados DD e FIO para escrita são consistentes (1,2 GB/s vs 1,219 GB/s)
- Validação cruzada confirma precisão das medições

#### 4.2.3 Latência
- Latência baixa e consistente (605μs média)
- Baixo desvio padrão (51,81μs) indica performance estável

### 4.3 Características do EXT4 Observadas

**Vantagens identificadas:**
- Alto throughput para operações sequenciais
- Baixa latência de acesso
- Performance estável e previsível
- Eficiência em ambiente virtualizado

**Considerações:**
- Performance limitada pelo overhead de virtualização
- Journaling impacta performance de escrita
- Direct I/O elimina cache effects para medições precisas

## 5. Fatores Limitantes

### 5.1 Virtualização
- VirtualBox introduz overhead de I/O
- Performance nativa seria superior
- Impacto na latência e throughput

### 5.2 Configuração do Teste
- Loop device pode introduzir overhead adicional
- Tamanho de bloco (1MB) otimizado para throughput sequencial
- Direct I/O bypassa page cache

## 6. Conclusões

O sistema de arquivos EXT4 demonstrou excelente performance em ambiente de teste:

1. **Throughput elevado:** 1,2 GB/s escrita, 4,5 GB/s leitura
2. **Latência baixa:** ~605μs média
3. **Consistência:** Resultados reproduzíveis entre ferramentas
4. **Estabilidade:** Baixo desvio padrão nas medições

### 6.1 Recomendações

- EXT4 adequado para workloads com I/O sequencial intensivo
- Performance de leitura superior à escrita (padrão esperado)
- Adequado para aplicações que demandam baixa latência

### 6.2 Trabalhos Futuros

- Comparação com outros sistemas (XFS, Btrfs, ZFS)
- Testes com I/O randômico
- Análise em hardware nativo (sem virtualização)
- Avaliação com diferentes tamanhos de bloco

## 7. Referências

- [Documentação oficial EXT4](https://www.kernel.org/doc/Documentation/filesystems/ext4.txt)
- [Manual FIO](https://fio.readthedocs.io/)
- [Linux I/O Performance](https://www.brendangregg.com/linuxperf.html)

## 8. Reprodutibilidade

### 8.1 Script de Teste Completo

```bash
#!/bin/bash
# Script para reprodução dos testes

echo "=== Preparação do ambiente ==="
sudo mkdir -p /mnt/ext4_test
sudo dd if=/dev/zero of=/tmp/ext4.img bs=1M count=1024
sudo mkfs.ext4 /tmp/ext4.img
sudo mount -o loop /tmp/ext4.img /mnt/ext4_test

echo "=== Verificação da montagem ==="
df -h | grep mnt

echo "=== Teste de escrita DD ==="
sudo dd if=/dev/zero of=/mnt/ext4_test/testfile bs=1M count=100 oflag=direct

echo "=== Teste de leitura DD ==="
sudo dd if=/mnt/ext4_test/testfile of=/dev/null bs=1M iflag=direct

echo "=== Teste FIO ==="
sudo fio --name=write_test --directory=/mnt/ext4_test --rw=write --bs=1M --size=100M --numjobs=1 --runtime=30 --group_reporting

echo "=== Limpeza ==="
sudo umount /mnt/ext4_test
sudo rm /tmp/ext4.img
```

### 8.2 Ambiente de Teste

Para reproduzir os resultados, utilize:
- Ubuntu 24.04.3 ou superior
- Mínimo 2GB RAM disponível
- Ferramentas: `dd`, `fio`, `mkfs.ext4`

---

**Autores:** Equipe de Análise de Sistemas de Arquivos  
**Data:** 18 de Setembro de 2025  
**Versão:** 1.0
````I'm waiting for your response to continue with creating the comprehensive scientific documentation for your EXT4 filesystem performance testing in your GitHub repository.

Based on your request, I'll create a detailed README.md file that includes:

- Complete system specifications (AMD Ryzen 9 5900XT, Ubuntu 24.04.3, VirtualBox environment)
- All the test commands and their outputs you provided
- Performance analysis of the dd and fio benchmark results
- Scientific methodology and conclusions
- Proper GitHub markdown formatting

The documentation will cover your filesystem testing research including the characteristics, advantages/disadvantages, and practical performance testing as requested for your academic project.
