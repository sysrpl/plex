Here's my setup:

* Home server running Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-93-generic x86_64)
* Plex Media Server debian package running on server
* [Netgear Nighthawk R6900](https://www.netgear.com/home/products/networking/wifi-routers/r6900.aspx) home router
* Dynamic hostname from [no-ip.org](http://www.noip.com/), which I'll use for this setup

## Prep

Complete up to the ["Generate the cert" section in this gist](https://gist.github.com/srilankanchurro/e56fa7aee3b2cf36f9c240c90f456494#generate-the-cert) 
and stop just before the "**Set up the certificate**" section.

Your ceritifcate files should now be in this directory: `/etc/letsencrypt/live/myhostname.no-ip.org/`

I also assume your Plex server is port-forwarded to be accessible via port 32400: http://myhostname.no-ip.org:32400

## Set up the certificate
Before we begin, we need to generate a PKCS #12 (.pfx) file from the Let's Encrypt certificate files. It's all the Let's Encrypt files archived, and bundled into one file.

### Create the PCKS #12 file:

1. Run the package command:

    ```bash
      sudo openssl pkcs12 -export -out ~/certificate.pfx \
        -inkey /etc/letsencrypt/live/myhostname.no-ip.org/privkey.pem \
        -in /etc/letsencrypt/live/myhostname.no-ip.org/cert.pem \
        -certfile /etc/letsencrypt/live/myhostname.no-ip.org/chain.pem
    ```
2. You'll first be prompted for your sudo password.
   
   Next you'll be asked to enter a password to encrypt the `.pfx` file. Enter a password you won't mind saving in the Plex settings in plaintext.

3. Hand it over to plex.
    ```bash
    sudo mv ~/certificate.pfx /var/lib/plexmediaserver
    sudo chown plex:plex /var/lib/plexmediaserver/certificate.pfx
    ```

### Have Plex use your PFX file
1. Visit the Plex UI on your server: http://myhostname.no-ip.org:32400

2. Go to Settings (icon on top right corner) > Server (tab) > Network (left navigation column).
   
   Click "SHOW ADVANCED" to see the necessary fields.
   
3. Enter the following values:
    * **Custom certificate location:** /var/lib/plexmediaserver/certificate.pfx
    * **Custom certificate encryption key:** The password you entered on step 2 of last section
    * **Custom certificate domain:** https://myhostname.no-ip.org:32400

4. Save your changes.

That's it. You don't even have to restart plex!

You can check the `Plex\ Media\ Server.log` file in `/var/lib/plexmediaserver/Library/Application\ Support/Plex\ Media\ Server/Logs` if you want to 
verify whether there were any errors.

Visit your server at https://myhostname.no-ip.org:32400 (**Custom certificate domain**) and see the HTTPS in action.
