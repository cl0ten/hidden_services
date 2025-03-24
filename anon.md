# Host your own Anon hidden service!

A super simple guide to spinning up a Anon hidden service.

Ubuntu 20.04 LTS was used for the making of this guide.

### Install Anon

you can install Anon using the following command

You can install the standalone Anon daemon using the following command

```sh
. /etc/os-release
sudo wget -qO- https://deb.en.anyone.tech/anon.asc | sudo tee /etc/apt/trusted.gpg.d/anon.asc
sudo echo "deb [signed-by=/etc/apt/trusted.gpg.d/anon.asc] https://deb.en.anyone.tech anon-live-$VERSION_CODENAME main" | sudo tee /etc/apt/sources.list.d/anon.list
sudo apt-get update
sudo apt-get install anon
```

### Config

you need to look for a `anonrc` file which will most probably be in the `/etc/anon` directory. open it in an editor. you'll need to use `sudo` because it is a protected file.

```sh
#nano
sudo nano /etc/anon/anonrc

#vim
sudo vim /etc/anon/anonrc
```

Look for the following lines.

```conf
############### This section is just for location-hidden services ###

## Once you have configured a hidden service, you can look at the
## contents of the file ".../hidden_service/hostname" for the address
## to tell people.
##
## HiddenServicePort x y:z says to redirect requests on port x to the
## address y:z.

#HiddenServiceDir /var/lib/anon/hidden_service/
#HiddenServicePort 80 127.0.0.1:80
```

you need to uncomment these lines and update accordingly

```conf
HiddenServiceDir /var/lib/anon/hidden_service/
HiddenServicePort 80 127.0.0.1:PORT
```

Where `PORT` is the port number on which your application is accessible on the localhost.

For this guide, we're going to use a simple directory listing server.

If you have NodeJS installed, you can use the following command to start the server.

```sh
npx serve -l 5000
```

Alternatively, if you have Python 3 installed, you can use the following command to start a python http server.

```sh
python3 -m http.server --bind 127.0.0.1 5000
```

So our `anonrc` file now looks like following

```conf
HiddenServiceDir /var/lib/anon/hidden_service/
HiddenServicePort 80 127.0.0.1:5000
```

### Start Anon

Now that everything is configured, you can restart the Anon service by using the following command.

```sh
sudo systemctl restart anon
```

Upon restarting anon with the hidden service configuration, a new `.anon` address will be generated for you. you can get the address using the following command

```sh
sudo cat /var/lib/anon/hidden_service/hostname
```

If you've changed the `HiddenServiceDir` setting in the `anonrc` file, you can find the hostname in `<HiddenServiceDir>/hostname` file.

Anyone can now visit this address on the Anyone Network to use your website.
