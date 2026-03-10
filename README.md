# lb-ent-geotool
MaxMind GeoIP integration for Loadbalancer.org Enterprise v8.13.4+ appliances

Sources:
* https://github.com/maxmind/libmaxminddb (git clone --recursive https://github.com/maxmind/libmaxminddb)
* https://github.com/maxmind/mod_maxminddb (git clone https://github.com/maxmind/mod_maxminddb.git)

Steps:
1. Unpack this archive in the root of LB filesystem
2. Run: `ldconfig`
3. Install geoipupdate from MaxMind
        `rpm -Uvh https://github.com/maxmind/geoipupdate/releases/download/v7.1.1/geoipupdate_7.1.1_linux_amd64.rpm`
4. Edit /etc/GeoIP.conf
*        Replace "YOUR_ACCOUNT_ID_HERE" with account ID
*        Replace "YOUR_LICENSE_KEY_HERE" with license key
*        Set: DatabaseDirectory /usr/local/share/GeoIP
5. Run: `geoipupdate -v`
6. Configure the WAF blocking in the WebUI;

## This section enables and sources the MaxMind database

```
MaxMindDBEnable On
MaxMindDBFile COUNTRY_DB /usr/local/share/GeoIP/GeoLite2-Country.mmdb
MaxMindDBEnv GEOIP_COUNTRY_CODE COUNTRY_DB/country/iso_code
```

## This section would block traffic from Poland and Russia, using the 2 letter country code per https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements

```
SecRule ENV:GEOIP_COUNTRY_CODE "!@pm PL RU" \
   "id:10000,\
   phase:2,\
   deny,\
   log,\
   logdata:'%{MATCHED_VAR}',\
   msg:'Denying access for IP from GeoIP country %{MATCHED_VAR}'"
```
