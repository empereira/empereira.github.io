# Samba ID Mapping Backends

## Tipos de Backends IDMAP

Os backends de mapeamento de IDs no Samba (idmap) são utilizados para gerenciar a correspondência entre os IDs de usuários e grupos no Windows e os IDs Unix (UIDs e GIDs). Três dos principais backends de idmap são `ad`, `rid` e `autorid`. Abaixo está um resumo de cada um deles com exemplos de configuração.

### idmap_ad

O backend `ad` utiliza os atributos RFC2307 do Active Directory para mapear usuários e grupos. Isso significa que os atributos `uidNumber` e `gidNumber` devem estar definidos no AD para que o mapeamento funcione corretamente. Este backend é adequado quando se deseja que os IDs sejam consistentes e gerenciáveis diretamente no AD.

**Exemplo de configuração:**

```ini
[global]
   security = ADS
   workgroup = SAMDOM
   realm = SAMDOM.EXAMPLE.COM

   idmap config * : backend = tdb
   idmap config * : range = 1000000-1999999

   idmap config SAMDOM : backend = ad
   idmap config SAMDOM : range = 1000-999999
   idmap config SAMDOM : schema_mode = rfc2307
   idmap config SAMDOM : unix_nss_info = yes
```

Neste exemplo, os atributos Unix (como shell e diretório home) são lidos diretamente do AD se estiverem presentes【11†source】【13†source】.

### idmap_rid

O backend `rid` calcula os UIDs e GIDs usando o RID (Relative Identifier) dos objetos do AD. Ele é simples de configurar e não requer que atributos específicos sejam definidos no AD. Este backend garante que os mesmos RIDs resultarão nos mesmos IDs Unix em diferentes membros do domínio Samba.

**Exemplo de configuração:**

```ini
[global]
   security = ADS
   workgroup = SAMDOM
   realm = SAMDOM.EXAMPLE.COM

   idmap config * : backend = tdb
   idmap config * : range = 1000000-1999999

   idmap config SAMDOM : backend = rid
   idmap config SAMDOM : range = 1000-999999
```

Neste exemplo, o backend `rid` é utilizado para o domínio `SAMDOM`, e os IDs são gerados automaticamente com base nos RIDs do AD.

### idmap_autorid

O backend `autorid` é uma versão mais automática do `rid`, ideal para ambientes com múltiplos domínios. Ele atribui blocos de IDs a diferentes domínios automaticamente, reduzindo a necessidade de administração manual. No entanto, ele não garante que os IDs sejam os mesmos em diferentes membros do domínio, a menos que a configuração seja cuidadosamente gerida.

**Exemplo de configuração:**

```ini
[global]
   security = ADS
   workgroup = SAMDOM
   realm = SAMDOM.EXAMPLE.COM

   idmap config * : backend = autorid
   idmap config * : range = 10000-24999999
   idmap config * : rangesize = 200000
```

Neste exemplo, o `autorid` é configurado para o domínio padrão, atribuindo IDs em blocos de 200.000 para cada domínio conforme necessário.

### Considerações Finais

- O backend `ad` é indicado quando se precisa de um controle rigoroso dos atributos Unix diretamente no AD.
- O backend `rid` é útil para uma configuração mais simples e direta, onde a consistência de IDs entre diferentes servidores é necessária.
- O backend `autorid` oferece uma configuração automática e é ideal para ambientes com múltiplos domínios, embora precise de um cuidado especial para evitar sobreposição de IDs.

Cada backend tem seus benefícios e desvantagens, e a escolha depende das necessidades específicas de gerenciamento de IDs e da infraestrutura de TI existente.

## Melhor escolha de backend para diferentes ambientes

### Ambiente com 1000 usuários, 2 ADs e 1 fileserver

Para um ambiente com 1000 usuários, utilizando 2 ADs e 1 fileserver, a melhor escolha de backend é o `autorid`.

**Razões:**
1. **Escalabilidade:** `autorid` gerencia múltiplos domínios automaticamente, ideal para configurações com mais de um AD.
2. **Simplicidade:** Minimiza a administração manual de ID ranges e evita sobreposição de IDs, simplificando a configuração.
3. **Consistência:** Garante a atribuição de IDs de forma automática e consistente para usuários e grupos em diferentes domínios e servidores.

### Configuração Exemplo:

```ini
[global]
   security = ADS
   workgroup = SAMDOM
   realm = SAMDOM.EXAMPLE.COM

   idmap config * : backend = autorid
   idmap config * : range = 10000-24999999
   idmap config * : rangesize = 200000
```

Essa configuração assegura que todos os usuários e grupos tenham IDs únicos e gerenciáveis automaticamente através dos dois ADs e o fileserver.

### Ambiente com 1000 usuários, 3 ADs e 2 fileservers

Para um ambiente com 1000 usuários, 3 ADs e 2 fileservers, a melhor escolha de backend é o `ad`.

**Razões:**
1. **Consistência de IDs:** Garante que os IDs de usuários e grupos sejam consistentes em todos os servidores e domínios.
2. **Atributos Unix do AD:** Permite a utilização de atributos Unix (UID, GID, shell, diretório home) diretamente do AD, facilitando a administração centralizada.
3. **Escalabilidade:** Suporta um ambiente com múltiplos DCs e fileservers, mantendo a consistência e a gestão centralizada.

### Configuração Exemplo:

```ini
[global]
   security = ADS
   workgroup = SAMDOM
   realm = SAMDOM.EXAMPLE.COM

   idmap config * : backend = tdb
   idmap config * : range = 1000000-1999999

   idmap config SAMDOM : backend = ad
   idmap config SAMDOM : range = 1000-999999
   idmap config SAMDOM : schema_mode = rfc2307
   idmap config SAMDOM : unix_nss_info = yes
```

Essa configuração permite que os atributos Unix sejam gerenciados diretamente no AD, garantindo consistência e facilidade de administração.

### Ambiente ideal para utilizar o backend `ad`

Você deve utilizar o backend `ad` em um ambiente onde:

- **Número mínimo de usuários:** 500 ou mais.
- **Número mínimo de DCs:** 2 (para alta disponibilidade e redundância).
- **Número mínimo de fileservers:** 2 (para garantir a consistência dos atributos Unix).

### Razões para usar `ad`:
1. **Consistência de IDs:** Garante que os IDs de usuários e grupos sejam consistentes em todos os servidores.
2. **Atributos Unix:** Permite a utilização de atributos Unix (UID, GID, shell, diretório home) diretamente do AD, facilitando a administração centralizada.
3. **Complexidade do ambiente:** Adequado para ambientes onde é necessário um controle preciso sobre os IDs e atributos Unix, especialmente com múltiplos DCs e fileservers.

### Configuração Exemplo:

```ini
[global]
   security = ADS
   workgroup = SAMDOM
   realm = SAMDOM.EXAMPLE.COM

   idmap config * : backend = tdb
   idmap config * : range = 1000000-1999999

   idmap config SAMDOM : backend = ad
   idmap config SAMDOM : range = 1000-999999
   idmap config SAMDOM : schema_mode = rfc2307
   idmap config SAMDOM : unix_nss_info = yes
```

Essa configuração permite que os atributos Unix sejam gerenciados diretamente no AD, garantindo consistência e facilidade de administração.