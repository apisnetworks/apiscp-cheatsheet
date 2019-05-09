This cheatsheet is a quick reference for common commands, alternate usage, and fast navigation from within the command-line. Feel free to submit a PR on any additional commands you deem useful.

**Legend**  
*S*: Site Admin, *A*: Appliance Admin, *B*: Both  
*FSPATH*: absolute filesystem path for site data  
*module*:*command-name* - API command  
*group*.*task* - [Scopes](https://gitlab.com/apisnetworks/apnscp/blob/master/docs/admin/Scopes.md)  
*DEFAULT*: resets service to default value if specified  

**CLI invocation**  
cpmd [-d domain][-u user] module:command arg1 '[key:value]' null true  
*null* converts to null type, *""* for empty string, *false* for boolean false  

# Management
## Versioning
(A) **Set update policy to edge**  
cpcmd config:set apnscp.update-policy edge  
(A) **Reset update policy to default**  
cpcmd config:set apnscp.update-policy major  
(A) **Update panel code**  
upcp  
(A) **Reset panel code**  
upcp --reset  

## Bootstrapper
(A) **Get all defaults**  
cpcmd config:get apnscp.bootstrapper  
(A) **Override default**  
cpcmd config:set apnscp.bootstrapper name val  
(A) **Run entire Bootstrapper**  
upcp -sb  
(A) **Run Bootstrapper component(s)**  
upcp -sb mail/rspamd mail/configure-postfix  

## Scopes
(A) **General usage**  
cpcmd config:set group.task value  
cpcmd config:get group.task  
cpcmd config:info group.task  

### Scope list
apache.block10 *boolean* - toggle HTTP/1.0 disallowance  
apache.evasive-whitelist *ip address* - whitelist an IP address, accepts wildcards (64.22.68.\*)  
apnscp.debug *boolean* - place panel in debug mode  
apnscp.headless *boolean* - toggle panel frontend  
apnscp.low-memory *boolean* - toggle low-memory mode  
mail.enabled *boolean* - toggle inbound mail services  
mail.smart-host *hostname* *username* *password* - set outbound mail relay  
net.hostname *hostname* - change system hostname  
system.integrity-check - run Bootstrapper, email results  


# API commands
All invoked using CLI, cpcmd *module*:*function*. Full list via https://api.apnscp.com/namespace-none.html

## General
(B) **Get panel version**  
misc:cp-version  
(B) **Commands available for role**  
misc:list-commands  

## Security
(A) **IP is banned**  
rampart:is-banned 64.22.68.1  
(A) **Unban IP**  
rampart:unban 64.22.68.1  
(A) **Whitelist IP** (3.1+)  
rampart:whitelist 64.22.68.1  

## Web
(S) **List/create/delete addon domains**  
aliases:list-shared-domains/aliases:add-domain *domain.com* *docroot*/aliases:remove-domain *domain.com*  
(S) **Update addon domain configuration**  
aliases:synchronize-changes  
(S) **List/create/delete subdomains**  
web:list-subdomains/web:add-subdomain *subdomain* *docroot*/web:remove-subdomain *subdomain*  

## Webapps
(A) **Reset, detect, update all sites**  
admin:reset-webapp-failure  
admin:locate-webapps '[site12,mydomain.com]'  
admin:update-webapps '[type:wordpress]'  
(S) **Setting write-access**  
wordpress:fortify mydomain.com / max  
(S) **Webapp modules**  
composer (3.1+), discourse, drupal, ghost, joomla, laravel, nextcloud (3.1+), wordpress  

### WordPress
(S) **Bypassing bypassing plugin/theme updates**  
*.wp-update-skip* in docroot, format: *name* or *theme:name* or *plugin:name* (newline delimited)  
(S) **Update core, theme, and plugins to latest**  
cpcmd wordpress:update-all mydomain.com /  
(S) **Install WordPress**  
cpcmd wordpress:install mydomain.com / '[ssl:true,email:contact@addr.com]'

## Alternative invocation
(B) **Via browser**  
apnscp.cmd('command_name', ['arg1', 'arg2'], {async: false});  
(B) **Afar**  
https://github.com/apisnetworks/beacon  

# Account management
(A) **Add site**  
AddDomain -c siteinfo,domain=domain.com -c siteinfo,admin_user=admin -c siteinfo,email=user@domain.com ...  
(A) **Edit site**  
EditDomain -c diskquota,quota=20 -c diskquota,units=GB mydomain.com  
(A) **Deactivate site**  
SuspendDomain domain.com *invoice also accepted*  
(A) **Reactivate site**  
ActivateDomain domain.com *invoice also accepted*  
(A) **Delete site**  
DeleteDomain domain.com *invoice also accepted*  

## DNS binding
(A) **Attach provider/key to site**  
EditDomain -c dns,provider=linode -c dns,key=abc1234567890  
(A) **Change global default provider**  
cpcmd config:set dns.default-provider linode  
cpcmd config:set dns.default-provider-key abc1235  
EditDomain -c dns,provider=DEFAULT -c dns,provider_key=DEFAULT domain.com  

## Resource throttling
(A) **Disable all throttles**  
EditDomain -c cgroup,enabled=0 domain.com  
(A) **Set memory limit**  
EditDomain -c cgroup,memory=1024 domain.com  
(A) **Set thread limit**  
EditDomain -c cgroup,proclimit=256 domain.com  
*1 thread = 1 process in non-threaded processes*  
(A) **Set CPU priority**  
EditDomain -c cgroup.cpuweight=2048 domain.com  
*1024 baseline, 2048 = 2x, 512 = 1/2x*  
(A) **Set 24-hour CPU seconds usage**  
EditDomain -c cgroup,cpu=10000 domain.com  

## Shell helpers
(A) **Get site configuration**  
get_config domain.com siteinfo email  
(A) **Override password**  
temp_password domain.com  
(A) **Switch to account admin**  
su domain.com  

# Locations
(A) **apnscp license**  
/usr/local/apnscp/config/license.pem  

## Apache
(A) /etc/httpd/conf/httpd-custom.conf - global config (requires Apache reload; "systemctl reload httpd")  
(A) /etc/httpd/conf/sslXX{,.ssl}/ - per-site global config + SSL-only config (compile config changes w/ "htrebuild"; requires Apache reload)  
(S) *FSPATH*/var/www - broadest application of .htaccess rules (inheritable)  
(S) *FSPATH*/var/www/domain-docroot/.htaccess - per-document root Apache directives (inheritable)  

## Logs
(S) **Per-site web traffic/errors**  
*FSPATH*/var/log/httpd/access_log + error_log  
(S) **Passenger errors**  
/.socket/passenger/logs/passenger.log  
(A) **Mail log**  
/var/log/maillog  
(A) **Rampart ban log**  
/var/log/fail2ban.log  
