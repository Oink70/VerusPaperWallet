# Verus Paperwallet Generator

**Are you looking for the old paperwallet tool? See [here](https://github.com/BloodyNora/VerusPaperWallet.deprecated)!**

A Paperwallet generator in Bash script and GNU tools, outputting to styled HTML formatted like a bank account statement, meant to be laminated and stored in bank account statement folders. The Paperwallets can either be created in manual mode or by interfacing with `verusd` via RPC, dubbed `daemon` mode. Manual operation has the disadvantage of being unable to verify the input data.

## What is a Paperwallet?

A Paperwallet is a paper copy of t-Addresses, z-Addresses, associated private keys or VerusID Information. A Paperwallet usually has a machine-readable representation of the address and private key data, often in form of a QR code. Private keys on Paperwallets can optionally be encrypted in one way or another.

However, Paperwallets - even encrypted ones - are not ideal for uncontrolled storage. While a safe at home or a safe-deposit box in a bank vault should be reasonably secure, storing plain-text verbatim copies of Paperwallets with friends, family or anybody else is not recommended. In that case, either put the resulting file of a simple `z_exportwallet` call or the paperwallets in PDF format onto properly encrypted media and give away that instead.

## Why would i want a Paperwallet?

Paperwallets are a machine-readable (ideally offline) backup of handpicked Verus t- and z-Addresses and Verus Identities. Paperwallets can protect against data corruption and generally provide a safety net in case of computer or disk damage as well as brown- or blackout events. Addresses and Identities whose associated private keys are not imported into any computer or mobile wallet are effectively in "cold storage"; while any funds on them will not benefit from staking revenues, these funds and identities are practically exempt from electronic theft. 

## Why is this thing so bloody complicated and not just a browser tool like it used to be?

Sadly, often times safety and useability don't mix very well. You will have to choose one. Since money is involved, defaulting to safety is reasonable.

Also, the old paperwallet tool didn't support anything but t-Addresses and overall made it too easy for a careless user to leak his paperwallet data to a number of parties.

## Suppported types of Paperwallets

All Paperwallet types share these properties:

  * Left binding margin w/ pre-marked punch-out holes
  * Date & time of document creation
  * (optional) The associated balance
  * (optional) A text display of the associated balance
  *  A little space for notes

### t-Address

This Paperwallet type has one t-Address as well as the WIF private key associated with that t-Address:

  * tAddress
  * WIF private key

  t-Addresses and in turn t-Paperwallets also can be generated automatically in a semi-random fashion by generating an arbitarily long passphrase using a supplied passphrase generator script and dictionary file. The resulting passphrase is then fed to the `convertpassphrase` API call of `verusd`, using the tAddress and WIF private key from the response to generate a Paperwallet.

### z-Address

This Paperwallet type has one z-Address as well as the `secret-extended-key` associated with that z-Address:

  * zAddress
  * `secret-extended-key`

  Note that for z-Addresses the Paperwallet generator just relies on the `z_getnewaddress` API call of `verusd` to generate new zAddresses, further randomization seeding is not possible there. However, z-Addresses and in turn z-Paperwallets can be generated automatically.

### VerusID

This Paperwallet type is  meant to be an addition to one z-Address and one or more t-Address Paperwallets, combining all necessary info about a VerusID referred to by these addresses:

  * VerusID Friendly Name
  * Verus Identity Address
  * associated zAddress
  * associated tAddress(es)

### Paperwallet Encryption

Instead of [`BIP0038`](https://github.com/bitcoin/bips/blob/master/bip-0038.mediawiki) which sadly also lacks support for zAddresses, OpenSSL is used as a more generic approach to (optional but highly recommended) data encryption. Please note that Paperwallet encryption only makes sense for t- and z-Addresses and thus is not supported for the VerusID template. 

**You are highly encouraged to encrypt all your t- and z-Paperwallets! Remember to use safe passphrases with at least 16 different words!**

## Prerequisites

The Verus Paperwallet generator incooperates different bits and pieces, some on board, some of which need to be preinstalled. As per usual, this guide is tailored to and has been tested on [`Debian 10 (Buster)`](https://debian.org) and thus should work on derivatives like Devuan or Ubuntu almost exactly the same. 

A running and fully synced mainnet Verus daemon to get your data from is the most important prerequisite. A default `verusd` config and (if desired) the `wallet.dat` file you want to generate Paperwallets off will do, however you can also start with a fresh wallet and generate Paperwallets off that as you go. The `verusd` RPC interface must be enabled and accessible in both cases.

Besides that, update your software and install the required packages as shown in the snippet below. 

**NOTE**: You are supposed to use [`chromium`](https://www.chromium.org/Home) **without plugins in offline mode** to generate your paperwallet printouts. `firefox` [does **not** work](https://bugzilla.mozilla.org/show_bug.cgi?id=760436). Other browsers may work but are untested and unsupported.

```bash
apt update
apt upgrade
apt install openssl awk qrencode zbar-tools jq chromium
```

## Installation

After installing all necessary prerequisites, just clone this repository. No more installation needed. 

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

### z- & t-Address

For z-Addresss, just replace the values in below example.

```bash
./paperwallet \
  --address RYEeZExoasXs1npLNR3A9cqyfa5UPuCU3W \
  --key Uqu4wkVekqnTCwR9SfrP6vmbFrt4D8stLC8EJG2FsqvcREHboiYL \
  --passphrase "In posterum veritas, in Verus sanitas!"
```

**I STRONGLY ADVISE AGAINST EVER DOING THAT,** but if you want to have plaintext instead of encrypted paperwallets, omit the `--passphrase` parameter and add `-e` to the commandline options instead.

### VerusID

For multi-signature VerusIDs, just specify multiple instances of `--verusid-taddress` on your commandline and add the respective number of minimum signatures needed using `--verusid-required-signatures`.

```bash
./paperwallet \
  --address iC6zMHgzHs9sZz9wZGDAn6bZZ8QgGFMX7t \
  --verusid-friendlyname "💩@" \
  --verusid-zaddress zs17yyf6ytam0kzzwm0n8w688mm62xkws9vyjaddwulnlrqdlv7jsvhzt62su9v67942w5ausv2z6t \
  --verusid-taddress RN8N9EkJBGj5pkYCujuoRRJduKZZWGkjD2 \
  --passphrase "In posterum veritas, in Verus sanitas!"
```

## Printing properly

**TODO**

## Wiping data on the computer

**TODO**

## Restoring from Paperwallets

**TODO**

1) Turn paperwallet into image file (scanner, camera)
2) Use `zbarimg` to extract QRcode data

## Protection & Security

**TODO**

### Physical damage, UV degradation

**TODO**

### Unauthorized physical or logical access

**TODO**

## Filing & long-term storage

**TODO**

## Documentation for supplied/contrib tools

### Passphrase generator 

Below example outputs a 16-word-long passphrase:

```bash
genphrase 16 ./words.txt
logwood posthexaplar stoechas fiddleback holey antirationally pelvis galloman periopis scarabaeiform vermiformous rebeller sawneys unreturnable insensately hydrogenium
```

### Dictionary file

The dictionary file has been taken from [here](https://github.com/dwyl/english-words). Feel free to replace `words.txt` with any other text file of your choice containing 1 word per line.

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
