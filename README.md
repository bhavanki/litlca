[OpenSSL]: http://www.openssl.org/

# LitlCA

## Introduction

Welcome to LitlCA. LitlCA lets you operate your own certificate authority.
It uses the [OpenSSL] toolkit's built-in certificate authority functions,
wrapping them in convenient scripts and setting up a standard directory
hierarchy for the various files involved. You must have OpenSSL installed in
order to use LitlCA.

OpenSSL's CA functions, and therefore LitlCA, are not intended for use in
establishing a large-scale certificate authority. Please refer to the "ca"
man page for an overview of the limitations.

### Conventions

LitlCA includes some functions which are user-based, such as generating
private keys and CSRs, and some which are CA-based, such as signing CSRs.
Sections of these instructions pertaining to a user-based function are marked
with {U}, while those pertaining to a CA-based function are marked with {C}.

## Installation

LitlCA wraps around the OpenSSL toolkit. You must install OpenSSL in order
to use LitlCA. Use the most recent version available.

LitlCA scripts are written for the bash shell. They have only been tested
on Linux and under Cygwin, but should function on other Unix variants.

To install, run `install`. The only required argument is the name of a
directory to install into.

    ./install icahome

By default, the installation supports both user-based and CA-based operations.
To only install the user side, pass the extra "useronly" argument.

    ./install icahome useronly

To only install the CA side, pass the extra "caonly" argument.

    ./install icahome caonly

## Actions

### Establishing a New Certificate Authority {C}

To establish a new certificate authority, run the `newCA` script. Pass in
the (directory) name of the CA to establish.

    ./newCA myCA

You can optionally pass in additional arguments: the pass phrase to set for
the authority's private key file, the CN to use for the CA's root certificate,
and a URL where the CA's CRL will be posted. An optional argument can be passed
only if previous optional arguments are too.

    ./newCA myCA myPassword "My CA" http://somewhere/myCA.crl

Depending on the arguments passed, much of the work is automatic. This includes
the creation of the root certificate for the authority, which requires the
creation of a new private key. If you do not supply a pass phrase for the
private key file on the command line, you will be prompted for one. Be sure to
select a very strong pass phrase, because if the CA key is ever compromised, the
trust established by the CA is rendered useless.

You must also create a distinguished name (DN) for the root certificate. Enter
information as you are prompted. Most values have defaults. If you passed in a
CN, a DN is automatically generated.

### Establishing a New Intermediate Certificate Authority {C}

To establish a new certificate authority which is subordinate to an existing
CA, run the `newSubCA` script. Pass in the (directory) name of the CA to
establish and the (directory) name of the superordinate CA.

    ./newSubCA mySubCA myCA

This script works much like `newCA` in what it does and what it prompts for.

### Setting up a New Certificate Authority {U}

In a user-side installation of LitlCA, the `newCA` script is not used to
establish a new certificate authority, but only to set up the necessary
directory structure to work with it (for example, to generate CSRs for it).
Pass in the (directory) name of the CA to set up.

    ./newCA myCA

All of the work is automatic and you are not prompted for anything.

To complete setting up a certificate authority, obtain the CA certificate chain
file and certificate file from the CA and place them in the CA directory.

### Generating a New Key Pair {U}

To generate a new key pair, run the `genkey` script. Pass in an "alias"
which will be used in the naming of files pertaining to the keys.

    ./genkey bob

You will be prompted for a pass phrase for the key file, or you can pass it as
an additional argument.

### Replacing an Existing Key Pair {U}

To replace an existing key pair, run the `replacekey` script. Pass in the
alias of the keys to replace.

    ./replacekey bob

The old key files are backed up, and the new ones take their place. You will
be prompted for a pass phrase for the new key file, or you can pass it as an
additional argument.

### Generating a Certificate Signing Request (CSR) {U}

Before generating a CSR, establish / set up a CA and generate a key pair.

There are two scripts that can generate CSRs.

* `gencsr` is used for the first CSR to be sent to a CA for a key pair.
* `genrenewcsr` is used for all subsequent CSRs to be sent to a CA for a
  key pair.

In other words, given some key pair and some CA, use `gencsr` for the first
CSR, and use `genrenewcsr` for all subsequent CSRs. Both scripts require the
key alias and the name of the CA to be passed in as arguments.

    ./gencsr bob myCA
    ./genrenewcsr bob myCA

You may optionally pass in additional arguments to `gencsr`: the pass phrase
for the key pair's private key file, and the CN to use for the CSR's DN. An
optional argument can be passed only if previous optional arguments are too.

In order to successfully generate a new CSR, you must enter the pass phrase for
the key file, unless you pass it as an additional argument.

When generating a new CSR, you must create a distinguished name (DN) for the
request, which will be part of the signed certificate. Enter information as you
are prompted. Most values have defaults. If you passed in a CN, a DN is
automatically generated.

If you do not pass in a CN, you are also prompted for a password that will
protect the CSR. You are free to leave this blank.

### Signing a CSR {C}

To sign a CSR, run the `signcsr` script. Pass in the key alias (which
identifies the CSR) and the name of the CA.

    ./signcsr bob myCA

In order to successfully sign a CSR, you must enter the pass phrase for the
CA's private key, which was determined when the CA was established. You can pass
in the pass phrase as an additional argument.

Follow the prompts (if any) to sign the CSR. The result will be a signed
certificate.

### Generating a PKCS#12 File {U}

To generate a PKCS#12 file containing a private keyt, run the `genpkcs12`
script. Pass in the key alias, the name of the CA which signed its trusted
certificate, and a "friendly name" which is used for display purposes by
applications that read the file.

    ./genpkcs12 bob myCA "Mr. Bob Smith"

In order to successfully generate a PKCS#12 file, you must enter the pass phrase
for the key file and a new password that will protect the PKCS#12 file. Both
passwords may be passed as additional arguments.

    ./genpkcs12 bob myCA "Mr. Bob Smith" bobPassword p12Password

To use the same password for both the key file and the PKCS#12 file, simply
leave off the PKCS#12 file password.

### Verifying a Certificate

To verify a signed certificate, run the `verify` script. Pass in the key alias
(which identifies the signed certificate) and the name of the CA.

    ./verify bob myCA

In order to successfully verify a signed certificate, the CA's certificate
chain must be present. This is normally the case for a CA-side installation, but
for a user-side installation the file must be obtained from the CA.

### Revoking a Certificate {C}

Before a signed certificate can be renewed through a renewal CSR, the old
certificate must be revoked. To revoke a signed certificate, run the `revoke`
script. Pass in the key alias (which identifies the signed certificate) and
the name of the CA.

    ./revoke bob myCA

In order to successfully revoke a signed certificate, you must enter the pass
phrase for the CA's private key, which was determined when the CA was
established. You can pass in the pass phrase as an additional argument.

### Generating a Certificate Revocation List (CRL) {C}

To generate a CRL, run the `gencrl` script. Pass in the name of the CA.

    ./gencrl myCA

In order to successfully generate a CRL, you must enter the pass phrase for the
CA's private key, which was determined when the CA was established. You can pass
in the pass phrase as an additional argument.

The CRL is generated in two different formats. Use whichever one is best for
your needs.

## Directory Structure

LitlCA has a specific directory structure. All the scripts rely on this
structure to find the files they need, and place new files into places which
are expected by other scripts. If you are running a full installation of
LitlCA, you don't need to deal with these details, for the most part.

However, if you have only a user-side or CA-side installation, then you will
need to place files generated by the "other side" into the expected locations
in order to work. This section explains the hierarchy and summarizes where
files need to go for partial installations to work.

### Scripts

All scripts are placed in the top installation directory.

### ca.config {C}

The CA configuration file is placed in the top installation directory.

### Customer Information

All customer information is placed in the "customers" directory. The "csrs" and
"signedcerts" subdirectories hold CSRs and signed certificates, respectively.
The "privatekeys" directory, which holds key files and PKCS#12 files, is not
present in a CA-side installation.

### CA Information

The information for each CA is placed in a directory with that CA's name. In a
user-side installation, the only file present in this directory is the CSR
configuration file.

## Running a User-Side Installation {U}

The following functions are unavailable in a user-side installation:

* establishing a new certificate authority (you can only set up one)
* signing CSRs
* revoking signed certificates
* generating CRLs

In order for the remaining functions to work, you must place files received
from a CA into certain locations.

* If a CA sets up a new CSR configuration file, you must replace your copy,
  which resides in the CA's directory in your installation.
* Signed certificates must be placed in the customers/signedcerts/*ca-dir*
  directory, where *ca-dir* is the name of the CA.
* The CA certificate file and the CA certificate chain file, which includes its
  certificate, must be placed in the *ca-dir* directory, where *ca-dir* is the
  name of the CA.

## Running a CA-Side Installation {C}

The following functions are unavailable in a CA-side installation:

* generating keys
* generating CSRs
* generating PKCS#12 files

In order for the remaining functions to work, you must place files received
from customers into certain locations.

* CSRs must be placed in the customers/csrs/*ca-dir* directory, where *ca-dir*
  is the name of the CA.

## File Formats

Files handled by LitlCA encompass a variety of formats.

### PEM

PEM is the primary format for files in LitlCA. A PEM file is base 64 encoded
BER or DER data surrounded by header and footer lines.

Files generated by all LitlCA scripts, with the exception of `genpkcs12`,
are in PEM format.

### PKCS#12

PKCS#12 files are used by LitlCA to store private keys along with associated
signed certificates. These files are supported by many applications which can
import private keys, such as web browsers.

The `genpkcs12` script generates PKCS#12 files.

### X.509

X.509 is a data format for certificates. The following files generated by
LitlCA contain data in X.509 format.

* CA root certificates
* signed certificates

### DER

DER is a specific encoding ("Distinguished Encoding Rules") for ASN.1 data.
It is often used for digital signatures.

The `gencrl` script generates a CRL in DER format (with suffix .crl) as
well as in PEM format.

### SSLeay traditional format

Private keys generated by LitlCA are encoded according to the traditional
key format defined by SSLeay. The private key for a CA and private keys for
customers are both encrypted: a CA private key is encrypted with DES, and a
customer private key is encrypted with Triple DES.

OpenSSL offers the ability to convert keys into the more secure PKCS#8 format;
see the man page for `pkcs8` for details.

## License

See LICENSE.txt or visit
[http://opensource.org/licenses/mit-license.php](http://opensource.org/licenses/mit-license.php).
