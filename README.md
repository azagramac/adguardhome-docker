<a href="https://snapcraft.io/adguard-home">
	<img alt="adguard-home" src="https://snapcraft.io/adguard-home/badge.svg"/>
</a>
</p>
<br/>
<p align="center">
    <img src="https://cdn.adtidy.org/public/Adguard/Common/adguard_home.gif" width="800"/>
</p>

### Requeriments
- Service docker running

### Install Docker
    sudo update && sudo apt upgrade -y
    sudo apt install git vim wget curl net-tools ca-certificates gnupg
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
	"$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
    sudo usermod -aG docker $USER
    sudo apt install docker-compose-plugin -y

### Test Docker
    $ docker version
    Client: Docker Engine - Community
     Version:           24.0.2
     API version:       1.43
     Go version:        go1.20.4
     Git commit:        cb74dfc
     Built:             Thu May 25 21:52:41 2023
     OS/Arch:           linux/arm64
     Context:           default

    $ docker compose version
    Docker Compose version v2.18.1

### Clone repo
    git clone https://github.com/AzagraMac/adguardhome-docker.git

### Running
    docker-compose up -d

### Check
    docker ps -a

## Adguard Home® configuration: <img src="https://github.com/JuanRodenas/Pihole_list/blob/main/assets/adguard_home_lightmode.svg" alt="AdGuard Home" width="100"/>
### Setting to have DNS over TLS or DNS over HTTPS enabled
In AdGuard settings, DNS settings:
- Upstream DNS servers, copy one of these URLs:

For Cloudfare DoH-DoT:
```
https://dns.cloudflare.com/dns-query
tls://1dot1dot1dot1.cloudflare-dns.com
```
For DoH-DoT de Quad9:
```
https://dns.quad9.net/dns-query
tls://dns.quad9.net
```

and check the option: "**Load balancing**", by default this option is checked.

- Boot DNS servers, we put the DNS of our choice:

Cloudflared in both IPv4 and IPv6:
```
1.1.1.1
1.0.0.1
2606:4700:4700::1111
2606:4700:4700::1001
```
Quad9 in both IPv4 and IPv6:
```
9.9.9.9
149.112.112.112
2620:fe::fe
2620:fe::fe:9
```

- DNS server configuration, check the option "**Enable DNSSEC**".

## Add domain for DoH and DoT:

### Create the certificate with Let's Encrypt
<details>
    <summary>Create the self-signed personal certificate with Let's Encrypt:</summary>

#### Create the self-signed personal certificate with Let's Encrypt:
Installing a free SSL certificate with CertBot:
1. We update the list of packages.
~~~
sudo apt-get update
~~~
2. Install the Certbot package
~~~
sudo apt-get install certbot
~~~
3. Run the following command modifying the valid email to acquire a Wildcard certificate:
~~~
certbot certonly --manual --preferred-challenges=dns --rsa-key-size 4096 --email usuario@ejemplo.com --agree-tos --server https://acme-v02.api.letsencrypt.org/directory -d "*.your_domain"

~~~
4. Finally, it will ask to make an <code>_acme-challenge</code> TXT record in our name server provider with the content it tells us:
It creates the following files, in the directory <code>/etc/letsencrypt/live/</code>:
- <code>fullchain.pem</code> – your SSL certificate encrypted in PEM.
- <code>privkey.pem</code> – your private key encrypted in PEM.

#### Configuración de Lets encrypt
Steps to follow after requesting the certificate:
* You will be prompted to enter the domain to be certified, enter it using <code>*.</code> plus the domain you wish to certify to obtain the Wildcard.
* Finally, it will ask you to register <code>_acme-challenge</code> TXT type in our name server provider with the content you indicate.

To check if the certificate will self-renew:
* Renewal test (simulación):<code>certbot renew --dry-run</code>
* Check the status of the Certbot timer service: <code>systemctl status certbot.timer</code>
* To renew a certificate: <code>certbot renew</code>
	* To force self-renewal: <code>--force-renewal</code>
* To list jobs: <code>systemctl list-timers --all</code> Debe aparecer el siguiente configurado para la renovación automática: <code>certbot.timer - certbot.service</code>
* Listing certificates: <code>certbot certificates</code>

To revoke a certificate:
* Delete a certificate completely: <code>certbot delete --cert-name example.com</code>
* From the account for which the certificate was issued: <code>certbot revoke --cert-path /etc/letsencrypt/archive/${YOUR_DOMAIN}/cert1.pem</code>
* Using the certificate's private key: <code>certbot revoke --cert-path /PATH/TO/cert.pem --key-path /PATH/TO/key.pem</code>

If you don't want to go through all these steps, you can obtain the certificate with [Zero SSL](https://zerossl.com/). but the wildcard certificate is via payment.
</details>

### Create the self-signed personal certificate with OPENSSL:
<details>
    <summary>Create the self-signed personal certificate:</summary>

#### Crear el certificado personal autofirmado:

Info: [INFO](https://www.busindre.com/comandos_openssl_utiles_para_certificados)
1. We update the list of packages.
~~~
sudo apt-get update
~~~
2. Install the openssl package
~~~
sudo apt-get install openssl
~~~
3. Create the directory where we want to store the certificates:
~~~
mkdir certs 
cd certs/
~~~
4. Create certificate with the following command, changing the certificate path or leave the name of the .key and dot crt to store it in the directory:
~~~
sudo openssl req -x509 -nodes -days 1825 -sha384 -newkey ec:secp384r1 -keyout privkey.key -out privcert.pem
~~~

* You may ask us these questions:
<ul><code>Country Name (2 letter code) [AU]: US</code></ul>
<ul><code>State or Province Name (full name) [Some-State]: New York</code></ul>
<ul><code>Locality Name (eg, city) []: New York City</code></ul>
<ul><code>Organization Name (eg, company) [Internet Widgits Pty Ltd]: Bouncy Castles, Inc.</code></ul>
<ul><code>Organizational Unit Name (eg, section) []: Ministry of Water Slides</code></ul>
<ul><code>Common Name (e.g. server FQDN or YOUR name) []: server_IP_address or domain</code></ul>
<ul><code>Email Address []: admin@your_domain.com</code></ul>

</details>


## Configure certificate in AdGuard Home:
1. Open the AdGuard Home web interface and go to configuration.
2. Scroll down the menu to settings: <code>Encryption settings</code>.
3. Enable check<code>Enable encryption (HTTPS, DNS via HTTPS and DNS via TLS)</code>.
4. Enable <code>Redirect to HTTPS automatically</code>.
5. Enter your domain name in <code>Server name</code>. If you are entering a wildcard, enter the domain name only<code>"example.com"</code>.
6. Copy/paste the contents of the file `fullchain.pem` in <code>Certificados</code>.
7. Copy / paste the contents of the file `privkey.pem` in <code>Private key</code>.
8. Click <code>Save configuration</code>.

## Configure the domain to allow private DNS DoH and DoT clients:
To create a zone in your domain for both <code>*.example.org</code> to enable clients, follow these steps:

#### Instructions for use:

1. Log into the control panel of your web hosting provider or domain registrar where you purchased the domain name.
2. Find the `DNS Zones` option.
3. Create a new `DNS Zones` entry. To add the entry for each client, e.g. `one.example.org`.
This will allow the client created in the `Client Configuration` panel to connect.
4. Configure `Settings/Client Configuration/Persistent clients`. Click `Add Clients` and under `Identifier` create a name.

<sup>Current instructions in the developer's documentation <a href="https://github.com/AdguardTeam/AdGuardHome/wiki/Clients#clientid">documentación</a>.</sup>


## Change password in Adguard
In order to change the password in Adguard we can access these websites and create a username and password:

- [web2generators](https://www.web2generators.com/apache-tools/htpasswd-generator)
- [ipvoid](https://www.ipvoid.com/htpasswd-generator/)
- [wtools](https://wtools.io/generate-htpasswd-online)

<p>We create the user and password. Once created, it has this format:</p>
<p><code>ser:$apr1$x4gcjzrl$qSvcJK46C2rQUGRl4z1kl0</code></p>

<p>Once the user and password have been created, we proceed to access the adguard configuration file, <code>AdGuardHome.yaml</code>.</p>
<p>We look for the following line in the configuration file and replace the created data.</p>
<ul>
<li>For the <code>user</code>: user</li>
<li>For the <code>password</code>: $qSvcJK46C2rQUGRl4z1kl0</li>
</ul>
<pre><code class="lang-yaml">users:
  - name: <span class="hljs-literal">user</span>
    password: <span class="hljs-variable">$apr1</span><span class="hljs-variable">$x4gcjzrl</span><span class="hljs-variable">$qSvcJK46C2rQUGRl4z1kl0</span>
</code></pre>

Once the data has been changed, restart adguard.

# List for Pihole <img src="https://github.com/JuanRodenas/Pihole_list/blob/main/assets/pihole.png" alt="Pi-Hole" width="40"/> and AdGuard Home <img src="https://github.com/JuanRodenas/Pihole_list/blob/main/assets/AdGuard_Logo.png" alt="AdGuard Home" width="32"/>

## Main safelist

| List | Link | Description |
| :-- | :--: | :-- |
| safelist repository | [Link](https://raw.githubusercontent.com/JuanRodenas/Pi-hole_list/main/Listas/whitelist.txt) | safelist JuanRodenas |
| safelist hagezi | [Link](https://raw.githubusercontent.com/hagezi/dns-blocklists/main/whitelist.txt) | safelist hagezi (Not tested) |


## Main Black Lists
<sup>Column Link: Pi-hole® | Adguard Home®.</sup>

#### Host
| List Host | Link | Description |
| :-- | :--: | :-- |
| List oisd | [Link](https://dbl.oisd.nl) &#124; [Link](https://abp.oisd.nl) | To Block host Adguard and domains [dbl.oisd](https://oisd.nl/) |
| The big list | [Link](https://big.oisd.nl/domains) &#124; [Link](https://big.oisd.nl/) | The big list [oisd](https://oisd.nl/) |
| urlhaus-filter-domains | [Link](https://malware-filter.gitlab.io/malware-filter/urlhaus-filter-domains.txt) &#124; [Link](https://malware-filter.gitlab.io/malware-filter/urlhaus-filter-agh.txt) | urlhaus-filter DEV [Link](https://gitlab.com/malware-filter/urlhaus-filter) |
| everything | [Link](https://blocklistproject.github.io/Lists/everything.txt) &#124; [Link](https://raw.githubusercontent.com/blocklistproject/Lists/master/adguard/everything-ags.txt) | To Block everything |
| energized pro | [Link](https://energized.pro/unified/formats/hosts.txt) &#124; [Link](https://block.energized.pro/ultimate/formats/hosts.txt) | To Block [energized](https://energized.pro/) |
| d3ward | [Link](https://raw.githubusercontent.com/d3ward/toolz/master/src/d3host.txt) &#124; [Link](https://raw.githubusercontent.com/d3ward/toolz/master/src/d3host.adblock) | [d3ward](https://github.com/d3ward) popular list |


#### Malware / Shock / Porn / Adult 
| List | Link | Description |
| :-- | :--: | :-- |
| The NSFW list | [Link](https://nsfw.oisd.nl/domains) &#124; [Link](https://nsfw.oisd.nl/) | The NSFW list [oisd](https://oisd.nl/) |
| Gambling-porn | [Link](https://raw.githubusercontent.com/JuanRodenas/Pi-hole_list/main/List/Gambling.txt) &#124; [Link](https://github.com/blocklistproject/Lists/blob/master/adguard/gambling-ags.txt) | To Block Gambling and porn |
| Malware | [Link](https://blocklistproject.github.io/Lists/malware.txt) &#124; [Link](https://raw.githubusercontent.com/blocklistproject/Lists/master/adguard/malware-ags.txt) | To Block malware |
| Ransomware | [Link](https://raw.githubusercontent.com/blocklistproject/Lists/master/ransomware.txt) &#124; [Link](https://raw.githubusercontent.com/blocklistproject/Lists/master/adguard/ransomware-ags.txt) | To Block ransomware |
| phishing | [Link](https://phishing.army/download/phishing_army_blocklist_extended.txt) | To Block phishing |


#### Tracking/Ads
| List Tracking/Ads | Link | Description |
| :-- | :--: | :-- |
| SmartTV | [Link](https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/SmartTV.txt) &#124; [Link](https://raw.githubusercontent.com/blocklistproject/Lists/master/adguard/smart-tv-ags.txt) | To Block SmartTV |
| WindowsSpyBlocker | [Link](https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/spy.txt) | To Block WindowsSpyBlocker |
| GoodbyeAds-Ultra | [Link](https://raw.githubusercontent.com/jerryn70/GoodbyeAds/master/Hosts/GoodbyeAds-Ultra.txt) &#124; [Link](https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.plus.txt) | To Block [hagezi](https://github.com/hagezi/dns-blocklists) and [jerryn70](https://github.com/jerryn70/GoodbyeAds) |
| ads-and-tracking-extended | [Link](https://www.github.developerdan.com/hosts/lists/ads-and-tracking-extended.txt) | To Block ads-and-tracking-extended |
| Adblock_Plus | [Link](https://raw.githubusercontent.com/notracking/hosts-blocklists/master/adblock/adblock.txt) &#124; [Link](https://raw.githubusercontent.com/JuanRodenas/Pi-hole_list/main/List/Adblock_Plus_Ads.txt) | To Block Tracking AdBlock |
| Android tracking | [Link](https://github.com/JuanRodenas/Pihole_list/blob/main/Listas/android-tracking.txt) | Android tracking for AdGuard Home |

#### Adguard team filters
| List Tracking/Ads | Link | Description |
| :-- | :--: | :-- |
| AdGuardSDNSFilter | [Link](https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt) | AdGuard team DNS filter |
| AdAway | [Link](https://adaway.org/hosts.txt) | AdAway default blocklist |
| Game Console Adblock List | [Link](https://raw.githubusercontent.com/DandelionSprout/adfilt/master/GameConsoleAdblockList.txt) | Game Console Adblock List |
| SmartTV-AGH | [Link](https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/SmartTV-AGH.txt) | Smart-TV Blocklist for AdGuard Home |
| Peter Lowe's List | [Link](https://pgl.yoyo.org/adservers/serverlist.php?hostformat=adblockplus&showintro=1&mimetype=plaintext) | Blocklist for use with Adblock Plus |

#### Services
| List Services | Link | Description |
| :-- | :--: | :-- |
| Youtube | [Link](https://raw.githubusercontent.com/blocklistproject/Lists/master/youtube.txt) &#124; [Link](https://raw.githubusercontent.com/blocklistproject/Lists/master/adguard/youtube-ags.txt) | To Block youtube |
| Facebook | [Link](https://github.com/jmdugan/blocklists/blob/master/corporations/facebook/all) | To Block Facebook/Instagram/Whatsapp |
| Whatsapp open | [Link](https://raw.githubusercontent.com/jmdugan/blocklists/master/corporations/facebook/all-but-whatsapp) | To Block Facebook/Instagram but leave Whatsapp open |
| Google | [Link](https://raw.githubusercontent.com/jmdugan/blocklists/master/corporations/google/all) | To Block Google |
| Mozilla | [Link](https://raw.githubusercontent.com/JuanRodenas/Pi-hole_list/main/List/mozilla.txt) &#124; [Link](https://raw.githubusercontent.com/JuanRodenas/Pi-hole_list/main/List/mozilla_adguard.txt) | To Block Mozilla tracking |
| Microsoft | [Link](https://raw.githubusercontent.com/jmdugan/blocklists/master/corporations/microsoft/all) | To Block Microsoft |
| VideoGamesAdiction | [Link](https://raw.githubusercontent.com/JuanRodenas/Pi-hole_list/main/List/VideoGamesAdiction.txt) | To Block VideoGames Adiction |


#### uBlock Origin uAssets
| List Services | Link | Link dev | Description |
| :-- | :--: | :--: | :-- |
| uBlock filters | [Link](https://raw.githubusercontent.com/uBlockOrigin/uAssets/master/filters/filters.txt) | [Link DEV](https://github.com/uBlockOrigin/uAssets/tree/master/filters) | uBlock filters |
| Badware risks | [Link](https://raw.githubusercontent.com/uBlockOrigin/uAssets/master/filters/badware.txt) | [Link DEV](https://github.com/uBlockOrigin/uAssets/tree/master/filters) | uBlock filters – Badware risks |
| Privacy | [Link](https://raw.githubusercontent.com/uBlockOrigin/uAssets/master/filters/privacy.txt) | [Link DEV](https://github.com/uBlockOrigin/uAssets/tree/master/filters) | uBlock filters – Privacy |
| Quick fixes list | [Link](https://raw.githubusercontent.com/uBlockOrigin/uAssets/master/filters/quick-fixes.txt) | [Link DEV](https://github.com/uBlockOrigin/uAssets/tree/master/filters) | Quick fixes list |
| Resource abuse | [Link](https://raw.githubusercontent.com/uBlockOrigin/uAssets/master/filters/resource-abuse.txt) | [Link DEV](https://github.com/uBlockOrigin/uAssets/tree/master/filters) | uBlock filters – Resource abuse |
| Unbreak | [Link](https://raw.githubusercontent.com/uBlockOrigin/uAssets/master/filters/unbreak.txt) | [Link DEV](https://github.com/uBlockOrigin/uAssets/tree/master/filters) | uBlock filters – Unbreak |
| i-dont-care-about-cookies | [Link](https://raw.githubusercontent.com/uBlockOrigin/uAssets/master/filters/unbreak.txt) | [Link DEV](https://www.i-dont-care-about-cookies.eu/) | i-dont-care-about-cookies |
| urlhaus-filter | [Link](https://malware-filter.gitlab.io/malware-filter/urlhaus-filter.txt) | [Link DEV](https://gitlab.com/malware-filter/urlhaus-filter) | urlhaus-filter |

<sup>A tab has been added for AdGuard with lists adapted to its format.</sup>


### Check your SelfHosted:

<details>
<summary>fivefilters:</summary>

<Original>&nbsp;Page to check your selfhosted from fivefilters</Original>

<p>  &nbsp;&nbsp;https://blockads.fivefilters.org/</p>
</details>
&nbsp;

<details>
<summary>d3ward:</summary>

<Original>&nbsp;Page to check your selfhosted from [d3ward](https://d3ward.github.io/toolz/)</Original>

<p>  &nbsp;&nbsp;https://d3ward.github.io/toolz/adblock.html</p>
</details>
&nbsp;

<details>
<summary>canyoublockit:</summary>

<Original>&nbsp;Page to check your selfhosted from canyoublockit</Original>

<p>  &nbsp;&nbsp;https://canyoublockit.com/</p>
</details>
&nbsp;

<details>
<summary>No more ads:</summary>

<Original>&nbsp;Page to check your selfhosted from No more ads</Original>

<p>  &nbsp;&nbsp;https://ads-blocker.com/es/pruebas/</p>
</details>
&nbsp;


<details>
<summary>AdBlock Tester:</summary>

<Original>&nbsp;Page to check your selfhosted from AdBlock Tester</Original>

<p>  &nbsp;&nbsp;https://adblock-tester.com/</p>
</details>
&nbsp;

### Check DoH, DoT and DDNSSEC:

<details>
<summary>1.1.1.1 de Cloudflare:</summary>

<Original>&nbsp;Page to check encryption of 1.1.1.1 de Cloudflare</Original>

<p>  &nbsp;&nbsp;https://1.1.1.1/help</p>
</details>
&nbsp;

<details>
<summary>Tenta VPN Browser:</summary>

<Original>&nbsp;Page to check encryption of Tenta VPN Browser</Original>

<p>  &nbsp;&nbsp;https://tenta.com/test/</p>
</details>
&nbsp;

<details>
<summary>Cloudflare:</summary>

<Original>&nbsp;Page to check encryption of Cloudflare</Original>

<p>  &nbsp;&nbsp;https://www.cloudflare.com/es-es/ssl/encrypted-sni/</p>

#### The technologies analysed are:
1. Secure DNS: a technology that encrypts DNS queries and includes DNS-over-TLS and DNS-over-HTTPS.
2. DNSSEC: a technology designed to verify the authenticity of DNS queries.
3. TLS 1.3: the latest version of the TLS protocol that includes many improvements and closes security holes from previous versions.
4. Encrypted SNI: stands for Server Name Indication encryption that reveals the hostname during a TLS connection. This technology aims to ensure that only the IP address can be leaked.
<p><sup>The only browser that supports all four technologies is Firefox.</sup></p>

#### To activate the technologies, go to `about:config` and activate:
<p>  &nbsp;&nbsp;<code>network.security.esni.enabled</code> - pulsamos en el <code>+</code> y se ponga en <code>true</code>.</p>
<p>  &nbsp;&nbsp;<code>network.trr.mode</code> – (valor 2)</p>
<p>  &nbsp;&nbsp;<code>network.trr.uri</code> – <a href="https://mozilla.cloudflare-dns.com/dns-query">valor en la web Mozilla.</a></p>
<p>  &nbsp;&nbsp;<code>HTTPS-Only Mode</code> - pulsamos en el <code>+</code> y se ponga en <code>true</code>.</p>
</details>
&nbsp;

<details>
<summary>DNSSEC Resolver Test:</summary>

<Original>&nbsp;Page to check DNSSEC</Original>

<p>  &nbsp;&nbsp;http://dnssec.vs.uni-due.de/</p>
<p>  &nbsp;&nbsp;http://www.dnssec-or-not.com/</p>
<p>  &nbsp;&nbsp;http://en.conn.internet.nl/connection/</p>
<p>  &nbsp;&nbsp;https://wander.science/projects/dns/dnssec-resolver-test/</p>

<Original>&nbsp;Page to check DNSSEC encryption</Original>
<p>  &nbsp;&nbsp;https://rootcanary.org/test.html</p>
</details>
&nbsp;

<details>
<summary>DNS leak test:</summary>

<Original>&nbsp;Page to check DNS leakage</Original>

<p>  &nbsp;&nbsp;https://www.dnsleaktest.com/</p>

</details>
&nbsp;

## Applications for Android or iOS.
Link to the developer of the application: [![GitHub](https://img.shields.io/static/v1.svg?color=blue&labelColor=555555&logoColor=ffffff&style=social&label=JGeek00&message=GitHub&logo=github)](https://github.com/JGeek00 "view the source for all of our repositories.")

### Adguard Home® android application
<a href="https://play.google.com/store/apps/details?id=com.jgeek00.adguard_home_manager"><img src="https://lh3.googleusercontent.com/q1k2l5CwMV31JdDXcpN4Ey7O43PxnjAuZBTmcHEwQxVuv_2wCE2gAAQMWxwNUC2FYEOnYgFPOpw6kmHJWuEGeIBLTj9CuxcOEeU8UXyzWJq4NJM3lg=s0" width="130px"></a>

<img src="https://github.com/AzagraMac/adguardhome-docker/assets/571796/1cadc751-565d-430a-a884-7c17384135b0" height="600"/> 

<img src="https://github.com/AzagraMac/adguardhome-docker/assets/571796/30c2bf8a-9106-49c3-918e-28eba88092b8" height="600"/> 


### Adguard Home® iOS application
<a href="https://apps.apple.com/es/app/adguard-home-remote/id1543143740"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3c/Download_on_the_App_Store_Badge.svg/640px-Download_on_the_App_Store_Badge.svg.png" width="130px"></a>

<img src="https://github.com/AzagraMac/adguardhome-docker/assets/571796/13c9771a-8847-4e56-9a1f-0327f6ba0caf" height="600"/> 

<img src="https://github.com/AzagraMac/adguardhome-docker/assets/571796/f3d2526c-018d-4eff-aa75-b10bd2625864" height="600"/>

<p><sub>Any and all rights and responsibilities pertaining thereto remain the property of the respective developer.</sub></p>

## HELP ME 🙌
<p> &nbsp;If you want to contribute to improve the lists, open a <code>issue</code> here:  <A HREF="https://github.com/AzagraMac/AdGuardDocker/issues"> ISSUE </A></p>

## Credits 🚀
This repository is made with all my love and affection.
#
[![GitHub](https://img.shields.io/static/v1.svg?color=blue&labelColor=555555&logoColor=ffffff&style=social&label=Follow%20AzagraMac&message=GitHub&logo=github)](https://github.com/AzagraMac "view the source for all of our repositories.")
[![GitHub](https://img.shields.io/static/v1.svg?color=blue&labelColor=555555&logoColor=ffffff&style=social&label=Follow%20JuanRodenas&message=GitHub&logo=github)](https://github.com/JuanRodenas "view the source for all of our repositories.")

# :coffee: Donations
<img src="https://github.com/AzagraMac/AzagraMac/assets/571796/f4f27bb8-cc3a-47e7-94a9-c4569d412a80" width="19" height="19" title="bitcoin"/> <code>1K7bU83Lw1LxzN2dKWrLrWjA51HDpfyzWm</code> <br>
<img src="https://github.com/AzagraMac/AzagraMac/assets/571796/59998222-1cc2-405e-b5f6-323d5e456ba9" width="19" height="19" title="ethereum"/> <code>0x9C4e7853cB77F57EFd834F540Bc31F4f06562A11</code> <br>
<img src="https://github.com/AzagraMac/AzagraMac/assets/571796/b22e20a6-5147-4615-93dd-08e8e2d3d25b" width="19" height="19" title="doge" /> <code>DJfiHJGmJK6iCB8iugG879a4L6ixNHtYg1</code> <br>
<img src="https://github.com/AzagraMac/AzagraMac/assets/571796/c21d91cb-6c03-4cdf-aff0-4bba4e1837bf" width="19" height="19" title="litecoin"/> <code>LgWSf87Vfcz5yejVjZJWvSbi5WwBRaRsZg</code>

# 🎉 ¡Ready!
&nbsp;

<sup>These files/texts are provided "AS IS", without warranties of any kind, express or implied, including, but not limited to, warranties of merchantability, fitness for a particular purpose and non-infringement. In no event shall the authors or copyright holders be liable for any claims, damages or other liability arising out of or relating to the files or the use thereof.</sup>

<sub>Any and all trademarks are the property of their respective owners.</sub>

<p><sup>I will be updating with information and adding procedures in my spare time.</sup></p>
