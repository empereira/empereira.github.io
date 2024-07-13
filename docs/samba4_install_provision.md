[Lista oficial do Samba](https://lists.samba.org/)

## Instalação e Atualização da Distro base

**Distro:** Debian 12

### Atualizando
```bash
apt update && apt full-upgrade
```

### Modificar o arquivo /etc/hosts
```plaintext
127.0.0.1        localhost
172.16.1.7       DC1.samdom.dominio.intra   DC1
```

### Modificar o arquivo /etc/hostname
Configure um nome de hostname:
```plaintext
DC1
```
**Fonte:** [Configuração do Samba como Controlador de Domínio](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller#Preparing_the_Installation)

Reinicie o servidor.

## Instalação do pacote Samba4

```bash
apt-get install gnupg
gpg --keyserver hkp://keyring.debian.org --recv-key 701B4F6B1A693E59
gpg --export --armor 804465C5 | gpg --dearmour -o /etc/apt/keyrings/804465C5.gpg
gpg --delete-key 804465C5
echo "deb [signed-by=/etc/apt/keyrings/804465C5.gpg] http://www.corpit.ru/mjt/packages/samba bookworm/samba-4.19/" | tee -a /etc/apt/sources.list.d/mjt-samba4.list
apt-get update
apt-get install --no-install-recommends krb5-user krb5-config libldb2 libnss-winbind libpam-winbind libwbclient0 python3-ldb python3-samba samba samba-common samba-common-bin samba-dsdb-modules samba-libs samba-vfs-modules winbind
```

## Provisionamento

### Parâmetros
Para conhecer todos os parâmetros, pode-se utilizar o comando:
```bash
samba-tool domain provision --help
```

### Comando para o provisionamento
Para provisionar, execute o comando abaixo.
```bash
samba-tool domain provision --use-rfc2307 --interactive
```

**Fonte:** [Configuração do Samba como Controlador de Domínio](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller)

## Restante das Configurações

Acesse o [link](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller#Configuring_the_DNS_Resolver) abaixo e siga o restante das configurações.

[Configurando o resolvedor DNS](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller#Configuring_the_DNS_Resolver)
