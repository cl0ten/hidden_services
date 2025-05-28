# Host your own Anon hidden service!

A super simple guide to spinning up a Anon hidden service.

Ubuntu 20.04 LTS was used for the making of this guide.

### Install Anon

You can install the standalone Anon daemon using the following command

```bash
. /etc/os-release
sudo wget -qO- https://deb.en.anyone.tech/anon.asc | sudo tee /etc/apt/trusted.gpg.d/anon.asc
sudo echo "deb [signed-by=/etc/apt/trusted.gpg.d/anon.asc] https://deb.en.anyone.tech anon-live-$VERSION_CODENAME main" | sudo tee /etc/apt/sources.list.d/anon.list
sudo apt-get update
sudo apt-get install anon
```

### Config

You need to look for a `anonrc` file which will most probably be in the `/etc/anon` directory. open it in an editor. you'll need to use `sudo` because it is a protected file.

```bash
#nano
nano /etc/anon/anonrc
```
```bash
#vim
vim /etc/anon/anonrc
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

So our `anonrc` file now looks like following

```conf
HiddenServiceDir /var/lib/anon/hidden_service/
HiddenServicePort 80 127.0.0.1:5000
```

## Start Anon

Now that everything is configured, you can restart the Anon service by using the following command.

```bash
systemctl restart anon
```

Upon restarting anon with the hidden service configuration, a new `.anon` address will be generated for you. you can get the address using the following command

```bash
cat /var/lib/anon/hidden_service/hostname
```

If you have Python 3 installed, you can use the following command to start a python http server.

First cd in to a directory that you want to start the http.server in.
```
mkdir webdir
cd webdir
echo -e "hello world" >> ./test.html
```

```bash
python3 -m http.server --bind 127.0.0.1 5000
```
Tip: Use a separate session to keep the other terminal free for further configuration or testing.


Anyone can now visit the address on the Anyone Network to use your website.

## Test Connectivity
If you want to test your connectivity through socks proxy on localhost, you can `curl` the hostname using `--socks-hostname`

Curl the test.file
```bash
curl --socks5-hostname 127.0.0.1:9050 http://<hostname>:80/test.html
```

Show document info only.
```bash
curl -I --socks5-hostname 127.0.0.1:9050 http://<hostname>:80
```

---
# Generating a vanity .anon address with [`mkp224o`](https://github.com/cl0ten/mkp224o)

This version of `mkp224o` is a fork of a tool originally designed to generate Tor v3 `.onion` addresses with custom prefixes, making it useful for creating branded or more memorable hidden service addresses. It is changed to work with compatible forks; such as the [Anyone protocol](https://github.com/anyone-protocol) tools. Allowing generation of `.anon` addresses in the same format.

## Installation

To install `mkp224o`, you'll need to compile it from source. Here's how you can do it on a Debian-based system:

```bash
# Install dependencies
apt install gcc libc6-dev libsodium-dev make autoconf

# Clone the repository
git clone https://github.com/cl0ten/mkp224o.git
cd mkp224o

# Prepare the build system
./autogen.sh
./configure

# Compile the tool
make
```
For other systems or more detailed instructions, refer to the [forked mkp224o GitHub repository](https://github.com/cl0ten/mkp224o).

## Generating a vanity address

Once installed, you can generate a vanity address using the following command:
```bash
./mkp224o test -t 4 -v -n 3 -y
```
Here's what each option means:

* `test` - The desired prefix for your .anon address. Replace this with your chosen prefix.
* `-t` 4 - Use 4 threads for processing. Adjust this number based on your CPU cores.
* `-v` - Enable verbose output to see the progress.
* `-n` 3 - Stop after finding 3 matching addresses.
* `-y` - Output the results in YAML format, which includes the keys and hostname.

The tool will output YAML-formatted keys for each matching address.

# Converting YAML Keys with [`yaml2hs`](https://github.com/cl0ten/yaml2hs)

After generating your desired vanity address, you'll have YAML-formatted keys. To convert these into the format anon expects, use the [`yaml2hs`](https://github.com/cl0ten/yaml2hs) script.

## Using `yaml2hs`

1. Interactive mode
   Run the script and input the keys when prompted.
```bash
python3 yaml2hs.py --interactive
```
   
2. Direct Input
   Provide the keys as command-line arguments.
```bash
python3 yaml2hs.py -p "<base64_public_key>" -s "<base64_secret_key>" -o output_directory
```
Replace `<base64_public_key>` and `<base64_secret_key>` with the corresponding values from the YAML output. The script will generate `hs_ed25519_public_key` and `hs_ed25519_secret_key` files in the specified directory (default is "`keys`").

## Integrating the Custom address into Your Anyone Hidden Service

1. Move a copy of the generated keys to your hidden service directory.
```bash
cp keys/hs_ed25519_* /var/lib/anon/hidden_service/
```
2. Apply the changes by restarting `anon`.
```bash
systemctl restart anon.service
```

You can find the new hostname in the `<HiddenServiceDir>/hostname` file.
```bash
cat /var/lib/anon/hidden_service/hostname
```
