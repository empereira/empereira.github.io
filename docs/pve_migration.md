# Migração de VM usando `qm remote-migrate`

## Comando `qm remote-migrate`

### Descrição

O comando `qm remote-migrate` migra uma máquina virtual para um cluster remoto, criando uma nova tarefa de migração. Esta é uma funcionalidade experimental.

### Sintaxe

```bash
qm remote-migrate <vmid> [<target-vmid>] <target-endpoint> --target-bridge <string> --target-storage <string> [OPÇÕES]
```

### Parâmetros

- **\<vmid\>**: ID único da VM (100 - 999999999).
- **\<target-vmid\>**: ID único da VM no destino (100 - 999999999).
- **\<target-endpoint\>**: Ponto de extremidade remoto no formato:
  ```plaintext
  apitoken=<PVEAPIToken=user@realm!token=SECRET>,host=<ADDRESS>[,fingerprint=<FINGERPRINT>][,port=<PORT>]
  ```
- **--bwlimit \<integer\>**: Limite de largura de banda de E/S (em KiB/s). Padrão: limite de migração da configuração do datacenter ou armazenamento.
- **--delete \<boolean\>**: Excluir a VM original e dados relacionados após migração bem-sucedida. Padrão: 0 (não excluir).
- **--online \<boolean\>**: Usar migração online/semi-online se a VM estiver em execução. Ignorado se a VM estiver parada.
- **--target-bridge \<string\>**: Mapeamento de bridges de origem para destino. Um único ID de bridge mapeia todas as bridges de origem para esta bridge. Valor especial 1 mapeia cada bridge de origem para ela mesma.
- **--target-storage \<string\>**: Mapeamento de storages de origem para destino. Um único ID de storage mapeia todos os storages de origem para este storage. Valor especial 1 mapeia cada storage de origem para ele mesmo.

### Exemplo de Uso

```bash
qm remote-migrate 101 102 apitoken=user@realm!token=SECRET,host=192.168.1.2 --target-bridge vmbr0 --target-storage local-lvm --bwlimit 512000 --online
```

## Script de Migração

### Descrição

Este script migra uma VM de um servidor Proxmox para outro usando `qm remote-migrate`.

### Script

```bash
#!/bin/bash

# Verificar se o número correto de argumentos foi fornecido
if [ "$#" -ne 3 ]; then
    echo "Uso: $0 <vmid> <target-vmid> <bwlimit>"
    exit 1
fi

# Definir variáveis
export APITOKEN="PVEAPIToken=user@realm!token=SECRET"
export FINGERPRINT="YOUR_FINGERPRINT"
export HOST="YOUR_HOST_ADDRESS"

# Capturar argumentos
VMID=$1
TARGET_VMID=$2
BWLIMIT=$3

# Executar o comando qm remote-migrate
qm remote-migrate $VMID $TARGET_VMID "apitoken=${APITOKEN},host=${HOST},fingerprint=${FINGERPRINT}" --target-bridge vmbr0 --target-storage local-lvm --online 1 --bwlimit $BWLIMIT
```

### Execução

```bash
./migrate.sh <vmid> <target-vmid> <bwlimit>
```

Substitua `YOUR_FINGERPRINT` e `YOUR_HOST_ADDRESS` pelos valores apropriados para seu ambiente. Certifique-se de que os valores sensíveis, como tokens e endereços, estejam seguros e não sejam expostos publicamente.