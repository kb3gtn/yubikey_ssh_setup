# Yubikey SSH Setup Instructions

Setup / configure  gnupg to work with yubikey and act as a ssh-agent.
This will allow you to use the PKI from the yubikey to authentication via SSH to remote hosts.

## Packages to install

Arch Distros:
```
sudo pacman -S pcsclite ccid gnupg opensc-p11-kit-module

```

Debian / Ubuntu Distros:
```
sudo apt install libpcsclite1 libccid gnupg gnupg-agent p11-kit-modules gnutls-bin opensc-pkcs11

```

## enable pcsclite deamon

Enable and start pcsc deamon
```
sudo systemctl enable pcscd
sudo systemctl start pcscd
```

On Arch distro, need to add opensc-pkcs11.so module manually (as root):
```
echo "module: opensc-pkcs11.so" > /usr/share/p11-kit/modules/opensc.module
 
```

## Check if p11tool can see card reader
```
p11tool --list-tokens

```
Should see a token entry with "Hardware Token" as the type using opensc-pkcs11.so.

## setup SSH Agent

create (if it doesn't already exist) ~/.gnupg/gpg.conf    
put the following in it:
```
personal-cipher-preferences AES256 AES192 AES
personal-digest-preferences SHA512 SHA384 SHA256
personal-compress-preferences ZLIB BZIP2 ZIP Uncompressed
default-preference-list SHA512 SHA384 SHA256 AES256 AES192 AES ZLIB BZIP2 ZIP Uncompressed
cert-digest-algo SHA512
s2k-digest-algo SHA512
s2k-cipher-algo AES256
charset utf-8
no-comments
no-emit-version
no-greeting
keyid-format 0xlong
list-options show-uid-validity
verify-options show-uid-validity
with-fingerprint
require-cross-certification
require-secmem
no-symkey-cache
armor
use-agent
throw-keyids
```

create (if it doesn't already exist) ~/.gnupg/gpg-agent.conf   
Put the following into it:   
```
#pinentry-program /usr/bin/pinentry-gnome3
#pinentry-program /usr/bin/pinentry-tty
#pinentry-program /usr/bin/pinentry-x11
#pinentry-program /usr/local/bin/pinentry-curses
#pinentry-program /usr/local/bin/pinentry-mac
#pinentry-program /opt/homebrew/bin/pinentry-mac
pinentry-program /usr/bin/pinentry-qt
enable-ssh-support
ttyname $GPG_TTY
default-cache-ttl 60
max-cache-ttl 120
```

# Create SSH_AGENT_SOCK environment variable
create a SSH_AGENT_SOCK environment variable to point to the GPG SSH AGENT socket service interface.
This is typically going to be something like "/run/user/1000/gnupg/S.gpg-agent"

for bash, put the following in your bash.rc file:
```
export SSH_AGENT_SOCK=/run/user/1000/gnupg/S.gpg-agent

```

For fish, create a new file in ~/.config/fish/conf.d/ssh_gpg_sock.fish
```
set -gx SSH_AUTH_SOCK /run/user/1000/gnupg/S.gpg-agent.ssh

```

Then Restart your shell..   


## Check to see if things are working

See if pcsc can see the smartcard / yubikey
```
pcsc-scan

```
Should see it pop up with your YUBIKEY information.

See if gpg can see your card:
```
gpg --card-status

```
Should get some text about the card.

See if SSH can see your key vai the agent connection:
```
ssh-add -L

```
Should spit out a list of keys available via gpg agent interface.  Yubikey should be one if plugged in.





