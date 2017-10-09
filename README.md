[![Build Status](https://travis-ci.org/dedis/drand.svg?branch=master)](https://travis-ci.org/dedis/drand)

# Drand 

Drand is a distributed randomness beacon written in Go. Drand emits a publicly
verifiable, unbiasable and unpredictable random value at a fixed interval. Having
a good random source is important in many protocols such as lottery, generating
random keys (SAY MORE).

**DISCLAIMER**: This software has NOT received a full audit and therefore should
NOT be put into production stage at this point. You have been warned.

## In a nutshell

Each participating node generates its own private key pair. A group file
describing all public keys of the participants is then created and distributed.
At that point, all nodes can run the distributed key generation part of drand to
generate a completely distributed keypair where each node have a share of the
distributed private key. After that, a special node called the leader starts a
beacon round at a fixed interval to produce a random value from a threshold of
online nodes. 

For more information about the cryptographic protocol, see PROTOCOL.md. For more
information about the design of drand, see DESIGN.md.

## Installation 

Two different ways of running drand are possible: either by installing via
Golang or using it in a Docker image. For the quickest setup, use the Docker
way.

### Installation via Docker

You need to have Docker installed on your system in order to continue.
Docker will download the image automatically the first time you launch the
image. For any drand command,  launch it as following:
```
docker run --rm --name drand -p <port>:<port> dedis/drand <command>
```

<port> is the port you want your container to expose in order to communicate
 with the other participants.

### Installation via Golang

You need to have Goland installed and setup you GOPATH. After that, a simple
```
go get github.com/dedis/drand
```
is enough to get you started.

## Usage

There are different stages that need to run in order to have a fully functional
drand beacon.

### Key generation

Each node generates their keypair using
```
drand keygen <address>
```
where address is in the form <ip>:<port>. The address is attached to the public
key so each node must be reachable at the address they specified.
By default, your keys are saved under `$HOME/.drand/drand_id.{secret,public}`.

### Group generation

To generate the group file listing all public keys, simply run:
```
drand group <pk1> <pk2> ... <pkn>
```
where <pkn> is the public key file of the n-th participant.
The group file is generated by default under `$HOME/.drand/drand_group.toml`.
That group file MUST be distributed to each of the node; it's best if the group
file is saved at the same place so there is no need for specifying the location
to drand.

### Beacon 

At the point, there needs to be a designated special node called the leader that
starts the protocol at a fixed interval. 
For the leader, run:
```
drand run --leader
```
or
```
drand run
```
for all other nodes.

The dealer can choose the period to wait between two runs with the `--period
DURATION` flag. DURATION can be for example `1mn` or `30s` (in fact, everything
understood by Golang's [duration
parsing](https://golang.org/pkg/time/#ParseDuration).

This command first runs first the Distributed Key Generation protocol, saves
the private share and the distributed public key in
`~/.drand/drand_id.{secret,public}`.
All signatures are by default saved under `~/.drand/beacons/<timestamp>.sig`.

**The distributed public key is generated under `~/.drand/drand_id.public`**.

### Verify a beacon

In order to verify that a beacon has been generated correctly, the verifier
needs two things:
 + the distributed public key generated during the DKG step. Default place is
   `~/.drand/drand_id.public`
 + the beacon signature, one in the default place `~/.drand/beacons/`.

 Simply run:
 ```
 drand verify --distkey <distkey_file> <beacon_file>
 ```

 The command outputs if the signature is valid or not, and returns 0 if the signature is valid, 1 otherwise. 

## What's this crypto magic ?

drand relies on well known protocol and concepts. 
+ drand uses pairing based cryptography for all its protocols. Drand
uses an optimized implementation of the [Barreto-Naehrig curves](https://github.com/dfinity/bn).
+ drand uses a distributed key generation (DKG) protocol to generate a distributed key
  where no node individual node can recover the private key but the public key
  is known. Only a threshold of nodes can actually recover the private key if
  they collude together. drand currently uses an implementation of the basic
  Pedersen which is currently being revised due to some implementation issues.
  The next goal would be to try with the Distributed Key Generation in the Wild
  algorithm from Aniket & Goldberg (SOURCE) allowing for more realistic
  network assumptions.
+ drand uses the BLS signature scheme but in the threshold setting. Instead of
  signing with a private key, each node signs with a private share of the
  distributed private key. A node collects at most a threshold of these partial
  signatures to reconstruct (using Lagrange interpolation) the BLS signature
  that can be verified under the distributed public key. For more info, see XXX.

## What's next

see the [TODO](https://github.com/dedis/drand/master/blob/TODO.md) file for more
information.

## Contribution

Thanks to Philipp Jovanovic for the long discussions and design decisions about drand.
