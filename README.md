# Verus Paperwallet Generator

**Are you looking for the old paperwallet tool? See [here](https://github.com/BloodyNora/VerusPaperWallet.deprecated)!**

A Paperwallet generator in Bash script and GNU tools, outputting to styled HTML formatted like a bank account statement, meant to be laminated and stored in bank account statement folders. The Paperwallets can either be created in manual mode or by interfacing with `verusd` via RPC, dubbed `daemon` mode. Manual operation has the disadvantage of being unable to verify the input data.

It is strongly advised that you read this document from start to finish once before doing anything.

## What is a Paperwallet, why would i want it?

A Paperwallet is a paper copy of t-Addresses, z-Addresses, associated private keys or VerusID Information. A Paperwallet usually has a machine-readable representation of the address and private key data, often in form of a QR code. Private keys on Paperwallets can optionally be encrypted in one way or another. 

Paperwallets are a machine-readable backup of handpicked Verus t- and z-Addresses and Verus Identities. Paperwallets can protect against data corruption and generally provide a safety net in case of computer or disk damage as well as brown- or blackout events, or plain data loss. Addresses and Identities whose associated private keys are not imported into any computer or mobile wallet are effectively in "cold storage"; while any funds on them will not benefit from staking revenues, these funds and identities are practically exempt from electronic theft. 

## Why is this thing so bloody complicated and not just a browser tool like it used to be?

Sadly, often times safety and useability don't mix very well. You will have to choose one. Since money is involved, defaulting to safety is reasonable.

Also, the old paperwallet tool didn't support anything but t-Addresses and overall made it too easy for a careless user to leak his paperwallet data to a number of parties.

A way to run this on windows - either natively or using something like Docker - is actively being worked. on.

## Paperwallet types and their properties

All Paperwallet types share these common properties:

  * Left binding margin w/ pre-marked punch-out holes
  * Date & time of document creation
  * (optional) The associated balance
  * (optional) A text display of the associated balance
  *  A little space for notes

### t-Address

This Paperwallet type has one t-Address as well as the WIF private key associated with that t-Address:

  * tAddress
  * WIF private key

### z-Address

This Paperwallet type has one z-Address as well as the `secret-extended-key` associated with that z-Address:

  * zAddress
  * `secret-extended-key`

### VerusID

This Paperwallet type is  meant to be an addition to one z-Address and one or more t-Address Paperwallets, combining all necessary info about a VerusID referred to by these addresses:

  * VerusID Friendly Name
  * Verus Identity Address
  * associated zAddress
  * associated tAddress(es)

### Paperwallet Encryption

Instead of [`BIP0038`](https://github.com/bitcoin/bips/blob/master/bip-0038.mediawiki) which sadly also lacks support for zAddresses, OpenSSL is used as a more generic approach to (optional but highly recommended) data encryption. Please note that Paperwallet encryption only makes sense for t- and z-Addresses and thus is not supported for the VerusID template. 

**You are highly encouraged to encrypt all your t- and z-Paperwallets! Remember to use safe passphrases with at least 16 different words! See the supplied `passphrase` tool and the section on it below if needed.**

## Prerequisites

The Verus Paperwallet generator incorporates different bits and pieces, some on board, some of which need to be preinstalled. As per usual, this guide is tailored to and has been tested on [`Debian 10 (Buster)`](https://debian.org) and thus should work on derivatives like Devuan or Ubuntu almost exactly the same. 

For daemon mode, a running and fully synced Verus daemon to get your data from is the most important prerequisite. A default `verusd` config and the `wallet.dat` file you want to generate Paperwallets off will do. The `verusd` RPC interface must be enabled and accessible, the `verus` CLI binary must be in the `${PATH}` variable of your terminal session.

Besides that, update your software and install the required packages as shown in the snippet below. 

**NOTE**: You are supposed to use [`chromium`](https://www.chromium.org/Home) **without plugins in offline mode** to generate your paperwallet printouts. `firefox` [does **not** work properly](https://bugzilla.mozilla.org/show_bug.cgi?id=760436). Other browsers may work but are untested and unsupported.

```bash
apt update
apt upgrade
apt install openssl awk qrencode zbar-tools jq chromium
```

After that is done, because of some limitations with shells and browsers and stuff, you will need to install these fonts onto your system: 

  * [Source Sans Pro](https://fonts.google.com/specimen/Source+Sans+Pro)
  * [Noto Mono](https://www.google.com/get/noto/) (search for `Noto Mono`)
  * [Noto Color Emoji](https://www.google.com/get/noto/) (search for `Noto Color Emoji`)

On GNU/Linux, usually unpacking the `.ttf` files and copying them to `~/.fonts/` and then rebuilding the fonts cache as seen below will do it: 

```bash
fc-cache -f -v
```

## Installation

After installing all necessary prerequisites, just clone this repository. 

```bash
git clone https://github.com/BloodyNora/VerusPaperWallet
cd VerusPaperWallet
```

## Generating Paperwallets

See below examples. For more options and the complete commandline option list refer to 

```bash
./paperwallet --help
```

### `daemon` Mode

This requires a running `verusd` that has access to the Addresses and/or Identities you want to export to paperwallets. Just use a valid, local z-, t- or i-Address. For identities, you will need to determine your Identities' i-Address first. Then just do this:

```bash
./paperwallet -d \
  --passphrase "In posterum veritas, in Verus sanitas!" \
  --address RYEeZExoasXs1npLNR3A9cqyfa5UPuCU3W
```

### manually: z- & t-Address

**NOTE:** This does not verify the specified data further than very basic sanity checks. 

For z-Addresss, just replace the values in below example.

```bash
./paperwallet \
  --address RYEeZExoasXs1npLNR3A9cqyfa5UPuCU3W \
  --key Uqu4wkVekqnTCwR9SfrP6vmbFrt4D8stLC8EJG2FsqvcREHboiYL \
  --passphrase "In posterum veritas, in Verus sanitas!"
```

**I STRONGLY ADVISE AGAINST EVER DOING THAT,** but if you want to have plaintext instead of encrypted paperwallets, omit the `--passphrase` parameter and add `-e` to the commandline options instead.

### manually: VerusID

**NOTE:** This does not verify the specified data further than very basic sanity checks. 

For multi-signature VerusIDs, just specify multiple instances of `--verusid-taddress` on your commandline and add the respective number of minimum signatures needed using `--verusid-required-signatures` which will end up in the `Notes` section of the paperwallet. Note that the number of signatures will *only* show up  if that number is greater than `1`.

```bash
./paperwallet \
  --address iC6zMHgzHs9sZz9wZGDAn6bZZ8QgGFMX7t \
  --verusid-friendlyname "💩@" \
  --verusid-zaddress zs17yyf6ytam0kzzwm0n8w688mm62xkws9vyjaddwulnlrqdlv7jsvhzt62su9v67942w5ausv2z6t \
  --verusid-taddress RN8N9EkJBGj5pkYCujuoRRJduKZZWGkjD2 \
  --passphrase "In posterum veritas, in Verus sanitas!"
```

## Printing properly

Generally, (color) **laser printers will give massively superiour** results over using inkjet technology. Don't cheap out on ink or toner, disable `draft mode` in the printer driver. Obviously, higher quality paper will be useful for longevity, but with inkjet printouts, the paper quality is about 100x more important than with laser printouts.

Greyscale instead of color works fine, too.

Other than that just keep in mind that your printer may require a margin set in the page setup of `chromium`.

## Restoring from Paperwallets

These are just rough basic instructions, a more automatic restore tool will soon follow.

1) Turn paperwallet into image file (scanner, camera)
2) Use `zbarimg` to extract QRcode data
3) *(optionally) Use `openssl` to decrypt the resulting private key if paperwallet was encrypted*
4) Import resulting private key into litemode (t-Addresses) or native mode (z-Addresses and VerusIDs)

For reference, here's a full commandline to decrypt some of the data you'd get from one of the (now obviously compromised) test keys used in development:

```bash
echo "U2FsdGVkX19IPKDlN0HERH1oKJ6f/UrMLe11jUBJ12HZDR0UlKANsS/LoeRyLk9WNEGW9Y47qEIZsFtTzqiB7Ixs4+KKJXEcBUHQACm1kYo=" | openssl enc -d -aes-256-cbc -a -salt -pbkdf2 -A -k "In posterum veritas, in Verus sanitas!"
```

## Protection & Security

This section talks about some possible damage and attack vectors and how to protect against them. This document can in no way list all possibilities in which a paperwallet could be compromised or damaged, it is **MANDATORY** that you use common sense!

### Physical damage, UV degradation

Liquids and UV radiation are easily your paperwallets' greatest "natural" enemies. You should **definitely** laminate your paperwallets **using UV-resistant laminating pouches**. Laminators and proper laminating pouches are easily found everywhere around the internet.

Remember to store them in a fire-proof location.

### Unauthorized physical access

Storing paperwallets - even encrypted ones - with friends and family certainly is less than ideal. If you absolutely have to do so and want to use nicely formatted paperwallets instead of just putting the output of `z_exportwallet` **onto encrypted media**, just use a browser to create PDFs from the resulting HTML paper wallets, put them next to the `z_exportwallet` **onto encrypted media** and give that away instead.

Storing plain-text printed paperwallets locally also can be a bad idea! 

1) Unencrypted paperwallet templates have a foldover-section (beware with Laminators) which you are supposed to fold over the front face side of the paperwallet, folding the noise pattern to the inside. The noise pattern isn't for optics, but for decreased readability.
2) If you have to use a Laminator (which you should) you can seperate the noise pattern from the paperwallet slip, laminate both seperately and glue them together with some seal tape.
3) Additionally, you can get "scratch off label" stickers (remember these lottery tickets or the slip your bank card PIN comes printed on?). The size of the private key QRcode is purposefully chosen to adhere to the maximum reasonably available sticker size considering the general output format.

### Unauthorized electronic access

To protect against electronic access, it is mandatory that you wipe any residue of your generated paperwallets or copies of the input data. **BE CAREFUL WITH THAT, YOU COULD KILL YOUR WALLET IF YOU ACT CARELESSLY!**

On GNU/Linux systems with `coreutils` installed, you can use the `shred(1)` utility to properly overwrite files before erasing them. See `man 1 shred` for more information. Example: 

```bash
shred --random-source /dev/urandom -n3 -z ./out.html
rm ./out.html
```

## Filing & long-term storage

Just get a "bank account statement" folder, these can be bought everywhere around the internet. Make use of the pre-marked punch-out holes and file your laminated paperwallets nicely in that folder.

## Documentation for supplied/contrib tools

### Passphrase generator 

Below example outputs a 16-word-long passphrase:

```bash
genphrase 16 ./words.txt
logwood posthexaplar stoechas fiddleback holey antirationally pelvis galloman periopis scarabaeiform vermiformous rebeller sawneys unreturnable insensately hydrogenium
```

#### Dictionary file

The dictionary file supplied to use with the `passphrase` tool has been taken from [here](https://github.com/dwyl/english-words). Feel free to replace `words.txt` with any other text file of your choice containing 1 word per line.

### `num2words`

The `num2words` script is inspired by [this StackExchange thread](https://unix.stackexchange.com/questions/413441/converting-numbers-into-full-writen-words/413475#413475) and converts full numbers up into the billions into an english language word representation of the number that was input, i.e.

```bash
# ./num2words 1024
one thousand twenty four
```

**NOTE:** Only integers are supported. The output of the script doesn't make any sense to use for precision digits, either.

## Conclusion

As stated in the `LICENSE` document: 

```
THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE
USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

Use some common sense.

**EOT**
