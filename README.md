Cold Wallet Generator
=====================

Given one or more Bitcoin addresses with private keys, generates a TeX or HTML file suitable for printing and putting into cold storage. Now with BIP 0038 symmetric encryption.

Requirements
============

Python and [jinja2](http://jinja.pocoo.org/docs/). For TeX: something that can render TeX files. For HTML: the Python libraries [PIL](http://www.pythonware.com/products/pil/) and [qrcode](https://github.com/lincolnloop/python-qrcode), as well as something that can render HTML (either your web browser or a direct-to-PDF tool like [wkhtmltopdf](https://code.google.com/p/wkhtmltopdf/)).

For format_key: sudo pip install base58 pycrypto ecdsa scrypt

Before you begin
===

Have a look at https://github.com/bpdavenport/btc as an alternative. It's substantially easier to use than this set of scripts, and it requires only Python libraries.

Usage
=====

1. Get a file of Bitcoin addresses with private keys that you want to put into cold storage (i.e., offline, not on a computer). The addresses should be one per line, in Electrum format: `address:private_key`.
2. `cat my_keys.txt | gen_cold_wallet > my_keys.tex`
3. Options include `--exclude-private-keys`, `--exclude-private-key-text`, and `--html`. Default is to print private keys, both as QR codes and as text, and to output as TeX.
4. Either run the TeX file through your favorite TeX processor and print the result, or print the HTML file.
5. Test the printed page with a QR-code scanner and make sure you can read the private keys. You might want to print another copy without the private keys so that you can easily send Bitcoin to the addresses later on.
6. Securely delete the input key file, the TeX/HTML file, and any intermediate files.
7. Put the printed paper somewhere safe.
8. Use your addresses as you wish.

Notes
======

While developing this script, I used format_key, based on [keyfmt](https://github.com/bkkcoins/misc/blob/master/keyfmt/keyfmt), and piped its output straight to the wallet generator:

`$ { for i in {1..20} ; do hexdump -v -e '/1 "%02X"' -n 32 /dev/urandom | format_key.py "%a:%w" ; done } |
gen_cold_wallet > /tmp/keys.tex`

(For real cold-storage addresses, if you're paranoid you'll use something other than /dev/urandom for your source of entropy.) On Linux, you probably want to install texlive and then run `pdflatex --shell-escape keys.tex`. On Mac, install [MacTex](http://tug.org/mactex/). No idea what to do on Windows.

If you don't want to worry about files sticking around on Linux, avoid writing them in the first place with `mkdir /tmp/ramdisk; chmod 777 /tmp/ramdisk; sudo mount -t tmpfs -o size=256M tmpfs /tmp/ramdisk/` and then do your work in that directory. Then `umount /tmp/ramdisk/` when you're done.

The TeX output is designed to be printed on 8.5-inch by 11-inch paper, 10 addresses per page, with the long end of the printed pages folded in half. Then the folded pages fit nicely in a small 5.5-inch x 8.5-inch binder. I couldn't figure out how to render the same thing in HTML, so I recommend a 2-per-page layout when printing, which ends up looking the same.

The create_new_cold_storage script is the one I use to create a fresh set of BIP 0038-encrypted keys and addresses. Run it as follows:

` PASSPHRASE=secret GPG_KEY_ID=1234ABCD create_new_cold_storage`

(Note that there's a space at the start of the command! If your Bash shell is set up with HISTCONTROL=ignorespace, this will keep your passphrase from being written to your Bash history.)

This will create the following:

1. A plaintext file containing the addresses.
2. A PDF of the addresses with scannable QR codes.
3. A GPG-encrypted file containing keys and addresses, where the GPG key is the one having the ID you specified earlier.
4. A PDF of BIP 0038-encrypted keys and addresses, using the passphrase you specified earlier.

Run the script on an offline computer, connect the computer by USB to a printer, print the PDF with the private keys, save that paper somewhere safe, then securely delete that PDF. Finally, move the other three files onto normal storage. For the GPG-encrypted file, keep it on well-replicated storage (Google Drive, Dropbox, email to friends, email to enemies, pastebin, anywhere you like).

FAQ
===

* **What's this about secure deletion?** On OSX, you want `srm file_I_never_want_to_see_again`. On Linux, it's `shred -u file_I_never_want_to_see_again`. I don't know how to do it on Windows.

* **TeX? Seriously? If I wanted low tech I'd carve my addresses on rocks.** It's the best toolchain I could think of that gave good control over page layout but didn't involve a web browser (which you might not trust not to leave traces behind). There's probably a better command-line page-layout tool that will generate printable content, but I'm not familiar with it.

* **This is just what I was looking for! May I send you a tiny thank-you?** Sure! Here's my address: [1BUGzQ7CiHF2FUxHVH2LbUx1oNNN9VnuC1](https://blockchain.info/address/1BUGzQ7CiHF2FUxHVH2LbUx1oNNN9VnuC1)
