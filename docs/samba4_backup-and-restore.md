## Backup com o Bacula

Pode-se configurar o bacula para backup dos dados e db do AD.

## Restaurando o arquivo de backup

O passo para restaurar um backup é semelhante à 'provisão de domínio' que você fez quando configurou pela primeira vez sua rede Samba, exceto que desta vez o backup contém todos os objetos de banco de dados que você adicionou desde então. Semelhante à realização de uma provisão, você precisa especificar um novo Controlador de Domínio (DC) ao executar o comando de restauração.

### Preparação do ambiente
Como será necessário outro nome de hostname, antes da execução do comando, troque o nome do hostname no arquivo `/etc/hostname` e também no `/etc/hosts`.

Para que os dados fiquem no diretorio default de instalação, terá que mover a pasta `/var/lib/samba` para outro local.

```bash
mv /var/lib/samba /root
```
### Executando o comando

Após mover, conseguirá executar o comando abaixo.
Observe que o diretório de destino especificado deve estar vazio (ou não existir).

```bash
samba-tool domain backup restore --backup-file=samba-backup-2023-12-31T10-06-04.581319.tar.bz2 --newservername=dc002 --targetdir=/var/lib/samba --site="Default-First-Site-Name" --host-ip=10.10.10.7
```
### Movendo e modificando arquivos e pastas

Depois do processo de restore, mova a seguinte pasta:

1. Dentro da pasta `/var/lib/samba/state` terá a pasta `sysvol` que deveerá ser movida para `/var/lib/samba`
2. No arquivo `/etc/samba/smb.conf` substitua o antigo `netbios name` para o novo nome, no caso do comando acima, `dc002`


### Fazendo o demote do antigo DC

```bash
samba-tool domain demote --remove-other-dead-server=DCXXX
```

## Colocando no dominio

Após todos os passos acima, terá que re-colocar no dominio. Para isso execute o comando abaixo.

```bash
samba-tool domain join campus.site.intra DC -U"CAMPUS\administrator" --option='idmap_ldb:use rfc2307 = yes' --dns-backend=BIND9_DLZ
```

Será solicitado a senha de `administrator`.

### Reconfigurando o DNS

Para reconfigurar o DNS execute o comando abaixo:

```bash
samba_upgradedns --dns-backend=BIND9_DLZ
```
Verifique se está tudo ok com o DNS.

```bash
samba_dnsupdate --verbose --all-names
```

### Reboot

Por fim, dê um reboot na maquina.

Teste os serviços do samba e do DNS com os comandos:

```bash
systemctl status samba-ad-dc.service 
systemctl status named.service
```

Verifique a replicação

```bash
samba-tool drs showrepl
```

Verifique os logs

```bash
tail -f /var/log/samba/log.samba
```