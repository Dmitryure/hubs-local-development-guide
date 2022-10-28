# Hubs local development guide

This guide features intallation for **WSL Windows 10/Linux** . 

If you are on Linux, you can go straight to installation guide, for Windows users, we first install WSL

## Installing WSL
Open **Windows PowerShell** as **administrator**.
Run `wsl --install` this will install Ubuntu as your Linux distribution.
Reboot your PC after seeing success message ![img1](https://user-images.githubusercontent.com/33320716/198296719-edff21cb-376e-4842-b381-6b9a6cc67c58.jpg)

Then you will need to install **Ubuntu** from Microsoft store https://www.microsoft.com/store/productId/9PDXGNCFSCZV
Running it for the first time will take longer than usual because it needs to be installed first.

Enter username and password that you won't forget and install some dependencies
```
sudo apt update
sudo apt-get install build-essential python3-pip
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
```

Next, we will change some code in Reticulum. We will do it with `nano`, but you can do it with Linux GUI for WSL

## Setting up Reticulum
First of all, as guide suggests, we need to install **Erlang** and **Elixir**. OTP Erlang can be of versions 22 or 23 will do, as for elixir you can select any version according to [this table of compatibility ](https://hexdocs.pm/elixir/1.12/compatibility-and-deprecations.html#compatibility-between-elixir-and-erlang-otp "this table of compatibility ")

Unfortunately, apt does not let us to choose a version of Erlang and Elixir. So I chose to do it with [asdf](https://asdf-vm.com/guide/getting-started.html#_2-download-asdf "asdf"). 

As for writing this post I needed to run 
`git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.10.2`

Then we make asdf seeable for our terminal. First, you need to edit **./~bashrc** file
Run `sudo nano ~/.bashrc` it will open up the file. Then insert the following lines at the bottom: 
```
. $HOME/.asdf/asdf.sh
. $HOME/.asdf/completions/asdf.bash
```
Now save and exit the file and run `. ~/.bashrc` to reload **~/.bashrc**
Install erlang: 
```
asdf plugin add erlang
asdf install erlang 22.3
asdf global erlang 22.3
```

**If something goes wrong during erlang installation you can try doing it with** [brew](https://formulae.brew.sh/formula/erlang@22)

Install elixir: 
```
asdf plugin add elixir
asdf install elixir 1.12
asdf global elixir 1.12
```

Check that everything is installed:
`elixir -v`

![image](https://user-images.githubusercontent.com/33320716/198309464-f17f589c-ea23-4391-8a66-a669b58820ea.png)

Before install reticulum itself we need to install **PostgreSQL**
```
apt install postgresql
sudo service postgresql start
```
Then enter psql as root with `sudo -u postgres psql` change the password `ALTER USER postgres WITH PASSWORD 'postgres';` and leave `\q`.

To work properly, **Reticulum** requires **Dialog** with properly configured certificates. Let's set them up together:
```
git clone https://github.com/mozilla/reticulum.git
git clone https://github.com/mozilla/dialog.git
mkdir dialog/certs
cp reticulum/priv/dev-ssl.cert dialog/certs/fullchain.pem
cp reticulum/priv/dev-ssl.key dialog/certs/privkey.pem
```
And that is not all, we need to set up authentication key as well. Go to https://travistidwell.com/jsencrypt/demo/ click "Generate keys" button. And paste left key to `perms_key` variable so it looks like this 
![image](https://user-images.githubusercontent.com/33320716/198327332-3da28aa4-7d5a-48e3-acac-b29c5fb2b0b8.png)
While you are at it change `dev_janus_port` and `dialog_port` variables so they have your local environment 


Right key goes to **perms.pub.pem** and looks something like this:
![image](https://user-images.githubusercontent.com/33320716/198328603-dcefe95d-7c9f-46cb-a130-302e420f4af6.png)

We can launch dialog right now:
```
cd dialog
npm i
npm start
```
If for some reason `npm i` throws an error, try running `npm i mediasoup@3`

Now we can continue to installing and running reticulum, open new terminal window and run:
```
cd reticulum
mkdir -p storage/dev
mix deps.get
mix ecto.create
```

Before running hubs, we need to edit `etc/hosts`:
```
cd /
sudo nano etc/hosts
```
Add these two lines:
```
127.0.0.1   hubs.local
127.0.0.1   hubs-proxy.local
```

We are almost ready, open new terminal window clone hubs and run it:
```
git clone https://github.com/mozilla/hubs.git
cd hubs
nvm install 16
nvm use 16
npm i
npm run local
```

Now you can just open the browser and go to https://hubs.local:8080
