# ipfs keystore proposal
Author[s]: [whyrusleeping](github.com/whyrusleeping)

Reviewer[s]: 

* * *

Note: This spec is currently in the `wip` phase, things are likely to change very quickly.

## Goals:
To have a secure, simple and user-friendly way of storing and managing keypairs
for use by ipfs. As well as the ability to share these keys, encrypt, decrypt,
sign and verify data.

## Planned Implementation
### Storage
Keys will be stored in a directory named `keys` under the `$IPFS_PATH`
directory. Each named keypair will be stored across two files, the private key
in `$NAME` and the public key in `$NAME.pub`. They will be encoded in PEM (or
similar) format, and optionally password encrypted.

### Interface
Several additions and modifications will need to be made to the ipfs toolchain to
accomodate the changes. First, the creation of an `ipfs key` subcommand:

```

    ipfs key - Interact with ipfs keypairs

SUBCOMMANDS:

    ipfs key gen                  - Generates a new named ipfs keypair
    ipfs key list                 - Lists out all local keypairs
    ipfs key info <key>           - Get information about a given key
	ipfs key rm	<key>             - Delete a given key from your keystore
	ipfs key rename <key> <name>  - Renames a given key

    ipfs key sign <data>          - Generates a signature for the given data with a specified key
    ipfs key verify <data> <sig>  - Verify that the given data and signature match
    ipfs key encrypt <data>       - Encrypt the given data
    ipfs key decrypt <data>       - Decrypt the given data

	ipfs key send <key> <peer>    - Shares a specified private key with the given peer

    Use 'ipfs key <subcmd> --help' for more information about each command.

DESCRIPTION:

    'ipfs key' is a command used to manage ipfs keypairs.

```

#### Some subcommands:

##### Key Gen
```

    ipfs key gen - Generate a new ipfs keypair

OPTIONS:

	-t, -type		string		- Specify the type and size of key to generate (i.e. rsa-4096)
	-p, -passphrase string		- Passphrase for encrypting the private key on disk
	-n, -name		string		- Specify a name for the key

DESCRIPTION:

    'ipfs key gen' is a command used to generate new keypairs.
	If any options are not given, the command will go into interactive mode and prompt
	the user for the missing fields.

```
##### Comments: 

Similar to ssh's `ssh-keygen` with the `-t` option and interactive prompts.

* * *

##### Key Send
```

    ipfs key send <key> <peer> - Send a keypair to a given peer

OPTIONS:

	-y, -yes		bool		- Yes to the prompt

DESCRIPTION:

    'ipfs key send' is a command used to share keypairs with other trusted users.

	It will first look up the peer specified and print out their information and 
	prompt the user "are you sure? [y/n]" before sending the keypair.

	Note: while it is still managed through the keystore, ipfs will prevent you from
			sharing your nodes private key with anyone else.

```

##### Comments:

Ensure that the user knows the implications of sending a key

* * *

##### Key Encrypt
```

    ipfs key encrypt <data> - Encrypt the given data with a specified key

ARGUMENTS:

	data						- The filename of the data to be encrypted ("-" for stdin)

OPTIONS:

	-k, -key		string		- The name of the key to use for encryption (default: localkey)
	-o, -output		string		- The name of the output file (default: stdout)

DESCRIPTION:

    'ipfs key encrypt' is a command used to encypt data so that only holders of a certain
	key can read it.

```

##### Comments:

This should probably just operate on raw data and not on DAGs.

* * *

##### Other Interface Changes

We will also need to make additions to support keys in other commands, these changes are as follows:

- `ipfs add`
    - Support for a `-encrypt-key` option, for block encrypting the file being added with the key
		- also adds an 'encrypted' node above the root unixfs node
	- Support for a `-sign-key` option to attach a signature node above the root unixfs node

- `ipfs block`
    - Support for a `-encrypt-key` option, for encrypting the block before hashing and storing
	
### Code Changes/Additions
An outline of which packages or submodules will be affected.

#### Repo

- add `keystore` concept to repo, load/store keys securely
- needs to understand PEM (or $CHOSEN_FORMAT) encoding

#### Unixfs

- new node types, 'encrypted' and 'signed', probably shouldnt be in unixfs, just understood by it
- if new node types are not unixfs nodes, special consideration must be given to the interop

- DagReader needs to be able to access keystore to seamlessly stream encrypted data we have keys for 
	- also needs to be able to verify signatures

#### Importer

- DagBuilderHelper needs to be able to encrypt blocks
	- Dag Nodes should be generated like normal, then encrypted, and their parents should
		link to the hash of the encrypted node

#### New 'Encrypt' package

Should contain code for crypto operations on dags.

#### New 'Keystore' package

Should contain a `KeyStore` object with methods for storing and retrieving keys.

