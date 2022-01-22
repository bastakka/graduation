# Multifunkční VMware obraz linuxové distribuce Debian

## Zavedení virtuálního počítače

Pomocí programu Hyper-V Manager byl vytvořen nový virtuální počítač druhé generace, což umožnilo počítači využívat celý výkon hostitelského systému. Toto řešení bylo optimální pouze pro instalaci systému. Do počítače byl přidán dynamicky alokovaný virtuální disk o maximální velikosti 127GB pomocí SCSI řadiče. Maximální velikost disku se dá libovolně měnit. Dále do počítače byla zavedena virtuální DVD mechanika a virtuální síťový adaptér s připojením na internet.

### Zavedení počítače do VMware

Doporučené nastavení počítače pro VMware jsou `4 jádra CPU a 6GB RAM`. Disk byl přiložen v této práci. Počítač nemá žádné Hardwarové požadavky, než funčkní síťovou kartu s připojením na internet. Počítač je třeba nastavit na režim EFI, jestliže tak nebylo možné učinit při jeho vytváření, je možné toto nastavit přidáním řádku `firmware = "efi"` do .vmx souboru počítače.

## Instalace operačního systému na disk

Z oficálních stránek [Debian](https://www.debian.org/CD/live/) byl stažen nejnovější `live` DVD obraz této lixunové distribuce. V době instalace se jednalo o verzi 11.1.0. Následně bylo DVD zavedeno do virtuální DVD mechaniky virtuálního počítače. Počítač byl nastaven na spuštění z tohoto DVD obrazu. V úvodní obrazovce GRUBu byla zvolena možnost instalátoru Debianu.

### Instalační proměnné

Jako výchozí jazyk byla nastavena angličtina. Prostředí bylo nastaveno na Spojené státy, jelikož Debian neobsahuje české prostředí v této verzi. Jazyk klávesnice byl nastaven na Americkou Angličtinu.

Název hostitele (hostname) byl nastaven na `debian`. Název domény byl nastaven na `sspbrno.cz`.  Dále bylo nastaveno heslo na uživatele root. Uživatel root bude později uzamčen a jeho heslo bude zneplatněno, tudíž není potřeba toto heslo uvádět. Byl vytvořen nový uživatel `administrator` s heslem `8i6vyAAB`.

### Formátování disku

Virtuální disk byl naformátován vcelku pomocí řízeného formátování. Všechny složky byly nastaveny, aby se nacházely v jednom oddíle. Na konci řízeného formátování by měla rekapitulace vypadat takto:

![oddíly](oddily.png)

Formátování disku bylo potvrzeno.

### Instalace operačního systému

Soubory operačního systému byly na disk přeneseny automaticky. Poté byl nastaven český repozitář `deb.debian.org` pro balíčkový manažer. Připojení bylo nastaveno bez využití proxy serveru.

### Instalace zavaděče

Zavadeč operačního systému byl nainstalován automaticky. Jedná se o EFI instalaci. Virtuální počítač automaticky změnil své výchozí spuštění z DVD na soubor `EFI/debian/shimx64.efi` umístěný v prvním oddíle na disku.

### Řešení zavaděče na VMware

Je možné, že BIOS VMware nebude vědět, že má automaticky spustit soubor `EFI/debian/shimx64.efi`. V takovém připadě je nutné vejít do EFI BIOSu daného počítače a nastavit spouštění z tohoto souboru.

## Nastavení prostředí

### Nastavení uživatelských účtů

Z uživatele root byla uživateli `administrator` byla přiřazena skupina `sudo` pro možnost spouštění příkazů se zvýšenými právy. Bez těchto práv by nebylo možné z tohoto účtu instalovat nové balíčky a provádět jiné potřebné příkazy.

```bash
root@debian:~$ sudo usermod -aG sudo administrator
```

```bash
sudo            Spustit příkaz se zvýšenými právy
usermod         Příkaz pro správu uživatelů
-a              Přidat vlastnost, nepřepisovat
-G              Změna skupiny
sudo            Název skupiny se zvýšenými právy
administrator   Jméno uživatele
```

Jestliže by chyběla možnost `-a` v příkazu, uživateli `administrator` by byli odebrány všechny ostatní skupiny.

Všechny další kroky byly prováděny z uživatelského účtu `administrator`. Nyní bylo nutné se znova na tento účet přihlásit pro zavedení přidání skupiny `sudo`

Účet root byl uzamčen a jeho heslo bylo zneplatněno z důvodu zabezpečení systému. Hlavně kvůli možnému zapomenutí otevřené konzole přihlášené na tomto účtě.

```bash
administrator@debian:~$ sudo passwd -d root
administrator@debian:~$ sudo passwd -l root
```

```bash
sudo        Spustit příkaz se zvýšenými právy
passwd      Příkaz pro správu hesel
-d          Smazat heslo uživatele
-l          Zamknout možnost přihlášení jako uživatel
root        Jméno uživatele
```

Příkazy je nutno napsat ve správném pořadí, jinak se nejříve zamkne možnost přihlášení jako uživatel a až poté se jeho heslo vymaže, čímž se umožní na uživatele přihlásit a dokonce i bez hesla!

Dále byl uživateli změněn výchozí shell, kde namísto výchozího prostředí `/bin/bash` byla nastavena hodnota `/usr/sbin/nologin`, což zapříčínilo, že i po přihlášení na tohoto uživatele, například pomocí příkazu `sudo su`, se místo výchozího prostředí ukáže akorát chybová hláška.

```bash
administrator@debian:~$ sudo usermod -s /usr/sbin/nologin root
```

```bash
sudo                Spustit příkaz se zvýšenými právy
usermod             Příkaz pro správu uživatelů
-s                  Změna shellu
/bin/sbin/nologin   Cesta k shellu
root                Jméno uživatele
```

Jestliže bychom se nyní pokusili přihlásit na uživatele root

```bash
administrator@debian:~$ sudo su
This account is currently not availible.
administrator@debian:~$
```

### Instalace balíčků

Balíčky v linuxové distribuci Debian má na stratosti balíčkový manažer `apt` , pomocí kterého budou standartně balíčky instalovány. U instalace každého balíčku bude přsto napsán příkaz, pomocí kterého byl balíček nainstalován pro případ, že by se balíček instaloval jiným způsobem.

Příklad instalace balíčku v manažeru apt

```bash
administrator@debian:~$ sudo apt install sl
```

```bash
sudo                Spustit příkaz se zvýšenými právy
apt                 Balíčkový manažer
install             Instalovat balíček
sl                  Název balíčku
```

Jestliže bychom chtěli balíček pouze stáhnout , ale neinstalovat, musíme přidat za tento příkaz příznak `--download-only`

Před instalací jakéhokoli balíčku je doporučeno provést aktulizaci balíčků.

### Aktulizace balíčků

Pro aktulizaci balíčků se bude používat dvojice příkazů.

#### Aktulizace databáze balíčků

První příkaz aktulizuje databázy balíčků z repozitářů specifikovaných v souboru `/etc/apt/sources.list` nebo v jakémkoli souboru končícím příponou `.list` v adresáři `/etc/apt/sources.list.d/`. Do tohoto adresáře budeme později přidávat repozitáře nové.

```bash
administrator@debian:~$ sudo apt update
```

```bash
sudo                Spustit příkaz se zvýšenými právy
apt                 Balíčkový manažer
update              Aktulizovat databázy balíčků
```

Jestliže by dva repozitáře obsahovali jeden stejný balíček, tak se priorizuje repozitář podle označení s vyšším číslem priority v konfiguračním souboru `/etc/apt/preferences` nebo v jakémkoli souboru v adresáři `/etc/apt/preferences.d/`. Tyto priority budeme také upravovat.

#### Stažení a instalace balíčků

Druhý příkaz stáhne a nainstaluje všechny balíčky, které mají v databázy balíčků verzi vyšší než je aktuálně nainstalovaná.

```bash
administrator@debian:~$ sudo apt upgrade
```

```bash
sudo                Spustit příkaz se zvýšenými právy
apt                 Balíčkový manažer
uprgade             Stáhnout a nainstalovat vyšší verzi balíčků podle databáze
```

Jestliže bychom chtěli pouze stáhnout nové verze balíčků bez jejich samotné instalace, musíme přidat za tento příkaz příznak `--download-only`

```bash
administrator@debian:~$ sudo apt upgrade --download-only
```

#### Odstranění balíčků

Pro odstranění balíčků bude používán tento příkaz

```bash
administrator@debian:~$ sudo apt remove sl
```

```bash
sudo                Spustit příkaz se zvýšenými právy
apt                 Balíčkový manažer
remove              Odstranit balíček
sl                  Název balíčku
```

Tento příkaz pouze balíček odinstaluje, ale instalační soubory a soubory aplikace nebudou odstraněny. Abychom tohoto dosáhli, musíme použít příznak `--purge`

```bash
administrator@debian:~$ sudo apt remove --purge sl
```

#### Automatické odstranění nepotřebných knihoven

Při instalaci programu pomocí `apt` se zároveň s ním instalují potřebné knihovny. Jestliže odstraníme program, kvůli kterému byly knihovny nainstalovány, zůstanou v počítači bez využití. Abychom nemuseli ručně mazat každou takovou knihovnu, můžeme použít následující příkaz.

```bash
administrator@debian:~$ sudo apt autoremove
```

```bash
sudo                Spustit příkaz se zvýšenými právy
apt                 Balíčkový manažer
autoremove          Automaticky odstranit nepotřebné knihovny
```

#### Přeinstalace balíčků

Programy je možné pomocí `apt` přeinstalovat.

```bash
administrator@debian:~$ sudo apt reinstall sl
```

```bash
sudo                Spustit příkaz se zvýšenými právy
apt                 Balíčkový manažer
reinstall           Přeinstalovat program
sl                  Název programu
```

### Správce procesů

Procesy aplikací, hlavně těch na pozadí, spravuje systémová služba `systemd`. Pomocí této služby je možné nastavit automatické spuštění aplikací na pozadí a sledovat zde jejich stav.

#### Nasatvení procesů

Pro spuštění procesu je nutné nastavit za jakého uživatele se má proces spustit, jakou cestu má proces hledat k spustitelnému souboru programu a příkazy k programu. Dále je zde možné nastavit podmínku spuštění např.: Pouze pokud je dostupné internetové připojení nebo spustit program každou hodinu jestliže se program sám po dokončení práce ukončí.

Veškeré mnou vytvořené procesy byly ukládany do adresáře `/etc/systemd/system/`. Procesy vytvořené instalací balíčků se ukládají do adresáře `/lib/systemd/system`.

Příklad takového nastavení

```systemd
[Unit]
Description=Příklad procesu
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=administrator
ExecStart=/cesta/k/programu argumenty

[Install]
WantedBy=multi-user.target
```

Vysvětlení parametrů

```systemd
[Unit]            Blok s popisem procesu
Description       Popis procesu
After             Nastevení podmínky spuštění po skončení jiného procesu
network.target    Podmímka dosáhnutí aktivního připojení k síti

[Service]         Blok s informacemi o řízení procesu
Type              Typ procesu
simple            Jednoduchý proces

Restart           Nastavení znovuspuštění aplikace po vyplnutí
always            Aplikace se znovu spustí pokaždé co skončí
RestartSec        Časový interval mezi vypnutím a znovuspuštěním procesu
User              Za jaký účet se má aplikace spustit
ExecStart         Cesta k programu, který má proces zapnout

[Install]         Blok se speciálními informacemi
WantedBy          Specifikace vztahu tohoto procesu k procesu jinému
                  V tomto případě je tento proces žádán jiným.
multi-user.target Název procesu
```

Pro více informací o nastavení procesů viz [DigitalOcean](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)

Po každé modifikaci nastavení procesu je nutné zavést změny do programu `systemd`.

```bash
administrator@debian:~$ sudo systemctl daemon-reload
```

#### Spuštění procesu

Následující příkazy jsou čistě demonstrativní, žádný takový proces na počítači v této části dokumentace není, bude ovšem v pozdější části dokumentace přidán.

Tímto příkazem se spustí proces bez ohledu na podmínky.

```bash
administrator@debian:~$ sudo systemctl start mysql
```

```bash
sudo                Spustit příkaz se zvýšenými právy
systemctl           Manažer procesů na pozadí
start               Spustit proces
mysql               Název procesu
```

Jestliže bychom chtěli proces spouštět automaticky podle podmínek.

```bash
administrator@debian:~$ sudo systemctl enable mysql
```

#### Stav procesu

```bash
administrator@debian:~$ sudo systemctl status mysql
● mariadb.service - MariaDB 10.5.12 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2021-10-19 15:26:19 CEST; 2 weeks 3 days ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
   Main PID: 1069 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 8 (limit: 2355)
     Memory: 157.9M
        CPU: 8min 8.633s
     CGroup: /system.slice/mariadb.service
             └─1069 /usr/sbin/mariadbd
```

Příkaž vypíše detalní popis procesu a poté log výstupů procesu.

#### Vypnutí procesu

```bash
administrator@debian:~$ sudo systemctl stop mysql
```

Tento příkaz ukončí proces a proces zůstane vypnutý až do restartu systému. Jestliže bychom chtěli vypnout automatický start procesu.

```bash
administrator@debian:~$ sudo systemctl disable mysql
```

#### Restart procesu

Tento příkaz ukončí a znovu spustí proces.

```bash
administrator@debian:~$ sudo systemctl restart mysql
```

### Osobní nastavení

Pro jednodušší a pohodlnější práci v linuxovém prostředí byly nainstalovány následující programy.

#### Git

Škálovatelý program pro správu verzí projektu. Umožňuje klonovat projekty z internetu a za pomocí protkolu `ssh` nebo `http` zaručuje bezpečný přenos i soukromých projektů do vlastních repozitářů. Obsahuje velké množství možností, ať už podpis verzí projektů pomocí `gpg` klíče nebo řešení rozdílů mezi verzemi od více lidí. Bude použit hlavně pro klonování projektů ze serveru `github.com`

```bash
administrator@debian:~$ sudo apt update
...
All packages are up to date.
administrator@debian:~$ sudo apt install git
```

#### Fish

Alternativa pro výchozí shell. Prostředí tohoto shellu se dá jednoduše upravovat. Automaticky navrhuje dokončení příkazů. Nabízí vlastní funkce a proměnné, umožňuje jejich tvorbu. Signulizuje v terminálu stav git repozitářů. Instalováno pomocí `apt`. Dokumentace ovšem bude stále používat příkazy pro shell `bash`.

```bash
administrator@debian:~$ sudo apt update
...
All packages are up to date.
administrator@debian:~$ sudo apt install fish
```

#### Oh-my-fish

Manažer vzhledů pro shell `Fish`. Obsahuje příkazy pro změnu vzhledu shellu. Instalováno pomocí skriptu pro fish.

```bash
curl https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install | fish
```

```bash
curl          Program pro přenos dat z adresy URL do terimálu
|             Přenos výstupu příkazu do jiného programu
fish          Alternativní shell
```

Pomocí tohoto manažeru byl změněn vzhled shellu na vzhled se jménem `slavic-cat`

```bash
administrator@debian:~$ fish -c 'omf install slavic-cat'
```

```bash
fish          Alternativní shell
-c            Spustit příkaz v alternativním shellu
omf           Manažer vzhledů fish shellu
install       Instalovat nový vzhled
slavic-cat    Název vzhledu
```

## Nastavení zabezpečení pro komunikaci po síti

Pro zabezpečení komunikace po síti byly nainstalovány dva programy

### UFW - Uncomplicated FireWall

Veškerá komunikace po IP se v linuxu řeší za pomocí tabulky pravidel. UFW je program upravující tyto tabulky v mnohem jednodušším prostředí.

#### Instalace UFW

```bash
administrator@debian:~$ sudo apt update
...
All packages are up to date. 
administrator@debian:~$ sudo apt install ufw
```

#### Konfigurace UFW

Tabulky byly nastaveny pro odmítnutí veškeré příchozí komunikace

```bash
administrator@debian:~$ sudo ufw default deny incoming
```

```bash
sudo                Spustit příkaz se zvýšenými právy
ufw                 Program upravující tabulky pravidel IP
default             Nastavit výchozí hodnotu
deny                Zamezit přiopjení
incoming            Příchozí připojení
```

Dále byly nastaveny pro schávelní všech odchozích připojení

```bash
administrator@debian:~$ sudo ufw default allow outgoing
```

Poté jsme nastavily výjimky pro komunikaci pomocí ssh protkolu, který využijeme později.

```bash
administrator@debian:~$ sudo ufw allow ssh
```

Ještě byly nastaveny výjimky pro komunikaci na portech 80 a 443 využívaných HTTP a HTTPS komunikací, aby byl možný budoucí přístup na webové stránky.

HTTP - Komunikce na portu 80 pomocí protkolu TCP

```bash
administrator@debian:~$ sudo ufw allow 80/tcp
```

HTTPS - Komunikace na portu 443 pomocí protokolu TCP, ale i UDP

```bash
administrator@debian:~$ sudo ufw allow 443/tcp
administrator@debian:~$ sudo ufw allow 443/udp
```

Pro odstranění pravidla je možné využít následující příkaz

```bash
administrator@debian:~$ sudo ufw delete allow ssh
```

#### Aktivace UFW

Po nastavení pravidel musí být zapnuta samotná aplikace UFW. Zapnutím programu se veškeré komunikace budou řídit těmito pravidly, tudíž je nutné vyhodnotit jestli jsou všechna potřebná pravidla nastavena a jestli se aktivací UFW nepřeruší již některá důležitá aktivní spojení.

```bash
administrator@debian:~$ sudo ufw enable
```

#### Aktuální stav UFW

```bash
administrator@debian:~$ sudo ufw status
```

#### Deaktivace UFW

```bash
administrator@debian:~$ sudo ufw disable
```

### Fail2ban

Program hlídající příchozí komunikace na otevřené porty. Jestliže komunikace bude klasifikována jako útok, bude IP adresa, ze které útok přichází, dočasně zablokována. Jako útok se můžou považovat tři neúspešné pokusy o připojení na počítač pomocí ssh protokolu.

#### Instalace Fail2ban

```bash
administrator@debian:~$ sudo apt update
...
All packages are up to date.
administrator@debian:~$ sudo apt install fail2ban
```

#### Konfigurace Fail2ban

Konfigurace programu nebyla změněna. Program obsahuje konfigurační soubory `/etc/fail2ban/jail.conf` a `/etc/fail2ban/jail.d/defaults-debian.conf`, které jsou ale při každé aktulizace balíčku přepisovány, proto se pro vlastní konfiguraci používají vlastní `.conf` a `.local` soubory v adresáři `/etc/fail2ban/jail.d/`. Tyto soubory mají jiné vlastnosti. Soubory `.local` přepisují hodnoty `.conf` souborů, tudíž nemusí obsahovat celou, ale pouze její změny. Pořadí čtení souborů při konfiguraci následující:

1. `/etc/fail2ban/jail.conf`
2. `/etc/fail2ban/jail.d/*.conf`
3. `/etc/fail2ban/jail.local`
4. `/etc/fail2ban/jail.d/*.local`

Pro vysvětlení konfiguračních souborů programu viz [ArchWiki](https://wiki.archlinux.org/title/Fail2ban). Po změně konfiguračních souborů je důležité program restartovat pro zavedení změn.

#### Zapnutí procesu Fail2ban

Proces programu byl automaticky spouštěn při instalaci programu, tudíž není nutné program aktivovat. Proces programu je řízen pomocí `systemd`. Název procesu je stejný jako název programu.

```bash
administrator@debian:~$ sudo systemctl start fail2ban
```

Pro automatický start procesu

```bash
administrator@debian:~$ sudo systemctl enable fail2ban
```

#### Aktuální stav procesu Fail2ban

```bash
administrator@debian:~$ sudo systemctl status fail2ban
```

#### Vypnutí procesu Fail2ban

```bash
administrator@debian:~$ sudo systemctl stop fail2ban
```

Pro vypnutí automatického startu

```bash
administrator@debian:~$ sudo systemctl disable fail2ban
```

## Nastavení vzdáleného přístupu

Ke vzdálenému přístupu na počítač bude využívano potokolu `ssh`.  Na počítači byl nainstalován server pro tuto komunikaci. Tato komunikace bude probíhat na výchozím portu 22. K veškeré budoucí práci na počítači byla od aktivace vzdáleného přistupu byla výhradně využívána tato služba.

```bash
administrator@debian:~$ sudo apt install openssh-server
```

### Konfigurace serveru vzdálelého přístupu

Před spuštěním serveru byla upravena jeho konfigurace v souboru `/etc/ssh/sshd_config`. Následující řádky byly v souboru upraveny.

```ssh
PermitEmptyPasswords no
```

Byly odmítnuty pokusy o příhlášení s prázdným heslem pro nezabezpečené účty odstraněním poznámkového znaku `#` na řádku 47.

```ssh
PermitRootLogin no
```

Byl přidán 112. řádek na konec souboru odmítající pokusy o přihlášení na uživatele `root`.

### Spuštění serveru vzdáleného přístupu

Proces programu je řízen pomocí `systemd`. Název procesu je `sshd`.

```bash
administrator@debian:~$ sudo systemctl start sshd
```

Byl nastaven automatický start procesu.

```bash
administrator@debian:~$ sudo systemctl enable sshd
```

### Aktuální stav serveru vzdáleného přístupu

```bash
administrator@debian:~$ sudo systemctl status sshd
```

### Deaktivace serveru vzdáleného přístupu

```bash
administrator@debian:~$ sudo systemctl stop sshd
```

Pro vypnutí automatického startu

```bash
administrator@debian:~$ sudo systemctl disable sshd
```

## Nastavení webového serveru

Pro pokrytí služeb webového serveru, byl nainstalován komplexní program nginx pomocí instalčního skriptu z [mého GitHub repozitáře](https://github.com/bastakka/nginx-autoinstall), který byl stažen do domovského adrsáře `/home/administrator/`.

```bash
administrator@debian:~$ git clone https://github.com/bastakka/nginx-autoinstall
Cloning into 'nginx-autoinstall'...
...
Resolving deltas: 100% (826/826), done.
administrator@debian:~$ sudo ./nginx-autoinstall/
```
