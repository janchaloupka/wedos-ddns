# Dynamická DDNS u Wedosu (Docker image)
Docker image pro automatickou změnu IP adresy domény. Jde o fork původního skriptu, který díky kontejnerizaci je možné spustit například v UnRAIDu. V ideálním případě doporučuji nastavit cron aby se tento image periodicky spouštěl.

Docker image je sestaven z poslední verze master větve v Docker Hubu pod názvem [janch32/wedos-ddns](https://hub.docker.com/repository/docker/janch32/wedos-ddns).

## Jak to funguje?
Skript zjistí svoji ip adresu počítače, na kterém běží. Poté přeloží doménové jméno na IP adresu a obě IP adresy porovná. Pokud se liší, skript *wedos-updatedns.py* změní přes Wedos WAPI (rozhraní) IP adresu, kam má doména směřovat. Pak standardně čekáte až hodinu, než se změny projeví na všech DNS serverech.

## Prerekvizity, které musíme mít připravené v administraci Wedosu.
- nastavenou a uloženou A doménu, kterou chcete skriptem udržovat aktuální [návod Youtube](https://youtu.be/TX9eJdxUDcI), [návod v textové nápovědě s typy DNS adres](https://kb.wedos.com/cs/dns/wedos-dns/wedos-dns-zaznamy-domeny/)
> Nejčastěji vás budou zajímat A domény a CNAME (alias = povedou na stejnou IP adresu). A doména je example.com směřující na konkrétní IP adresu. CNAME můžete nastavit jako subdoménu something.example.com s adresou na example.com. Vřele doporučuji dát alias *.example.com , což přesměruje všechny subdomény na váš server a nemusíte je ručně vyjmenovávat. Případně vás budou zajímat ještě MX záznamy pro emailové adresy. Za zmínku stojí ještě formát AAA, což je to samé co A záznam, jen pro IPv6. Teoreticky by mohl tento skript fungovat pro IPv4 i IPv6, ale neměl jsem možnost IPv6 vyzkoušet.
- nastavené WAPI rozhraní ([návod k WAPI](https://kb.wedos.com/cs/wapi-api-rozhrani/zakladni-informace-wapi-api-rozhrani/wapi-aktivace-a-nastaveni/), rozsahy IP adres českých poskytovatelů: [tomasbenda.cz](https://www.tomasbenda.cz/2016/08/27/rozsah-ipv4-adres-pridelenych-pro-ceskou-republiku/) či [nirsoft.net](https://www.nirsoft.net/countryip/cz.html))

## Parametry spuštění
Parametry se zadávají jako proměnné prostředí při vytváření nebo spouštění kontejneru
 - `LOGIN` - E-mailová adresa vašeho Wedos účtu
 - `PASSWORD` - Heslo k Wedos WAPI účtu
 - `DOMAIN` - hlavní doména pod kterou změny provádíme
 - *(volitelné)* `SUBDOMAIN` - poddoména u které se má A záznam nastavit. Již musí existovat A záznam pod touhle doménou. V případě vynechání se záznam aplikuje na doménu druhého řádu.
### Příklady spuštění
1. Dynamické nastavení IP adresu A záznamu na doméně `subdomain.example.com`

`docker run -it --rm -e LOGIN=user@example.com -e PASSWORD=passW0rd! -e DOMAIN=example.com -e SUBDOMAIN=subdomain janch32/wedos-ddns`

2. Dynamické nastavení IP adresu A záznamu na doméně `example.com`

`docker run -it --rm -e LOGIN=user@example.com -e PASSWORD=passW0rd! -e DOMAIN=example.com janch32/wedos-ddns`

## Automatické spouštění skriptu
1. otevřeme správce úloh ```$ crontab -e```
2. na konec souboru přidáme tyto dva řádky:
```
@reboot     ...sem zadejte váš docker příkaz...
0 * * * *   ...sem zadejte váš docker příkaz...
```
> Skript se spustí při každém (re)startu počítače a pak každou hodinu. Čas si můžete upravit pomocí [konfigurátoru](https://crontab.guru/).

### Automatické spuštění v UnRAIDu
V případě běhu v UnRAIDu doporučuji plugin [User Scripts](https://forums.unraid.net/topic/48286-plugin-ca-user-scripts/), který umožňuje nastavit cronjoby. V nastavní pluginu stačí přidat nový skript, který může vypadat například takto: (`wedos-ddns` je název vytvořeného docker kontejneru v UnRAIDu)

```sh
#!/bin/bash
/usr/bin/docker start wedos-ddns
```
