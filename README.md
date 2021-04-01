Ceema
=====
This is the ceema library, which provides a C++ implementation of the [Threema](https://threema.ch/)
communication protocol (and, in the future, a C interface to it). The aim is to
provide a somewhat simple interface to hook the Threema protocol in client
applications such as multi-protocol IM clients.

This is a fork of https://github.com/hbruintjes/ceema which was modified to use
mbedTLS instead of OpenSSL to be compliant with GPL.


If you are looking for a ready-to-use Threema desktop application, consider
[openMittsu](https://github.com/blizzard4591/openMittsu).

Building
========
Since this is CMake based, building the library consists of the following steps

    git clone https://github.com/myxor/ceema
    git submodule init
    git submodule sync
    git submodule update --init --recursive
    mkdir build && cd build
    cmake ../
    make


Installing as pidgin plugin
=================

    cp src/libthreepl.so ~/.purple/plugins
    sudo cp icons/threema-app-icon_neutral-16.png /usr/share/pixmaps/pidgin/protocols/16/threema.png


Dependencies
============
This library requires the following dependencies to be installed (both headers
and libraries):

* `curl`
* `jsoncpp`
* `mbedTLS`
* `sodium`
* `boost`

Library layout
==============
Under `src` you will find:
* `client`: Structures for dealing with clients/contact. The primary class is
`Client`, and a utility class `ContactStore`
* `encoding`: Some utilities for massaging data (hashing, padding, ...)
* `logging`: What it says on the tin
* `protocol`: Main protocol handling. Goes from packets, to messages, to
payloads
* `socket`: Low-level network interface
* `types`: Type definitions used throughout the library

Protocol
========
The protocol is fairly straightforward. First, a client-server handshake is
performed to set-up the encryption (using sodium/NaCl). This is handled by the
Session class. Next, the client and
server will communicate in (encrypted) packets as follows:
* A (2 byte) length prefix is sent, followed by an encrypted packet
* A packet consists of a (4 byte) type prefix, followed by an optional payload.
There are roughtly 4 types of packets:
  * Session status packets, indicating start (after any pending messages have
  been sent), or end (server generally ends the session when the client
  connects using the same credentials form another source). Normally sent by the
  server only. The end packet contains a UTF-8 encoded string payload
  * Keep-alive/ping packets, containing an arbitrary payload that must be in the
  reply. Can be sent both by the server and the client
  * Messages (see below), identified by a random ID
  * Acknowledgements, sent by both the server and the client in response to
  receiving a message, containing the sender and ID of the message being
  acknowledged. Without an acknowledgement, the server will keep retrying to
  deliver the message.
* Messages contain both a sender and recipient, some arbitrary message ID,
a timestamp (seconds since epoch), the ID or nickname of the sender and some
payload. They are prefixed by a (1 byte) type identifier, which determines
the type of payload. The payload is encrypted using the keys of the sender and
recipient, after adding padding according to PKCS#7. The following types of
messages are known:
  * Client message:
    * Text: Payload is a UTF-8 encoded string.
    * Picture
    * Location: Payload is UTF-8 encoded string with coordinates: Two lines,
    separated by LF, first in decimal, then minutes
    * Video
    * Audio
    * Poll
  * User status message, either typing, or typing stopped
  * Client message-status message, can be received, seen, agree/disagree
  * Group message:
    * Create
    * Sync
    * Leave
    * Title
    * Text
    * Picture
    * Location
    * Video
    * Audio
    * Poll

Backup string
=============
The Threema client allows for backup of the private key and client ID using a
backup string. Its is a base32 encoded string consisting of a salt and the
encrypted data. The salt and password generate a hash which can be used to
decrypt the data, which will yield the client ID, private key, and 2 bytes of
the SHA-256 hash of both which can be used to validate the decryption.


Protocol icon
=============
(c) Threema GmbH
