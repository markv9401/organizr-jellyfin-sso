# organizr-jellyfin-sso
A workaround to have a functioning SSO in Organizr + Jellyfin

# credits
All credits for all the hard work in both Jellyfin & Organizr go for their awesome devs:
* https://github.com/jellyfin/jellyfin
* https://github.com/causefx/Organizr

# how does it work?
* Jellyfin's auth currently doesn't support external tokens, header auths etc. necessary for proper SSO. When the Jellyfin server authenticates you the client receives an `AccessToken` which it saves to the browser's `localStorage`. Starting from that point, all client (web frontend) -> Jellyfin backend requests use that to prove identity
* I simply used Organizr's SSO hooks / apis to implement a Jellyfin pseudo-sso which does the authentication, receives the `AccessToken` and then crafts a special `cookie` with it which is then converted to a `localStorage` item by the Organizr frontend
* TL;DR it works just fine, so long you're withing its limitations

# limitations
* since `localStorage` hack-around is all we've got for now, and that's specific to a (sub)domain, your Organizr and your Jellyfin need to be on the same (sub)domain. For example `home.org/organizr` + `home.org/jellyfin` will play nice but `home.org/organizr` + `jellyfin.home.org` will NOT! You can, in fact keep them on seperate (sub)domains but you need an access to Jellyfin on the same domain as Organizr. (More about this on the setup section)
* any slight change in the Jellyfin authentication process / api and this is broken. This isn't much likely very soon and when it happens it probably does for the best, for an actual, great SSO / token support so no worries here imo
* using my "setup" will intentionally break Organizr's otherwise smart deployment which pulls & uses the newest git repo every time the container is deployed

# setup / how to
* Start with a working Jellyfin and Organizr setup
* edit your `/config/www/organizr/api/config.php` and include an option: `'ssoJellyfin': true,` *(or start from ground up via the included default config default.php)*
* make sure you have Jellyfin accessible on the same (sub)domain as Organizr (for example `domain.com/organizr` and `domain.com/jellyfin`). If you use different (sub)domains for different applications, please include an additional reverse proxy entry so that Jellyfin is accessible on the same root, too. For example
```
...
# main Jellyfin access: jellyfin.yourdomain.org
server {
   server_name jellyfin.yourdomain.org;
   include /etc/nginx/ssl_pub.conf;
   location / {
      proxy_pass                http://localhost:8096;
   }

...

# Organizr
server {
   server_name organizr.yourdomain.org;
   location / {
      proxy_pass                http://localhost:80;
   }
   
   # alternative Jellyfin access for localStorage workaround with Organizr
   location /jellyfin/ {
       proxy_pass         http://localhost:8096;
   }
 
...
```
* If you just had to and created a secondary jellyfin access like `https://organizr.yourdomain.org/jellyfin` then please set the `base url` to `jellyfin` inside `Jellyin - Admin - Networking` *(it will NOT break your normal `https://jellyfin.yourdomain.org` access. It will, however, add an additional `/jellyfin/` suffix automatically)*
* Set up your Organizr settings to access Jellyfin on the same domain Organizr is at *(like the newly created one)*
* make sure you use the *same* username / password for Organizr and Jellyfin *(either by using Jellyfin backend or simply setting it up to be the same manually)*
* Jellyfin should be accessible without a login inside an Organizr tab now :) *(Jellyfin, Organizr restarts may be necessary. If still not working try in a private browser tab and if that solves, clean your cache)*
